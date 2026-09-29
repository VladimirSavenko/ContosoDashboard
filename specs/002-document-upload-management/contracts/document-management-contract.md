# Document Management Service Contract

This contract describes the internal service-layer interface for the document feature. The repository does not yet expose a public REST API; the design is therefore framed as a Blazor-friendly service contract used by pages and future controllers or endpoints.

## Core Operations

### UploadDocumentAsync

Request payload:

```json
{
  "title": "Quarterly Review",
  "description": "Final review deck",
  "category": "Reports",
  "projectId": 12,
  "taskId": 45,
  "tags": ["finance", "quarterly"],
  "fileName": "review.pdf",
  "contentType": "application/pdf",
  "fileStream": "binary stream"
}
```

Response:

```json
{
  "documentId": 101,
  "status": "Available",
  "storageKey": "doc_8d5f0d8a-2f21-4e1a-99a5-1f2b9bb60250.pdf",
  "message": "Upload completed successfully"
}
```

Validation rules:
- file size must be <= 25 MB
- file type must be permitted
- title and category are required
- the file must pass the malware/virus safety gate before becoming available

### ListAuthorizedDocumentsAsync

Query object:

```json
{
  "category": "Reports",
  "projectId": 12,
  "dateFrom": "2026-09-01",
  "dateTo": "2026-09-30",
  "searchText": "quarterly",
  "sortBy": "uploadDate",
  "sortDirection": "desc"
}
```

Response:

```json
[
  {
    "documentId": 101,
    "title": "Quarterly Review",
    "category": "Reports",
    "fileSizeBytes": 123456,
    "uploadedBy": "Camille Nicole",
    "projectName": "Operations",
    "uploadedAt": "2026-09-28T12:00:00Z"
  }
]
```

### GetDocumentDetailsAsync

Returns document metadata and current permission state for an authorized user.

### ShareDocumentAsync

Request:

```json
{
  "documentId": 101,
  "recipientUserIds": [4, 8],
  "recipientTeamIds": [2],
  "shareReason": "Project collaboration"
}
```

Rules:
- duplicate grants are ignored
- team membership is evaluated dynamically
- notifications are created only for newly granted recipients

### ReplaceDocumentAsync

Replaces the stored file while preserving metadata and authorization.

### DeleteDocumentAsync

Permanently removes the file and content from user-accessible storage.

Audit rule:
- non-content activity metadata remains available to administrators after deletion.

## Security Contract

All document operations must require an authenticated actor and verify access at the service boundary before returning metadata or file bytes. Authorization checks must include:
- ownership
- project membership
- explicit share grants
- administrator role

No document bytes, metadata, or audit records may be returned to unauthorized users.

## Storage Contract

The implementation will use an interface such as `IDocumentStorageService` with methods to:
- store a file with a generated unique key
- retrieve the file stream for authorized access
- delete the stored file
- replace an existing stored file

This abstract interface allows a later cloud-based implementation without changing the business logic or document service API.
