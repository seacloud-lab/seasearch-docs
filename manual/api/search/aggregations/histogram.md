# Histogram aggregation

Group numeric values into equal intervals.

```json title="Group prices into intervals of 10"
{"size":0,"aggs":{"price_steps":{"histogram":{"field":"price","interval":10,"min_doc_count":1,"size":20}}}}
```

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `field` | string | — | Required numeric, aggregatable field. |
| `interval` | number | — | Required positive interval. |
| `offset` | number | `0` | Bucket shift; use `0 <= offset < interval`. |
| `min_doc_count` | integer | `0` | Zero fills gaps; positive values remove buckets below the threshold. |
| `size` | integer | `1000` | Positive bucket limit, configurable through `ZINC_AGGREGATION_TERMS_SIZE`. |
| `extended_bounds` | object | None | Numeric `min`/`max` extending gap generation. |
| `hard_bounds` | object | None | Numeric `min`/`max` used for gap-generation limits; see warning below. |

Buckets use numeric interval keys and `doc_count`. Nested `aggs` are supported. `keyed` and ordering options do not control output.

!!! note "Bounds and ordering"
    Bounds do not reliably remove existing out-of-range buckets; use a query filter to restrict data. Bucket ordering compares generated key strings, not a guaranteed numeric order, and decimal keys can lose numeric fidelity in response conversion. Do not assume Elasticsearch histogram ordering or hard-bound semantics.

Use bounded intervals and sensible limits: gap filling can generate many buckets before truncation. Invalid intervals and non-numeric fields cause search errors. See [common rules](overview.md#common-rules).
