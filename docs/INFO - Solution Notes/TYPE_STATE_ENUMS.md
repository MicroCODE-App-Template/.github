# Entity Type & State Enums Reference

This document defines the `type` and `state` enum values for all database entities.

## Standard Entities

### Account
- **Type**: `['undefined', 'master', 'owner', 'personal']`
  - Default: `'undefined'`
  - Usage: Categorizes account ownership level
- **State**: `['undefined', 'active', 'inactive', 'suspended', 'archived']`
  - Default: `'undefined'`
  - Usage: Account operational status

### User
- **Type**: `['undefined', 'admin', 'developer', 'owner', 'user']`
  - Default: `'undefined'`
  - Usage: User permission/role level
- **State**: `['undefined', 'active', 'inactive', 'suspended', 'archived']`
  - Default: `'undefined'`
  - Usage: User account status

### Email
- **Type**: `['undefined', 'transactional', 'billing', 'authentication', 'system', 'custom']`
  - Default: `'transactional'`
  - Usage: Email template category
- **State**: `['undefined', 'draft', 'active', 'inactive', 'archived']`
  - Default: `'active'`
  - Usage: Template availability status

### Event
- **Type**: `['undefined', 'user_action', 'system', 'error', 'custom']`
  - Default: `'user_action'`
  - Usage: Event category
- **State**: `['undefined', 'logged', 'processed', 'archived']`
  - Default: `'logged'`
  - Usage: Event processing status

### Feedback
- **Type**: `['undefined', 'feature_request', 'bug_report', 'general', 'complaint']`
  - Default: `'general'`
  - Usage: Feedback category
- **State**: `['undefined', 'pending', 'reviewed', 'resolved', 'archived']`
  - Default: `'pending'`
  - Usage: Feedback processing status

### Invite
- **Type**: `['undefined', 'user', 'admin', 'developer']`
  - Default: `'user'`
  - Usage: Invited user permission level (matches `permission` field)
- **State**: `['undefined', 'pending', 'accepted', 'expired', 'cancelled']`
  - Default: `'pending'`
  - Usage: Invitation status

### Key (API Key)
- **Type**: `['undefined', 'api', 'test', 'production']`
  - Default: `'api'`
  - Usage: API key environment/purpose
- **State**: `['undefined', 'active', 'inactive', 'revoked', 'expired']`
  - Default: `'active'`
  - Usage: Key validity status

### Log
- **Type**: `['undefined', 'info', 'warning', 'error', 'debug']`
  - Default: `'info'`
  - Usage: Log severity level
- **State**: `['undefined', 'logged', 'archived']`
  - Default: `'logged'`
  - Usage: Log retention status

### Login
- **Type**: `['undefined', 'password', 'social', 'magic', 'api']`
  - Default: `'password'`
  - Usage: Authentication method used
- **State**: `['undefined', 'success', 'failed', 'suspicious', 'blocked']`
  - Default: `'success'`
  - Usage: Login attempt result

### Notification
- **Type**: `['undefined', 'new_signin', 'plan_updated', 'card_updated', 'invite_accepted', 'custom']`
  - Default: `'custom'`
  - Usage: Notification category (matches `name` field patterns)
- **State**: `['undefined', 'enabled', 'disabled']`
  - Default: `'enabled'`
  - Usage: User preference status

### Token
- **Type**: `['undefined', 'app', 'google', 'facebook', 'twitter']`
  - Default: `'app'`
  - Usage: Authentication provider (matches `provider` field)
- **State**: `['undefined', 'valid', 'expired', 'revoked']`
  - Default: `'valid'`
  - Usage: Token validity status

### Usage
- **Type**: `['undefined', 'metered', 'flat']`
  - Default: `'metered'`
  - Usage: Billing measurement type
- **State**: `['undefined', 'open', 'closed', 'reported']`
  - Default: `'open'`
  - Usage: Usage period reporting status

## PLC/Manufacturing Entities

### Mfg_Company (Manufacturing Company)
- **Type**: `['undefined', 'manufacturer']`
  - Default: `'manufacturer'`
  - Usage: Company category
- **State**: `['undefined', 'active', 'inactive', 'archived']`
  - Default: `'active'`
  - Usage: Company status in system

### PLC_Family
- **Type**: `['undefined', 'controller_family']`
  - Default: `'controller_family'`
  - Usage: Entity category
- **State**: `['undefined', 'active', 'inactive', 'archived']`
  - Default: `'active'`
  - Usage: Family availability status

### PLC_Mfg (PLC Manufacturer)
- **Type**: `['undefined', 'controller_model']`
  - Default: `'controller_model'`
  - Usage: Entity category
- **State**: `['undefined', 'active', 'inactive', 'archived']`
  - Default: `'active'`
  - Usage: Model availability status

### PLC_Program
- **Type**: `['undefined', 'user_upload', 'sample', 'template']`
  - Default: `'user_upload'`
  - Usage: Program source/purpose
- **State**: `['undefined', 'uploaded', 'processing', 'complete', 'error', 'archived']`
  - Default: `'uploaded'`
  - Usage: Processing workflow status

## Implementation Notes

### MongoDB
- Schema defaults automatically apply on document creation
- Enum validation ensures only valid values are stored
- Use `default` property in schema for automatic assignment

### SQL
- Values must be explicitly set in `create` functions
- Migration adds columns with `DEFAULT 'undefined'` for existing records
- Update create functions to use meaningful defaults

### Best Practices
1. Always use meaningful defaults appropriate to entity purpose
2. Update CREATE functions when adding new record types
3. Use `'undefined'` only when the actual type/state is truly unknown at creation time
4. Prefer specific enum values (e.g., `'owner'` instead of `'undefined'`) when context is clear

## Updating Models

After modifying schemas in `model/mongo/*.mongo.js` or `model/sql/*.sql.js`:

**Server:**
```bash
npm run mongo:update  # For MongoDB models
npm run sql:update    # For SQL models
```

**Admin:**
```bash
cd admin
npm run mongo:update  # For MongoDB models
npm run sql:update    # For SQL models
```

## Complete MongoDB Reset

To completely reset your MongoDB development environment (clear DB, update models, create master, seed data, start services):

**Server:**
```bash
npm run mongo:reset
```

This executes the full sequence:
1. `db:init` - Clear database
2. `mongo:update` (server) - Update server models
3. `mongo:update` (admin) - Update admin models
4. `create:master` (admin) - Create master account
5. `seed:all` - Seed all data
6. `all` - Start development environment
