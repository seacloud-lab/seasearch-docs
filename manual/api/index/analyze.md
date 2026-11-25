# Analyze text

Inspect the tokens produced by an analyzer or a supplied tokenization pipeline.

=== "ES-compatible"

    ```http
    POST /es/_analyze
    POST /es/{target}/_analyze
    ```

=== "Native"

    ```http
    POST /api/_analyze
    POST /api/{target}/_analyze
    ```

`target`, when present, selects an index's custom analyzers and mappings. No query parameters are consumed.

## Request body

| Parameter | Type | Description |
| --- | --- | --- |
| `text` | string | Text to analyze; supply this value. Arrays of strings are not supported. |
| `analyzer` | string | Optional built-in/custom analyzer name. |
| `field` | string | Optional mapped field on the selected index; used to select analysis when `analyzer` is absent. |
| `tokenizer` | string or object | Optional tokenizer name or definition with `type`. |
| `char_filter` | array | Optional character filter names/definitions. |
| `token_filter` | array | Optional token filter names/definitions; `filter` is an alias. |

Choose an analyzer, a mapped field with analysis, or a tokenizer pipeline. Without a usable tokenizer the request fails. See [Text analysis](text_analysis.md) for names and configuration.

## Example

```sh title="Inspect tokens with the keyword analyzer"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X POST "$SS_URL/es/_analyze" \
  -H 'Content-Type: application/json' \
  --data-binary '{"analyzer":"keyword","text":"SeaSearch"}'
```

HTTP 200 returns a `tokens` array:

```json title="HTTP 200 · Analyzed tokens"
{"tokens":[{"token":"SeaSearch","start_offset":0,"end_offset":9,"position":1,"type":"AlphaNumeric","keyword":false}]}
```

Token metadata depends on the tokenizer. Offsets identify the token's span; `position` reports its position increment, not a cumulative token index. `keyword` is the analyzer's keyword marker.

Invalid JSON, unknown components, a missing index, or a pipeline without a tokenizer returns 400. Analyze does not create an index. When experimenting with extra filters, prefer a standalone pipeline over appending filters to an index's reusable custom analyzer.
