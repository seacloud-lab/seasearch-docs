# terms query

Match any of several exact values on one field.

```json title="Match either category"
{"query":{"terms":{"category":["reference","travel"]}}}
```

Supply one field with a nonempty value array. Use string values for keyword terms, JSON numbers for numeric fields, and booleans for boolean fields. Unlike `term`, value-array element types select the query representation; do not use numeric strings as a substitute for numeric values.

Ordinary Search also accepts a nonnegative `boost` beside the field:

```json title="Ordinary Search · Optional scoring boost"
{"query":{"terms":{"category":["reference","travel"],"boost":2}}}
```

There is no analyzer, terms lookup, or `terms_set` support. Input is not analyzed. Use [bool](bool.md) to combine fields. Unsupported value types cause errors; avoid empty arrays and multiple field keys.

Unified Search does not support `boost` on a `terms` scoring clause. Omit it when using [cluster search](../cluster.md).
