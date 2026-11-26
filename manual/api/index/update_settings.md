# Update settings

=== "ES-compatible"

    ```http
    PUT /es/{target}/_settings
    ```

=== "Native"

    ```http
    PUT /api/{target}/_settings
    ```

!!! warning "Configure analyzers at index creation"
    This endpoint accepts text-analysis settings, but **does not support changing analyzers on an existing index**. Prefer configuring them when [creating the index](create_index.md).

`target` is one index name. No query parameters are consumed. Send a settings object directly, without an outer `settings` wrapper; its supported member is `analysis`.

## Example

For a **new** index named `api_books_analysis`:

```sh title="Configure analysis on a new index"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X PUT "$SS_URL/es/api_books_analysis/_settings" \
  -H 'Content-Type: application/json' \
  --data-binary '{"analysis":{"analyzer":{"default":{"type":"standard"}}}}'
```

HTTP 200:

```json title="HTTP 200 · Settings accepted"
{"message":"ok"}
```

A missing index is created and receives the settings. On an existing index, a nonempty analyzer map returns 400 (`can't update analyzer for existing index`); other settings can be accepted without changing configuration. A successful empty/no-op request does not mean hot updates are supported.

Invalid JSON/analysis returns 400; creation/storage errors can return 500. Replica/shard configuration and other Elasticsearch index settings are not supported by this operation. See [Text analysis](text_analysis.md) for the supported configuration shape.
