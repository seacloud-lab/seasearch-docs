# prefix query

Find indexed terms beginning with a prefix. Input is not analyzed.

```json title="String shorthand · Indexed-term prefix"
{"query":{"prefix":{"category":"ref"}}}
```

Expanded form accepts required string `value` and optional nonnegative `boost` (default 1):

```json title="Expanded form · Prefix and boost"
{"query":{"prefix":{"category":{"value":"ref","boost":1}}}}
```

Use one keyword/text field per object. On text, the prefix applies to individual indexed tokens, not the original whole source string. Case must match indexed terms. `rewrite` and `case_insensitive` are not supported options.

Multiple fields and non-string values can cause errors. Broad prefixes can be expensive. Unified Search does not support this scoring query; see [Cluster API usage](../cluster.md).
