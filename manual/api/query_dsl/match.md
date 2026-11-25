# match query

Analyze a string and search one text field for the resulting terms. Use `term` rather than `match` for exact keyword or typed-value comparisons.

## Syntax

String shorthand:

```json title="String shorthand"
{"query":{"match":{"title":"search guide"}}}
```

Expanded form:

```json title="Expanded form · Require all analyzed terms"
{
  "query":{"match":{"title":{
    "query":"search guide",
    "operator":"AND",
    "analyzer":"standard"
  }}}
}
```

Use one field per `match` object. Combine separate queries with `bool` to search multiple fields.

## Parameters

Options below belong inside the field object.

| Parameter | Type | Default | Description |
| --- | --- | --- | --- |
| `query` | string | — | Text to search; supply this value. |
| `analyzer` | string | Field/index configuration | Analyzer used for the input text. |
| `operator` | string | `OR` | `OR` matches any analyzed term; `AND` requires all terms. Case-insensitive. |
| `minimum_should_match` | number or string | At least one term | Required number of token clauses with `OR`; ignored with `AND`. |
| `boost` | number | `1` | Nonnegative scoring weight. |
| `fuzziness` | number or string | No fuzzy matching | Edit distance, or `AUTO`. |
| `prefix_length` | integer | `0` | Unchanged prefix length for fuzzy matching. |

For custom analysis, explicitly set both `analyzer` and `search_analyzer` in the [field mapping](../index/mapping.md), or specify the query analyzer. If analysis produces no tokens, no documents match.

### minimum_should_match

The threshold counts **analyzed token clauses**, not the original words. With five tokens, `"75%"` requires three matches and `"-25%"` requires four.

| Form | Example | Meaning |
| --- | --- | --- |
| Integer | `3` or `"3"` | Require three clauses. |
| Negative integer | `-2` | Allow two clauses to be missing. |
| Percentage | `"75%"` | Require the rounded-down percentage of clauses. |
| Negative percentage | `"-25%"` | Allow the rounded-down percentage to be missing. |
| Conditional | `"2<-25%"` | Require all up to two clauses; otherwise apply `-25%`. |
| Multiple conditions | `"2<-25% 9<-3"` | Apply thresholds in ascending order; above nine, allow three missing clauses. |

For a nonempty token list the result is clamped between 1 and the number of clauses. Thus `0` does not make every term optional. Send a number or string, not an array.

### Fuzzy matching

Use a small explicit distance such as `1`, optionally with `prefix_length`. `AUTO` selects distance 0, 1, or 2 from the longest analyzed token in the query, using byte-length thresholds of 3 and 6. `AUTO:low,high` changes the thresholds, with `2 <= low < high`.

!!! note
    `AUTO` uses one distance for the query and measures bytes, not Unicode characters. Fuzzy options are not available for cluster Unified Search; see [Cluster API usage](../cluster.md).

## Example

Using the [tutorial dataset](getting_started.md), require two of three title terms:

```sh title="Require two of three analyzed title terms"
curl -u "$SS_USER:$SS_PASSWORD" \
  -X POST "$SS_URL/es/api_books/_search" \
  -H 'Content-Type: application/json' \
  --data-binary '{
    "query":{"match":{"title":{"query":"practical search guide","minimum_should_match":2}}},
    "size":10
  }'
```

Results use the [Search response format](../search/search.md#responses).

## Errors and limitations

Multiple fields, a non-string shorthand, or an unknown operator/analyzer cause errors. Unknown options may instead be ignored. Options such as `max_expansions`, `lenient`, and `zero_terms_query` are not supported.
