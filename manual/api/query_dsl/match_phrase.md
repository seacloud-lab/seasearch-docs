# match_phrase query

Analyze text and match its terms as a phrase, using indexed term positions.

```json title="Match an analyzed phrase"
{"query":{"match_phrase":{"title":"search guide"}}}
```

The expanded form accepts `query` (required string), `analyzer` (optional string), and `boost` (optional nonnegative number, default 1):

```json
{"query":{"match_phrase":{"title":{"query":"search guide","analyzer":"standard","boost":2}}}}
```

Use one text field per object. Analysis follows the same field/custom analyzer considerations as [match](match.md). Empty analyzed input matches no documents. `slop` and other Elasticsearch phrase options are not exposed; supplying an unknown option does not enable it.

Use [Search](../search/search.md) to submit the body. Multiple fields, non-string input, or an unknown explicit analyzer can cause parsing errors.
