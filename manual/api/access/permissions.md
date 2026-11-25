# Permissions

```http
GET /api/permissions
```

List registered permission identifiers. Requires `auth.ListPermissions`; see [Authentication](../authentication.md). No body, path parameters, or query parameters are used.

```sh title="List registered permission identifiers"
curl -u "$SS_USER:$SS_PASSWORD" "$SS_URL/api/permissions"
```

HTTP 200 returns a string array, for example:

```json title="HTTP 200 · Selected permission names"
["index.Create","index.CreateES","search.SearchDSL"]
```

The example shows selected names. Use exact, case-sensitive strings in a role's `permission` array. The endpoint lists available identifiers, not the current user's effective permissions.

The registered list can differ by entry point. For coordinated cluster search, consult [Cluster API usage](../cluster.md), not just the ordinary search permission.

## Operation identifiers

The following table collects Server permissions so individual API pages do not need to repeat authentication details. An ES/native distinction matters only where listed.

| Operation | Permission |
| --- | --- |
| List indexes / names | `index.List` / `index.IndexNameList` |
| Create index (native / ES) | `index.Create` / `index.CreateES` |
| Get / delete / check index | `index.Get` / `index.Delete` / `index.Exists` |
| Refresh | `index.Refresh` |
| Get mapping (native / ES) | `index.GetMapping` / `index.GetESMapping` |
| Add mapping | `index.SetMapping` |
| Get / set settings | `index.GetSettings` / `index.SetSettings` |
| Analyze | `index.Analyze` |
| List / get templates | `index.ListTemplate` / `index.GetTemplate` |
| Create or update / delete template | `index.CreateTemplate` / `index.DeleteTemplate` |
| Update / get aliases | `index.AddOrRemoveESAlias` / `index.GetESAliases` |
| Index document (native / ES, including _create) | `document.Create` / `document.CreateUpdate` |
| Get or HEAD / update / delete document | `document.Get` / `document.Update` / `document.Delete` |
| Bulk / Bulk V2 (native) | `document.Bulk` |
| Bulk (ES) | `document.ESBulk` |
| Multi ingestion | `document.Multi` |
| Native SearchV1 / DSL Search | `search.SearchV1` / `search.SearchDSL` |
| Multi-search / delete by query | `search.MultipleSearch` / `search.DeleteByQuery` |
| Unified Search | `search.unified_search` |
| Vector search / seal / recall | `search.SearchVector` / `search.Seal` / `search.vectorRecall` |
| Create or update / list / delete user | `auth.CreateUpdateUser` / `auth.ListUser` / `auth.DeleteUser` |
| Create or update / list / delete role | `auth.CreateUpdateRole` / `auth.ListRole` / `auth.DeleteRole` |
| List permissions | `auth.ListPermissions` |
| Data-stream PUT / GET or HEAD | `elastic.PutDataStream` / `elastic.GetDataStream` |

These identifiers are case-sensitive. Login, version/health, client probes, Cluster Manager management endpoints, and diagnostics have the exceptions described on their pages. Internal cluster-call permissions are documented only as role-configuration requirements on the cluster page, not as client request protocols.
