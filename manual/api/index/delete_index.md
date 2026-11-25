# Delete an index

Delete an index and its associated data.

!!! danger "Permanent deletion"
    **This cannot be undone.**

```http
DELETE /api/index/{target}
```

`target` is an index name or comma-separated names. There is no registered `DELETE /es/{target}` equivalent. No query parameters or request body are used.

## Example

Run only against an index you intend to remove:

```sh title="Destructive · Delete api_books"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X DELETE "$SS_URL/api/index/api_books"
```

HTTP 200:

```json title="HTTP 200 · Deletion response"
{"message":"deleted"}
```

## Errors and limitations

Deletion errors generally return 400; metadata-list failures return 500. Multiple deletions are not transactional: earlier indexes may already be deleted when a later deletion fails.

The handler also accepts `*` patterns, but their matching is broader than a literal name list and is not Elasticsearch glob syntax. Prefer explicit names to avoid deleting unintended indexes. There is no dry-run or confirmation parameter.
