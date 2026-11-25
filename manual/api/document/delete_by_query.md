# Delete by query

Search an index and delete documents from the returned result page.

```http
POST /es/{target}/_delete_by_query
```

Use one concrete index as `target`. No URL query parameters are consumed. The body uses the [Search request format](../search/search.md), including `query`, `from`, and `size`.

!!! warning "Limited deletion window"
    This operation does not iterate through all matches. The default size is 10, and the server result-size cap applies. Its reported `total` and `deleted` are calculated from search total matches, so they can **overstate the number actually selected for deletion**. Verify remaining matches instead of trusting those counters.

## Example

This request deletes data. Run only against a disposable index or an approved deletion target:

```sh title="Destructive · Delete the selected travel documents"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X POST "$SS_URL/es/api_books/_delete_by_query" \
  -H 'Content-Type: application/json' \
  --data-binary '{"query":{"term":{"category":"travel"}},"size":100}'
```

HTTP 200 has the following shape; the counters shown are illustrative:

```json title="HTTP 200 · Illustrative deletion counters"
{"took":1,"time_out":false,"total":1,"deleted":1,"batches":0,"version_conflicts":0,"noops":0,"failures":[],"retries":{"bulk":0,"search":0},"throttled_millis":0,"requests_per_second":-1,"throttled_until_millis":0}
```

`took` is milliseconds. `failures` contains failed document IDs. The key is `time_out`, not Search's `timed_out`. Batch, retry, conflict, and throttle fields are compatibility values, not an asynchronous task-management feature.

Invalid JSON/search errors generally return 400; deletion failures can be reported in the 200 response's `failures` array. There is no task API, automatic paging, throttling, conflict policy, or transactional rollback. Omitted query means match-all: always specify the intended condition explicitly.
