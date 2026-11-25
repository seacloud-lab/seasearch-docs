# Roles

Roles contain exact permission identifiers. The special role `admin` grants all checked operations and cannot be created/updated with this API.

## Create or update

```http
POST /api/role
PUT /api/role
```

Both methods create or replace the role's name and permission list by `_id`. No query parameters are used.

| Body parameter | Type | Description |
| --- | --- | --- |
| `_id` | string | Required ID; normalized to lowercase. `admin` is reserved. |
| `name` | string | Display name. |
| `permission` | string array | Complete permission list; omission does not preserve an old list. |

```sh title="Create or replace the reader role"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X POST "$SS_URL/api/role" \
  -H 'Content-Type: application/json' \
  --data-binary '{"_id":"reader","name":"Book reader","permission":["search.SearchDSL","document.Get"]}'
```

HTTP 200:

```json title="HTTP 200 · Role saved"
{"message":"ok","id":"reader"}
```

The list is not an index-level access policy. Use exact IDs from [Permissions](permissions.md); wildcards such as `search.*` are not expanded.

## List and delete

```http
GET /api/role
DELETE /api/role/{id}
```

No body or query parameters. GET returns role objects with `_id`, `name`, `role`, `permission`, `created_at`, and `updated_at`. DELETE returns HTTP 200 with `{"message":"deleted","id":"reader"}`. Deleting a role does not reassign its users.

## Errors and permissions

Create/update, list, and delete require `auth.CreateUpdateRole`, `auth.ListRole`, and `auth.DeleteRole`. Invalid JSON/missing ID returns 400. Reserved-admin or metadata errors can return 500 with an `error` string. Protect administrative credentials carefully.
