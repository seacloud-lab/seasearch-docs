# Recall

Evaluate IVF-PQ approximate neighbors against exact comparisons on sampled stored vectors. A recall value closer to 1 indicates better agreement for that sample.

```http
POST /api/{target}/_recall
```

`target` is one index name. No query parameters are consumed.

| Body parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `field` | string | — | Required vector field. |
| `k` | integer | `10` | Positive neighbors per sample query. |
| `nprobe` | integer | `5` | Positive IVF clusters to probe. |
| `query_count` | integer | `100` | Positive number of sampled query vectors. |

```sh title="Evaluate IVF-PQ recall on sampled vectors"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X POST "$SS_URL/api/api_vectors/_recall" \
  -H 'Content-Type: application/json' \
  --data-binary '{"field":"vec","k":10,"nprobe":5,"query_count":100}'
```

HTTP 200 example:

```json title="HTTP 200 · Illustrative recall value"
{"recall":0.95}
```

Results depend on the sampled data, trained segments, and search parameters.

!!! note "Supported index types"
    For `flat`, the endpoint returns 1 directly rather than running this evaluation. **HNSW recall evaluation is not supported**, even though HNSW search is available since v1.1.

## Evaluation limits

- Use representative, nonempty data with enough vectors for `k`.
- `query_count` must not exceed the number of available vectors in any evaluated segment.
- Growing segments report recall 1 without approximate evaluation.
- Larger query counts and probes increase evaluation cost.

Invalid JSON, missing/incorrect fields, and nonpositive counts return 400; evaluation failures return 500.
