# Create or update an index template

Templates supply mappings and analysis to newly created matching indexes. Updating a template does not modify existing indexes.

```http
PUT /es/_index_template/{target}
POST /es/_index_template
```

`target` is the template name. With POST, provide `name` in the body. A path name takes precedence. No query parameters are consumed.

## Request body

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `name` | string | — | Required only without a path name. |
| `index_patterns` | string array | — | Required matching patterns; use nonempty patterns with a literal prefix, such as `books-*`. |
| `priority` | integer | `0` | Highest-priority matching template is selected. |
| `template` | object | — | Required object containing optional `settings` and `mappings`. |

Mappings and analysis use the same objects as [Create index](create_index.md). Alias definitions and template composition are not supported. `data_stream` is accepted for compatibility but does not enable data-stream management.

## Example

```sh title="Create the books index template"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X PUT "$SS_URL/es/_index_template/books" \
  -H 'Content-Type: application/json' \
  --data-binary '{
    "index_patterns":["books-*"],
    "priority":10,
    "template":{"mappings":{"properties":{"category":{"type":"keyword"}}}}
  }'
```

HTTP 200:

```json title="HTTP 200 · Template saved"
{"message":"ok","template":"books"}
```

## Errors and limitations

Invalid JSON, missing name/patterns/template, unsupported options, or conflicting equal-priority stored patterns returns 400. Use distinct priorities where patterns overlap. Pattern matching is not full Elasticsearch glob behavior; avoid empty or leading-wildcard patterns. There is no simulation endpoint or composed/component-template support.
