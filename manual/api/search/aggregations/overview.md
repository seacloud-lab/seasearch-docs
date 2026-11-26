# Aggregations

Summarize documents matching a search query. Put named definitions under `aggs` in a [Search](../search.md) body; use `size: 0` when no document hits are needed.

```sh title="Count documents by category"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X POST "$SS_URL/es/api_books/_search" \
  -H 'Content-Type: application/json' \
  --data-binary '{"size":0,"aggs":{"by_category":{"terms":{"field":"category","size":10}}}}'
```

The response adds an `aggregations` object keyed by your names. Metric results normally contain `value`; bucket results contain a `buckets` array with `key` and `doc_count`.

## Types

<div class="grid cards" markdown="1">

- **[Metrics](metrics.md)**

    Compute `avg`, `weighted_avg`, `min`, `max`, `sum`, `count`, and `cardinality` summaries.

- **[Terms](terms.md)**

    Group documents by an indexed field value.

- **[Range and date range](range.md)**

    Define explicit numeric or date intervals.

- **[Histogram](histogram.md)**

    Group numeric values into equal intervals.

- **[Date histogram](date_histogram.md)**

    Group dates by a fixed or calendar interval.

- **[Auto date histogram](auto_date_histogram.md)**

    Select a date interval from a target bucket count.

</div>

## Common rules

- Map fields before indexing and enable `aggregatable`. Text fields default to false; keyword fields are usually better for categories.
- Specify one aggregation type per definition. Unsupported aggregation names may be ignored; `ip_range` is explicitly rejected.
- Use names other than `count`, `duration`, and `max_score`, which are reserved by result processing.
- `terms`, `histogram`, `date_histogram`, and `auto_date_histogram` accept nested `aggs` beside their type definition. Range aggregations do not apply nested definitions.
- `keyed` does not change buckets to an object, and pipeline/scripted aggregations are not implemented.
- Use single-valued fields for predictable bucket counts. Multi-valued fields do not necessarily have Elasticsearch's deduplication/grouping semantics.
- Empty numeric metric results can be normalized to zero instead of Elasticsearch-style nulls. Do not infer the presence of source values from a zero result alone.

Results may be merged from multiple readers and truncated by bucket size limits; do not assume exact global top-bucket accounting. Cluster Proxy search has additional aggregation restrictions; see [Cluster API usage](../../cluster.md).
