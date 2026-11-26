# Version

Read the version and build information of a SeaSearch Server.

## Request

```http
GET /version
```

Available directly on SeaSearch Server, without authentication. See [Cluster API usage](../cluster.md) for endpoint availability.

No request parameters or body are required.

## Responses

HTTP 200 returns an object containing string build fields:

| Field | Meaning |
| --- | --- |
| `version` | SeaSearch version. |
| `build` | Build identifier. |
| `commit_hash` | Source commit identifier. |
| `branch` | Build branch. |
| `build_date` | Build date string. |

Values depend on the build. A build without supplied metadata returns these defaults:

```json title="Build metadata · Defaults when not injected"
{"version":"v0.0.0","build":"0","commit_hash":"0","branch":"0","build_date":"0"}
```

## Example

Point `SS_SERVER_URL` directly at the Server endpoint you want to inspect:

```sh title="Read Server build metadata"
export SS_SERVER_URL='http://localhost:4080'
curl "$SS_SERVER_URL/version"
```
