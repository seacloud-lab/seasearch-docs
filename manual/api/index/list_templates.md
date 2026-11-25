# List index templates

```http
GET /es/_index_template
```

The optional string query parameter `pattern` (default empty) selects templates containing that **exact stored pattern**. It is not a wildcard search over template names. No body is required.

```sh title="List index templates"
curl -u "$SS_USER:$SS_PASSWORD" "$SS_URL/es/_index_template"
```

HTTP 200 returns an array of objects with `name` and `index_template`, where `index_template` has the [template definition](get_template.md). An empty list is `[]`. Metadata errors return 400 with an `error` string.
