# Unified search

Search several indexes with shared query statistics so relevance scores can be combined.

```http
POST /api/unified_search
```

No path or query parameters are consumed. Send `index_queries`, a nonempty array of objects containing an `index` string and a `query` object. Each `query` is a **full Search body**, not just the inner DSL condition.

## Example

Using two existing indexes with consistent mappings:

```sh title="Search two indexes with shared query statistics"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X POST "$SS_URL/api/unified_search" \
  -H 'Content-Type: application/json' \
  --data-binary '{"index_queries":[
    {"index":"api_books","query":{"query":{"match":{"title":"search"}},"size":10}},
    {"index":"api_books_archive","query":{"query":{"match":{"title":"search"}},"size":10}}
  ]}'
```

HTTP 200 uses the [Search response shape](search.md#responses), merging hits by descending score. Set the same positive `size` in every entry; unlike ordinary Search, this handler does not initialize each entry's size to 10.

## Limitations

Keep scoring queries, size, mappings, and analysis consistent across indexes; only filters should differ. Field sorting is not a global sort order, and aggregation results are not merged. Missing indexes can be skipped; do not use an empty response as validation of query syntax.

Unified scoring supports fewer query forms than ordinary search. In particular, omit `match.fuzziness` and `match.prefix_length` and put range filters inside `bool.filter`. Cluster coordination and permissions are explained in [Cluster API usage](../cluster.md).

Invalid JSON, no indexes, or a missing entry query returns 400. Statistics/execution failures can return 500. This endpoint is not multi-search for unrelated scoring queries.
