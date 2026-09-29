# Quickstart: Document Upload and Management Validation

## Prerequisites

- .NET 10 SDK installed
- Repository checked out locally
- Application can run in the current feature branch
- At least one authenticated user exists in the mock auth system

## Setup

1. Open a terminal in the repository root.
2. Restore and run the app:

```bash
dotnet restore
cd ContosoDashboard
dotnet run
```

3. Sign in using one of the mock users in the app.

## Validation Scenarios

### 1. Upload a valid document

- Navigate to the document management area.
- Choose a small PDF or image file below 25 MB.
- Enter a title and category.
- Submit the upload.
- Expected result: upload succeeds, a success message appears, and the document appears in the user’s document list.

### 2. Validate rejection and retry flow

- Attempt to upload a file larger than 25 MB or an unsupported extension.
- Expected result: the upload is rejected with a clear message and no document record is created in the accessible list.

### 3. Search and filter authorized documents

- Create documents with distinct titles and categories.
- Use title search and category/project filters.
- Expected result: only documents authorized for the current user appear in the results.

### 4. Share and access rules

- Share a personal document with another user or team.
- Log in as the recipient and open the shared-documents view.
- Expected result: the document appears to the recipient and remains hidden to unauthorized users.

### 5. Remove access and verify security

- Remove a team member from the team or revoke access.
- Expected result: the removed user can no longer see the document without a new grant.

### 6. Delete and audit

- Delete a document after confirmation.
- Open the administrator report or activity log.
- Expected result: the file/content is removed from user-visible access, but audit metadata remains available for review.

## Expected Outcome

The feature is considered ready when the upload, browse, share, delete, and authorization scenarios all behave according to the feature spec and the constitution’s security and UX principles.
