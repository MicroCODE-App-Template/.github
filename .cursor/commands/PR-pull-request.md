# PULL REQUEST - Create PRs for all repos

## Phase: R - PULL REQUEST

**Purpose**: Create pull requests for all repositories with changes.

## Instructions

Create a PULL REQUEST for each affected Repo in this GitHub Organization to completely save this change.

### Critical Requirements

- **Be sure to update the overall change log** found in `.github/CHANGELOG.md`, including a tag lock to this PR
- **Ensure a git tag matching the PR number or release name is added post-merge**

### Prerequisites

Ensure all work is complete:

- Implementation done (I: IMPLEMENT)
- Linting passed (L: LINT)
- Tests passing (T: TEST)
- Documentation complete (M: DOCUMENT)

### Process

1. **Commit all changes** in each affected repo:

   ```bash
   git add .
   git commit -m "feat: [descriptive message]"
   git push origin feature/[branch-name]
   ```

2. **Create pull requests** for each affected repo:

   - Navigate to the repository on GitHub
   - Create PR from feature branch to main/master
   - Use consistent PR title: `[Type]: [Description]`
   - Reference issue: `Refs MicroCODE-App-Template/.issue#[number]`
   - Include summary of changes
   - Link to related PRs in other repos (if applicable)

3. **Update CHANGELOG.md**:

   - Add entry for this change
   - Include tag lock reference to PR number
   - Follow changelog format

4. **Document PR creation**:
   - List each PR with link
   - Note PR numbers
   - Document any PR dependencies
   - Note review status
   - Document git tag plan (matching PR number or release name)

### Document the PRs

- **If using Issue Template**: Update the working file in `.issue/.github/ISSUE_TEMPLATE/`
- **If using AIN document**: Navigate to the R: PULL REQUEST section and update it
- **Update Status in Metadata** to "PULL REQUEST"

## Context

- **CHANGELOG**: `.github/CHANGELOG.md` - **MUST UPDATE** with tag lock to PR
- **PR Template**: `.github/PULL_REQUEST_TEMPLATE.md`
- **Issue Tracking**: `MicroCODE-App-Template/.issue` repository
- **Git Workflow**: `.github/docs/DEV/DEVELOPER-GIT.md`
- **Issue Template**: `.issue/.github/ISSUE_TEMPLATE/` - Update the working file
- **AIN Workflow Template**: `.github/docs/AI/AIN [20YY-MM-DD] WORKFLOW_TEMPLATE.md` (alternative workflow)

## Important

- **MUST update CHANGELOG.md** with tag lock to PR
- **Ensure a git tag matching the PR number or release name is added post-merge**
- Create PRs for ALL repos with changes
- Reference the issue number in PR description
- Link related PRs together if they depend on each other

## PR Checklist

- [ ] All changes committed
- [ ] Branches pushed to remote
- [ ] PRs created for all affected repos
- [ ] Issue referenced in PR description
- [ ] PR descriptions complete
- [ ] Related PRs linked (if applicable)
- [ ] PRs ready for review

## PR Title Format

- `feat: Add notification settings`
- `fix: Resolve account scoping issue`
- `docs: Update API documentation`
- `refactor: Reorganize entity management dialogs`
