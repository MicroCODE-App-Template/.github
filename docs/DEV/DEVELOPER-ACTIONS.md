# GitHub Actions Workflows

This document describes the GitHub Actions workflows used in the MicroCODE App Template solution.

## Overview

GitHub Actions provide automated CI/CD, validation, and workflow automation across all repositories in the MicroCODE App Template organization.

## Available Workflows

### 1. CI Workflow (Reusable)

**Location:** `.github/workflows/ci.yml`

**Purpose:** Reusable CI workflow that can be called from repository-specific workflows.

**Inputs:**

- `node-version` - Node.js version to use (default: "22")
- `working-directory` - Working directory for the job (default: ".")
- `install-command` - Command to install dependencies (default: "npm ci")
- `lint-command` - Command to run linter (default: "npm run lint")
- `test-command` - Command to run tests (default: "npm test")
- `build-command` - Command to build the project (default: "npm run build")

**Jobs:**

- `lint` - Runs ESLint/linter checks
- `test` - Runs test suite
- `build` - Builds the project

**Usage:**

```yaml
jobs:
  ci:
    uses: MicroCODE-App-Template/.github/workflows/ci.yml@main
    with:
      node-version: "22"
      working-directory: "."
```

### 2. Branch Name Validation Workflow

**Location:** `.github/workflows/validate-branch-name.yml` (reusable)

**Purpose:** Validates that branch names match the issue number format from the centralized `.issue` repository.

**Validation Rules:**

- Branch name must match pattern: `{type}/{issue#}--{description}`
- Issue number must be zero-padded to 4 digits (e.g., `0123`)
- Issue number must reference a valid issue in `.issue` repository
- Valid types: `feature`, `bugfix`, `hotfix`, `security`, `release`

**Examples:**

- ✅ Valid: `feature/0123--add-user-auth`
- ✅ Valid: `bugfix/0045--fix-login-error`
- ✅ Valid: `hotfix/0067--security-patch`
- ❌ Invalid: `feature/123--add-user-auth` (not zero-padded)
- ❌ Invalid: `feature/0123-add-user-auth` (single dash, not double)
- ❌ Invalid: `feature/9999--invalid-issue` (issue doesn't exist)

**Behavior:**

- **FAILS PR** if branch name doesn't match format or issue doesn't exist
- Provides clear error message with expected format
- Checks issue existence in `.issue` repository via GitHub API

**Usage:**

Repository-specific workflows call this reusable workflow:

```yaml
name: Branch Validation

on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  validate-branch:
    uses: MicroCODE-App-Template/.github/workflows/validate-branch-name.yml@main
    with:
      branch-name: ${{ github.head_ref }}
      issue-repo: ".issue"
```

### 3. Dependency Update Workflow

**Location:** `.github/workflows/dependency-update.yml`

**Purpose:** Automated dependency updates using Dependabot or similar tools.

**Note:** Dependabot issues are created in the `.issue` repository with format: `[repo] Dependency update: package-name`

### 4. Release Workflow

**Location:** `.github/workflows/release.yml`

**Purpose:** Automated release process and version tagging.

### 5. Security Scan Workflow

**Location:** `.github/workflows/security-scan.yml`

**Purpose:** Security vulnerability scanning and reporting.

## Branch Name Validation

### Purpose

Enforces consistency across all repositories by requiring branch names to:

1. Reference valid issues in the centralized `.issue` repository
2. Use zero-padded format (4 digits) for compatibility with existing branches
3. Follow the standard naming convention: `{type}/{issue#}--{description}`

### Implementation

The branch validation workflow:

1. Extracts branch name from PR
2. Parses branch name to extract issue number
3. Validates format (zero-padded, double-dash separator)
4. Verifies issue exists in `.issue` repository
5. Fails PR if validation fails

### Error Messages

If validation fails, the workflow provides clear error messages:

```
❌ Branch name validation failed

Branch: feature/123--add-user-auth
Error: Issue number must be zero-padded to 4 digits

Expected format: feature/0123--add-user-auth
```

### Bypassing Validation

**Not Recommended:** Branch validation should not be bypassed except in exceptional circumstances.

If absolutely necessary, use a branch name that doesn't match the pattern (e.g., `main`, `develop`, `release/bM.F.0`), but note that these branches are protected and have different validation rules.

## Workflow Best Practices

### Consistency

- All workflows follow MicroCODE coding standards
- Consistent naming conventions across all repos
- Reusable workflows reduce duplication

### Error Handling

- Workflows fail fast on validation errors
- Clear error messages guide developers
- Non-blocking warnings for non-critical issues

### Performance

- Workflows run in parallel when possible
- Caching dependencies to speed up runs
- Minimal workflow execution time

## Creating New Workflows

When creating new workflows:

1. Follow the reusable workflow pattern when possible
2. Use consistent naming: `{purpose}.yml`
3. Document inputs, outputs, and behavior
4. Include error handling and clear messages
5. Test workflows in a test repository first

## Troubleshooting

### Workflow Not Running

- Check workflow file is in `.github/workflows/` directory
- Verify workflow syntax is valid YAML
- Check repository has Actions enabled
- Verify branch protection rules allow workflow execution

### Validation Failing

- Verify branch name matches format: `{type}/{issue#}--{description}`
- Check issue number is zero-padded (4 digits)
- Confirm issue exists in `.issue` repository
- Review error message for specific failure reason

### Performance Issues

- Check workflow dependencies and caching
- Review workflow execution time in Actions tab
- Optimize workflow steps to run in parallel
- Consider splitting large workflows into smaller ones

## References

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [MicroCODE Git Workflow](./DEVELOPER-GIT.md)
- [MicroCODE Software Support Process](./DEVELOPER-SSP.md)

---

**Note:** This is an internal MicroCODE, Inc. document. All workflows are confidential.
