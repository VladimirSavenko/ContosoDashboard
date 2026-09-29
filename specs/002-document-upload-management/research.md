# Research: Document Upload and Management

## Decision

Implement document storage using a local filesystem abstraction managed through a document service, with metadata stored in SQLite and authorization enforced through project membership, document access grants, and user ownership rules. Document lifecycle states and audit events will be recorded as first-class entities to support compliance and reporting without exposing content to unauthorized actors.

## Rationale

The application is an offline-first training system with an existing ASP.NET Core + Blazor Server + SQLite pattern. The feature requirements emphasize secure access control, local storage compatibility, and a future cloud migration path. That makes a storage abstraction and service-layer authorization the least risky and most consistent design decision.

## Alternatives Considered

1. Store files directly in the web content folder
   - Rejected because it exposes file paths and makes authorization harder to enforce consistently. It also mixes user content with static assets and complicates non-public storage requirements.

2. Put all document state directly into the UI layer or Razor page logic
   - Rejected because it violates the layered architecture and would reduce testability and security enforcement.

3. Use a cloud storage model from the start
   - Rejected because the training environment is intentionally offline and the constitution requires local-first behavior without external services.

4. Treat project documents and personal documents as the same access model
   - Rejected because the specification explicitly distinguishes between project-visible documents and personal documents, and the access rules differ significantly.

## Research Findings

### 1. Storage model

A document should have a unique, non-user-controlled storage identifier generated before persistence. This avoids path guessing and protects against sensitive metadata and file names being exposed through the filesystem or URL. The storage key should be persisted and used only through the service layer.

### 2. Access model

Access should be determined from a combination of:
- document owner
- project membership
- explicit document sharing grants
- administrator privileges

This supports the requirement that project-associated documents are visible to authorized project members by default while personal documents require explicit sharing.

### 3. Team membership rules

Because team membership may change over time, the authorization check should evaluate current membership rather than static sharing snapshots. This matches the requirement that a member who leaves the team loses access without re-sharing.

### 4. Audit and deletion

The design should keep document content and file storage permanently removed after confirmation while retaining non-content activity metadata for admin reporting. This preserves auditability without leaving an accessible file behind.

### 5. Lifecycle state handling

The document domain should support at least these states:
- Draft / Pending validation
- Quarantined
- Available
- Shared
- Replaced
- Deleted

This allows upload, scanning, quarantine, and post-delete operations to be represented cleanly and prevents documents from being treated as usable while they are still unreachable or pending approval.

## Final Approach

The implementation will add a `Document` aggregate with metadata, a `DocumentAccessGrant` relationship model, and a `DocumentActivity` audit log. A document storage service will hide the actual file-system logic behind an interface, while UI pages will interact through service methods and role-based authorization checks. This keeps the feature aligned with the project’s architecture and security constraints while preserving a straightforward migration path to cloud storage in a later implementation.
