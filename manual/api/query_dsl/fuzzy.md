# fuzzy query

Find indexed terms near a supplied value by edit distance. Unlike [match](match.md), the supplied value is not tokenized into separate match clauses.

```json title="Find terms within one edit of serch"
{"query":{"fuzzy":{"title":{"value":"serch","fuzziness":1,"prefix_length":1}}}}
```

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `value` | string | — | Required term. |
| `fuzziness` | number or string | Backend distance 1 | Positive edit distance or `AUTO`; see [fuzzy matching](match.md#fuzzy-matching) for AUTO thresholds. |
| `prefix_length` | integer | `0` | Unchanged leading prefix. |
| `boost` | number | `1` | Nonnegative scoring weight. |

String shorthand is accepted, for example `{"fuzzy":{"title":"serch"}}`. Use one text/keyword field and the case/form of its indexed terms. Prefer small edit distances such as 1 or 2. Specifying 0 does not reliably disable fuzziness because nonpositive values leave the backend default unchanged; use `term` for exact matching.

ES expansion/rewrite options are not supported. Malformed values can cause errors; broad fuzzy expansion can be costly. Unified Search does not support this scoring query; see [Cluster API usage](../cluster.md).
