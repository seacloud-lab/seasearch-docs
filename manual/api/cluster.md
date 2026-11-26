# Cluster API usage

Use SeaSearch-Proxy as the client endpoint for cluster workloads. It routes writes to assigned Servers and coordinates searches. The ordinary API pages describe single-node behavior; this page collects the important cluster differences.

For deployment instructions, see [Cluster deployment](../usage/deploy_cluster.md).

## Endpoint availability

Most index/document paths are forwarded, while some operations are handled directly by the Proxy. Do not assume every Server route is also registered on it:

- `/version`, `/healthz`, `/es/`, `/_license` and `/_xpack` under `/es/`, aliases, and data-stream compatibility routes are not Proxy routes.
- Server diagnostics are not exposed through Proxy routes.
- Vector seal is registered on the Server as `POST /api/{target}/{field}/_seal`. The current Proxy registers `POST /api/{target}/{field}/_rebuild` instead, without a matching Server route; do not rely on either path being transparently forwarded through the Proxy.

An HTTP 406 from a Server can indicate that the requested index is assigned elsewhere. Use the Proxy rather than guessing the assigned Server.

## Authentication

Use the same [Basic credentials](authentication.md) on client requests. Forwarded operations are authenticated by the backend. Operations handled directly by the Proxy have their own checks.

One exception is Proxy `HEAD /es/{target}`, which reads index metadata without the ordinary Server authentication middleware. Do not treat index-existence responses as an access-control boundary.

Coordinated DSL search uses the backend permissions `search.internal.UnifySearch` and, when statistics are collected across nodes, `search.internal.GetStats`. Granting `search.SearchDSL` alone does not authorize this workflow. These permissions are listed here to configure roles; the internal request protocol is not a public client API.

## Search differences

Proxy `POST /es/{target}/_search` and `POST /es/_search` use the Unified Search chain. This differs from ordinary Server search:

| Feature | Cluster behavior |
| --- | --- |
| Scoring | Uses shared query statistics for coordinated search. |
| Targets | Prefer concrete names or comma-separated names. Do not rely on Server alias or wildcard expansion. |
| Sorting | Final results are merged by descending score. Field-sort requests do not provide equivalent global ordering. |
| Aggregations | Multi-index aggregations are rejected; the final result merger does not preserve aggregation results even for a single-index DSL request. Use a suitable direct Server request where applicable. |
| Query support | Scoring queries must support Unified term-statistics collection. Ordinary DSL support alone is insufficient. |

In particular, omit `match.fuzziness` and `match.prefix_length`: their presence is rejected during statistics collection, even with a value of 0. Place range conditions used only to restrict documents inside `bool.filter`.

Use the same scoring query and compatible mappings/analyzers across indexes; only document filters should differ. For the explicit multi-index request format, see [Unified search](search/unified_search.md).

## Vector search

The Proxy selects one index by name for vector search. Flat and HNSW requests are forwarded to an assigned Server; sufficiently segmented IVF-PQ indexes can use parallel segment searches.

!!! warning "Parallel IVF-PQ request options"
    The parallel path does not pass `filter_query`, `return_fields`, or `_source` selection through to the equivalent single-Server workflow. Do not rely on these options to filter or redact results when parallel vector execution is possible. Apply authorization outside the query and use a deployment path whose behavior you have verified.

Parallel vector requests can require `search.internal_vector_search` and `search.internal.UnifySearch` backend permissions. Ordinary forwarded vector requests use `search.SearchVector`. HNSW partial-segment search is not supported, and the Proxy does not use it for HNSW.

## Management and diagnostics

The Proxy forwards `GET`, `POST`, `PUT`, and `DELETE /api/cluster/nodes` to Cluster Manager. These management routes do **not** enforce SeaSearch Basic authentication. Restrict them with network controls or an authenticated administrative gateway.

`POST /api/cluster/clean` is available only directly on Manager and deletes membership/assignment metadata. It is not normal document cleanup.

See [Cluster nodes](system/cluster_nodes.md), [Cluster cleanup](system/cluster_cleanup.md), and the [Operations appendix](system/operations.md). Keep Manager and backend diagnostic ports private.
