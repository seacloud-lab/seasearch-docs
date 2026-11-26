# Get settings

=== "ES-compatible"

    ```http
    GET /es/{target}/_settings
    ```

=== "Native"

    ```http
    GET /api/{target}/_settings
    ```

Read text-analysis settings for one index. No query parameters or body are needed.

```sh title="Read index settings"
curl -u "$SS_USER:$SS_PASSWORD" "$SS_URL/es/api_books/_settings"
```

HTTP 200 returns an object keyed by index name; an index without analysis settings returns:

```json title="HTTP 200 · No analysis settings"
{"api_books":{"settings":{}}}
```

Configured analysis appears under `settings.analysis`; see [Text analysis](text_analysis.md). A missing index returns 400 with an `error` string.
