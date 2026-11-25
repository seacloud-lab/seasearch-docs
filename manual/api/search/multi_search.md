# Multi-search

Run multiple search requests in one NDJSON body.

```http
POST /es/_msearch
POST /es/{target}/_msearch
```

`target` supplies default comma-separated index names. No URL query parameters are consumed.

## Body format

Alternate a metadata line and a [Search body](search.md#request-body), ending with a newline. Metadata `index` can be a string or string array; omit it to use the path default. Specify indexes explicitly for predictable scope.

```sh title="Run two searches with metadata/body NDJSON"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X POST "$SS_URL/es/_msearch" \
  -H 'Content-Type: application/x-ndjson' \
  --data-binary @- <<'NDJSON'
{"index":"api_books"}
{"query":{"match":{"title":"search"}},"size":10}
{"index":["api_books"]}
{"query":{"term":{"category":"travel"}},"size":10}
NDJSON
```

## Responses and errors

HTTP 200 contains `responses`, an array of Search responses. An individual failed search can instead contain an `error` string. Inspect every response, not only the HTTP status.

Malformed metadata/body pairs and oversized lines are not consistently reported as one error per request. Do not rely on response position matching malformed input. Each line is limited by `ZINC_MAX_DOCUMENT_SIZE` (default 1 MiB). Multi-search is not transactional, and per-item URL options, Elasticsearch routing metadata, and concurrency parameters are not supported.

Cluster handling differs; see [Cluster API usage](../cluster.md).
