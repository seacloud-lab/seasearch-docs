# Metric aggregations

Compute a scalar summary over an aggregatable field.

```json title="Compute numeric summaries and counts"
{
  "size":0,
  "aggs":{
    "average_price":{"avg":{"field":"price"}},
    "highest_price":{"max":{"field":"price"}},
    "matching_books":{"count":{}},
    "categories":{"cardinality":{"field":"category"}}
  }
}
```

| Type | Required options | Result |
| --- | --- | --- |
| `avg` | `field` string | Average numeric value. |
| `min` | `field` string | Minimum numeric value. |
| `max` | `field` string | Maximum numeric value. |
| `sum` | `field` string | Sum of numeric values. |
| `weighted_avg` | `field`, `weight_field` strings | Weighted numeric average, using a separate weight field. |
| `count` | None | Matching document count; a supplied `field` does not change it. |
| `cardinality` | `field` string | Distinct-value estimate; no precision parameter is exposed. |

Use numeric fields for numeric metrics and compatible aggregatable fields for distinct values. `weighted_avg` uses `weight_field`, not Elasticsearch's nested `value`/`weight` objects:

```json title="Weighted average using a separate weight field"
{"size":0,"aggs":{"weighted_price":{"weighted_avg":{"field":"price","weight_field":"quantity"}}}}
```

Each result appears under `aggregations.<name>`, normally as `{"value":25}`. Missing/non-numeric values and empty datasets do not necessarily produce Elasticsearch-style nulls; NaN results are normalized to zero.

There are no `script`, `missing`, `format`, or precision-control options. Metrics do not accept nested aggregations. See [Aggregation common rules](overview.md#common-rules).
