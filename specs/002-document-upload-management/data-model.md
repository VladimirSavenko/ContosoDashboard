# Data Model: Document Upload and Management

## Core Entities

### Document

Represents an uploaded work item and its metadata.

| Field | Type | Required | Notes |
|---|---|---:|---|
| DocumentId | int | Yes | Primary key |
| Title | string | Yes | User-visible name; required |
| Description | string | No | Optional details and search text |
| Category | enum/string | Yes | One of Project Documents, Team Resources, Personal Files, Reports, Presentations, Other |
| Tags | string[]/CSV | No | Searchable metadata |
| FileName | string | Yes | Original filename for display only |
| StorageKey | string | Yes | Globally unique storage identifier, not user-controlled |
| FileSizeBytes | long | Yes | Must be <= 25 MB |
| ContentType | string | Yes | MIME type, validated against allow list |
| OwnerUserId | int | Yes | Owning user |
| ProjectId | int | No | Optional project association |
| TaskId | int | No | Optional task association |
| Status | enum | Yes | Draft, Quarantined, Available, Replaced, Deleted |
| CreatedAt | DateTime | Yes | Audit timestamp |
| UpdatedAt | DateTime | Yes | Last modification |
| DeletedAt | DateTime | No | Non-content retention for audit use |

Relationships:
- Many-to-one with `User` (owner)
- Optional many-to-one with `Project`
- Optional many-to-one with `TaskItem`
- One-to-many with `DocumentAccessGrant`
- One-to-many with `DocumentActivity`

Validation rules:
- `Title` is required and should be trimmed.
- `Category` must be from the predefined list.
- `FileSizeBytes` must be greater than zero and no more than 25 MB.
- `StorageKey` must be unique and generated server-side.
- `ContentType` must be recognized and allowed.
- Storage and database record must remain transaction-consistent.

### DocumentAccessGrant

Represents explicit access to a document beyond the default project visibility rules.

| Field | Type | Required | Notes |
|---|---|---:|---|
| DocumentAccessGrantId | int | Yes | Primary key |
| DocumentId | int | Yes | Related document |
| UserId | int | No | User-level grant |
| TeamId | int | No | Team-level grant |
| GrantedByUserId | int | Yes | Actor who granted access |
| CreatedAt | DateTime | Yes | Grant creation time |
| IsActive | bool | Yes | Current active state |

Rules:
- A grant is valid only for one recipient type (user or team).
- Duplicate grants for the same recipient are prevented.
- Team grants must evaluate current team membership when authorization is checked.

### DocumentActivity

Captures business and admin audit events for the document lifecycle.

| Field | Type | Required | Notes |
|---|---|---:|---|
| DocumentActivityId | int | Yes | Primary key |
| DocumentId | int | Yes | Related document |
| ActorUserId | int | Yes | The user performing the action |
| ActionType | enum/string | Yes | Upload, Download, Share, Replace, Delete, Preview, View, Restore |
| ActionTime | DateTime | Yes | Time of action |
| Details | string | No | Machine-readable or human-readable summary |

Rules:
- Non-content metadata remains after permanent deletion for admin reporting.
- Only administrators can view audit summaries.

### Category

Enumerated classification used by document metadata and filtering.

Allowed values:
- Project Documents
- Team Resources
- Personal Files
- Reports
- Presentations
- Other

### Project and Task Association

These are existing application concepts reused for document context rather than introducing a separate document workflow domain.

- A document may belong to a single project or task context.
- Authorized project members see project-associated documents by default.
- Task uploads inherit the task’s project association when available.

## State Transitions

### Document lifecycle

- Upload started -> validation
- Validation fails -> rejected
- Validation succeeds -> quarantined while malware scan is pending
- Scan succeeds -> available
- Share granted -> available / shared access
- Replace by owner -> available with updated file and preserved metadata
- Delete confirmed -> deleted and removed from user-accessible storage
- Admin audit record remains

## Relationships Summary

```text
User 1---* Document
Project 1---* Document
TaskItem 1---* Document
Document 1---* DocumentAccessGrant
Document 1---* DocumentActivity
```

## Key Validation Notes

- A document cannot be available if the file is missing or the database state points to no stored blob.
- A file may be quarantined if scanning is unavailable or incomplete.
- Shared access must not create duplicate access or notifications for an already-authorized recipient.
- Access checks must use the latest project membership state, not a stale snapshot.
