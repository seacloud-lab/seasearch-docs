# match_bool_prefix query

Analyze input into terms and use the final token as a prefix. Earlier tokens become exact-term alternatives; this is not an ordered phrase query.

```json title="Match analyzed terms with a final prefix"
{"query":{"match_bool_prefix":{"title":"search gui"}}}
```

Expanded form:

```json
{"query":{"match_bool_prefix":{"title":{"query":"search gui","analyzer":"standard","boost":1}}}}
```

`query` is a required string. Optional `analyzer` selects text analysis; optional nonnegative `boost` defaults to 1. Use one text field per object. There is no supported `operator`, `minimum_should_match`, or fuzzy configuration for this query: earlier terms and the final prefix are combined as `should` clauses.

Malformed field values, multiple fields, and unknown analyzers can cause errors. This scoring query is not available in Unified Search; see [Cluster API usage](../cluster.md).
