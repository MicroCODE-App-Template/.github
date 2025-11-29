# MicroCODE Web App Workflow

This document defines the standard development workflow for changes across the `app-template` repositories.

The lifecycle is:

- ALT-SHIFT-**C → D → P → V → B → I → L → T → M → R** are defined as `AutoHotKey` shortcuts.
- Concept → Design → Plan → Review → Branch → Implement → Lint → Test → Document → PR/Release

---

## C – Concept

I have a **CONCEPT** I want to talk to you about.

- Do **not** generate any changes, files, or code; just talk until explicitly instructed to start development.
- Make no assumptions about when that time comes.
- If you cannot access a file or resource I reference, **STOP** and ask how to access it.
- Do not explore or read other files unless I explicitly ask you to.

At the start of this phase, the assistant must ask two questions:

1. Is there an existing GitHub Issue # assigned?
2. Is this a **FEATURE**, **BUGFIX**, **HOTFIX**, or **RELEASE**?

If there **is** an Issue #:

- Start a new ISSUE markdown file from the appropriate template under:
  - `.github/ISSUEs/bugfix/`
  - `.github/ISSUEs/feature/`
  - `.github/ISSUEs/hotfix/`
  - `.github/ISSUEs/release/`
- This ISSUE document is the place to write the design, document tests, capture results, etc.
- It lives with the change until the Pull Request stores it permanently.

If there is **already** an ISSUE markdown started:

- Open it and read what we have so far to get started (but do not open any other files unless explicitly asked).

---

## D – Design

**DESIGN** a detailed solution to implement this change.

- Follow the rules documented in `docs/AI/AI-RULES.md`.
- Do **not** change any code or files in this phase.
- Refine the concept into a concrete, technical design that is ready for planning and implementation.
- Capture the design primarily in the related ISSUE markdown (and linked artifacts as needed).

Design activities typically include:

- Selecting or refining architecture and patterns for this change.
- Defining data models, API contracts, validation rules, and error handling behavior.
- Identifying cross-cutting concerns (security, logging, observability, performance, i18n, etc.).
- Mapping responsibilities across repos (`admin/`, `client/`, `server/`, `portal/`, `app/`).

Testing aspects of the design:

- Identify test categories (unit, integration, smoke) relevant to the change.
- For each basic feature or scenario, define test cases that can be implemented by **extending the existing `/test` structure** in each affected repository.
  - Reuse existing patterns, helpers, and conventions already present under `*/test`.
  - Avoid introducing new test frameworks unless explicitly agreed.
- Document these proposed test cases (names, purpose, inputs, expected outputs) in the ISSUE markdown.

Only proceed to **P – Plan** once the design is documented and aligned with `docs/AI/AI-RULES.md`.

---

## P – Plan

**PLAN** a detailed implementation of this design. During this phase:

- Do **not** change any code or files.
- Outline affected files, modules, and expected refactor areas.
- Include, as needed:
  - Diagrams (architecture, sequence, data flow, etc.).
  - Interface contracts (types, DTOs, API shapes, etc.).
  - User stories / use cases.
  - Any other documents required for a solid, secure, and complete implementation.
- Make no assumptions: if something is unclear, provide a list of questions to clarify before proceeding.
- Do **not** proceed with any code or file changes until the user gives an explicit command to implement.

Output of this phase should be captured primarily in the ISSUE markdown (and linked artifacts if relevant).

---

## V – Review

**REVIEW** and validate the plan.

- Summarize the plan and key decisions.
- Provide a confidence rating from **0% to 100%** on ability to implement this plan in code.
- If there are any assumptions or questions remaining, list them and aim to move to at least **95% confidence**.
- Do **not** proceed to branching or implementation until the user explicitly accepts the plan and/or resolves the questions.

---

## B – Branch

**BRANCH** the required repositories.

- Changes may affect multiple repos in this directory/collection (e.g., `admin/`, `client/`, `server/`, `portal/`, `app/`).
- Before creating branches, list which repos you intend to modify and get confirmation from the user if there is any doubt.
- Create a Git branch for **each** affected repo.
- Branch names **must** follow the Git Flow implementation rules defined in `/docs/DEV/DEVELOPER-GIT.md` (e.g., `feature/...`, `bugfix/...`, `hotfix/...`, `release/...`).
- If you are not positive about the branch names or prefixes, **stop and ask** the user.

---

## I – Implement

**IMPLEMENT** the approved plan.

- Begin code and file changes only after explicit user approval.
- Be elegant and follow MicroCODE’s philosophy:

  > Code like a Machine — Consistently and Explicitly, Simply and for Readability. Hail CAESAR!

- Follow the **MicroCODE JavaScript Style Guide** (`MicroCODE JavaScript Style Guide.pdf`).
- Keep changes focused on the scoped Issue / concept.
- Update or create tests as required by the plan, but major test strategy changes should be discussed.

---

## L – Lint

**LINT** everything.

- Run the central lint script:

  - `server/bin/lint.all.js`

- Check for any warnings or errors and fix them **all**.
- Do **not** modify any `eslint.config.js` rules.
- If linting fails due to configuration limitations that cannot reasonably be fixed in the scope of this change:
  - Raise a warning in the logs/ISSUE markdown.
  - Do **not** modify the lint rules; instead, document the limitation.

---

## T – Test

**TEST** the solution using the code as designed and implemented.

- Use the **Mocha** tests as defined by each repo’s `package.json` and held in `*/test` under each repo.
- Run appropriate test suites (unit, integration, smoke) as applicable to the change.
- Generate logs and Markdown files with the results.
  - Logs should be both human-readable and machine-readable (e.g., JSON and/or Markdown summaries).
- Pass Criteria:
  - **All tests must pass** to call this phase complete.
  - Tests should **not** be edited merely to make them pass without a prior discussion.
- If a repo has no tests:
  - Explicitly note this in the ISSUE markdown.
  - Optionally propose tests to add, but do not create them without explicit approval.
- Do **not** proceed with any code or file storage in GitHub (e.g., PR creation) until the user gives an explicit command to create a Pull Request.

---

## M – Document

**DOCUMENT** the implemented and successfully tested solution.

- Update all affected `README.md` files (per repo, per component as appropriate).
- Ensure new or updated APIs are covered in relevant OpenAPI/Swagger schemas where applicable.
- Update or add:
  - High-level architecture notes if relevant.
  - Developer notes or migration steps if the change affects deployment, configuration, or data.
- Ensure the ISSUE markdown reflects:
  - Final design decisions.
  - Test approach and results.
  - Any known limitations or follow-up tasks.

---

## R – Pull Request / Release

**Create a PULL REQUEST** for each affected repo.

- For each repo in this GitHub organization that was changed:
  - Open a PR from the feature/bugfix/hotfix/release branch to the appropriate base branch (per Git Flow).
- Update the overall change log found in `.github/CHANGELOG.md`:
  - Include a reference to the Issue #.
  - Include a tag lock / note tying this entry to the PR.
- After merge:
  - Ensure a **Git tag** matching the PR number or release name is added.
  - Confirm that release notes and tags remain consistent with `.github/CHANGELOG.md`.

---

## Summary

- **C – Concept:** Requirements, scope, constraints, and initial design discussion.
- **P – Plan:** Design and impact analysis (files, modules, refactors) without touching code.
- **V – Review:** Validate the Plan, targeting ≥95% confidence before coding.
- **B – Branch:** Proper Git Flow setup across all affected repos.
- **I – Implement:** Actual coding aligned with MicroCODE style/philosophy.
- **L – Lint:** Static analysis / style enforcement, without changing lint rules.
- **T – Test:** Unit/integration/smoke, with human- and machine-readable logs/MD.
- **M – Document:** README, API docs, OpenAPI/Swagger, and ISSUE markdown.
- **R – PR:** Code review, changelog updates, tagging, and release linkage.

Effectively: **Discuss → Design → Review → Branch → Build → Verify → Document → Release**.

---

## Quickstart (Typical Commands)

> Exact scripts may vary per repo; always check each `package.json` for the authoritative commands.

### 1. Environment & Install

From the monorepo root (`app-template/`):

- Install dependencies for each app as needed:
  - `admin/`: `cd admin && npm install`
  - `client/`: `cd client && npm install`
  - `server/`: `cd server && npm install`
  - `portal/`: `cd portal && npm install`
  - `app/`: `cd app && npm install`

### 2. C → P → V (Concept, Plan, Review)

- Start in GitHub with / by confirming an **Issue #**.
- Use the appropriate ISSUE template under `.github/ISSUEs/` (`bugfix`, `feature`, `htfix`, `release`).
- Capture design, questions, decisions, and test strategy directly in that ISSUE markdown.

### 3. B (Branch)

From each affected repo directory (example: `server/`):

- Ensure you are on the correct base branch (e.g., `develop`, `main`, or per `docs/DEV/DEVELOPER-GIT.md`).
- Create and switch to a feature/bugfix/hotfix/release branch:

  - `git checkout -b feature/XXXX--short-description`

### 4. I (Implement)

- Implement changes per the plan.
- Use repo-specific scripts (examples; confirm via `package.json`):
  - `npm run dev` – run local dev server (where available).
  - `npm run build` – build for production.

### 5. L (Lint)

- From the monorepo root or `server/` (per existing setup):

  - `node server/bin/lint.all.js`

- Fix all reported issues without changing `eslint.config.js` rules.

### 6. T (Test)

- From each affected repo (examples):
  - `npm test` or `npm run test`
  - Or any Mocha-specific script defined in `package.json` (e.g., `npm run test:unit`)
- Ensure all tests pass before moving to documentation and PR.

### 7. M (Document)

- Update relevant `README.md` files in affected repos.
- Update OpenAPI/Swagger specs where API changes occurred.
- Ensure the ISSUE markdown is up to date with:
  - Final design
  - Test runs and outcomes
  - Any follow-up work

### 8. R (PR & Release)

- From each affected repo, push your branch:

  - `git push -u origin feature/XXXX--short-description`

- Open a Pull Request in GitHub:
  - Link the Issue #.
  - Reference this workflow where helpful.
- After merge:
  - Add/update entries in `.github/CHANGELOG.md`.
  - Ensure a Git tag is created that matches the PR number or release name, as per `docs/DEVDEVELOPER-GIT.md`.

For more details, follow this document from top to bottom during each change.
