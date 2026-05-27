---
title: Workfront Planning API Documentation
description: A guide to using Workfront Planning APIs
---

<Hero slots="heading, text"/>

# Workfront Planning API

The Adobe Workfront Planning API provides a REST-ful architecture to simplify integrations with Planning, utilizing HTTP methods for operations.

## Overview

The Adobe Workfront Planning API facilitates integration with the Planning feature via a REST-ful design over HTTP. It is designed for users familiar with REST and JSON, manipulating objects via unique URIs using standard HTTP methods.

More details can be found in the [Adobe Workfront Planning API Basics](https://experienceleague.adobe.com/en/docs/workfront/using/adobe-workfront-planning/adobe-workfront-planning-general-information/planning-api-basics).

## API References

<DiscoverBlock width="100%" slots="heading, link, text"/>

### Get Started

Familiarize yourself with the Workfront Planning API: HTTP requests, CRUD operations, and available resources.

### Versions
 - **[V1](/api/v1/)**
 - **[V2](/api/v2/)**

---

## V1 vs V2 — What's New

Version 2 of the Workfront Planning API is a significant expansion of what you can build. If you're new to the API, V2 is where to start. If you're on V1, this page covers everything that changed and exactly what to update.

### New capabilities in V2

These features are available for the first time in V2 and are not available in V1.

#### Full platform CRUD

V1 was limited to reading workspaces and record types, with CRUD only for records. V2 gives you full create, update, and delete access across the entire Planning object model.

| Resource     | V1                       | V2                                   |
| ------------ | ------------------------ | ------------------------------------ |
| Workspaces   | `GET` only               | `GET`, `POST`, `PUT`, `PATCH`, `DELETE` |
| Record types | `GET` only               | `GET`, `POST`, `PUT`, `PATCH`, `DELETE` |
| Fields       | Not available            | `GET`, `POST`, `PUT`, `PATCH`, `DELETE` |
| Records      | `GET`, `POST`, `PUT`, `DELETE` | `GET`, `POST`, `PUT`, `PATCH`, `DELETE` |
| Views        | Not available            | `GET`, `POST`, `PUT`, `PATCH`, `DELETE` |

This means you can now build integrations that create and manage workspaces, record types, fields, records and views entirely through the API.

#### Partial updates with PATCH

V2 adds `PATCH` support with JSON merge-patch semantics. Only the fields you include in the request body are updated — everything else is left unchanged. No more sending a full resource payload to change a single field.

```
PATCH /v2/workspaces/{id}
{
  "description": "Updated description only"
}
```

#### Bulk record operations

V2 introduces dedicated bulk endpoints for records, allowing you to create, update, patch, or delete large volumes of records in a single API call.

| Operation    | Endpoint                                          | Max per request |
| ------------ | ------------------------------------------------- | --------------- |
| Bulk create  | `POST /v2/record-types/{id}/records/bulk`         | 100             |
| Bulk update  | `PUT /v2/record-types/{id}/records/bulk`          | 100             |
| Bulk patch   | `PATCH /v2/record-types/{id}/records/bulk`        | 100             |
| Bulk delete  | `DELETE /v2/record-types/{id}/records/bulk`       | 100             |

Bulk endpoints return `201 Created` when every row succeeds and `207 Multi-Status` for partial success — inspect the per-row results envelope to identify which records failed and why.

#### Reorder records

Reorder up to 100 records within a record type by placing them immediately after or before an anchor record. Provide exactly one of `putAfterRecordId` or `putBeforeRecordId`. The operation is all‑or‑nothing — on failure no records are reordered.

```
POST /v2/record-types/{recordTypeId}/records/move
```

**Request body — place after an anchor**

```json
{
  "recordIds": ["Rc6796aaaa0000000000000001", "Rc6796aaaa0000000000000002"],
  "putAfterRecordId": "Rc67a1bbbb0000000000000010"
}
```

**Request body — place before an anchor**

```json
{
  "recordIds": ["Rc6796aaaa0000000000000003"],
  "putBeforeRecordId": "Rc67a2cccc0000000000000020"
}
```

#### Record thumbnails

Upload or remove a thumbnail image for a record:

```
POST   /v2/records/{id}/thumbnail   → 201 Created
DELETE /v2/records/{id}/thumbnail   → 204 No Content
```

#### Global record types

Retrieve the list of available global record types that you can add to a workspace:

```
GET /v2/workspaces/{workspaceId}/global-record-types
```

#### Delete a global record type from a secondary workspace

Use this endpoint to delete a global record type from a secondary workspace. Note that all records created in that workspace will be permanently deleted:

```
POST /v2/workspaces/{workspaceId}/record-types/{recordTypeId}/detach
```

#### Retrieve record changes history

Retrieve a cursor‑paginated list of field change events for a specific record. Each entry captures who changed which field, when, and the before/after values along with a snapshot of the field's metadata at that moment.

```
GET /v2/records/{id}/history
```

#### Permissions API

V2 introduces a dedicated Permissions API for reading and managing access programmatically, including workspace and record type member management, access request workflows, and permission inheritance, using the below endpoints:

```
GET    /v2/permissions/{resourceType}/{resourceId}
GET    /v2/permissions/{resourceType}/{resourceId}/members
GET    /v2/permissions/{resourceType}/{resourceId}/inheritance
GET    /v2/permissions/{resourceType}/{resourceId}/requests
DELETE /v2/permissions/{resourceType}/{resourceId}/requests
PATCH  /v2/permissions/{resourceType}/{resourceId}/members
POST   /v2/permissions/{resourceType}/{resourceId}/requests
```

In these endpoints `resourceType` represents the entity type, and `resourceId` represents the specific entity for which the permissions are being checked. The `resourceType` can take one of the following values: `workspaces`, `record-types`, `records`, `views`.

For example, the endpoint below will return the list of people who have access to the workspace with ID `Ws6a0475de4ecced960185e1e1`:

```
GET /v2/permissions/workspaces/Ws6a0475de4ecced960185e1e1/members
```

#### Platform limits

V2 surfaces and enforces explicit platform limits. Plan integrations accordingly:

| Limit                                      | Value   |
| ------------------------------------------ | ------- |
| Records per bulk request                   | 100     |
| Fields per record type (total)             | 500     |
| `PARAGRAPH` fields per record type         | 20      |
| `FORMULA` fields per record type           | 20      |
| `REFERENCE` fields per record type         | 30      |
| Personal views per record type             | 100     |
| Workspaces per tenant                      | 100,000 |

### V1 vs V2 — what changed

For developers migrating from V1, the table below covers every breaking change and where it affects your integration.

| Area                  | V1                                          | V2                                                          | Impact                  |
| --------------------- | ------------------------------------------- | ----------------------------------------------------------- | ----------------------- |
| Base URL              | `/maestro/api/v1/`                          | `/maestro/api/v2/`                                          | All requests            |
| Record type listing   | `GET /v1/record-types?workspaceId={id}`     | `GET /v2/workspaces/{id}/record-types`                      | URL update required     |
| Record creation       | `POST /v1/records` (`recordTypeId` in body) | `POST /v2/record-types/{id}/records`                        | URL + body update       |
| Record search         | `GET` or `POST /v1/records/search`          | `GET` or `POST /v2/record-types/{id}/records/search`        | URL update              |
| Create status code    | `200 OK`                                    | `201 Created`                                               | Error handling update   |
| Delete status code    | `200 OK`                                    | `204 No Content`                                            | Error handling update   |
| Pagination            | `offset`/`limit` in request body            | Cursor-based (listings) or `page`/`size` query params (search) | Pagination logic update |
| Pagination response   | `{ records, totalCount }`                   | `{ records, page }` (search) or `{ content, cursor }` (listings) | Response parsing update |
| Filter syntax         | Mongo-style `$and`, `$is`, `$contains`      | Typed enums `AND`, `IS`, `CONTAINS`                         | Filter rebuild required |
| Filter location       | `recordTypeId` in body                      | `recordTypeId` in URL path                                  | URL + body update       |
| Partial updates       | `PUT` only (full replacement)               | `PATCH` available (merge-patch)                             | Optional adoption       |
| Error code format     | Numeric `type` (e.g. `40001`)               | String `errorCode` (e.g. `VALIDATION_FAILED`)               | Error handling update   |
| Error structure       | `report.detailedMessage` wrapper            | RFC 7807 problem+json with `errors[]` array                 | Error parsing update    |
| Field projection      | `?attributes=`                              | `?fieldIds=` or `?fieldAliases=`                            | Query param update      |
| Sorting key           | `sorting` array                             | `sort` array                                                | Request body update     |
| `createdBy` / `updatedBy` | Plain ID string                         | Object `{ id, name }`                                       | Response parsing update |
| `customerId` / `imsOrgId` | Present in responses                    | No longer returned in responses                             | Response parsing update |
| Bulk operations       | Not available                               | `/records/bulk` endpoints                                   | New capability          |
| Permissions API       | Not available                               | `/v2/permissions/`                                          | New capability          |
| Permissions in response | Embedded in resource response             | Removed — use Permissions API                               | Response parsing update |

### Migration guide — updating from V1 to V2

Work through the checklist below in order. Each step includes before/after examples.

#### 1. Update your base URL

Change the version segment in all request URLs from `/v1/` to `/v2/`. Single-resource GETs by ID keep the same path pattern (e.g. `GET /v2/workspaces/{id}` and `GET /v2/record-types/{id}`), but the **listing** path for record types did change — see step 2.

```
Before: https://{customer-domain}/maestro/api/v1/workspaces
After:  https://{customer-domain}/maestro/api/v2/workspaces
```

#### 2. Update how the list of record types in a workspace is retrieved

Move `workspaceId` from a query parameter to the URL path.

```
Before: GET /v1/record-types?workspaceId={id}
After:  GET /v2/workspaces/{id}/record-types
```

#### 3. Update record creation and search

Move `recordTypeId` from the request body to the URL path for both create and search.

Create:

```
Before: POST /v1/records             { "recordTypeId": "Rt123", ... }
After:  POST /v2/record-types/Rt123/records  { ... }
```

Search:

```
Before: POST /v1/records/search                      { "recordTypeId": "Rt123", ... }
After:  POST /v2/record-types/Rt123/records/search   { ... }
```

Both `GET` and `POST` variants of search remain supported in V2 at `/v2/record-types/{id}/records/search`.

#### 4. Handle new HTTP status codes

Update any logic that checks response status codes for create and delete operations.

```
POST (create):  200 → 201 Created
DELETE:         200 → 204 No Content
Bulk (partial): — → 207 Multi-Status
```

Bulk endpoints return `201` when every row succeeds and `207 Multi-Status` when some rows fail — make sure your client inspects per-row results in the response envelope.

#### 5. Update pagination

Workspaces, record types, fields, and views use cursor-based pagination. Records use page number and page size parameters.
Remove `offset`/`limit` from the request body. Move to query parameters and update response parsing.

Records:

```
Before (body):     { "offset": 0, "limit": 50 }
After  (query):    ?page=0&size=50

Before (response): { "records": [...], "totalCount": 150 }
After  (response): {
                     "records": [...],
                     "groups":  [...],   // present only on grouped searches
                     "page": { "size": 50, "number": 0, "totalElements": 150, "totalPages": 3 }
                   }
```

Workspaces, record types, fields and views (cursor-based):

```
GET /v2/workspaces?limit=20&cursor={nextCursor}

Response: { "content": [...], "cursor": { "hasMore": true, "nextCursor": "..." } }
```

#### 6. Update search filter syntax

Rebuild filters using typed enum operators. Move pagination params and `recordTypeId` out of the filter body.

Before (V1):

```
{
  "recordTypeId": "Rt123",
  "filters": {
    "$and": [
      { "fieldId": { "$is": "Active" } },
      { "fieldId": { "$contains": "marketing" } }
    ]
  },
  "offset": 0,
  "limit": 50
}
```

After (V2):

```
POST /v2/record-types/Rt123/records/search?page=0&size=50
{
  "filter": {
    "operator": "AND",
    "conditions": [
      { "fieldId": "<fieldId>", "condition": "IS", "value": "Active" },
      { "fieldId": "<fieldId>", "condition": "CONTAINS", "value": "marketing" }
    ]
  }
}
```

#### 7. Update error handling

Replace numeric `type` checks with string `errorCode` checks. Update any code that reads from the `report` wrapper. V2 error responses are RFC 7807-compatible (`application/problem+json`) with Planning-specific extensions.

Before (V1):

```
{
  "type": 40001,
  "title": "Invalid Data",
  "report": { "detailedMessage": "Field name cannot be empty" }
}
```

After (V2):

```
{
  "title": "Validation failed",
  "status": 400,
  "detail": "Request validation failed.",
  "errorCode": "VALIDATION_FAILED",
  "messageArguments": { },
  "requestId": "b7c3...e1",
  "errors": [
    { "field": "name", "message": "must not be blank" }
  ]
}
```

Switch on the stable `errorCode` enum (e.g. `VALIDATION_FAILED`, `NOT_FOUND`, `CONFLICT`, `FORBIDDEN`, `UNAUTHORIZED`, `INTERNAL_ERROR`) rather than the HTTP status or `title`. Always capture `requestId` in client logs — support tickets are resolved much faster with it.

#### 8. Use Permissions API to check access

V2 no longer includes permission level in resource responses. Switch to the Permissions API for any integration that reads the permission field from workspace or record type responses.

```
GET /v2/permissions/{resourceType}/{resourceId}
```

#### 9. Update response parsing for system fields

V1 included the `customerId` and `imsOrgId` in both headers and request/response bodies. V2 request/response bodies no longer contain that information and only rely on headers.

Also, `createdBy` and `updatedBy` fields are returned as objects instead of a plain string.
```
Before: "createdBy": "user123"
After:  "createdBy": { "id": "user123", "name": "Jane Doe" }
```

#### 10. Update field parameters

In V2, filtering the response by a specific field is only supported for records. All other resources always return their full information.

For records, replace `?attributes=` with `?fieldIds=` for field IDs or `?fieldAliases=` for field aliases. System fields such as Created By and Last Updated By are always included in the response regardless of what you specify.
Replace the `?attributes=` query parameter with `?fieldIds=` or `?fieldAliases=`.

```
Before: /v1/records/search?attributes=F123,F456,createdBy
After:  /v2/record-types/{id}/records/search?fieldIds=F123,F456
```

#### 11. Update sorting key

Rename the `sorting` array key to `sort` in search request bodies.

```
Before: { "sorting": [{ "fieldId": "F123", "direction": "asc" }] }
After:  { "sort":    [{ "fieldId": "F123", "direction": "asc" }] }
```

#### 12. Adopt PATCH for partial updates (recommended)

Not a breaking change — `PUT` still works — but switching updates to `PATCH` reduces payload size and eliminates the risk of accidentally nulling fields.

#### 13. Adopt bulk endpoints for high-volume record operations (recommended)

If your integration creates, updates, or deletes records in loops, replace individual calls with the `/records/bulk` endpoints for significantly better performance. Remember to handle `207 Multi-Status` for partial success.

### Next steps

- [Full API V2 reference](/api/v2/)
- [API Basics](https://experienceleague.adobe.com/en/docs/workfront/using/adobe-workfront-planning/adobe-workfront-planning-general-information/planning-api-basics) — authentication, filtering, pagination, and field types
