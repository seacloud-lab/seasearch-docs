# Index a document

Insert a JSON document, or replace a document using its ID. Missing indexes can be created automatically.

=== "ES-compatible"

    ```http
    POST /es/{target}/_doc
    PUT /es/{target}/_doc/{id}
    PUT /es/{target}/_create/{id}
    POST /es/{target}/_create/{id}
    ```

=== "Native"

    ```http
    POST /api/{target}/_doc
    PUT /api/{target}/_doc
    PUT /api/{target}/_doc/{id}
    ```

`target` is one index name. `id` is optional on the ID-less paths; an automatic ID is generated if none is supplied. A string `_id` in the body overrides the path ID. No query parameters are consumed.

The body is the document itself, not a `doc` wrapper.

## Example

```sh title="Insert or replace book-1"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X PUT "$SS_URL/es/api_books/_doc/book-1" \
  -H 'Content-Type: application/json' \
  --data-binary '{"title":"A practical search guide","category":"reference","price":25}'
```

HTTP 200:

```json title="HTTP 200 · Indexing response"
{"message":"ok","id":"book-1","_id":"book-1","_index":"api_books","_version":1,"_seq_no":0,"_primary_term":0,"result":"created"}
```

`id`/`_id` identify the document. Version, sequence, primary-term, and `result` fields are compatibility values; `result: created` does not distinguish replacement from insertion.

## Errors and limitations

- `_create` uses the same replacement behavior as `_doc`; it does **not** enforce create-only conflicts or return 409 when an ID exists.
- Replacements remove omitted fields. For the existence-checking variant, see [Update document](update_document.md).
- Invalid JSON returns 400; indexing, type conversion, or storage errors can return 500.
- Vector indexes require canonical base62 IDs; use generated IDs unless you deliberately manage valid vector IDs. `book-1` is only for this non-vector example.
- Acknowledgment is not a search-visibility guarantee; see [Document overview](overview.md).
