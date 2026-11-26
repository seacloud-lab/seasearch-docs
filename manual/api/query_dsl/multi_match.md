# multi_match query

Search the same text across an explicit list of fields. Each field produces a match query and the queries are combined as alternatives.

```json title="Search title and summary"
{"query":{"multi_match":{"query":"search guide","fields":["title","summary"],"operator":"OR"}}}
```

## Parameters

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `query` | string | — | Supply the text to analyze. |
| `fields` | string array | Empty | Supply plain field names. |
| `analyzer` | string | Field/index analysis | Optional shared analyzer. Use consistent field analysis. |
| `operator` | string | `OR` | `OR` or `AND` within each field. |
| `minimum_should_match` | number or string | Backend optional-clause behavior | Threshold across **field queries**, not tokens per field. Uses the [common threshold forms](match.md#minimum_should_match). |
| `boost` | number | `1` | Nonnegative weight on the combined query. |

`type` is accepted but does not select Elasticsearch `best_fields`, `cross_fields`, phrase, or other modes. Field syntax such as `title^2` and wildcard field expansion are not implemented. Use [explicit boosted match clauses](getting_started.md#5-search-more-than-one-field) when different field weights are needed.

Invalid operator values cause errors; use correct JSON types rather than relying on coercion. Fields with different custom analyzers can behave inconsistently, especially in Unified statistics collection. Prefer compatible analysis across fields and consult [Cluster API usage](../cluster.md).
