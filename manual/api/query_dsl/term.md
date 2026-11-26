# term query

Find an exact indexed term or a numeric/boolean value. Input strings are **not analyzed**: use a keyword field for whole-string equality.

```json title="String shorthand · Exact keyword term"
{"query":{"term":{"category":"reference"}}}
```

Expanded form:

```json title="Expanded form · Value and boost"
{"query":{"term":{"category":{"value":"reference","boost":1}}}}
```

Use one field per object. `value` is required and may be a string, number, or boolean appropriate for the field. Optional nonnegative `boost` defaults to 1. Numeric fields use exact numeric equality; booleans use their boolean representation.

`case_insensitive` is accepted but ignored. Match the indexed keyword's case, or configure analysis and use the appropriate text query. `term` does not parse dates as a date-equality query. Multiple fields or invalid value conversions can cause errors.

Use [Search](../search/search.md) for execution and result controls.
