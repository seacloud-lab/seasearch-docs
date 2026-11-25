# Search

Search documents with a JSON Query DSL request.

## Request

```http
POST /es/{target}/_search
POST /es/_search
```

`target` selects an index or comma-separated indexes. Omit it to search all local indexes. Exact aliases and simple leading/trailing `*` index patterns are also recognized on a single Server; prefer explicit names when possible.

The native `/api/{target}/_search` endpoint uses a separate request format.

There are no supported URL query parameters; send search options in the JSON body.

## Request body

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `query` | object | Match all | [Query DSL](../query_dsl/overview.md) condition. Omit it or use `{"match_all":{}}`; an empty query object is invalid. |
| `from` | integer | `0` | Number of hits to skip. Use a nonnegative value. |
| `size` | integer | `10` | Maximum hits returned; capped by server `ZINC_MAX_RESULTS` (default 10000). |
| `sort` | string or array | Score descending | Field ordering; see [Pagination and sorting](#pagination-and-sorting). |
| `_source` | boolean or string array | `true` | Return all, none, or selected source keys. |
| `fields` | array | None | Return selected source values in a separate `fields` object. |
| `highlight` | object | None | Return fragments from highlightable text fields. |
| `aggs` | object | None | Named [aggregations](#aggregations). |
| `explain` | boolean | `false` | Include scoring details in each hit's `explain` field. |
| `timeout` | integer | `0` | Execution deadline in **seconds**; 0 means no explicit deadline. Use a number, not `"1s"`. |

All parameters are optional. Send `{}` for a default search, not an empty HTTP body.

## Example

Using the index and data from the [tutorial](../query_dsl/getting_started.md):

```sh title="Search the tutorial dataset"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X POST "$SS_URL/es/api_books/_search" \
  -H 'Content-Type: application/json' \
  --data-binary '{
    "query":{"match":{"title":{"query":"search guide","operator":"AND"}}},
    "size":10,
    "_source":["title","category","price"]
  }'
```

## Responses

HTTP 200 returns the following shape. Counts, timing, and scores are illustrative:

```json title="HTTP 200 · Search response"
{
  "took":1,
  "timed_out":false,
  "hits":{
    "total":{"value":1},
    "max_score":0.8,
    "hits":[{
      "_index":"api_books",
      "_type":"_doc",
      "_id":"book-1",
      "_score":0.8,
      "_source":{"title":"A practical search guide","category":"reference","price":25}
    }]
  }
}
```

- `took` is execution time in milliseconds.
- `hits.total.value` counts matches, while `hits.hits` contains only the requested page.
- Hits identify the index and document, report `_score`, and normally include `_source`.
- Requested `fields`, `highlight`, and `explain` are included when available. Custom results appear under `aggregations`.
- There is no `_shards` summary, `hits.total.relation`, or sort cursor array.

## Pagination and sorting

Use `from` and `size` for offset pagination. `size: 0` returns no hits but still computes counts and aggregations. There is no scroll, point-in-time, or `search_after` pagination.

```json
{"from":0,"size":10,"sort":[{"price":"asc"},"-_score"]}
```

A sort string names a field; prefix it with `-` for descending order. An array may also contain single-field objects with `asc` or `desc`. `_score` as a string selects descending relevance. Fields used for sorting must be mapped as sortable; text fields are not sortable by default.

## Source selection and returned fields

```json
{"_source":["title","category"],"fields":["price"]}
```

Set `_source` to false to omit it, true to return it all, or a string array to select keys. Selection matches exact top-level keys or a prefix ending in `*`. An empty array returns the whole source. Includes/excludes objects and recursive nested-path selection are not supported.

`fields` reads source values and returns each selected value wrapped in an array. It does not retrieve Elasticsearch doc values or synthesize indexed subfields. Entries can also use `{"field":"published","format":"2006-01-02"}` for date formatting; layouts use Go's reference date, not Elasticsearch patterns.

## Highlighting

Map the field with `highlightable: true` before indexing, then request it by name:

```json
{
  "highlight":{
    "number_of_fragments":3,
    "pre_tags":["<mark>"],
    "post_tags":["</mark>"],
    "fields":{"title":{}}
  }
}
```

`fields` contains field option objects. The default fragment count is 3; fields can override `number_of_fragments` and the tag pair. Only the first item of each tag array is used. A fragment count of 0 does not request the whole field. `fragment_size` is accepted but does not control fragment length.

## Aggregations

Use `aggs` to map result names to aggregation definitions. For example, count documents by keyword category:

```json
{"size":0,"aggs":{"categories":{"terms":{"field":"category","size":10}}}}
```

Supported types are `avg`, `weighted_avg`, `min`, `max`, `sum`, `count`, `cardinality`, `terms`, `range`, `date_range`, `histogram`, `date_histogram`, and `auto_date_histogram`. Fields must be mapped appropriately and be aggregatable. `count` counts matching documents, not field values. Bucket results are arrays; `terms.order` and `keyed` do not change them.

See the [Aggregation reference](aggregations/overview.md) for options, examples, and limits for each type.

## Errors and limitations

Invalid JSON, unavailable indexes, unsupported queries, and search errors generally return HTTP 400. The `error` value may be a string or an object with `type`, `reason`, and `cause`. Missing indexes do not necessarily return 404.

`track_total_hits` has no effect. `knn`, `min_score`, `retriever`, `post_filter`, and rescore are not supported. A timeout can fail the request instead of returning partial results.

For multiple indexes, keep mappings and analysis consistent. Cluster search has additional restrictions; see [Cluster API usage](../cluster.md).
