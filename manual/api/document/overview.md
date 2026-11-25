# Document

Documents are JSON objects identified by an ID within an index. Prefer batch ingestion for larger workloads.

## Document reference

<div class="grid cards" markdown="1">

- **Write a document**

    ---

    [Index a document](index_document.md) with an automatic or explicit ID, or [update a document](update_document.md) with a complete replacement.

- **Read or delete**

    ---

    [Get or check a document](get_document.md) by ID, [delete one document](delete_document.md), or [delete a search result page](delete_by_query.md).

</div>

## Choose a batch format

| API | Request format | Use |
| --- | --- | --- |
| [Bulk](bulk.md) | Action/source NDJSON | Index, create, update, and delete actions. |
| [Bulk V2](bulk_v2.md) | JSON object with a records array | Insert or replace multiple documents. |
| [Multi](multi.md) | Source-only NDJSON | Index one document per line, without action metadata. |

Writes can automatically create missing indexes. Create mappings first when analysis or field types matter. Updates replace documents; they do not perform Elasticsearch partial-document merging or scripts.

!!! note "Write acknowledgment and visibility"
    With WAL enabled, a write response can precede search visibility. Do not rely on `refresh=wait_for`; verify the intended results as shown in the [tutorial](../query_dsl/getting_started.md#wait-for-search-visibility).
