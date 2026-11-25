# Native search

SearchV1 provides a native request format. For JSON Query DSL requests, use the [Search API](search.md).

```http
POST /api/{target}/_search
```

`target` is one index name. No URL query parameters are consumed.

## Request body

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `search_type` | string | Match all | `match`, `matchphrase`, `term`, `prefix`, `wildcard`, `fuzzy`, `querystring`, `daterange`, `matchall`, or `alldocuments`. Use these spellings without underscores. |
| `query` | object | Empty | Search input as described below. |
| `max_results` | integer | `10` | Maximum hits; server result cap applies. |
| `from` | integer | `0` | Hits to skip. |
| `sort_fields` | string array | Score descending | Field names; `-` prefix for descending. |
| `_source` | boolean or string array | `true` | Source selection. |
| `highlight` | object | None | Same shape as [Search highlighting](search.md#highlighting). |
| `explain` | boolean | `false` | Include scoring details. |
| `aggs` | object | None | Native aggregation definitions. |

`query.field` selects a field and `query.term` supplies a string for term/text/pattern searches. `query.start_time` and `query.end_time` are optional RFC 3339 timestamps; when provided they also filter `@timestamp` (start inclusive, end exclusive). With `daterange`, a different `field` adds a range on that field too.

## Example

```sh title="Search with the native SearchV1 format"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X POST "$SS_URL/api/api_books/_search" \
  -H 'Content-Type: application/json' \
  --data-binary '{"search_type":"match","query":{"field":"title","term":"search"},"max_results":10}'
```

HTTP 200 uses the ordinary [Search response](search.md#responses).

## Aggregations

Each `aggs` entry has `agg_type` and relevant `field`, `weight_field`, `size`, `ranges` (numeric from/to objects), or `date_ranges` (RFC 3339 from/to objects). Types are `term`/`terms`, `range`, `date_range`, `max`, `min`, `avg`, `weighted_avg`, `sum`, and `count`. Prefer DSL Search when writing new aggregation clients; nested native aggregations are not reliable.

Invalid JSON, a missing index, unsupported aggregation, or query execution error returns 400. An unknown `search_type` falls back to match-all rather than rejecting the request. `query.boost` and `query.terms` are not used by the conversion; do not treat them as supported controls.
