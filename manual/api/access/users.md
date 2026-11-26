# Users

Manage API accounts and their assigned roles. See [Authentication](../authentication.md) for the shared authentication mechanism.

## Create or update

```http
POST /api/user
PUT /api/user
```

Both methods create or update by `_id`. No query parameters are used.

| Body parameter | Type | Description |
| --- | --- | --- |
| `_id` | string | Required user ID; normalized to lowercase. |
| `name` | string | Optional display name. |
| `role` | string | Assigned role ID. Supply it on both create and update. |
| `password` | string | Supply a nonempty password for new users. On update, omitted/empty retains the current password. |

Name and role are replaced on update, not merged selectively. Do not omit them when they should be retained.

```sh title="Create or update book_reader"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X POST "$SS_URL/api/user" \
  -H 'Content-Type: application/json' \
  --data-binary '{"_id":"book_reader","name":"Book reader","role":"reader","password":"replace-with-a-strong-password"}'
```

HTTP 200:

```json title="HTTP 200 · User saved"
{"message":"ok","id":"book_reader"}
```

## List

```http
GET /api/user
```

No body or query parameters. Returns an array of users containing `_id`, `name`, `role`, `created_at`, and `updated_at`; passwords and salts are omitted. There is no pagination parameter.

## Delete

```http
DELETE /api/user/{id}
```

No body or query parameters. HTTP 200 example:

```json title="HTTP 200 · User deleted"
{"message":"deleted","id":"book_reader"}
```

## Errors and permissions

Creating/updating, listing, and deleting require `auth.CreateUpdateUser`, `auth.ListUser`, and `auth.DeleteUser`, respectively. Invalid JSON/missing ID returns 400; account-storage errors return 500.
