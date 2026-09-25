<!--
Sync Impact Report
- Version change: unversioned scaffold -> 1.0.0
- Modified principles: five scaffold placeholders -> Offline-First Architecture,
	Security by Design, Testable Requirements, Layered Simplicity, and Observable UX
- Added sections: Security and Architecture Constraints; Development Workflow and
	Quality Gates
- Removed sections: none
- Follow-up TODOs: Confirm the original ratification date.
-->

# ContosoDashboard Constitution

## Core Principles

### I. Offline-First Architecture
Features MUST work with the local SQLite database and local infrastructure abstractions
without requiring external services. Infrastructure dependencies MUST be accessed through
interfaces so that cloud implementations can be substituted without changing business
logic. This preserves reliable offline training and a credible migration path.

### II. Security by Design
Authentication and authorization MUST be enforced at the page and service boundaries.
Services MUST verify access to user-owned or project-scoped data, preventing IDOR and
unauthorized disclosure. New security-sensitive behavior MUST include a focused test or
documented verification step. Training shortcuts MUST remain clearly separated from
production security requirements.

### III. Testable Requirements
Every feature MUST define observable acceptance behavior, including empty, error, and
unauthorized states where applicable. Business rules MUST be covered by automated tests
or by a documented manual verification procedure when automation is impractical. A change
is incomplete until its relevant checks pass.

### IV. Layered Simplicity
Changes MUST respect the separation between pages, services, data access, and models.
Business logic MUST reside in services rather than UI components or persistence details.
The simplest design that satisfies the requirements MUST be preferred; added abstraction
or complexity MUST have a documented reason.

### V. Observable UX
User-facing workflows MUST communicate loading, success, empty, and failure states clearly.
Displayed aggregates, dates, permissions, and currency values MUST be unambiguous and
reconcilable with their source data. Navigation and feedback MUST remain usable on the
supported desktop and mobile layouts.

## Security and Architecture Constraints

The application MUST remain suitable for offline training: SQLite is the default data
store, external service calls are out of scope unless explicitly specified, and mock
authentication MUST NOT be represented as production-ready identity. Changes MUST retain
the existing ASP.NET Core, Blazor Server, Entity Framework Core, and dependency-injection
patterns unless a feature specification explicitly justifies a change. Secrets and
credentials MUST NOT be committed.

## Development Workflow and Quality Gates

Work MUST begin with a written feature specification when behavior or scope changes.
Plans and task breakdowns MUST remain consistent with that specification. Before a change
is considered complete, the author MUST run the narrowest relevant tests or build checks,
verify authorization-sensitive paths, and document any unavailable validation. Reviews
MUST check requirements coverage, security boundaries, layering, and meaningful empty or
error states.

## Governance
<!-- Example: Constitution supersedes all other practices; Amendments require documentation, approval, migration plan -->

This constitution governs feature specifications, plans, implementation, and reviews in
this repository. Amendments MUST be made through the constitution workflow, include a
Sync Impact Report, explain affected principles, and update the amendment date. Versioning
follows semantic versioning: MAJOR for incompatible governance changes, MINOR for added
or materially expanded principles, and PATCH for clarifications or non-semantic wording
changes. Every review MUST assess compliance with these principles; any exception MUST
state its scope, rationale, risk, and follow-up owner or task. The constitution MUST be
reviewed whenever the technology stack, security model, or training purpose changes.

**Version**: 1.0.0 | **Ratified**: TODO(RATIFICATION_DATE): confirm original adoption date | **Last Amended**: 2026-09-25
