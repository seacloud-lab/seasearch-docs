# Multi

Index multiple source documents, one JSON object per line. Unlike Bulk, there are **no action lines**.

```http
POST /api/{target}/_multi
```

`target` is the required index name. No query parameters are consumed. Missing indexes can be created automatically.

## Example

```sh title="Index source-only NDJSON"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X POST "$SS_URL/api/api_books/_multi" \
  -H 'Content-Type: application/x-ndjson' \
  --data-binary @- <<'NDJSON'
{"_id":"book-1","title":"A practical search guide","price":25}
{"_id":"book-2","title":"Vector search handbook","price":40}
NDJSON
```

Include a final newline. Omit `_id` to generate an ID; a supplied string ID replaces the previous document.

HTTP 200:

```json title="HTTP 200 · Multi response"
{"message":"multiple data inserted","record_count":2}
```

## Errors and limitations

Each line is limited by `ZINC_MAX_DOCUMENT_SIZE` (default 1 MiB). Malformed JSON lines are skipped, and the response does not provide per-line errors or generated IDs. Processing failures generally return 500 and may leave earlier writes applied.

This is source-only ingestion, not multi-get or multi-search. Use [Bulk](bulk.md) for delete actions and [Multi-search](../search/multi_search.md) for queries. Follow the [visibility guidance](overview.md) before dependent searches.
