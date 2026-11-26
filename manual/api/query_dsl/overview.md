# Query DSL

SeaSearch Query DSL is a JSON language for matching indexed documents. A search request places one query object under `query` and puts result controls, such as `size`, alongside it:

```json
{
  "query": {"match": {"title": "search guide"}},
  "size": 10
}
```

!!! tip "New to Query DSL?"
    Start with the [full-text search and filters tutorial](getting_started.md), then use the [match reference](match.md) and [Search API](../search/search.md) for exact syntax and response behavior.

## Queries and fields

- **Full-text queries** analyze the input into tokens. Use them with `text` fields and compatible search/index analyzers.
- **Term-level queries** work on indexed terms or typed values. Use a `keyword` field for exact whole-string matching and a `numeric` field for numeric comparisons. A `term` query does not analyze a text string for you.
- **Compound queries** combine conditions. `bool.must` requires conditions, `bool.should` supplies alternatives, `bool.must_not` excludes matches, and `bool.filter` adds required conditions with zero boost.
- **Result controls** such as source selection, sorting, highlighting, and aggregations are search-body options, not query types.

Map fields explicitly for the operations you need; see [Mapping and field types](../index/mapping.md).

## Defaults and unsupported syntax

An omitted or null `query` means match-all. Use `{"match_all":{}}` when specifying it explicitly. `{"query":{}}` is rejected; an empty object does not mean match-all.

`exists`, `boosting`, `combined_fields`, `terms_set`, and geographic queries are not supported. Each query reference lists its available options.

## Query reference

<div class="grid cards" markdown="1">

- **Match everything or nothing**

    ---

    [match_all / match_none](match_all.md)

- **Full-text queries**

    ---

    [match](match.md) · [match_phrase](match_phrase.md) · [match_bool_prefix](match_bool_prefix.md) · [match_phrase_prefix](match_phrase_prefix.md) · [multi_match](multi_match.md)

- **Exact values**

    ---

    [term](term.md) · [terms](terms.md) · [range](range.md) · [ids](ids.md)

- **Term patterns**

    ---

    [prefix](prefix.md) · [wildcard](wildcard.md) · [regexp](regexp.md) · [fuzzy](fuzzy.md)

- **Compound conditions**

    ---

    [bool](bool.md)

- **Expression strings**

    ---

    [query_string](query_string.md) · [simple_query_string](simple_query_string.md)

</div>

## Execution contexts

Available search interfaces:

- Ordinary Server DSL search uses `POST /es/{target}/_search` or `POST /es/_search`.
- Cluster search has additional restrictions described in [Cluster API usage](../cluster.md).
- `POST /api/{target}/_search` is the native SearchV1 protocol, not this request format.
- [Vector search](../vector_search_api.md) uses a separate request format with a `filter_query` condition.

See the [Search reference](../search/search.md) for request options and responses.
