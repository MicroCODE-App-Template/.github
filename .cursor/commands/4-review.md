# REVIEW - Review and validate the implementation plan

## Phase: 4: REVIEW

**Purpose**: Review and validate the implementation plan before proceeding to implementation.

## Instructions

REVIEW and validate our plan so far.

### Confidence Rating

Give me a **confidence rating from 0% to 100%** on your ability to implement this plan in code.

### Review Checklist

Validate the plan thoroughly:

- **Completeness** - Are all steps covered?
- **Dependencies** - Are steps in correct order?
- **Security** - account_id scoping, input validation
- **Consistency** - Follows existing patterns?
- **Architecture** - Matches layered structure?
- **File naming** - Follows entity-centric conventions?
- **Test coverage** - Are test cases included?

### Questions and Assumptions

- **If there are any assumptions or questions remaining**, give me a list of questions to move us to at least **95% confidence**
- **Do not proceed to CODE IMPLEMENTATION** until I explicitly accept the plan or answer/waive the remaining questions
- **IMPORTANT**: Continue editing the workflow document itself (AIN files or Issue Template derivatives) without stopping - this approval requirement only applies to actual code implementation, not workflow document editing

### Check Against Standards

- JavaScript Style Guide: https://github.com/MicroCODEIncorporated/JavaScriptSG.git
- AI-RULES.md coding standards
- Component-specific rules in `.cursor/rules/`

### Review Output

Document:

- **Confidence rating** (0-100%)
- **What looks good** - Strengths of the plan
- **What needs adjustment** - Areas that need clarification
- **Remaining questions** - List of questions to reach 95% confidence
- **Any concerns or risks** - Potential issues to address
- **Approval status** - Ready to proceed or needs clarification

### Document the Review

- **If using Issue Template**: Update the working file in `.issue/.github/ISSUE_TEMPLATE/`
- **If using AIN document**: Navigate to the V: REVIEW section and update it

## Context

- **AI Rules**: `.github/docs/AI/AI-RULES.md`
- **Issue Template**: `.issue/.github/ISSUE_TEMPLATE/` - Update the working file
- **AIN Workflow Template**: `.github/docs/AI/AIN [20YY-MM-DD] WORKFLOW_TEMPLATE.md` (alternative workflow)
- **Component Rules**: `.cursor/rules/[component].mdc`
- **Style Guide**: https://github.com/MicroCODEIncorporated/JavaScriptSG.git

## Important

- **95% confidence required** before proceeding to BRANCH phase
- **List all remaining questions** - don't proceed with assumptions
- **Wait for explicit acceptance** before creating branches or implementing code
- **Update Status in Issue Template** when plan is approved
- Review should be thorough before proceeding to implementation
- Address any concerns before moving to BRANCH phase
- **Workflow Document Editing**: Continue editing the AIN document (`.github/docs/AINs/*.md`) continuously without stopping for approval. See `.github/docs/AI/AI-RULES-WORKFLOW-EDITING.md` for details.
