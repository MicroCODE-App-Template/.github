# IMPLEMENT - Execute the plan

## Phase: 6: IMPLEMENT

**Purpose**: Execute the implementation plan step by step.

## Instructions

IMPLEMENT - this plan looks solid, go ahead and proceed with the implementation.

### Implementation Philosophy

Be elegant and follow our philosophy of **'Code like a Machine'**: **'Consistently and Explicitly, Simply and for Readability. Hail CAESAR!'**

### Critical Rules

- **Do not make any assumptions while coding**
- **If you are not 100% confident on any implementation points after opening the code, STOP, and let's figure it out together interactively**
- Follow the MicroCODE Style Guide: [MicroCODE JavaScript Style Guide.pdf] and the rules in `.github/docs/AI/AI-RULES.md`

### Follow the Plan

1. **Review the PLAN section** to understand the implementation steps

2. **Follow the plan step by step**:
   - Implement each step in order
   - Track progress as you complete each step
   - Note any deviations from the plan and why
   - Document any blockers or issues encountered

### Coding Standards

- Use `account_id` scoping for all database queries
- Validate inputs in controllers using `utility.validate()` with Joi
- Use `use()` HOF wrapper for API routes
- Follow entity-centric file naming: `{entity}.route.js`, `{entity}.controller.js`, `{entity}.model.js`
- Add JSDoc headers to all exported functions
- Use async/await (no callbacks or nested promises)

### Reference Component Rules

- Server: `.cursor/rules/server.mdc`
- Client: `.cursor/rules/client.mdc`
- App: `.cursor/rules/app.mdc`
- Admin: `.cursor/rules/admin.mdc`
- Portal: `.cursor/rules/portal.mdc`

### Document Progress

- **If using Issue Template**: Update the working file in `.issue/.github/ISSUE_TEMPLATE/`
- **If using AIN document**: Navigate to the I: IMPLEMENT section and update it

## Context

- **AI Rules**: `.github/docs/AI/AI-RULES.md` - Follow these rules strictly
- **Style Guide**: [MicroCODE JavaScript Style Guide.pdf] - Follow CAESAR philosophy
- **Issue Template**: `.issue/.github/ISSUE_TEMPLATE/` - Update the working file
- **AIN Workflow Template**: `.github/docs/AI/AIN [20YY-MM-DD] WORKFLOW_TEMPLATE.md` (alternative workflow)
- **Component Rules**: `.cursor/rules/[component].mdc`
- **Architecture**: `.github/.cursor/instructions.md`

## Important

- **STOP if not 100% confident** - let's figure it out together interactively
- **Do not make assumptions** while coding
- **Follow CAESAR philosophy**: "Consistently and Explicitly, Simply and for Readability"
- Follow the plan but document deviations
- Do not make any automatic commits
- Test as you go (don't wait until the end)
- Update Status in Issue Template (or AIN document) as you progress

## Quality Checklist

- [ ] All database queries use `account_id` scoping
- [ ] Inputs validated in controllers
- [ ] Error handling with `use()` wrapper
- [ ] JSDoc headers on exported functions
- [ ] Follows existing code patterns
- [ ] No ESLint errors (will check in LINT phase)
