# List indexes

```http
GET /api/index
```

List index metadata, optionally filtered, sorted, and paginated. No request body is needed.

## Query parameters

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `name` | string | Empty | Case-sensitive substring of the index name. |
| `sort_by` | string | `name` | `name`, `doc_num`, `shard_num`, `storage_size`, `storage_type`, or `wal_size`. Unknown values fall back to name. |
| `desc` | boolean | `false` | Descending sort. |
| `page_num` | integer | `1` | One-based page number. |
| `page_size` | integer | `0` | Items per page; 0 returns all. Use nonnegative values. |

## Example and response

```sh title="List a page of matching indexes"
curl -u "$SS_USER:$SS_PASSWORD" \
  "$SS_URL/api/index?name=api_books&page_num=1&page_size=20"
```

HTTP 200 returns `list` containing [index metadata objects](get_index.md), and `page`:

```json title="HTTP 200 · Empty result page"
{"list":[],"page":{"page_num":1,"page_size":20,"total":0}}
```

`total` is the number of indexes after filtering, not the page length. Metadata errors return 500 with an `error` string. Use ordinary in-range page numbers; this API does not provide cursor pagination.
