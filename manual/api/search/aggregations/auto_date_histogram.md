# Auto date histogram aggregation

Choose a date interval from the data range and a target bucket count.

```json title="Select a date interval for a target bucket count"
{"size":0,"aggs":{"publication_trend":{"auto_date_histogram":{"field":"published","buckets":10,"minimum_interval":"day","format":"2006-01-02"}}}}
```

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `field` | string | — | Required date, aggregatable field. |
| `buckets` | integer | `10` | Desired bucket count; nonpositive values fall back to 10. Not a guarantee of exactly this many buckets. |
| `minimum_interval` | string | `second` | `second`, `minute`, `hour`, `day`, `month`, or `year`. |
| `format` | string | RFC 3339 | Go output layout. Unlike date_histogram, `epoch_millis` is not a working special rendering mode here. |
| `time_zone` | string | UTC | Accepted/validated, but not consistently applied to boundaries and output. Use UTC for predictable results. |

The result contains `buckets` and an `interval` string identifying the selected interval. Each bucket contains `key` and `doc_count`. Nested `aggs` are supported; `keyed` is ignored. There are no exposed bounds, offset, or minimum-document-count options.

Use chronological output layouts and test representative data ranges: this is SeaSearch's interval selection, not a promise of Elasticsearch's interval table or rounding behavior. Invalid fields, unsupported minimum intervals, or invalid zones cause search errors. See [common rules](overview.md#common-rules).
