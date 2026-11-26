# Bulk V2

Insert or replace multiple documents using one JSON object rather than action/source NDJSON.

```http
POST /api/_bulkv2
POST /api/{target}/_bulkv2
```

No ES-compatible alias or query parameters are registered.

## Request body

| Parameter | Type | Description |
| --- | --- | --- |
| `index` | string | Required on the global path; the path `target` takes precedence when present. |
| `records` | object array | Documents to index. A string `_id` selects an ID; otherwise one is generated. |

```sh title="Index a JSON records array"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X POST "$SS_URL/api/api_books/_bulkv2" \
  -H 'Content-Type: application/json' \
  --data-binary '{"records":[{"_id":"book-1","title":"A practical search guide","price":25}]}'
```

HTTP 200:

```json title="HTTP 200 · Bulk V2 response"
{"message":"v2 data inserted","record_count":1}
```

Existing IDs are replaced; missing indexes are created automatically. Invalid JSON returns 400 and processing errors generally return 500. This endpoint returns a count, not per-record outcomes or generated IDs. It is not transactional; do not assume a failure leaves no changes. Empty records are not a useful ingestion operation and can still create the index.

Use complete document sources, and observe the [write visibility guidance](overview.md). Vector indexes require valid base62 IDs when IDs are supplied.
