# feature/0003--use-mongodb-ids (repo: app-template)

| Issue # | PR #    | Requested by | Implemented by  | Date       | Release # |
| ------- | ------- | ------------ | --------------- | ---------- | --------- |
| `#0003` | `#pppp` | MongoDB      | Timothy McGuire | 2025-11-27 | `a0.0.0`  |

- **Source Branch:** `main`
- **Working Branch:** `feature/0003--use-mongodb-ids`
- **Target Branch:** `develop`
- **Tag:** `a0.0.0`

## Request

Summary description of the requested feature or change. Include links to product requirements, customer tickets, or GitHub issues as applicable.

Change all primary keys from Gravity's "id" to MongoDB ObjectIDs "\_id" across the application.

## Current Behavior (AS-IS)

Describe in detail the current behavior, including screenshots, logs, or relevant references.

Kyle's original implementation uses custom random IDs for all entities.
This requires additional logic to manage ID generation and uniqueness.

- **Current UI, Workflow or Code:**
<p align="left"><img src="./.images/feature-0003--before.png" width="720" title="Current behavior" style="border: 0.5px solid lightgray;"></p>

## Requested Behavior (TO-BE)

Describe in detail the desired behavior. Include acceptance criteria, UX/UI mocks, API changes, and any non-functional requirements.

MongoDB recommends using ObjectIDs as primary keys for documents.
Switching to ObjectIDs will simplify the codebase and improve performance.
It will also enhance compatibility with MongoDB features and tools,
and move us to our standard key naming convention, '\_id' and 'parent_id'.

- **Proposed UI, Workflow or Code:**
<p align="left"><img src="./.images/feature-0003--after.png" width="720" title="Requested behavior" style="border: 0.5px solid lightgray;"></p>

## Reason for Change

Describe the business reason for the change. Capture measurable benefits, customer impact, compliance obligations, or other drivers.

MongoDB ObjectIDs are optimized for performance and storage efficiency.
Using them will reduce complexity in our codebase and improve maintainability.

## Implementation Plan

Describe the change to code, process, or configuration required to achieve the TO-BE state. Reference affected repos, modules, and components. Include testing strategy and rollout notes.

This change will involve updating the data models, database access layers,
and any related business logic to use MongoDB ObjectIDs. I've already down this once to
Kyle's original implementation, in the original LADDERS code, so this will be a reversion to that approach.

- **Repositories / Modules affected:** `server`, `admin`
- **Dependencies / External systems:** none
- **Database Changes:** Update schemas to use ObjectIDs
- **Testing Plan:** CRUD operations tests, integration tests
- **Risks & Mitigations:** None at this time, not deploying to production yet.
- **Rollout Plan:** Deploy to staging, monitor for issues, then deploy to production.
- **Back-out Plan:** Roll back to previous data model and code versions if issues arise during staging or production deployment.
