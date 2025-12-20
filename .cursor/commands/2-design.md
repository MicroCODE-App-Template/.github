# DESIGN - Design detailed solution

## Phase: 2: DESIGN

**Purpose**: Design a detailed solution approach before implementation.

## Instructions

DESIGN a detailed solution to implement this change following the rules documented in `.github/docs/AI/AI-RULES.md` file.

### Design Requirements

- **Include test cases** for basic features by extending the existing `/test` structure in each repository
- **Make no changes at this time** - simply create a detailed high-level DESIGN for the change
- **The details of the implementation** will come after you have no questions about how to update the existing code in all repos for this change
- **We must reach a 95% or higher confidence level** in the design before proceeding

### File Access Rules

- **During this phase you have full access to read all files** in the repos
- **Do NOT change anything** except our referenced ISSUE_TEMPLATE (or AIN document if using that workflow)
- Read existing code patterns, architecture, and implementations
- Understand how changes will affect all components

### Design Components

Consider all components in the monorepo:

- `server/` - Backend API, controllers, models
- `client/` - React web frontend
- `app/` - React Native mobile app
- `admin/` - Admin console
- `portal/` - Astro marketing site

Include in your design:

- Architecture and structure
- Component interactions
- Data flow and state management
- API design (if applicable)
- Database schema changes (if applicable)
- UI/UX considerations (if applicable)
- Security considerations
- Error handling approach
- **Test cases** for basic features

### Document the Design

- **If using Issue Template**: Update the D: DESIGN section in the working file in `.issue/.github/ISSUE_TEMPLATE/`
- **If using AIN document**: Navigate to the D: DESIGN section and update it

## Context

- **AI Rules**: `.github/docs/AI/AI-RULES.md` - Follow these rules strictly
- **Issue Templates**: `.issue/.github/ISSUE_TEMPLATE/` - Update the working file only
- **AIN Workflow Template**: `.github/docs/AI/AIN [20YY-MM-DD] WORKFLOW_TEMPLATE.md` (alternative workflow)
- **Component Rules**: `.cursor/rules/[component].mdc`
- **Architecture**: `.github/.cursor/instructions.md`

## Important

- **95% confidence required** before proceeding to PLAN phase
- **Read all files** to understand existing patterns
- **Only modify** the ISSUE_TEMPLATE working file (or AIN document)
- **Include test cases** in the design
- **No code changes** - design only
- **Wait for explicit permission** before proceeding to PLAN phase
