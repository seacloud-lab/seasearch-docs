<span id="api-authentication"></span>

# Authentication

Protected SeaSearch APIs use **HTTP Basic authentication**. Send your username and password with each request; no login token is required.

## Using curl

The examples throughout this reference use these shell variables:

```sh title="Set credentials and send an authenticated request"
export SS_URL='http://localhost:4080'
export SS_USER='your-user'
export SS_PASSWORD='your-password'

curl -u "$SS_USER:$SS_PASSWORD" "$SS_URL/api/index"
```

Alternatively, send `Authorization: Basic <base64(username:password)>`. Base64 is an encoding, not encryption.

!!! warning "Protect credentials and administrative endpoints"
    Use HTTPS across untrusted networks. Do not expose backend, Cluster Manager, or debugging ports directly to the public internet. Management and diagnostics have different access boundaries; see [Cluster API usage](cluster.md) and the [Operations appendix](system/operations.md).

## Roles and permissions

Users are assigned a role. The `admin` role bypasses individual permission checks; other roles need the exact operation permission. Native and ES-compatible paths can require different permissions even when they perform the same operation.

`GET /api/permissions` lists registered permission identifiers. Common examples include:

| Operation | Native permission | ES-compatible permission |
| --- | --- | --- |
| Create index | `index.Create` | `index.CreateES` |
| Index document | `document.Create` | `document.CreateUpdate` |
| Bulk | `document.Bulk` | `document.ESBulk` |
| Search | `search.SearchV1` | `search.SearchDSL` |

Permissions control operations, not per-index ownership. Do not treat an index name as an authorization boundary.

For user/role administration, see [Users](access/users.md), [Roles](access/roles.md), and [Permissions](access/permissions.md). [Login](access/login.md) validates credentials but does not create a session for subsequent requests.

## Authentication errors

| Status | Meaning | Typical response |
| --- | --- | --- |
| 401 | No credentials | `{"auth":"Missing credentials"}` |
| 401 | Credentials invalid | `{"auth":"Invalid credentials"}` |
| 403 | Role lacks permission | `{"error":"No permission:search.SearchDSL"}` |

Version, health, and client-probe routes do not all use this middleware. Administrative and diagnostic exceptions are described on their own pages. Cluster search permissions are explained centrally in [Cluster API usage](cluster.md).

## Regular Users

See [Users](access/users.md) to create a non-administrator account and [Roles](access/roles.md) to assign operation permissions.
