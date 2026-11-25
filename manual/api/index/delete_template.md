# Delete an index template

```http
DELETE /es/_index_template/{target}
```

`target` is the template name. No body or query parameters are used. Deleting a template does not delete indexes previously created from it.

```sh title="Delete the books template"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X DELETE "$SS_URL/es/_index_template/books"
```

HTTP 200:

```json title="HTTP 200 · Template deleted"
{"message":"ok"}
```

Metadata deletion errors return 400 with an `error` string. There is no separate update-only or bulk-delete template operation.
