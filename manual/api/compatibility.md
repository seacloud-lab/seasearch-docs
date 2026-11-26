# Client compatibility

SeaSearch implements selected Elasticsearch-compatible endpoints, not a complete Elasticsearch server. Use the methods, body formats, and supported options in this reference. A product header or successful client probe is not a guarantee that every subsequent client operation works.

## Probe endpoints

The following Server routes require no Basic credentials and take no body or query parameters:

| Method and path | Response |
| --- | --- |
| `GET /es/` | Product/version information. |
| `HEAD /es/` | Product probe; inspect status/headers rather than a body. |
| `GET /es/_license` | `{"status":"active"}`. |
| `GET /es/_xpack` | `{"build":{},"features":{},"license":{"status":"active"}}`. |

These routes add `X-elastic-product: Elasticsearch`. License/XPack responses are compatibility placeholders, not Elastic licensing or XPack feature implementations.

`GET /es/` returns `name`, `cluster_name`, `cluster_uuid`, a `version` object, and `tagline`. Version fields include `number`, `build_flavor`, `build_hash`, `build_date`, `build_snapshot`, `lucene_version`, `minimum_wire_version`, and `minimum_index_compatibility`. Several values are placeholders. The reported version can be selected from an Elastic client User-Agent or overridden with `ZINC_PLUGIN_ES_VERSION`; use [Version](system/version.md) for actual SeaSearch build identity.

```sh
curl "$SS_URL/es/"
```

## Data-stream compatibility responses

```http
PUT /es/_data_stream/{target}
GET /es/_data_stream/{target}
HEAD /es/_data_stream/{target}
```

These routes use normal [authentication](authentication.md): PUT requires `elastic.PutDataStream`, and GET/HEAD require `elastic.GetDataStream`. `target` supplies the returned name; request bodies and query parameters are not used.

PUT returns HTTP 200:

```json
{"name":"example-stream","message":"ok"}
```

GET returns HTTP 200:

```json
{"data_streams":[{"name":"example-stream","timestamp_field":{"name":"@timestamp"}}]}
```

These responses do not create or inspect a persisted data stream. They do not provide backing indexes, rollover, lifecycle management, or reliable existence checks. HEAD follows the same placeholder behavior.

## Practical limits

- Do not translate an arbitrary ES API by prepending `/es/`: methods and available paths differ.
- Extra request options may be ignored. Use only documented options; successful responses alone do not validate compatibility.
- Version/sequence/acknowledgment fields in compatible write responses do not imply ES concurrency or replica guarantees.
- Proxy endpoint availability and cluster search restrictions are documented in [Cluster API usage](cluster.md).
- No guarantee is made for all Elasticsearch client versions. Test the exact client operations and deployment you intend to use.
