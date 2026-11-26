# wildcard query

Match indexed terms using `*` for zero or more characters and `?` for one character. Input is not analyzed.

```json title="String shorthand · Indexed-term wildcard"
{"query":{"wildcard":{"category":"ref*"}}}
```

Expanded form accepts required string `value` and optional nonnegative `boost` (default 1):

```json title="Expanded form · Pattern and boost"
{"query":{"wildcard":{"category":{"value":"ref*","boost":1}}}}
```

Use one keyword/text field per object. The pattern applies to indexed terms, not the entire original text. Match the indexed case; `case_insensitive` and ES rewrite controls are not supported. Leading or broad wildcards can be expensive.

Malformed field values can cause errors. This is not regex syntax; use [regexp](regexp.md) for regular expressions. Unified Search does not support this scoring query; see [Cluster API usage](../cluster.md).
