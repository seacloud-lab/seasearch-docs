# Cluster cleanup

Remove Cluster Manager's node and assignment metadata. This is a destructive administrative reset operation, **not** a document cleanup API.

!!! danger "Do not run against an active cluster"
    The handler deletes the `/cluster/nodes` key and recursively deletes `/cluster/assign/`. It does not coordinate a safe shutdown, ask for confirmation, or provide rollback. Preserve the required metadata and follow an operator-controlled recovery procedure before considering this request.

## Request

```http
POST /api/cluster/clean
```

Available directly on **Cluster Manager**, not through Server or SeaSearch-Proxy.

### Authentication and permissions

No SeaSearch authentication middleware or permission check is registered. Restrict access at the network and administrative gateway layers. This endpoint must not be publicly reachable.

### Path parameters

None.

### Query parameters

None are consumed. There is no dry-run or confirmation parameter.

### Request body

None is consumed. A request body cannot restrict which assignments are deleted.

## Responses

HTTP 200 with an **empty body** when both delete calls complete without an error.

HTTP 500 with an **empty body** if either call fails. The first deletion can already have succeeded when the second fails: the handler does not implement a transaction or restore earlier state.

## Request example

This example is intentionally marked as destructive. Run it only as part of an approved reset/recovery procedure, against the verified Manager address:

```sh title="Destructive · Reset cluster control metadata"
# DESTRUCTIVE: deletes cluster membership and assignment metadata.
curl -X POST "$SS_MANAGER_URL/api/cluster/clean"
```

There is no JSON success object. Inspect the HTTP status and verify the intended recovery state independently.

## Errors

The handler logs metadata deletion errors and returns 500 without exposing their details in the response. Consult Manager logs. Do not blindly retry after a failure: inspect which metadata remains first.

## Compatibility and limitations

This operation does not explicitly delete user indexes, documents, users, or roles. That does **not** make it safe for normal operation: removing membership and assignments disrupts the cluster's control metadata. For ordinary membership changes, use [Cluster nodes](cluster_nodes.md) instead.
