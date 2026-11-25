# Create an index

Create an index with optional field mappings and text-analysis settings. All methods below create a **new** index; `PUT` does not update an existing index.

## Request

=== "ES-compatible"

    ```http
    PUT /es/{target}
    ```

=== "Native"

    ```http
    POST /api/index
    PUT /api/index
    PUT /api/index/{target}
    ```

`target` is the index name. Names may contain ASCII letters, digits, `_`, `.`, and `-`, but must not start with `_`.

There are no supported query parameters. Send a JSON object, using `{}` when the path supplies the name and no configuration is needed.

## Request body

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `name` | string | Without a path name | Index name. If provided, it takes precedence over `target`; keep the two consistent. |
| `mappings` | object | No | Field definitions under `properties`. See [Mapping and field types](mapping.md). |
| `settings` | object | No | Text-analysis configuration under `analysis`. See [Text analysis](text_analysis.md). |

Unspecified mappings and settings use the server defaults. `aliases`, `storage_type`, `shard_num`, shard counts, and replica counts are not creation-body options.

## Example

Examples use the service address and credentials from [Authentication](../authentication.md).

```sh title="Create api_books with explicit field mappings"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X PUT "$SS_URL/es/api_books" \
  -H 'Content-Type: application/json' \
  --data-binary '{
    "mappings": {
      "properties": {
        "title": {"type":"text","analyzer":"standard","search_analyzer":"standard","highlightable":true},
        "summary": {"type":"text","analyzer":"standard","search_analyzer":"standard"},
        "category": {"type":"keyword"},
        "price": {"type":"numeric"}
      }
    }
  }'
```

HTTP 200:

```json title="HTTP 200 · ES-compatible"
{"acknowledged":true,"shards_acknowledged":true,"index":"api_books"}
```

Native paths return a simpler response:

```json title="HTTP 200 · Native"
{"message":"ok","index":"api_books"}
```

`index` is the created name. The ES-compatible acknowledgment flags are compatibility values, not replica-allocation status.

## Errors and limitations

- Invalid JSON, an invalid name, a duplicate index, or invalid mapping/analysis configuration returns HTTP 400 with an `error` string.
- Repeating the request does not update the index. Use [Update mapping](update_mapping.md) or [Update settings](update_settings.md) instead.
- Matching [index templates](create_template.md) can contribute defaults.
- Creating an index does not make later asynchronous document writes immediately searchable. Follow the [tutorial](../query_dsl/getting_started.md) for importing and checking data.
