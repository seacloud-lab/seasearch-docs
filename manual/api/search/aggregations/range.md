# Range aggregations

Define explicit buckets over numeric or date fields. These are aggregation definitions, not [range queries](../../query_dsl/range.md).

## Numeric range

```json title="Define explicit price bands"
{"size":0,"aggs":{"price_bands":{"range":{"field":"price","ranges":[{"from":10,"to":30},{"from":30,"to":50}]}}}}
```

`field` is a required numeric field; `ranges` is a required nonempty array. Each entry has numeric `from` and `to`. Lower endpoints are inclusive and upper endpoints exclusive. Supply both endpoints: omitted numeric endpoints decode as 0 rather than an unbounded range.

## Date range

```json title="Define an explicit publication-date band"
{"size":0,"aggs":{"publication_bands":{"date_range":{"field":"published","format":"2006-01-02","ranges":[{"from":"2026-01-01","to":"2027-01-01"}]}}}}
```

`field` must be a date field. `ranges` contains string `from`/`to` endpoints. Optional `format` defaults to the mapped format or RFC 3339, and optional `time_zone` defaults to UTC. Use Go layouts, not Elasticsearch patterns. Date math is not supported; use explicit dates. Omitted date endpoints are passed as open endpoints to the backend.

## Responses and limitations

Results have a `buckets` array; each bucket has a generated `key` and `doc_count`. Do not depend on Elasticsearch `from`, `to`, or `*_as_string` response members. `keyed` is ignored and custom bucket names are not supported. Nested `aggs` are not applied for either range type.

Missing ranges, incorrect field types, invalid dates, or invalid zones cause search errors. See [common rules](overview.md#common-rules).
