# Cluster nodes

List or modify the node membership stored by Cluster Manager. This is a management API, not an index or document API.

!!! warning "Restrict management access"
    These routes do not register SeaSearch HTTP Basic authentication. SeaSearch-Proxy forwards them to Cluster Manager without adding API permission checks. Protect both endpoints with network restrictions and, where needed, an authenticated administrative gateway. Sending a Basic header does not make the underlying routes access-controlled.

## Requests

| Method and path | Operation |
| --- | --- |
| `GET /api/cluster/nodes` | List configured nodes and heartbeat presence. |
| `POST /api/cluster/nodes` | Add node entries. |
| `PUT /api/cluster/nodes` | Replace URLs of matching node IDs. |
| `DELETE /api/cluster/nodes` | Remove matching node IDs. |

All four paths are registered on Cluster Manager and forwarded by SeaSearch-Proxy. They are not registered by the ordinary Server router. No API permission identifier is required by these route registrations.

### Path parameters

None.

### Query parameters

None are consumed by the Manager handlers.

### Request body

GET requires no body. For writes, send a JSON object with `Content-Type: application/json`.

#### Add nodes: POST

```json
{"nodes":[{"id":1,"url":"http://server-1:4080"}]}
```

| Parameter | Type | Required | Behavior |
| --- | --- | --- | --- |
| `nodes` | object array | Supply it | Node entries to append. The handler does not require a nonempty array; omission is not a useful add operation. |
| `nodes[].id` | integer | Yes for each entry | Must be nonzero and not already present in the stored membership. There is no positive-only validation. Use deliberate, distinct IDs. |
| `nodes[].url` | string | Yes for each entry | Nonempty backend address. It is passed to Go URL parsing, but the handler does not verify reachability or require an absolute HTTP URL. Supply a reachable absolute `http://` or `https://` address. |

An existing ID is rejected, not updated. Duplicate new IDs within one request are not reliably rejected by the existing-membership check; do not submit them. No node connectivity test is performed before storing the list.

#### Update nodes: PUT

The body has the same `nodes`, `id`, and `url` shape as POST. Every entry must have a nonzero ID and nonempty parseable URL. Matching stored entries have their URL replaced; an ID absent from the stored list is rejected with HTTP 400, not added. Entries omitted from the request are retained.

#### Delete nodes: DELETE

```json
{"node_ids":[1]}
```

| Parameter | Type | Required | Behavior |
| --- | --- | --- | --- |
| `node_ids` | integer array | Supply it | IDs to remove. Missing IDs are ignored. An omitted/empty array removes no entries. |

This is a DELETE request **with a JSON body**, not a path containing the ID. Do not assume clients or intermediaries that discard DELETE bodies are suitable.

## Responses

### GET: HTTP 200

```json
[{"node_id":1,"url":"http://server-1:4080","alive":true}]
```

| Field | Type | Meaning |
| --- | --- | --- |
| `node_id` | integer | Stored node ID. Note that the response uses `node_id`, while write entries use `id`. |
| `url` | string | Stored node address. |
| `alive` | boolean | Whether a heartbeat entry exists for the node when this request lists heartbeats. It is not an active HTTP probe. |

The handler does not impose response ordering. With no configured entries, the current handler can serialize its uninitialized list as JSON `null` rather than `[]`.

### POST, PUT, DELETE: HTTP 200

Successful Manager write handlers return an **empty body**. They do not return `acknowledged`, a modified-node list, or a task identifier. Read the membership back with GET when verification is needed.

## Examples

The following examples use a restricted management endpoint, not the ordinary Server URL. Add whatever authentication your administrative gateway requires.

Read membership without changing it:

```sh title="Read membership from the management endpoint"
export SS_MANAGER_URL='http://cluster-manager:4081'
curl "$SS_MANAGER_URL/api/cluster/nodes"
```

The following request **changes stored membership**. Replace the example ID and address with an intended, prepared cluster node; do not run it as a connectivity test:

```sh title="Administrative change · Add node 2"
curl -X POST "$SS_MANAGER_URL/api/cluster/nodes" \
  -H 'Content-Type: application/json' \
  --data-binary '{"nodes":[{"id":2,"url":"http://server-2:4080"}]}'
```

Success is HTTP 200 with no response body. PUT uses the same request shape to update node 2; DELETE uses `{"node_ids":[2]}` to remove its membership entry.

## Errors

| Status | Cause |
| --- | --- |
| 400 | Invalid JSON, zero node ID, missing URL, URL parsing error, an already-stored ID on POST, or an unknown ID on PUT. |
| 500 | Membership/heartbeat read or metadata write failure; Proxy forwarding failures can also occur. |

Errors are not uniform: some branches return an empty body, while validation branches return `{"error":"..."}`. Check the HTTP status before parsing JSON. A duplicate existing ID on POST can produce:

```json
{"error":"node id 2 already exist"}
```

## Compatibility and limitations

- These operations update membership metadata. A successful response is not proof of data movement, completed reassignment, a healthy backend, or zero-downtime topology changes.
- PUT is not a replacement of the entire membership list; DELETE is not index deletion or cluster cleanup.
- There are no request parameters for timeouts, revision preconditions, dry-run validation, or rollback. Do not assume concurrent updates are safely merged by these HTTP handlers.
- For the separate destructive metadata-reset operation, see [Cluster cleanup](cluster_cleanup.md).
