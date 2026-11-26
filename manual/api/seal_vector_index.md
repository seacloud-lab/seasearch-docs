# Seal a vector index

Queue sealing of an eligible growing IVF-PQ segment. Sealing prepares a segment for trained approximate search; it is not a general flush or document-visibility operation.

```http
POST /api/{target}/{field}/_seal
```

`target` is one index name and `field` is its vector field. No request body or query parameters are consumed.

## Example

Run against an IVF-PQ index with enough vectors:

```sh title="Submit an IVF-PQ sealing task"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X POST "$SS_URL/api/api_vectors/vec/_seal"
```

!!! note "Submission is not completion"
    Success is HTTP 200 with an **empty body**. It means a task was submitted, not that training/sealing finished. Background checks also submit eligible work.

## Requirements and errors

- Only `ivf_pq` supports sealing. Flat and HNSW do not need this operation.
- A growing segment must reach `SS_VECTOR_IVFPQ_THRESHOLD`, default 100000 vectors.
- A missing index returns 404. A missing/non-vector field, an ineligible index, insufficient vectors, or already-sealing state returns 400 with an `error` string.

There is no task-status or wait-for-completion parameter. Monitor index metadata/logs before treating training as complete. Proxy route differences are documented in [Cluster API usage](cluster.md).
