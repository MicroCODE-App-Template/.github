# Contributing to MicroCODE App

Thank you for your interest in contributing to MicroCODE App! This document provides guidelines and instructions for contributing to the platform.

## Table of Contents

- [Code of Conduct](#code-of-conduct)
- [Getting Started](#getting-started)
- [Development Workflow](#development-workflow)
- [Coding Standards](#coding-standards)
- [Commit Guidelines](#commit-guidelines)
- [Pull Request Process](#pull-request-process)
- [Testing Requirements](#testing-requirements)
- [Documentation](#documentation)
- [Review Process](#review-process)

## Code of Conduct

This project adheres to the [Code of Conduct](./CODE_OF_CONDUCT.md). By participating, you are expected to uphold this code. Please report unacceptable behavior to dev-team@mcode.com.

## Getting Started

### Prerequisites

- Node.js (v20.x or later recommended)
- npm or yarn
- Git
- MongoDB or SQL database (depending on configuration)
- Redis (for caching features)
- Access to MicroCODE-App GitHub organization

### Initial Setup

1. **Clone the Repository:**

    ```bash
    git clone https://github.com/MicroCODE-App/[repository-name].git
    cd [repository-name]
    ```

2. **Install Dependencies:**

    ```bash
    npm install
    ```

3. **Set Up Environment:**
    - Copy `.env.example` to `.env` (if available)
    - Configure environment variables (see [DEVELOPER-ENV.md](../.issue/DEV/DEVELOPER-ENV.md))
    - Set up database connection

4. **Run Setup Scripts:**

    ```bash
    npm run setup
    ```

5. **Verify Installation:**
    ```bash
    npm run dev
    ```

For detailed setup instructions, see [DEVELOPER-SETUP.md](../.issue/DEV/DEVELOPER-SETUP.md).

## Development Workflow

### Branch Strategy

- **main/master:** Production-ready code
- **develop:** Integration branch for features
- **feature/\***: New features
- **bugfix/\***: Bug fixes
- **hotfix/\***: Critical production fixes
- **release/\***: Release preparation

### Creating a Branch

```bash
# For a new feature
git checkout -b feature/your-feature-name

# For a bug fix
git checkout -b bugfix/issue-number-short-description

# For a hotfix
git checkout -b hotfix/issue-number-short-description
```

### Branch Naming Convention

- Use lowercase letters and hyphens
- Prefix with type: `feature/`, `bugfix/`, `hotfix/`
- Include issue number if applicable: `bugfix/1234-fix-auth-bug`
- Keep names descriptive but concise

## Coding Standards

### General Principles

1. **Readability First:** Code should be self-documenting
2. **Consistency:** Follow existing patterns and conventions
3. **Simplicity:** Prefer simple, clear solutions
4. **DRY:** Don't Repeat Yourself
5. **SOLID Principles:** Follow object-oriented design principles

### JavaScript/Node.js Standards

1. **Use ESLint:**

    ```bash
    npm run lint
    npm run lint:fix
    ```

2. **Follow Style Guide:**
    - Use 2 spaces for indentation
    - Use single quotes for strings
    - Use semicolons
    - Maximum line length: 100 characters
    - Use meaningful variable and function names

3. **Code Organization:**
    - Group related functionality
    - Separate concerns (UI, UX, DB, IO layers)
    - Use consistent file naming conventions

4. **Error Handling:**
    - Always handle errors appropriately
    - Use try-catch for async operations
    - Provide meaningful error messages
    - Log errors appropriately

### Example Code Structure

```javascript
// Good: Clear, self-documenting code
async function getUserById(userId) {
    try {
        const user = await UserModel.findById(userId);
        if (!user) {
            throw new Error(`User ${userId} not found`);
        }
        return user;
    } catch (error) {
        logger.error('Error fetching user', { userId, error });
        throw error;
    }
}

// Avoid: Unclear, poorly named code
async function get(u) {
    return await U.findById(u);
}
```

### File Naming

- Use kebab-case for files: `user-controller.js`
- Use descriptive names: `account-settings-view.jsx`
- Match file purpose: `.model.js`, `.controller.js`, `.route.js`

## Commit Guidelines

### Commit Message Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types

- **feat:** New feature
- **fix:** Bug fix
- **docs:** Documentation changes
- **style:** Code style changes (formatting, no logic change)
- **refactor:** Code refactoring
- **perf:** Performance improvements
- **test:** Adding or updating tests
- **chore:** Maintenance tasks
- **ci:** CI/CD changes

### Examples

```
feat(server): add user authentication middleware

Implement JWT-based authentication middleware for API routes.
Includes token validation and user context injection.

Closes #123
```

```
fix(client): resolve 2FA toggle styling issue

Fix Switch component styling to match Apple design guidelines.
Green for ON state, gray for OFF state.

Fixes #456
```

## Pull Request Process

### Before Submitting

1. **Update Documentation:**
    - Update README if needed
    - Add/update code comments
    - Update API documentation
    - Update CHANGELOG.md

2. **Run Tests:**

    ```bash
    npm test
    npm run lint
    ```

3. **Self-Review:**
    - Review your own code
    - Check for typos and errors
    - Verify all tests pass
    - Ensure no console.log statements remain

### PR Checklist

- [ ] Code follows MicroCODE standards
- [ ] Tests added/updated and passing
- [ ] Documentation updated
- [ ] Linter checks pass
- [ ] No merge conflicts
- [ ] Related issues linked
- [ ] Breaking changes documented (if applicable)

### PR Description

Use the [Pull Request Template](./PULL_REQUEST_TEMPLATE.md) and include:

- Clear description of changes
- Related issue numbers
- Testing performed
- Screenshots (if UI changes)
- Breaking changes (if any)

## Testing Requirements

### Test Coverage

- **Unit Tests:** Test individual functions and components
- **Integration Tests:** Test API endpoints and workflows
- **E2E Tests:** Test complete user flows (where applicable)

### Writing Tests

```javascript
// Example test structure
describe('UserController', () => {
    describe('getUserById', () => {
        it('should return user when found', async () => {
            // Arrange
            const userId = '123';
            const expectedUser = { id: userId, name: 'Test User' };

            // Act
            const result = await getUserById(userId);

            // Assert
            expect(result).toEqual(expectedUser);
        });

        it('should throw error when user not found', async () => {
            // Arrange
            const userId = '999';

            // Act & Assert
            await expect(getUserById(userId)).rejects.toThrow();
        });
    });
});
```

### Running Tests

```bash
# Run all tests
npm test

# Run tests in watch mode
npm run test:watch

# Run tests with coverage
npm run test:coverage
```

## Documentation

### Code Comments

- **Function Comments:** Describe what the function does, parameters, and return value
- **Complex Logic:** Explain why, not just what
- **TODO Comments:** Include issue number and context

```javascript
/**
 * Fetches a user by ID from the database
 * @param {string} userId - The unique identifier of the user
 * @returns {Promise<Object>} The user object
 * @throws {Error} If user is not found
 */
async function getUserById(userId) {
    // Implementation
}
```

### Documentation Updates

- Update README for new features or setup changes
- Update API documentation for endpoint changes
- Update developer guides for workflow changes
- Keep CHANGELOG.md updated

## Review Process

### Review Criteria

1. **Code Quality:**
    - Follows coding standards
    - Is readable and maintainable
    - Has appropriate error handling

2. **Functionality:**
    - Works as intended
    - Handles edge cases
    - Doesn't break existing functionality

3. **Testing:**
    - Has adequate test coverage
    - Tests are meaningful and pass

4. **Documentation:**
    - Code is well-commented
    - Documentation is updated
    - Changes are clearly described

5. **Security:**
    - No security vulnerabilities introduced
    - Follows security best practices
    - Handles sensitive data appropriately

### Review Timeline

- **Initial Review:** Within 2 business days
- **Follow-up Reviews:** Within 1 business day
- **Critical Issues:** Immediate attention

### Addressing Review Feedback

1. **Respond to Comments:**
    - Acknowledge all feedback
    - Ask questions if unclear
    - Discuss alternatives if needed

2. **Make Changes:**
    - Address all requested changes
    - Update code based on feedback
    - Re-request review when ready

3. **Be Open to Feedback:**
    - Reviews are collaborative
    - Feedback improves code quality
    - Learn from each review

## Entity Configuration

When working with the entity system:

1. **Update `__config.js`** for entity changes
2. **Regenerate Code** using CLI tools
3. **Review Generated Code** for correctness
4. **Customize as Needed** for specific requirements
5. **Test Thoroughly** all CRUD operations

## Additional Resources

- [Developer Setup Guide](../.issue/DEV/DEVELOPER-SETUP.md)
- [Git Workflow Guide](../.issue/DEV/DEVELOPER-GIT.md)
- [Environment Configuration](../.issue/DEV/DEVELOPER-ENV.md)
- [Code of Conduct](./CODE_OF_CONDUCT.md)
- [Security Policy](./SECURITY.md)

## Questions?

If you have questions about contributing:

- **Email:** dev-team@mcode.com
- **Slack:** #microcode-app-template
- **Documentation:** Check the [docs](../DOCs/) directory

---

**Thank you for contributing to MicroCODE App!**

**Last Updated:** 2025-01-XX
