# Terms aggregation

Group by a mapped keyword, numeric, boolean, or explicitly aggregatable text field.

```json title="Count documents by category"
{"size":0,"aggs":{"by_category":{"terms":{"field":"category","size":10}}}}
```

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `field` | string | — | Required mapped field. |
| `size` | integer | `1000` | Positive maximum returned buckets; default configurable with `ZINC_AGGREGATION_TERMS_SIZE`. |

Buckets are normally ordered by descending count. `order` is accepted but ignored; tie ordering is not guaranteed. There are no exposed `shard_size`, include/exclude, missing-value, or error-bound controls.

Illustrative response fragment:

```json title="Response fragment · Category buckets"
{"aggregations":{"by_category":{"buckets":[{"key":"reference","doc_count":2},{"key":"travel","doc_count":1}]}}}
```

Numeric-looking bucket names can also produce a numeric `key` and `key_as_string`; do not infer the original mapping type from that response alone. For arrays/text tokens, grouping does not guarantee all-value Elasticsearch behavior. Prefer single-valued keywords for categories.

Nested `aggs` are supported beside `terms`; see [common rules](overview.md#common-rules). Unsupported field types cause a search error.
