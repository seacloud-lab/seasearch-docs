# Vector search API

Search embeddings stored in a mapped vector field. Use document APIs to index vectors, then query the nearest records. See the [complete example](vector_search_example.md).

## Create a vector index

<span id="mapping-request"></span>

Configure a vector field at [index creation](index/create_index.md) or with [Update mapping](index/update_mapping.md):

```json title="Mapping · Four-dimensional flat vector field"
{"mappings":{"properties":{
  "vec":{"type":"vector","dims":4,"vec_index_type":"flat"},
  "category":{"type":"keyword"}
}}}
```

<span id="mapping-parameters"></span>

| Mapping option | Type | Description |
| --- | --- | --- |
| `type` | string | Required: `vector`. |
| `dims` | integer | Required positive vector dimension. All indexed/query vectors must match it. |
| `vec_index_type` | string | Required: `flat`, `ivf_pq`, or `hnsw` (supported since v1.1). |
| `m` | integer | Required for `ivf_pq`: positive, and must divide `dims`. |
| `nbits` | integer | Required positive PQ code width for `ivf_pq`; use a backend-supported width, commonly 4 or 8. |
| `store_with_float16` | boolean | Optional, default false. Reduces precision of stored vectors on supported storage paths. |

<span id="choose-an-index-type"></span>

`flat` uses exact distance comparison and is suitable for small datasets. `ivf_pq` reduces memory using approximate search over trained/sealed segments; growing segments are searched directly. `hnsw`, available since v1.1, uses a graph with background rebuilding. Choose an index type based on dataset size, memory usage, latency, and recall requirements.

For IVF-PQ, eligible growing segments are sealed in the background after reaching `SS_VECTOR_IVFPQ_THRESHOLD` (default 100000). See [Seal vector index](seal_vector_index.md). HNSW rebuilding is automatic and does not use this seal endpoint.

## Index documents containing vectors

Use [Index document](document/index_document.md), [Bulk](document/bulk.md), or other ingestion APIs with complete numeric arrays. Omit document IDs to generate valid vector IDs; supplied IDs must be canonical base62 strings.

Vectors are stored separately and normally removed from `_source`; retaining an ordinary source field does not require `store: true`. Replacements must include the vector when it should remain part of the record.

## Search vectors

```http
POST /api/{target}/_search/vector
```

Use a concrete index name as `target`, or compatible indexes with the same vector field/dimension. No URL query parameters are consumed.

<span id="search-parameters"></span>

| Body parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `query_field` | string | — | Required mapped vector field. |
| `vector` | number array | — | Required query vector with exactly `dims` entries. |
| `k` | integer | `10` | Positive maximum nearest results to select. |
| `nprobe` | integer | `1` | Positive number of IVF clusters to probe; ignored by flat/HNSW search. |
| `filter_query` | object | None | Inner Query DSL condition used to select candidate documents first. |
| `return_fields` | array | None | Selected source fields returned under each hit's `fields`; same entries as Search `fields`. |
| `_source` | boolean or string array | `true` | Ordinary source-field selection. |

```sh title="Find nearest vectors within a category"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X POST "$SS_URL/api/api_vectors/_search/vector" \
  -H 'Content-Type: application/json' \
  --data-binary '{"query_field":"vec","vector":[1,0,0,0],"k":5,"return_fields":["name"],"filter_query":{"term":{"category":"reference"}}}'
```

## Responses

HTTP 200 uses the [Search response shape](search/search.md#responses). Hits are sorted by vector similarity; scores are calculated as `1 / (1 + distance)`. Search uses L2-based distance; cosine and dot-product modes are not configurable.

Only documents retrieved from the ordinary document index can appear as hits. Vector selection can therefore return fewer than `k` records. Use globally distinct document IDs when querying multiple indexes to avoid ambiguous ID-based result association.

## Filtering and source options

`filter_query` is a Query DSL object, for example `{"term":{"category":"reference"}}`, not a full Search body. On a single Server, candidates come from ordinary search, capped by `ZINC_MAX_RESULTS` (default 10000), sorted by ID before vector comparison. A broad filter can therefore exclude valid neighbors beyond that candidate window.

`return_fields` and `_source` use [Search source/field selection](search/search.md#source-selection-and-returned-fields); they do not return the stored embedding itself. Cluster vector execution has separate limits; see [Cluster API usage](cluster.md).

## Errors and limitations

- Invalid JSON, a missing/non-vector field, or a dimension mismatch returns 400. No matching index returns 404. Execution/storage failures generally return 500.
- `k` and `nprobe` must be positive integers.
- Approximate search does not guarantee exact top-k ordering. Tune IVF `nprobe` and evaluate with [Recall](recall.md).
- Recall and seal are IVF-PQ maintenance tools. For HNSW cluster search, see [Cluster API usage](cluster.md#vector-search).
- Write acknowledgment and vector/full-text visibility can differ; verify both before a dependent workflow.

## Recall Query

The recall operation has moved to [Recall](recall.md), with supported index types, defaults, and example requests.
