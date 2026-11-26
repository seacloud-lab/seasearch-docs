# Update a document

Replace a document by ID.

!!! warning "Updates replace the complete document"
    This operation does not merge a partial JSON object into the previous document.

=== "ES-compatible"

    ```http
    POST /es/{target}/_update/{id}
    ```

=== "Native"

    ```http
    POST /api/{target}/_update/{id}
    ```

`target` selects an index and `id` identifies the document. A string `_id` in the body overrides the path ID.

## Query parameter

`insert` is an optional boolean, default false. Set `insert=true` to insert when the ID is absent; otherwise a missing document causes an error. A missing index can still be created before the document check.

## Example

Send the **complete replacement document**, without an outer `doc` member:

```sh title="Replace book-1, inserting if absent"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X POST "$SS_URL/es/api_books/_update/book-1?insert=true" \
  -H 'Content-Type: application/json' \
  --data-binary '{"title":"A practical search guide","category":"reference","price":28}'
```

HTTP 200:

```json title="HTTP 200 · Update response"
{"message":"ok","id":"book-1"}
```

Invalid JSON or an empty ID returns 400; a missing document without `insert=true`, conversion, and write errors can return 500. There is no script, no-op detection, partial merge, or optimistic-concurrency support. For unconditional insertion/replacement, use [Index document](index_document.md).
