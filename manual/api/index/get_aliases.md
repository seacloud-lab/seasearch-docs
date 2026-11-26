# Get index aliases

```http
GET /es/_alias
GET /es/{target}/_alias
GET /es/_alias/{target_alias}
```

`target` filters concrete index names; `target_alias` filters alias names. Both accept comma-separated names on their respective routes. There is no registered route combining both path filters. No body or query parameters are consumed.

```sh title="Read books_current associations"
curl -u "$SS_USER:$SS_PASSWORD" "$SS_URL/es/_alias/books_current"
```

HTTP 200 returns an index-keyed map with alias objects, for example:

```json title="HTTP 200 · Index-keyed aliases"
{"api_books":{"aliases":{"books_current":{}}}}
```

An empty match returns an empty map rather than an Elasticsearch missing-alias error. See [Update aliases](update_aliases.md) to change associations.
