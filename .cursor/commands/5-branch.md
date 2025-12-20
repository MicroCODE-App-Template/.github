# BRANCH - Create Git branches for required repos

## Phase: 5: BRANCH

**Purpose**: Create Git branches for all repositories that need changes.

## Instructions

BRANCH the required Repos. We are going to work on a change that may affect multiple Repos found in this directory.

### Branch Naming Rules

Follow our Git Workflow implementation rules in `.github/docs/DEV/DEVELOPER-GIT.md` - **Section 5: Branch Naming Conventions (GitHub Organizations with Multiple Repos)**.

**If you are not positive about the names for the branches, stop and ask me.**

### Process

1. **List the repos you intend to branch** in one place, with your intended names, **before actually creating branches** so I can confirm the scope.

2. **Wait for confirmation** before creating any branches.

3. **Create branches** following the naming convention:

   - Format: `{type}/{ISSUE_NUMBER}--{short-name}` (zero-padded 4-digit issue number)
   - Types: `feature/`, `bugfix/`, `hotfix/`, `release/`
   - Example: `feature/0003--ladders-integration`
   - See DEVELOPER-GIT.md for complete naming rules

4. **Create branches in each affected repo**:

   - `.github` - If documentation changes needed
   - `server` - If backend changes needed
   - `client` - If web frontend changes needed
   - `app` - If mobile app changes needed
   - `admin` - If admin console changes needed
   - `portal` - If marketing site changes needed

5. **Document branch creation**:
   - List each repo and branch name
   - Note which branch is primary (if applicable)
   - Document any branch dependencies

### Document the Branches

- **If using Issue Template**: Update the working file in `.issue/.github/ISSUE_TEMPLATE/`
- **If using AIN document**: Navigate to the B: BRANCH section and update it

## Context

- **Git Workflow**: `.github/docs/DEV/DEVELOPER-GIT.md` - See Section 5 for branch naming conventions
- **Issue Tracking**: All issues in `MicroCODE-App-Template/.issue` repository
- **Issue Template**: `.issue/.github/ISSUE_TEMPLATE/` - Update the working file
- **AIN Workflow Template**: `.github/docs/AI/AIN [20YY-MM-DD] WORKFLOW_TEMPLATE.md` (alternative workflow)
- **Branch Naming**: Zero-padded 4-digit issue numbers

## Important

- All workflow phases go in **ONE document** - do NOT create separate files
- Use consistent branch names across all repos
- Reference issue number if applicable: `Refs MicroCODE-App-Template/.issue#[number]`
- Create branches before starting implementation

## Example

```bash
# In .github repo
git checkout -b feature/0003--ladders-integration

# In server repo
git checkout -b feature/0003--ladders-integration

# In client repo
git checkout -b feature/0003--ladders-integration
```
