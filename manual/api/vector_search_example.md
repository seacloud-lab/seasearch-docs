# Vector search example

Create a small four-dimensional dataset and search it without an external embedding model. Use the credentials and `SS_URL` from [Authentication](authentication.md). Choose a fresh disposable `api_vectors` index.

## Creating the SeaSearch Index and Vector Index

```sh title="1. Create a four-dimensional flat vector index"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X PUT "$SS_URL/es/api_vectors" \
  -H 'Content-Type: application/json' \
  --data-binary '{"mappings":{"properties":{
    "vec":{"type":"vector","dims":4,"vec_index_type":"flat"},
    "name":{"type":"keyword"},
    "category":{"type":"keyword"}
  }}}'
```

`flat` keeps this example independent of training thresholds. To use HNSW, supported since v1.1, select `hnsw` on a new index instead. See [Vector mapping options](vector_search_api.md#create-a-vector-index).

## Indexing Data

Use generated IDs; arbitrary strings such as `book-1` are not valid vector-index IDs.

```sh title="2. Import three vectors with generated IDs"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X POST "$SS_URL/es/api_vectors/_bulk" \
  -H 'Content-Type: application/x-ndjson' \
  --data-binary @- <<'NDJSON'
{"index":{"_index":"api_vectors"}}
{"name":"search-guide","category":"reference","vec":[1,0,0,0]}
{"index":{"_index":"api_vectors"}}
{"name":"vector-handbook","category":"reference","vec":[0.8,0.2,0,0]}
{"index":{"_index":"api_vectors"}}
{"name":"travel-guide","category":"travel","vec":[0,1,0,0]}
NDJSON
```

Inspect the Bulk response for errors and save each returned `_id`. Writes can become searchable asynchronously. Before continuing, repeat a match-all Search with `size: 0` until `hits.total.value` is 3, using a bounded wait in automation:

```sh title="3. Check document search visibility"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X POST "$SS_URL/es/api_vectors/_search" \
  -H 'Content-Type: application/json' \
  --data-binary '{"query":{"match_all":{}},"size":0}'
```

## Searching Data

```sh title="4. Search the nearest reference vectors"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X POST "$SS_URL/api/api_vectors/_search/vector" \
  -H 'Content-Type: application/json' \
  --data-binary '{"query_field":"vec","vector":[1,0,0,0],"k":2,"return_fields":["name"],"filter_query":{"term":{"category":"reference"}}}'
```

After both document and vector data are available, the exact-match vector should rank first. The response uses Search hits; a selected field is returned as `"fields":{"name":["search-guide"]}`. Embeddings themselves are not returned in `_source`. Do not assert exact timing or generated IDs in tests.

## Maintain vector data

### Update a document directly

Use a saved generated ID and send the complete replacement, including its vector:

```sh title="Replace a saved vector document"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X PUT "$SS_URL/es/api_vectors/_doc/$VECTOR_DOC_ID" \
  -H 'Content-Type: application/json' \
  --data-binary '{"name":"search-guide","category":"reference","vec":[0.9,0.1,0,0]}'
```

Set `VECTOR_DOC_ID` to an actual `_id` from your response before running this command.

### Query Then Update

If the ID was not saved, find it through an ordinary keyword query:

```json title="Find a document ID by business name"
{"query":{"term":{"name":"search-guide"}},"size":10}
```

Submit that body to `POST /es/api_vectors/_search`, inspect all intended matches, and use the returned IDs for replacements.

### Replace all vector documents for a paper

For chunked data, add a keyword business identifier such as `paper-id`. Multiple vector records can share it, but each retains its own document ID. Search the business identifier, enumerate the intended records within pagination limits, delete those IDs, and index new vectors. Bulk deletion/insertion is not transactional; verify completion before relying on the new dataset. Do not use Delete by query's counters as proof that all old chunks were removed.
