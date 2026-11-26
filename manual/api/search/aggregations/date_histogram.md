# Date histogram aggregation

Group date values by a fixed or calendar interval.

```json title="Group publication dates by calendar month"
{"size":0,"aggs":{"by_month":{"date_histogram":{"field":"published","calendar_interval":"month","format":"2006-01","min_doc_count":1}}}}
```

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `field` | string | — | Required date, aggregatable field. |
| `calendar_interval` | string | — | One calendar unit: `second`, `minute`, `hour`, `day`, `week`, `month`, `quarter`, or `year`; corresponding `1s`, `1m`, `1h`, `1d`, `1w`, `1M`, `1q`, `1y` are accepted. |
| `fixed_interval` | string | — | Positive duration such as `1s`, `15m`, `1h`, or `1d`. Supply one interval mode. |
| `format` | string | RFC 3339 | Go output layout, or `epoch_millis`. |
| `time_zone` | string | UTC | Zone used for calendar boundaries and rendering. |
| `min_doc_count` | integer | `0` | Zero fills gaps; positive values filter sparse buckets. |
| `size` | integer | `1000` | Positive bucket limit, configurable through `ZINC_AGGREGATION_TERMS_SIZE`. |
| `extended_bounds`, `hard_bounds` | object | None | Numeric `min`/`max` in epoch milliseconds for gap generation. |

`interval` is an older alias for `fixed_interval`. If both calendar and fixed modes are supplied, calendar mode takes precedence; use only one for clarity.

Results contain a `buckets` array with formatted keys and `doc_count`. Nested `aggs` are supported. `keyed` is ignored. There is no separate guaranteed epoch key plus formatted string unless `epoch_millis` is selected.

## Limits

Second/minute/hour/day calendar names are implemented as fixed-duration boundaries. Weeks begin on Sunday. Do not assume Elasticsearch timezone/DST rounding semantics. Bucket sorting uses formatted key strings; choose a chronological layout such as `2006-01`.

Bounds control gap generation rather than reliably excluding existing buckets. Use a date query filter to restrict documents. Avoid large empty ranges with tiny intervals. Missing interval, invalid zone/duration, or an incorrect field type causes errors.
