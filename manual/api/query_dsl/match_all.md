# match_all and match_none

`match_all` matches every document; `match_none` matches none.

```json title="Match every document"
{"query":{"match_all":{}}}
```

```json title="Match no documents"
{"query":{"match_none":{}}}
```

Use empty objects. There are no supported query-specific parameters; a `boost` inside these objects is not applied. An omitted or null Search `query` also means match-all, but `"query": {}` is invalid.

Submit these bodies with [Search](../search/search.md); standard result controls such as `size`, `_source`, and `aggs` remain available.
