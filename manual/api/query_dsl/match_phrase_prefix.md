# match_phrase_prefix query

Analyze text, require a phrase for all but the final token, and also require a prefix match for the final token.

```json title="Match a phrase with a final prefix"
{"query":{"match_phrase_prefix":{"title":"practical search gui"}}}
```

Expanded form accepts `query` (required string), `analyzer` (optional string), and nonnegative `boost` (default 1):

```json
{"query":{"match_phrase_prefix":{"title":{"query":"practical search gui","analyzer":"standard"}}}}
```

!!! note "Prefix position"
    The final prefix is a separate required condition on the field. It is not constrained to immediately follow the preceding phrase. With two tokens, the query requires the first token and the final prefix anywhere in the field.

Use one text field per object. `slop` and `max_expansions` are not supported controls. Invalid field values or an unknown analyzer can cause errors. Unified Search does not support this scoring query; see [Cluster API usage](../cluster.md).
