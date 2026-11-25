# SeaSearch API

<div class="ss-hero" markdown="1">

Index · Query · Discover
{ .ss-eyebrow }

SeaSearch provides document indexing, full-text search, and vector search through native APIs and a supported subset of Elasticsearch-compatible APIs.

[Start the tutorial](query_dsl/getting_started.md){ .md-button .md-button--primary }
[Explore Query DSL](query_dsl/overview.md){ .md-button }

</div>

## Start here

<div class="grid cards" markdown="1">

- **1. Connect**

    ---

    Set your service address and credentials before sending a request.

    [Configure authentication →](authentication.md)

- **2. Index your data**

    ---

    Create an index with explicit mappings, then import a small dataset.

    [Create an index →](index/create_index.md)

- **3. Run a search**

    ---

    Follow a complete example of full-text queries and exact-value filters.

    [Follow the tutorial →](query_dsl/getting_started.md)

</div>

## Reference

<div class="grid cards" markdown="1">

- **[Index](index/overview.md)**

    Mappings, field types, text analysis, settings, templates, and aliases.

- **[Document](document/overview.md)**

    Individual and batch writes, document retrieval, updates, and deletion.

- **[Search](search/search.md)**

    Requests, responses, pagination, highlighting, and aggregations.

- **[Query DSL](query_dsl/overview.md)**

    Full-text queries, exact values, term patterns, and compound conditions.

- **[Vector search](vector_search_api.md)**

    Vector search requests, [sealing](seal_vector_index.md), and [recall](recall.md).

- **Access and operations**

    [Users](access/users.md), [roles](access/roles.md), [permissions](access/permissions.md), [version](system/version.md), [health](system/health_check.md), and [operations](system/operations.md).

</div>

!!! tip "Running a cluster?"
    Read [Cluster API usage](cluster.md) for management access and search differences before using these examples with SeaSearch-Proxy.

## Request conventions

The ordinary reference pages describe single-node behavior. For a cluster, use SeaSearch-Proxy and consult the [cluster differences](cluster.md).

The `/es/` prefix identifies supported compatible routes, not an automatic translation of every Elasticsearch API. For example, DSL search uses:

```http
POST /es/api_books/_search
Content-Type: application/json
```

```json
{"query":{"match_all":{}},"size":10}
```

For the native request format, see [Native search](search/native_search.md).

Examples use a disposable `api_books` index and the shell variables defined in [Authentication](authentication.md). Dynamic scores, times, IDs, and statistics are illustrative. Write acknowledgment does not always mean search visibility; inspect batch errors and verify data before dependent operations.

Functionality introduced in a later version is marked where relevant, such as HNSW support since v1.1. Use [Version](system/version.md) to identify a running Server.
