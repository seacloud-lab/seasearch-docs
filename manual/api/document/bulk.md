# Bulk

Send multiple document actions as newline-delimited JSON (NDJSON).

=== "ES-compatible"

    ```http
    POST /es/_bulk
    POST /es/{target}/_bulk
    PUT /es/{target}/_bulk
    ```

=== "Native"

    ```http
    POST /api/_bulk
    POST /api/{target}/_bulk
    ```

`target` provides a default index. No query parameters are consumed.

## Body format

Use `Content-Type: application/x-ndjson`. Each line must be one complete JSON object; include a final newline.

| Action | Lines | Meaning |
| --- | --- | --- |
| `index` | Action + source | Insert or replace a document. |
| `create` | Action + source | Same write behavior; not create-only. |
| `update` | Action + source | Full replacement, not partial merge or an ES `doc` wrapper. |
| `delete` | Action only | Delete by ID. |

Action metadata uses `_index` (string) and `_id` (string). Supply `_index` explicitly for clarity, especially on global paths. A metadata index overrides the path default. Missing indexes can be created automatically. Omit `_id` for an automatic ID on writes; deletes require it.

## Example

```sh title="Index two documents with action/source NDJSON"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X POST "$SS_URL/es/api_books/_bulk" \
  -H 'Content-Type: application/x-ndjson' \
  --data-binary @- <<'NDJSON'
{"index":{"_index":"api_books","_id":"book-1"}}
{"title":"A practical search guide","category":"reference","price":25}
{"index":{"_index":"api_books","_id":"book-2"}}
{"title":"Vector search handbook","category":"reference","price":40}
NDJSON
```

## Responses

Native success is HTTP 200:

```json title="HTTP 200 · Native response"
{"message":"bulk data inserted","record_count":2}
```

ES-compatible paths return an object containing `took`, `errors`, `items`, and an optional top-level `error`. Items are action-keyed objects carrying `_index`, `_type`, `_id`, `_version`, `result`, `_shards`, `status`, `_seq_no`, `_primary_term`, and an optional `error`. These compatibility fields do not implement ES versioning or conflict detection; the `errors` flag alone is not a reliable failure summary.

## Errors and limitations

!!! warning "Check more than HTTP status"
    ES-compatible Bulk can return HTTP 200 with a top-level `error`. Malformed JSON lines can be skipped without a corresponding failed item. Do not assume a one-to-one input/item mapping, accurate per-item status, or that a successful HTTP response proves all intended documents were indexed.

Native worker failures generally return 500; input-format and processing failures can occur after earlier writes have taken effect. Batches are not transactional. WAL and direct-write modes can differ in item order and reported results. `create` does not reject an existing ID, and all update-like actions require complete replacement source.

Each line is limited by server `ZINC_MAX_DOCUMENT_SIZE` (default 1 MiB). With WAL enabled, acknowledgment can precede visibility. See [Bulk V2](bulk_v2.md) for JSON-array ingestion and [Multi](multi.md) for source-only NDJSON.
