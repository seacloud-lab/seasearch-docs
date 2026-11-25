# Index

An index groups documents, field mappings, and text-analysis settings. Create it explicitly when you need predictable field types or analyzers; document writes can otherwise create indexes and infer mappings.

## Index reference

<div class="grid cards" markdown="1">

- **Index lifecycle**

    ---

    [Create an index](create_index.md), [get an index](get_index.md), [check existence](check_exists.md), or [delete an index](delete_index.md).

- **Discover indexes**

    ---

    [List indexes](list_indexes.md) with metadata and pagination, or look up [index names](list_index_names.md).

- **Mappings and field types**

    ---

    Review [field types](mapping.md), [get mapping](get_mapping.md), or [add mapped fields](update_mapping.md).

- **Settings and analysis**

    ---

    [Get settings](get_settings.md), [update settings](update_settings.md), and configure [text analysis](text_analysis.md).

- **Analysis tools**

    ---

    Explore [analysis components](analysis_components.md), configure [Chinese analysis](chinese_analyzer.md), and [inspect tokens](analyze.md).

- **Templates and aliases**

    ---

    [Create templates](create_template.md), [get](get_template.md), [list](list_templates.md), or [delete](delete_template.md) them. [Update aliases](update_aliases.md) or [read associations](get_aliases.md).

</div>

!!! note "Refresh and search visibility"
    The [Refresh](refresh.md) operation closes index resources for reopening; it is not a guarantee that asynchronous writes have become searchable.

See [Authentication](../authentication.md) for credentials and [Cluster API usage](../cluster.md) for deployment differences.
