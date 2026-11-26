# bool query

Combine required, alternative, excluded, and filter conditions.

```json title="Combine full-text search, filters, and exclusions"
{
  "query":{"bool":{
    "must":{"match":{"title":"search"}},
    "filter":[{"term":{"category":"reference"}},{"range":{"price":{"gte":20,"lte":30}}}],
    "must_not":{"ids":{"values":["book-9"]}},
    "should":[{"match":{"summary":"practical"}}],
    "minimum_should_match":1
  }}
}
```

## Parameters

| Parameter | Type | Meaning |
| --- | --- | --- |
| `must` | Query or query array | All conditions must match; they contribute to scoring. |
| `should` | Query or query array | Alternative scoring clauses. Set a threshold explicitly when required. |
| `must_not` | Query or query array | Exclude matching documents. |
| `filter` | Query or query array | All conditions must match, with zero boost. |
| `minimum_should_match` | number or string | Required number of `should` clauses; accepts the [threshold forms](match.md#minimum_should_match). |

All clauses are optional, but an empty bool is not a match-all query. Use a meaningful condition. A bool containing only `must_not` excludes from all documents. There is no supported bool-level `boost` option; unknown members are rejected.

For nonempty should lists, an explicitly computed threshold is clamped to at least 1. It counts `should` clauses, not tokens inside a child match. Filters do not imply Elasticsearch query-cache behavior.

Malformed clauses or thresholds cause errors. In Unified Search, filter clauses are excluded from scoring-term statistics; this permits document-only filters even when the same query type is unsupported as a scoring clause. See [Cluster API usage](../cluster.md).
