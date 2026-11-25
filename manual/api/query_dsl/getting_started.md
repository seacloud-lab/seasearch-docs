# Full-text search and filters

Create a small book catalog, import three documents, and combine full-text search with exact and numeric filters.

Use a disposable index named `api_books`. The tutorial does not delete or overwrite an existing index automatically. If that name is already in use, choose another name consistently in every request.

## 1. Set the endpoint and credentials

```sh title="Set the service address and credentials"
export SS_URL='http://localhost:4080'
export SS_USER='your-user'
export SS_PASSWORD='your-password'
```

Use HTTPS on an untrusted network and an account authorized to create indexes, ingest documents, and search. See [Authentication](../authentication.md) for setup.

No login request is needed to obtain a session token: [HTTP Basic authentication](../authentication.md) sends credentials on each protected request.

## 2. Create explicit field mappings

`title` and `summary` are full-text fields. `category` holds whole-string terms. `price` stores numbers and supports numeric filters and sorting.

```sh title="Create api_books with explicit field types"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X PUT "$SS_URL/es/api_books" \
  -H 'Content-Type: application/json' \
  --data-binary '{
    "mappings": {
      "properties": {
        "title": {"type":"text","analyzer":"standard","search_analyzer":"standard","highlightable":true},
        "summary": {"type":"text","analyzer":"standard","search_analyzer":"standard"},
        "category": {"type":"keyword"},
        "price": {"type":"numeric"}
      }
    }
  }'
```

Successful creation returns HTTP 200 with this shape:

```json title="HTTP 200 · Index created"
{"acknowledged":true,"shards_acknowledged":true,"index":"api_books"}
```

Repeated creation returns an error; `PUT` is not an index-settings update. See [Create an index](../index/create_index.md).

## 3. Import three documents

Bulk uses newline-delimited JSON (NDJSON), not a JSON array. Each action line is followed by one complete source line. The quoted heredoc preserves JSON literally, and supplies a final newline. These IDs are for the tutorial's non-vector index.

```sh title="Import three books using NDJSON"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X POST "$SS_URL/es/api_books/_bulk" \
  -H 'Content-Type: application/x-ndjson' \
  --data-binary @- <<'NDJSON'
{"index":{"_index":"api_books","_id":"book-1"}}
{"title":"A practical search guide","summary":"Full text search with filters","category":"reference","price":25}
{"index":{"_index":"api_books","_id":"book-2"}}
{"title":"Vector search handbook","summary":"A practical guide to embeddings","category":"reference","price":40}
{"index":{"_index":"api_books","_id":"book-3"}}
{"title":"A coastal travel guide","summary":"Walking routes by the sea","category":"travel","price":15}
NDJSON
```

The ES-compatible bulk response contains `items`. Inspect its top-level `error` as well as item results; HTTP 200 alone does not guarantee that every intended operation was processed. Retrying with the same IDs can replace documents: this is not an append-only import.

### Wait for search visibility

Writes can be acknowledged before they become searchable, particularly with WAL enabled. `refresh=wait_for` is not a supported visibility guarantee.

Repeat the following read-only request until `hits.total.value` is **3** before continuing. Use a bounded wait in automation; stop and inspect the import response/logs if the count does not converge. No fixed delay is guaranteed for every deployment.

```sh title="Check visibility · Wait for three matches"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X POST "$SS_URL/es/api_books/_search" \
  -H 'Content-Type: application/json' \
  --data-binary '{"query":{"match_all":{}},"size":0}'
```

On a fresh index after all three documents become searchable, the response has this shape; timing is illustrative:

```json title="HTTP 200 · All three documents are searchable"
{"took":1,"timed_out":false,"hits":{"total":{"value":3},"max_score":1,"hits":[]}}
```

## 4. Search full text with match

```sh title="Match either analyzed title term"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X POST "$SS_URL/es/api_books/_search" \
  -H 'Content-Type: application/json' \
  --data-binary '{"query":{"match":{"title":"search guide"}},"size":10}'
```

The standard analysis produces terms including `search` and `guide`. The default `OR` operator allows either term, so all three sample titles can match. This is not an exact-string comparison.

Require both terms by using the expanded form:

```sh title="Require both analyzed title terms"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X POST "$SS_URL/es/api_books/_search" \
  -H 'Content-Type: application/json' \
  --data-binary '{
    "query":{"match":{"title":{"query":"search guide","operator":"AND"}}},
    "size":10
  }'
```

Only `book-1` has both terms in its title. See [match](match.md) for analyzer selection, optional thresholds, and Unified Search restrictions.

## 5. Search more than one field

Use plain field names with `multi_match`:

```sh title="Search title and summary for embeddings"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X POST "$SS_URL/es/api_books/_search" \
  -H 'Content-Type: application/json' \
  --data-binary '{
    "query":{"multi_match":{"query":"embeddings","fields":["title","summary"]}},
    "size":10
  }'
```

The term occurs in `book-2`'s summary. SeaSearch combines a match query for each listed field. It does **not** implement Elasticsearch's `multi_match.type` modes, and `title^2` is not parsed as a weighted field name.

To express different scoring weights, combine explicit `match` clauses with `bool.should` and a per-query `boost`:

```sh title="Apply different boosts to title and summary"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X POST "$SS_URL/es/api_books/_search" \
  -H 'Content-Type: application/json' \
  --data-binary '{
    "query":{"bool":{
      "should":[
        {"match":{"title":{"query":"practical guide","boost":2}}},
        {"match":{"summary":{"query":"practical guide","boost":1}}}
      ],
      "minimum_should_match":1
    }},
    "size":10
  }'
```

A boost adjusts scoring; it does not guarantee that a title match always ranks first or that final scores double. Scores also depend on document and query statistics.

## 6. Add exact and range filters

`term` searches the exact keyword term `reference`; it does not analyze that value. `range` compares the numeric `price`. This example deliberately uses positive bounds: the current numeric-range implementation does not correctly honor zero/negative bounds as general numeric endpoints.

```sh title="Combine full-text search with exact and range filters"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X POST "$SS_URL/es/api_books/_search" \
  -H 'Content-Type: application/json' \
  --data-binary '{
    "query":{"bool":{
      "must":{"match":{"title":"search"}},
      "filter":[
        {"term":{"category":"reference"}},
        {"range":{"price":{"gte":20,"lte":30}}}
      ]
    }},
    "size":10
  }'
```

This keeps `book-1`: it matches `search`, belongs to the reference category, and costs between 20 and 30 inclusive. `bool.filter` requires the conditions while applying zero boost. It does not imply Elasticsearch's filter-cache behavior.

For deployment-specific restrictions, see [Cluster API usage](../cluster.md).

## 7. Require optional clauses explicitly

```sh title="Require both optional scoring clauses"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X POST "$SS_URL/es/api_books/_search" \
  -H 'Content-Type: application/json' \
  --data-binary '{
    "query":{"bool":{
      "filter":{"term":{"category":"reference"}},
      "should":[
        {"match":{"title":"practical"}},
        {"match":{"summary":"filters"}}
      ],
      "minimum_should_match":2
    }},
    "size":10
  }'
```

Here the threshold counts the **two `should` queries**, so both must match. `book-1` satisfies them. In a `match` field object, the same option instead counts analyzed token clauses. See the [supported threshold forms](match.md#minimum_should_match); a computed threshold of 0 is clamped to at least 1 for a nonempty clause list.

## 8. Read the results and continue

`hits.total.value` is the total number of matches, while `hits.hits` contains the page of returned documents. Each hit identifies its index and document ID, includes a score, and normally returns `_source`. Scores and ordering are not stable values to copy into tests.

For the first book, a hit has this illustrative shape:

```json title="Illustrative hit · book-1"
{
  "_index":"api_books",
  "_type":"_doc",
  "_id":"book-1",
  "_score":0.8,
  "_source":{"title":"A practical search guide","summary":"Full text search with filters","category":"reference","price":25}
}
```

Continue with the Search reference for:

- [Pagination and sorting](../search/search.md#pagination-and-sorting).
- [Source selection and returned fields](../search/search.md#source-selection-and-returned-fields).
- [Highlighting](../search/search.md#highlighting), using the tutorial's highlightable title field.
- [Aggregations and deployment limitations](../search/search.md#aggregations).

The tutorial leaves the test index in place. Do not run a destructive cleanup against an index name shared with another workload.
