# Update index aliases

Add or remove associations between alias names and existing indexes.

```http
POST /es/_aliases
```

No query parameters are consumed. Send an `actions` array; each action contains an `add` or `remove` object:

| Member | Type | Description |
| --- | --- | --- |
| `index` | string | One index name; takes precedence over `indices`. |
| `indices` | string array | Alternative list of index names. |
| `alias` | string | One alias name; takes precedence over `aliases`. |
| `aliases` | string array | Alternative list of alias names. |

## Example

```sh title="Associate books_current with api_books"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X POST "$SS_URL/es/_aliases" \
  -H 'Content-Type: application/json' \
  --data-binary '{"actions":[{"add":{"index":"api_books","alias":"books_current"}}]}'
```

HTTP 200:

```json title="HTTP 200 · Alias acknowledgment"
{"acknowledged":true}
```

## Limitations

Missing indexes are skipped rather than created. Use explicit names; supported wildcard matching is not full Elasticsearch glob syntax. Actions are not a transaction: additions are applied before removals, rather than preserving an interleaved input order.

Alias filters, routing, and write-index selection are not supported. Alias expansion is limited to the Server search, multi-search, and delete-by-query target middleware; do not use aliases as general document-write paths. Cluster availability is covered in [Cluster API usage](../cluster.md).

Malformed JSON returns 400 and metadata-list failures can return 500. Verify associations with [Get aliases](get_aliases.md), rather than treating acknowledgment as validation of every intended association.
