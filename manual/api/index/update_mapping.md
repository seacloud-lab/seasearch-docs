# Update mapping

Add new mapped fields to an index. Existing fields cannot be modified, even by resubmitting the same definition.

=== "ES-compatible"

    ```http
    PUT /es/{target}/_mapping
    ```

=== "Native"

    ```http
    PUT /api/{target}/_mapping
    ```

`target` is one index name. No query parameters are consumed. The body is a [mapping object](mapping.md) with `properties`, **without** an outer `mappings` wrapper.

## Example

```sh title="Add the edition field"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X PUT "$SS_URL/es/api_books/_mapping" \
  -H 'Content-Type: application/json' \
  --data-binary '{"properties":{"edition":{"type":"numeric"}}}'
```

HTTP 200:

```json title="HTTP 200 · Mapping updated"
{"message":"ok"}
```

## Errors and limitations

Invalid mappings or any already-mapped field return 400. Index creation/storage failures can return 500. A missing index can be created automatically.

Adding a field does not reindex older documents. Define custom analyzers together with mappings at [index creation](create_index.md); this endpoint does not resolve arbitrary existing custom analysis definitions when validating new fields.
