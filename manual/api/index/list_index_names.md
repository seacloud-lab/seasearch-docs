# List index names

```http
GET /api/index_name
```

Return up to 30 names, optionally filtered by the case-insensitive substring query parameter `name` (string, default empty). No request body is needed.

```sh title="Look up index names containing books"
curl -u "$SS_USER:$SS_PASSWORD" "$SS_URL/api/index_name?name=books"
```

HTTP 200 example:

```json title="HTTP 200 · Matching names"
["api_books"]
```

This is a short name lookup, not a complete paginated catalog. Use [List indexes](list_indexes.md) when more than 30 indexes must be enumerated. Metadata errors return 500 with an `error` string.
