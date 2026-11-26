# Get an index template

```http
GET /es/_index_template/{target}
HEAD /es/_index_template/{target}
```

`target` is one template name. No body or query parameters are needed.

```sh title="Read the books template"
curl -u "$SS_USER:$SS_PASSWORD" "$SS_URL/es/_index_template/books"
```

GET returns HTTP 200 with the template definition directly: `index_patterns`, `priority`, `template`, `created_at`, and `updated_at`. The nested `template` contains configured `settings` and `mappings`. This is not an Elasticsearch `index_templates` response wrapper.

HEAD uses the same lookup; inspect status rather than a body. Missing templates return 404; lookup errors return 400. See [Create template](create_template.md) for the definition format.
