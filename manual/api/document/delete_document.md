# Delete a document

=== "ES-compatible"

    ```http
    DELETE /es/{target}/_doc/{id}
    ```

=== "Native"

    ```http
    DELETE /api/{target}/_doc/{id}
    ```

`target` and `id` identify the document. No query parameters or body are used.

```sh title="Delete book-1"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X DELETE "$SS_URL/es/api_books/_doc/book-1"
```

HTTP 200:

```json title="HTTP 200 · Deletion response"
{"message":"deleted","index":"api_books","id":"book-1"}
```

Missing indexes/documents and deletion failures generally return 400 with an `error` string. Deletion of vector-only records has different existence handling; do not use successful deletion as proof that a full-text document existed. With WAL enabled, verify subsequent visibility rather than assuming the response has synchronously removed search hits.
