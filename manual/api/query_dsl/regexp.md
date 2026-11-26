# regexp query

Search indexed terms with a regular-expression pattern. Input is not analyzed.

```json title="Expanded form · Indexed-term regular expression"
{"query":{"regexp":{"category":{"value":"ref.*"}}}}
```

Use one keyword/text field per object. A string shorthand is also accepted:

```json title="String shorthand"
{"query":{"regexp":{"category":"ref.*"}}}
```

The field object accepts `value` (required string) and `boost` (optional nonnegative number, default 1). `flags` is accepted but ignored. Do not assume Elasticsearch/Lucene regex syntax and flags are portable; use simple indexed-term patterns and test your intended expression. Invalid patterns can fail at search execution.

This does not search arbitrary substrings of the original document source. Broad expressions can be expensive. Unified Search does not support this scoring query; see [Cluster API usage](../cluster.md).
