# query_string query

Search using a compact expression string. Use explicit field prefixes for predictable field selection.

```json title="Require a title term and exclude a category"
{"query":{"query_string":{"query":"+title:search -category:travel"}}}
```

## Syntax

| Expression | Meaning |
| --- | --- |
| `title:search` | Analyze and match text on a named field. |
| `title:"search guide"` | Match a phrase. Escape the quotes inside JSON. |
| `+title:search` | Required clause. |
| `-category:travel` | Excluded clause. |
| `title:search summary:guide` | Optional clauses, rather than an implicit AND. |
| `category:ref*` | Wildcard term pattern; `?` matches one character. |
| `category:/ref.*/` | Regular expression on indexed terms. |
| `title:serch~1` | Fuzzy text query with a numeric distance. |
| `title:search^2` | Clause scoring weight. |
| `price:>=20` | Numeric comparison; also supports `>`, `<`, and `<=`. |
| `published:>="2026-01-01T00:00:00Z"` | Date comparison with a quoted RFC 3339 timestamp. |

Use spaces between clauses. The parser does not implement Elasticsearch's full Boolean expression grammar: `AND`, `OR`, `NOT`, grouping parentheses, bracket ranges, and date math are not supported operators. Use [bool](bool.md) and [range](range.md) when you need explicit structured conditions.

## Request options

`query` is the required string. An empty string matches no documents.

Other accepted members do not provide their Elasticsearch meanings:

- `fields` registers field analyzers; it does **not** expand unqualified text across that list or parse `title^2` weights. Put field names in the expression.
- `default_field`, `default_operator`, `boost`, and `analyze_wildcard` do not control the query as their names suggest. Use expression prefixes and clause weights instead.
- `analyzer` does not reliably override default analysis. Prefer compatible mapped field analyzers and explicit field-qualified queries.

Unknown option names are rejected. Invalid expressions, unmatched quotes, or malformed numeric/date values cause errors.

## Escaping and example

Backslashes escape reserved expression characters; JSON needs another level of escaping. For example, `"title:sea\\:search"` sends an escaped colon inside a term. Escaping a wildcard does not reliably make it literal, because wildcard recognition happens after unescaping; use structured queries for literal special-character matching.

```sh
curl -u "$SS_USER:$SS_PASSWORD" \
  -X POST "$SS_URL/es/api_books/_search" \
  -H 'Content-Type: application/json' \
  --data-binary '{"query":{"query_string":{"query":"+title:search -category:travel"}},"size":10}'
```

Results use the [Search response](../search/search.md#responses). Unified Search does not support this scoring query; see [Cluster API usage](../cluster.md).
