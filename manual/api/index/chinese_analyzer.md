# Chinese analyzer

SeaSearch provides `gse_standard` and `gse_search` for Chinese tokenization. Enable dictionary loading in the **Server** environment and restart the service:

```sh
ZINC_PLUGIN_GSE_ENABLE=true
ZINC_PLUGIN_GSE_DICT_EMBED=BIG
```

`ZINC_PLUGIN_GSE_ENABLE` defaults to false. `DICT_EMBED` defaults to `small`; `BIG` loads the larger embedded Chinese dictionary. The configuration names above use the `ZINC_` prefix.

## Field configuration

Set both analyzers when creating a text field:

```json
{
  "mappings":{"properties":{"content":{
    "type":"text",
    "analyzer":"gse_standard",
    "search_analyzer":"gse_standard"
  }}}
}
```

Use `gse_search` when search-mode segmentation is appropriate for your data. Inspect the actual tokens with [Analyze](analyze.md); dictionary and mode choices affect matching.

## Custom dictionaries

Set `ZINC_PLUGIN_GSE_DICT_PATH` to a directory containing:

- `user.txt`: custom dictionary entries.
- `stop.txt`: custom stop words.

The default directory is `./plugins/gse/dict`. Dictionary loading must be enabled. `ZINC_PLUGIN_GSE_ENABLE_STOP` defaults to true and controls embedded/default stop-word loading. Keep configuration and dictionary files consistent across Servers, and restart after changes.

Analyzer changes do not rewrite existing indexed terms. Create a new index and reindex when changing analysis for existing data.
