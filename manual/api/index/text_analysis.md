# Text analysis

An analyzer converts text into searchable tokens. Its pipeline is: character filters → tokenizer → token filters. Index-time and search-time pipelines should produce compatible terms.

Define custom analysis under `settings.analysis` when [creating an index](create_index.md). Existing analyzers cannot be changed through Update settings.

## Configuration

```json
{
  "settings":{"analysis":{
    "analyzer":{"book_text":{
      "type":"custom",
      "tokenizer":"standard",
      "token_filter":["lowercase"]
    }}
  }},
  "mappings":{"properties":{"title":{
    "type":"text",
    "analyzer":"book_text",
    "search_analyzer":"book_text"
  }}}
}
```

| Analysis member | Shape | Purpose |
| --- | --- | --- |
| `analyzer` | Name → definition | Built-in or custom analyzers. |
| `tokenizer` | Name → object with `type` | Configured tokenizer instances. |
| `char_filter` | Name → object with `type` | Configured character filters. |
| `token_filter` | Name → object with `type` | Configured token filters; `filter` is an alias when `token_filter` is absent. |

Analyzer definitions accept optional `type`, `tokenizer`, and ordered string arrays `char_filter` and `token_filter` (`filter` alias). A custom analyzer needs a tokenizer. Component names can refer to built-ins or definitions in the corresponding map. Unknown names return errors.

## Built-in analyzers

Common names are `standard`, `simple`, `keyword`, `web`, `pattern` (`regexp`), `stop`, and `whitespace`. Chinese analysis provides `gse_standard` and `gse_search`; see [Chinese analyzer](chinese_analyzer.md).

Language analyzers include `arabic`, `cjk`, `sorani`, `danish`, `german`, `english`, `spanish`, `persian`, `finnish`, `french`, `hindi`, `hungarian`, `italian`, `dutch`, `norwegian`, `portuguese`, `romanian`, `russian`, `swedish`, and `turkish`. The corresponding language-code aliases are also recognized.

Built-in types can be configured under a custom name. `standard`, `stop`, and `pattern` accept a string-array `stopwords`; `pattern` also accepts `pattern` and `lowercase`. The regular expression selects tokens rather than specifying an Elasticsearch split pattern. Explicitly set `lowercase` when configuring a pattern analyzer.

## Components

Common tokenizers: `standard` (`unicode`), `letter` (`simple`), `keyword` (`single`), `whitespace`, `web`, `lowercase`, `character`, `char_group`, `ngram`, `edge_ngram`, `path_hierarchy`, `pattern` (`regexp`), `exception`, `gse_standard`, and `gse_search`.

Common token filters: `lowercase`, `uppercase`, `stop`, `trim`, `truncate`, `length`, `ngram`, `edge_ngram`, `shingle`, `reverse`, `unique`, `camel_case`, `apostrophe`, `porter` (`stemmer`), `keyword_marker`, `elision`, `dict`, `pattern_replace` (`regexp`), `unicodenorm`, and `gse_stop`. Language-specific stemmers and normalization filters are also available.

Character filters: `asciifolding` (`ascii_folding`), `html_strip` (`html`), `zero_width_non_joiner`, `mapping`, and `pattern_replace` (`pattern`/`regexp`).

See [Analysis components](analysis_components.md) for per-type options and defaults. Arbitrary Elasticsearch plugin settings are not supported. Use [Analyze](analyze.md) to inspect tokens before indexing important data. Changing query-time analysis does not reindex existing terms.
