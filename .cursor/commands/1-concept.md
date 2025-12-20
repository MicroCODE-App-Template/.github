# CONCEPT - Discuss ideas without generating code

## Phase: 1: CONCEPT/CHANGE/CORRECTION

**Purpose**: Discuss ideas, requirements, and concepts without generating code.

## Instructions

I have a CONCEPT/CHANGE/CORRECTION I want to talk to you about. **DO NOT generate any changes, files, or code** - just talk to me until I'm ready to start the development process with you. Make no assumptions about when that time comes.

### Critical Rules

- **If you cannot access a file or resource I reference, STOP and ask me how to access it**
- **Do not explore or read other files unless I explicitly ask you to**
- **No code generation** - this phase is for discussion only

### Initial Questions

Ask me two questions to start:

1. **Is there an existing GitHub Issue # assigned?**
2. **Is this a FEATURE, BUGFIX, HOTFIX, RELEASE, or simple TASK?**

### Issue Document Management

**If there is an Issue #:**

- Start a new file from the existing templates under: `.issue/.github/ISSUE_TEMPLATE/{bugfix|feature|hotfix|release}/`
- This document will be our place to capture our standard workflow documentation.
- It will live with this change until the Pull Request stores it permanently

**If there is already an ISSUE Markdown started:**

- Open it and read what we have so far to get started
- Continue the conversation based on existing context

### Alternative: AIN Document Workflow

If I tell you there is no GitHub ISSUE #:

1. **Locate or create the AIN document** for this task:
   - Ask for a simple TASK NAME, use taht to search for an exisitng AIN doc.
   - Use the task name as TASK_NAME - upper snake case.
   - If starting a new task: Copy `.github/docs/AI/AIN [20YY-MM-DD] WORKFLOW_TEMPLATE.md` to create `AIN [20YY-MM-DD] [TASK_NAME].md`
   - If continuing an existing task: Open the existing `AIN [20YY-MM-DD] [TASK_NAME].md` document
2. **Navigate to the C: CONCEPT/CHANGE/CORRECTION section** in the document
3. **Update the document** with your discussion in that section

## Context

- **Issue Templates**: `.issue/.github/ISSUE_TEMPLATE/` - Use these for Issue-based workflow
- **AIN Workflow Template**: `.github/docs/AI/AIN [20YY-MM-DD] WORKFLOW_TEMPLATE.md` - Alternative workflow
- **AI Rules**: `.github/docs/AI/AI-RULES.md`
- **Component Rules**: `.cursor/rules/[component].mdc`

## Important

- All workflow phases go in **ONE document** - do NOT create separate files
- This is a discussion phase - no code generation yet
- Document all ideas, questions, and considerations
