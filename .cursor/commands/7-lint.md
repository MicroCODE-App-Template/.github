# LINT - Check and fix linting issues

## Phase: 7: LINT

**Purpose**: Check and fix all ESLint errors and warnings before declaring work complete.

## Instructions

LINT everything using `server/bin/lint.all.js`, check for any warnings or errors, and fix them all **WITHOUT modifying any eslint.config.js rules**.
Furthermore, do **NOT** use file level exceptions to rules to stop warnings or errors.

### Critical Rules

- **If linting fails due to config limitations, raise a warning in the log but do not modify the rules**
- **All ESLint errors must be fixed** before proceeding
- **Do not modify eslint.config.js files** - work within existing rules

### Linting Process

1. **Run the lint-all script**:

   ```bash
   cd server
   node bin/lint.all.js
   ```

2. **Then run ESLint** for each affected component individually:

   ```bash
   # In server directory
   cd server
   npm run lint

   # In client directory
   cd client
   npm run lint

   # In app directory
   cd app
   npm run lint

   # In admin directory
   cd admin
   npm run lint

   # In portal directory
   cd portal
   npm run lint
   ```

3. **Fix all errors and warnings**:

   - Use `npm run lint:fix` or `npm run fix` if available
   - Manually fix issues that can't be auto-fixed
   - Ensure code follows the JavaScript Style Guide
   - **Do NOT modify eslint.config.js** - work within existing rules

4. **Document linting results** using the required structure (see "Documentation Structure" below)

5. **Verify no errors remain**:
   - Run lint again to confirm
   - All errors must be fixed before proceeding

### Documentation Structure

**MANDATORY**: All LINT sections must follow this exact structure:

#### Step 1: Initial Repo-Wide Linting Results

- **Command**: `cd server && node bin/lint.all.js`
- **Document**: Create a table showing initial repo-wide status with:
  - Repo name (SERVER, CLIENT, APP, ADMIN Server, ADMIN Console, PORTAL)
  - Errors count
  - Warnings count
  - Status (✅ No problems found / ⚠️ X problems)
  - **TOTAL** row with aggregate counts
- **Note**: Document any pre-existing errors unrelated to this implementation

#### Step 2: Individual Repo Issues (Modified Files Only)

- **Command**: `cd {repo} && npm run lint` (for each repo with modified files)
- **Document**: For each repo with modified files:
  - List all modified files checked
  - Show initial issues found (errors and warnings)
  - Include file paths, line numbers, and issue descriptions

#### Step 3: Fixes Applied

- **Document**: For each fix:
  - File path
  - Line number
  - Issue type (error/warning)
  - Description of the fix applied

#### Step 4: Final Repo-Wide Linting Results

- **Command**: `cd server && node bin/lint.all.js` (after fixes)
- **Document**: Create a table showing final repo-wide status with:
  - Same structure as Step 1
  - Show changes from initial (e.g., "1 warning fixed")
  - Note pre-existing errors that remain unchanged

#### Step 5: Final Individual Repo Status (Modified Files)

- **Command**: `cd {repo} && npm run lint` (for each repo with modified files)
- **Document**: Verify all modified files pass after fixes

#### Summary

- **BEFORE (Modified Files Only)**: Show errors/warnings counts per repo
- **AFTER (Modified Files Only)**: Show final errors/warnings counts per repo
- **Corrections Made**:
  - Count of warnings fixed
  - Count of errors fixed
- **Status**: Ready to proceed to next phase

### Document Results

- **If using Issue Template**: Update the working file in `.issue/.github/ISSUE_TEMPLATE/`
- **If using AIN document**: Navigate to the L: LINT section and update it with the structure above

## Context

- **Lint Script**: `server/bin/lint.all.js` - Use this for comprehensive linting
- **ESLint Configs**: Each repo has its own `eslint.config.js` - **DO NOT MODIFY**
- **Style Guide**: https://github.com/MicroCODEIncorporated/JavaScriptSG.git
- **AI Rules**: `.github/docs/AI/AI-RULES.md` - "After every edit of a module you must ESLint"
- **Issue Template**: `.issue/.github/ISSUE_TEMPLATE/` - Update the working file
- **AIN Workflow Template**: `.github/docs/AI/AIN [20YY-MM-DD] WORKFLOW_TEMPLATE.md` (alternative workflow)

## Important

- **MANDATORY**: All ESLint errors must be fixed before declaring work "done"
- **DO NOT modify eslint.config.js rules** - work within existing rules
- **If config limitations prevent fixes**, raise a warning in the log but do not modify rules
- Use `server/bin/lint.all.js` for comprehensive linting
- Use `{repo}/bin/lint.js` for detailed issue reporting
- Run ESLint with each repo's specific config
- Document any acceptable warnings with justification

## Common Issues

- Missing JSDoc headers on exported functions
- Incorrect file naming (should be snake_case for files)
- Missing input validation
- Direct database queries from controllers
- Missing `account_id` in queries
