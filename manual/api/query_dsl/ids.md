# ids query

Find documents whose IDs match any string in `values`:

```json title="Match documents by ID"
{"query":{"ids":{"values":["book-1","book-2"]}}}
```

`values` is the required string array. Use strings even for IDs made only of digits. There are no supported analyzer, type, or boost parameters.

This queries the indexed `_id` field; it does not fetch records from a separate document store. Use [Get document](../document/get_document.md) for a single-ID retrieval endpoint. Non-string array entries or malformed input can cause errors.
