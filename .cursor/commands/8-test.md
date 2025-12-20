# TEST - Run tests

## Phase: 8: TEST

**Purpose**: Run tests to verify the implementation works correctly.

## Instructions

TEST the solution using the code as designed and implemented.

### Test Requirements

- **Generate logs and MD files with the results**
- **Unit, integration, and smoke tests** as applicable
- **Log output should be in both human-readable and machine-readable** (e.g., JSON/MD) formats
- **Use the MOCHA tests** defined by `package.json` and held in `*/test` under each repo

### Pass Criteria

- **All Tests must pass** to call this phase complete
- **Tests should not be edited in order to pass** without a discussion
- **If no tests exist for a repo, report that explicitly** and propose at least one test to add (but do not create it without my approval)

### Critical Rules

- **Do not proceed with any code or file storage in GitHub** until I give you an explicit command to create a Pull Request

### Testing Process

1. **Run tests** for each affected component:

   ```bash
   # In server directory
   cd server
   npm test

   # In client directory
   cd client
   npm test

   # In app directory
   cd app
   npm test
   ```

2. **Test the implementation**:

   - Unit tests for new functions
   - Integration tests for API endpoints
   - Component tests for UI changes
   - End-to-end tests if applicable

3. **Verify functionality**:

   - Test happy paths
   - Test error cases
   - Test edge cases
   - Test account scoping (multi-tenant security)
   - Test input validation

4. **Generate test reports**:

   - Human-readable format (markdown/logs)
   - Machine-readable format (JSON)
   - Save test results to files

5. **Document test results**:
   - Tests that passed
   - Tests that failed (and fixes applied)
   - New tests added
   - Test coverage information
   - Repos with no tests (and proposed tests)

### Document Results

- **If using Issue Template**: Update the working file in `.issue/.github/ISSUE_TEMPLATE/`
- **If using AIN document**: Navigate to the T: TEST section and update it

## Context

- **Test Framework**: MOCHA tests defined by `package.json` and held in `*/test` under each repo
- **Test Location**: `server/test/` directory (and similar in other repos)
- **AI Rules**: `.github/docs/AI/AI-RULES.md`
- **Issue Template**: `.issue/.github/ISSUE_TEMPLATE/` - Update the working file
- **AIN Workflow Template**: `.github/docs/AI/AIN [20YY-MM-DD] WORKFLOW_TEMPLATE.md` (alternative workflow)

## Important

- **All Tests must pass** to call this phase complete
- **Tests should not be edited in order to pass** without a discussion
- **If no tests exist for a repo**, report that explicitly and propose at least one test to add (but do not create it without approval)
- **Do not proceed with any code or file storage in GitHub** until explicit command to create Pull Request
- **Generate logs in both human-readable and machine-readable formats** (JSON/MD)
- Test account scoping to prevent data bleeds
- Test input validation and error handling

## Test Checklist

- [ ] All existing tests pass
- [ ] New functionality has tests
- [ ] Account scoping verified (multi-tenant security)
- [ ] Input validation tested
- [ ] Error handling tested
- [ ] Edge cases covered
