# PLAN - Create implementation plan

## Phase: 3: PLAN

**Purpose**: Create a step-by-step implementation plan based on the design.

## Instructions

PLAN a detailed implementation of this design. **Do NOT change any code or files at this point.**

### Plan Requirements

This phase should outline:

- **Affected files** - List all files that will be created or modified
- **Modules** - Identify which modules/components are affected
- **Expected refactor areas** - Note any code that needs refactoring
- **Diagrams** - Include visual representations of the solution
- **Interface contracts** - Define APIs, function signatures, data structures
- **User stories** - Document user-facing changes and workflows
- **Any other documents** required to implement a solid, secure, and complete implementation

### Critical Rules

- **Make no assumptions here** - if something is unclear, give me a list of questions to clarify before proceeding
- **Do not proceed with any code or file changes** until I give you an explicit command to implement
- **Be thorough** - this plan should be detailed enough that implementation is straightforward

### Plan Organization

Organize by component if working across multiple repos:

- **Server changes** (API, controllers, models, migrations)
- **Client changes** (components, views, routes, locales)
- **App changes** (mobile components, native features)
- **Admin changes** (if applicable)
- **Admin/Console changes** (if applicable)
- **Portal changes** (if applicable)

Include for each:

- Step-by-step tasks
- Order of implementation
- Files to create/modify
- Dependencies between steps
- Testing approach
- Database migrations (if needed)
- API endpoint changes
- Component creation/modification
- Locale file updates
- Configuration changes

### Document the Plan

- **If using Issue Template**: Update the working file in `.issue/.github/ISSUE_TEMPLATE/`
- **If using AIN document**: Navigate to the P: PLAN section and update it

## Context

- **AI Rules**: `.github/docs/AI/AI-RULES.md` - Follow coding standards
- **Issue Template**: `.issue/.github/ISSUE_TEMPLATE/` - Update the working file
- **AIN Workflow Template**: `.github/docs/AI/AIN [20YY-MM-DD] WORKFLOW_TEMPLATE.md` (alternative workflow)
- **Component Rules**: `.cursor/rules/[component].mdc`
- **File Naming**: See AI-RULES.md for entity-centric naming conventions

## Important

- **No code changes** - planning only
- **Ask questions** if anything is unclear - make no assumptions
- **Include diagrams and documentation** in the plan
- **Wait for explicit command** to proceed to IMPLEMENT phase
- Plan should be actionable and specific
- Consider account_id scoping for all database operations
- Follow the layered architecture: API → Controller → Model
- Reference existing patterns in the codebase
