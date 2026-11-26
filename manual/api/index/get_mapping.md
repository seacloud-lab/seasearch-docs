# Get mapping

=== "ES-compatible"

    ```http
    GET /es/{target}/_mapping
    ```

=== "Native"

    ```http
    GET /api/{target}/_mapping
    ```

`target` is one index name. No query parameters or body are needed.

```sh title="Read field mappings"
curl -u "$SS_USER:$SS_PASSWORD" "$SS_URL/es/api_books/_mapping"
```

HTTP 200 returns an object keyed by index name:

```json title="HTTP 200 · Selected mapping fields"
{"api_books":{"mappings":{"properties":{"category":{"type":"keyword"}}}}}
```

The example shows a selected field; actual output includes other configured and system fields. Native output retains SeaSearch field options and flattened names. ES-compatible output reconstructs a nested properties representation and exposes fewer field options; it is not a complete metadata round-trip format.

A missing index returns 400 with an `error` string. See [Mapping and field types](mapping.md) for definitions and [Update mapping](update_mapping.md) to add fields.
