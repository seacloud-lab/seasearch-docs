# Refresh

Close an index's resources so they can be reopened on subsequent access.

=== "ES-compatible"

    ```http
    POST /es/{target}/_refresh
    ```

=== "Native"

    ```http
    POST /api/index/{target}/refresh
    ```

`target` is one index name. No body or query parameters are required.

```sh title="Close index resources for reopening"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X POST "$SS_URL/es/api_books/_refresh"
```

HTTP 200:

```json title="HTTP 200 · Refresh response"
{"message":"ok"}
```

A missing index or close error returns 400 with an `error` string.

!!! note "Asynchronous writes"
    Refresh does not wait for pending asynchronous writes to become searchable. Verify expected results before a dependent operation; see the [tutorial's visibility check](../query_dsl/getting_started.md#wait-for-search-visibility).
