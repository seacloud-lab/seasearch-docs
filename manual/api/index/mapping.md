# Mapping and field types

Mappings define how document values are indexed. Supply `mappings.properties` when [creating an index](create_index.md), or a `properties` object when [adding mappings](update_mapping.md).

```json
{
  "properties":{
    "title":{"type":"text","analyzer":"standard","search_analyzer":"standard"},
    "category":{"type":"keyword"},
    "price":{"type":"numeric"},
    "published":{"type":"date","format":"2006-01-02"}
  }
}
```

## Field types

| Type | Use | Accepted aliases |
| --- | --- | --- |
| `text` | Analyzed strings for full-text search | `match_only_text` |
| `keyword` | Whole-string terms for exact matches, sorting, and aggregation | `constant_keyword` |
| `numeric` | Numeric comparisons, sorting, and metrics | `integer`, `double`, `long`, `short`, `int`, `float` |
| `bool` | Boolean values | `boolean` |
| `date` | Dates and timestamps | `time`, `datetime` |
| `vector` | Embeddings with explicit dimensions | See [Vector search](../vector_search_api.md) |

Aliases use the corresponding SeaSearch type; they do not implement the distinct storage/precision semantics of every Elasticsearch type.

Unmapped values can create dynamic mappings: strings become text (or dates when recognized), numbers become numeric, and booleans become bool. Map important fields explicitly rather than relying on inference.

!!! note "Exact strings and text subfields"
    The default configuration does not create `.keyword` subfields. Map exact-match fields as `keyword`, or explicitly add a keyword subfield to a text field.

## Field options

All options except `type` are optional.

| Option | Type | Default | Purpose |
| --- | --- | --- | --- |
| `type` | string | — | Required for a leaf field; selects a type above. |
| `index` | boolean | `true` | Include the value in the searchable index. |
| `store` | boolean | `false` | Store a separate field value; vector fields default to true. Not required for `_source`. |
| `sortable` | boolean | False for text, true otherwise | Enable field sorting. |
| `aggregatable` | boolean | False for text, true otherwise | Enable aggregation values. |
| `highlightable` | boolean | `false` | Enable highlighting; also enables stored values. |
| `analyzer` | string | No explicit analyzer | Text index-time analyzer. |
| `search_analyzer` | string | No explicit analyzer | Text search-time analyzer. Set both names when using custom analysis. |
| `format` | string | RFC 3339 for date parsing | Date layout; see below. |
| `time_zone` | string | UTC | Zone used when parsing dates without an explicit zone. |
| `fields` | object | None | Additional indexed representations of a text field. |

For custom analyzer definitions, see [Text analysis](text_analysis.md). Vector-specific options are documented with [Vector search](../vector_search_api.md).

## Text subfields

```json
{"properties":{"title":{"type":"text","fields":{"exact":{"type":"keyword","index":true,"sortable":true,"aggregatable":true}}}}}
```

This indexes `title.exact` from the title value without adding that key to `_source`. Only text fields support `fields`; nested subfields within a subfield are ignored. Set subfield flags explicitly as shown.

## Dates

Use Go layouts, for example `2006-01-02` or `2006-01-02T15:04:05Z07:00`, not patterns such as `yyyy-MM-dd`. The default string layout is RFC 3339. `epoch_millis` is also recognized, but numeric timestamp ingestion uses magnitude-based unit detection; explicit formatted strings are clearer when precision matters.

Query dates must follow the relevant query's format rules. Date math such as `now-1d/d` is not generally supported.

## Objects and limitations

Nested `properties` are flattened to dot-separated field names. This does not provide nested-document query semantics. Do not use `object`, `nested`, geographic, IP, or other Elasticsearch-only field types as supported SeaSearch types: some are ignored rather than rejected.

Existing field mappings cannot be modified through Update mapping. Adding a mapping does not reindex existing documents. `_id` and `@timestamp` have system mappings; avoid redefining them for unrelated data.
