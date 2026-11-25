# Get or check a document

```http
GET /api/{target}/_doc/{id}
HEAD /api/{target}/_doc/{id}
```

`target` and `id` identify one document. There is no equivalent GET/HEAD `/es/` document route. No query parameters or body are used.

```sh title="Read book-1 by ID"
curl -u "$SS_USER:$SS_PASSWORD" "$SS_URL/api/api_books/_doc/book-1"
```

HTTP 200 returns a hit object with `_index`, `_type`, `_id`, `_score`, and available `_source`. For example:

```json title="HTTP 200 · Document hit"
{"_index":"api_books","_type":"_doc","_id":"book-1","_score":0,"_source":{"title":"A practical search guide","category":"reference","price":25}}
```

The score is not a relevance ranking for this ID lookup. HEAD uses the same lookup; inspect the status instead of a response body.

## Errors and limitations

A missing ID normally returns 400, not 404. Initialization/storage failures can return 500. The handler can create a missing index while trying to read a document, so it is not a side-effect-free index existence check. Use [Check index existence](../index/check_exists.md) for that purpose.
