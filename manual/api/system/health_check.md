# Health check

Check that the Server can respond to an HTTP request.

## Request

```http
GET /healthz
```

Available directly on SeaSearch Server, without authentication. See [Cluster API usage](../cluster.md) for endpoint availability.

No request parameters or body are required.

## Responses

HTTP 200:

```json title="HTTP 200 · Liveness response"
{"status":"ok"}
```

`status` is a string with the fixed value `ok`.

## Example

```sh title="Check Server liveness"
export SS_SERVER_URL='http://localhost:4080'
curl "$SS_SERVER_URL/healthz"
```

## Limitations

This endpoint does not check storage, metadata, indexes, or cluster dependencies. Use application-level read/write checks when those dependencies must be verified for readiness.
