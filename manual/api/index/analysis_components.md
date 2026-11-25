# Analysis components

Define named components under `settings.analysis.tokenizer`, `token_filter`, or `char_filter`, then reference their names from an [analyzer](text_analysis.md). Each configured component is an object with a required string `type`. The tables list consumed options; unspecified options are optional unless marked required.

Use integer values for lengths and counts, valid positive gram bounds, and small test inputs in [Analyze](analyze.md) before indexing. Unknown options are not proof of an extension being supported.

## Tokenizers

| Type | Options and defaults |
| --- | --- |
| `standard` / `unicode` | No configurable options; Unicode tokenization. |
| `letter` / `simple` | No options; letter-based tokenization. |
| `keyword` / `single` | No options; one token for the input. |
| `whitespace`, `web`, `lowercase` / `lower_case` | No exposed options. |
| `pattern` / `regexp` | String `pattern`, default `\\w+`; regex matches select tokens. |
| `character` | Required string `char`: `graphic`, `print`, `control`, `letter`, `mark`, `number`/`digit`, `punct`/`punctuation`, `space`/`whitespace`/`white_space`, or `symbol`. |
| `char_group` | Required string array `tokenize_on_chars`; split using named character groups or literal separators. |
| `ngram`, `edge_ngram` | Integer `min_gram` = 1, `max_gram` = 2, optional string array `token_chars`. Minimum must not exceed maximum. |
| `path_hierarchy` | String `delimiter` = `/`, string `replacement` = delimiter, integer `skip` = 0. Use single-byte delimiter/replacement characters. |
| `exception` | Required string array `patterns`, combined into a regex exception matcher. |
| `gse_standard`, `gse_search` | No per-component options; see [Chinese analyzer](chinese_analyzer.md). |

Character groups include Unicode categories such as letter, number, punctuation, whitespace, and symbol; not all Elasticsearch tokenizer settings apply. In particular, ngram `token_chars` restricts accepted grams, not a separate preprocessing pipeline.

## Token filters

| Type | Options and defaults |
| --- | --- |
| `lowercase` / `lower_case`, `uppercase` / `upper_case` | No options; change case. |
| `trim`, `reverse`, `unique`, `camel_case` / `camelcase`, `apostrophe` | No exposed options. |
| `porter` / `stemmer` | No exposed language setting; Porter stemming, not a generic configurable ES stemmer. |
| `stop` | String array `stopwords`; supply an explicit list when defining a custom filter. |
| `keyword` / `keyword_marker` | Required string array `keywords`; mark dictionary entries. |
| `length` | Integer `min` = 1, `max` = 2; retain token lengths in that range. |
| `truncate` | Integer `length` = 10. |
| `ngram` | Integer `min_gram` = 1, `max_gram` = 2. |
| `edge_ngram` | Same gram bounds; string `side` = `front`, optionally `back`. |
| `elision` | Optional string array `articles`; omission uses built-in French articles. |
| `dict` | Required string array `words`; integer `min_word_size` = 5, `min_sub_word_size` = 2, `max_sub_word_size` = 15; boolean `only_longest_match` = false. |
| `pattern_replace` / `regexp` | Required string `pattern`; string `replacement` defaults to empty. |
| `shingle` | Integers `min_shingle_size` = 2, `max_shingle_size` = 2; string `token_separator` = space, `filler_token` = `_`; boolean `output_original` = true. |
| `unicodenorm` | Required string `form`: `NFC`, `NFD`, `NFKC`, or `NFKD` (case-insensitive). |
| `gse_stop` | Uses the configured GSE dictionary/stop words. |

Gram and length minimums must not exceed maximums. Zero values select the documented defaults in these constructors; do not use negative values to bypass validation.

Language filters also have no exposed per-filter options:

- `arabic_normalization`, `arabic_stemmer` (`ar_*` aliases).
- `cjk_bigram`, `cjk_width`.
- `sorani_normalization`, `sorani_stemmer` (`ckb_*`).
- `danish_stemmer` (`da_stemmer`).
- `german_normalization`, `german_stemmer`, `german_light_stemmer` (`de_*`).
- `english_possessive_stemmer`, `english_stemmer` (`en_*`).
- `spanish_stemmer`, `spanish_light_stemmer` (`es_*`).
- `persian_normalization` (`fa_normalization`), `finnish_stemmer` (`fi_stemmer`).
- `french_elision`, `french_stemmer`, `french_light_stemmer`, `french_minimal_stemmer` (`fr_*`).
- `irish_elision` (`ga_elision`).
- `hindi_normalization`, `hindi_stemmer` (`hi_*`), `hungarian_stemmer` (`hu_stemmer`).
- `indic_normalization` (`in_normalization`).
- `italian_elision`, `italian_stemmer`, `italian_light_stemmer` (`it_*`).
- `dutch_stemmer` (`nl_stemmer`), `norwegian_stemmer` (`no_stemmer`).
- `portuguese_stemmer` / `portuguese_light_stemmer` / `pt_light_stemmer`, `romanian_stemmer` / `ro_stemmer`.
- `russian_stemmer` (`ru_stemmer`), `swedish_stemmer` (`sv_stemmer`), `turkish_stemmer` (`tr_stemmer`).

## Character filters

| Type | Options and defaults |
| --- | --- |
| `asciifolding` / `ascii_folding`, `html_strip` / `html`, `zero_width_non_joiner` | No exposed options. |
| `pattern_replace` / `pattern` / `regexp` | Required string `pattern`; optional string `replacement`, default empty. |
| `mapping` | Required nonempty string array `mappings`, with rules such as `":) => happy"`. Include spaces around `=>`. |

Regex patterns use Go syntax. Elasticsearch regex flags, filter file paths, plugin names, and options not listed above are not supported configuration controls.

## Example

```json
{
  "settings":{"analysis":{
    "analyzer":{"title_text":{"type":"custom","tokenizer":"standard","token_filter":["lowercase","book_stop"]}},
    "token_filter":{"book_stop":{"type":"stop","stopwords":["a","the"]}}
  }},
  "mappings":{"properties":{"title":{"type":"text","analyzer":"title_text","search_analyzer":"title_text"}}}
}
```

Use this body with [Create index](create_index.md). Built-in analyzer types may also accept `stopwords`; pattern analyzers accept `pattern` and `lowercase`. Settings changes do not rewrite indexed terms.
