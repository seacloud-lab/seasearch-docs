# Get an index

Read an index's configuration and stored metadata.

```http
GET /api/index/{target}
```

`target` is one concrete index name. No query parameters or request body are needed.

## Example

```sh title="Read api_books metadata"
curl -u "$SS_USER:$SS_PASSWORD" "$SS_URL/api/index/api_books"
```

HTTP 200 returns the index metadata object. Main fields include:

| Field | Meaning |
| --- | --- |
| `name` | Index name. |
| `storage_type` | Storage backend. |
| `mappings`, `settings` | Configured fields and analysis. |
| `stats` | `doc_num`, `storage_size`, `wal_size`, `doc_time_min`, and `doc_time_max`. Sizes are bytes; document times use internal nanosecond timestamps. |
| `shard_num`, `shards` | Physical shard metadata. |
| `vec_indexes` | Vector index metadata, when present. |
| `version` | Stored index format/build metadata. |

Statistics are not a substitute for verifying search visibility. Internal layout fields can change; use dedicated mapping/settings APIs when only configuration is needed.

## Errors

A missing index returns 404 with an `error` string; metadata errors return 500. There is no equivalent `GET /es/{target}` route.
