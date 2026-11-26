# range query

Compare a mapped numeric or date field with bounds. Keyword/text ranges are not supported.

```json title="Numeric range · Inclusive positive bounds"
{"query":{"range":{"price":{"gte":20,"lte":30}}}}
```

Use one field per object. Bounds `gt`, `gte`, `lt`, and `lte` are optional; supply at least one useful bound. `gt`/`lt` exclude endpoints, `gte`/`lte` include them. Do not specify both forms for the same endpoint. Optional nonnegative `boost` defaults to 1.

## Numeric bounds

!!! warning "Use positive numeric bounds"
    **Zero and negative bounds are not reliably honored**: the current implementation starts at 0 (exclusive), ignores nonpositive supplied bounds, and uses an internal large positive maximum when no positive upper bound is applied. This is not a general unbounded numeric-range implementation.

## Date bounds

Use date strings with the mapped format, default RFC 3339. Optional `format` overrides it with a Go time layout, or `epoch_millis`; optional string `time_zone` overrides the mapping's zone, default UTC.

```json title="Date range · Explicit Go time layout"
{"query":{"range":{"published":{"gte":"2026-01-01","lt":"2027-01-01","format":"2006-01-02"}}}}
```

With no upper date bound, the implementation uses the current time, not positive infinity. Date math such as `now-1d/d` is not supported. Invalid dates, zones, or unsupported field types cause errors.

Use range conditions inside `bool.filter` for document-only filtering. Unified Search does not support range as a scoring query; see [Cluster API usage](../cluster.md).
