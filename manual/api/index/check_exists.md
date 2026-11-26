# Check index existence

=== "ES-compatible"

    ```http
    HEAD /es/{target}
    ```

=== "Native"

    ```http
    HEAD /api/index/{target}
    ```

`target` is one concrete index name. No query parameters or request body are needed.

```sh title="Check whether api_books exists"
curl -I -u "$SS_USER:$SS_PASSWORD" "$SS_URL/es/api_books"
```

Check the HTTP status, not a JSON body:

| Status | Meaning |
| --- | --- |
| 200 | Index metadata exists. |
| 404 | Index not found. |
| 500 | Metadata lookup failed. |

Existence does not prove data is searchable or storage is healthy. For cluster access differences, see [Cluster API usage](../cluster.md).
