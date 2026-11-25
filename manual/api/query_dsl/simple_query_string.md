# simple_query_string query

SeaSearch routes `simple_query_string` to the same parser as [query_string](query_string.md). It is **not** Elasticsearch's separate, error-tolerant simple query language.

```json title="Expression parsed with the query_string parser"
{"query":{"simple_query_string":{"query":"+title:search -category:travel"}}}
```

Use the syntax, escaping rules, accepted options, and limitations on the [query_string page](query_string.md). Invalid syntax can fail the request rather than being silently discarded. `flags`, `all_fields`, and Elasticsearch-specific simple-query options are not available.

Submit the body through [Search](../search/search.md). This scoring query is not supported by Unified Search.
