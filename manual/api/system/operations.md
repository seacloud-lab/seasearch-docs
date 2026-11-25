# Operations appendix

SeaSearch Server exposes optional Prometheus metrics and Go pprof runtime diagnostics.

!!! warning "Diagnostics are not protected by API roles"
    pprof and enabled Prometheus endpoints are accessible without SeaSearch authentication. They can expose runtime details or consume significant resources. Keep Server ports private and restrict administrative access. These diagnostics are available directly on Server, not through Proxy or Cluster Manager.

## Prometheus metrics

### Request

```http
GET /metrics
```

Enable metrics by starting the Server with `ZINC_PROMETHEUS_ENABLE=true` (default: `false`). No request parameters or body are required.

### Responses and version differences

HTTP 200 returns metrics in Prometheus exposition format. Metric values and available time series depend on runtime activity.

| Version | Application metric prefix | Example custom metric |
| --- | --- | --- |
| v1.0 | `zinc` | `zinc_index_stats{index="...",field="..."}` |
| v1.1 | `seasearch` | `seasearch_index_stats{index="...",field="..."}` |

HTTP metrics use the same prefix, including `requests_total`, `request_duration_seconds`, `request_size_bytes`, and `response_size_bytes` families. Go/process metrics use their own names. Update dashboard selectors from `zinc_*` to `seasearch_*` when moving to v1.1.

The custom index statistics metric is a labeled gauge family. A missing series is not necessarily zero. HTTP request metrics include URL-related labels; consider cardinality and sensitive path information when retaining or sharing scrapes.

### Example

```sh title="Scrape enabled Prometheus metrics"
curl "$SS_SERVER_URL/metrics"
```

The endpoint is unavailable when metrics are disabled.

## pprof runtime diagnostics

### Paths

The following pprof routes are enabled independently of `ZINC_PROFILER`.

| Method | Path | Purpose |
| --- | --- | --- |
| GET | `/debug/pprof/` | Profile index. |
| GET | `/debug/pprof/cmdline` | Process command line. |
| GET | `/debug/pprof/profile` | CPU profile collection. |
| GET | `/debug/pprof/symbol` | Symbol information endpoint. |
| POST | `/debug/pprof/symbol` | Symbol address resolution endpoint. |
| GET | `/debug/pprof/trace` | Runtime execution trace collection. |
| GET | `/debug/pprof/allocs` | Allocation profile. |
| GET | `/debug/pprof/block` | Blocking profile. |
| GET | `/debug/pprof/goroutine` | Goroutine profile. |
| GET | `/debug/pprof/heap` | Heap profile. |
| GET | `/debug/pprof/mutex` | Mutex profile. |
| GET | `/debug/pprof/threadcreate` | Thread-creation profile. |

Profiling parameters and symbol-resolution payloads follow the Go runtime's pprof interface.

### Responses and example

The profile index is HTML, profile/trace downloads are binary, and some diagnostic output is text.

The following lists available profiles without starting CPU or trace collection:

```sh title="List available runtime profiles"
curl "$SS_SERVER_URL/debug/pprof/"
```

Run profile and trace collection as part of an operator-controlled diagnostic procedure.

## Limitations

- Diagnostics describe one Server process, not the entire cluster.
- Prometheus enablement does not enable or disable pprof.
