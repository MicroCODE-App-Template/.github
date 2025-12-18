# AIN - BUGFIX - NOTIFICATION_SETTINGS

## Metadata

- **Type**: BUGFIX
- **Issue #**: [if applicable]
- **Created**: 2025-01-27
- **Status**: CONCEPT

---

## C: CONCEPT - Discuss ideas without generating code

### Current State Analysis

#### Notification System (Legacy)

- **Model**: `server/model/mongo/notification.mongo.js`
- **Controller**: `server/controller/notification.controller.js`
- **Routes**: `server/api/notification.route.js`
- **Schema**: Separate MongoDB collection with documents per user/account/notification type
- **Structure**: Each notification is a document with:
  - `id`, `name`, `active` (boolean), `user_id`, `account_id`
  - Permission-based creation (only creates notifications user has permission for)
- **Methods**:
  - `create()` - Creates default notifications for new user based on permission level
  - `get()` - Returns notification preferences (array or boolean for single name)
  - `update()` - Updates notification preferences (only updates changed values)
- **Usage Locations**:
  - `user.controller.js:142` - Creates notifications when user is created
  - `user.controller.js:197` - Checks `invite_accepted` notification before sending email
  - `account.controller.js:111` - Creates notifications when account is created
  - `account.controller.js:551,790,899` - Checks notifications before sending emails (`plan_updated`, `card_updated`)
  - `auth.controller.js:170` - Checks `new_signin` notification before sending email
  - `social.controller.js:224` - Creates notifications for social signups
- **Frontend**:
  - `client/src/views/account/notifications.jsx` - Reads from `/api/notification` but saves to `user.settings.messages.*.*`
  - `app/views/account/notifications.js` - Same pattern

#### User Settings System (New)

- **Model**: `server/model/mongo/user.mongo.js` (settings methods)
- **Controller**: `server/controller/user.controller.js` (settings methods)
- **Routes**: `server/api/user.route.js` (settings endpoints)
- **Schema**: Nested object in user document: `user.settings.messages.{channel}.{notificationName}`
- **Structure**: Hierarchical key path format: `subsystem.feature.setting`
  - Example: `messages.email.new_signin` = boolean
  - Current channels: `email`, `text` (from `config/default.json`)
- **Methods**:
  - `settings.get()` - Get setting by hierarchical key path
  - `settings.set()` - Set single setting by hierarchical key path
  - `settings.setAll()` - Replace entire settings object
- **Default Values**: Defined in `server/config/default.json` under `settings.messages.*.*`
- **Frontend**: Already using `user.settings.messages.*.*` for storage, but still calls `/api/notification` to discover available notifications

### Gap Analysis

#### Functionality Comparison

| Feature                      | Notification System                                   | User Settings System                 | Gap?    |
| ---------------------------- | ----------------------------------------------------- | ------------------------------------ | ------- |
| Permission-based filtering   | ✅ Creates only notifications user has permission for | ❌ No permission checking            | **YES** |
| Account-scoped               | ✅ Each notification tied to `user_id` + `account_id` | ❌ Settings are user-scoped only     | **YES** |
| Default values               | ✅ From `config.notifications`                        | ✅ From `config.settings.messages`   | No      |
| Lazy creation                | ✅ Creates on first update                            | ✅ Initialized on user creation      | No      |
| List available notifications | ✅ Returns array of available notifications           | ❌ No equivalent endpoint            | **YES** |
| Single notification check    | ✅ Returns boolean for single name                    | ✅ Can get single setting            | No      |
| Bulk update                  | ✅ Updates multiple notifications                     | ✅ Can update entire settings object | No      |

#### Key Architectural Differences

1. **Scoping Model**:

   - **Notifications**: Account-scoped (`user_id` + `account_id` pair)
   - **Settings**: User-scoped (no account context)
   - **Impact**: A user with multiple accounts would have different notification preferences per account in the old system, but unified preferences in the new system

2. **Permission Filtering**:

   - **Notifications**: Only creates notifications the user has permission for (from `config.permissions`)
   - **Settings**: No permission checking - all settings available to all users
   - **Impact**: Need to ensure permission-based filtering when reading/creating settings

3. **Discovery Endpoint**:
   - **Notifications**: `/api/notification` returns list of available notifications for current user
   - **Settings**: No equivalent endpoint to discover available notification types
   - **Impact**: Frontend currently relies on `/api/notification` to know which notifications exist

### Questions & Decisions Needed

1. **Account Scoping**: Should notification settings be account-scoped or user-scoped?

   - **Option A**: Keep account-scoped (settings per account)
     - Requires: `user.settings.{account_id}.messages.*.*` structure
     - Pros: Matches current notification behavior
     - Cons: More complex data structure
   - **Option B**: Make user-scoped (unified across accounts)
     - Requires: `user.settings.messages.*.*` (current structure)
     - Pros: Simpler, user has one preference
     - Cons: Different from current notification behavior

2. **Permission Filtering**: How should permission-based filtering work?

   - **Option A**: Filter at read time (only return notifications user has permission for)
   - **Option B**: Filter at write time (prevent setting notifications user doesn't have permission for)
   - **Option C**: Both

3. **Discovery Endpoint**: How should frontend discover available notifications?

   - **Option A**: Create new endpoint `/api/user/settings/notifications` that returns available notifications based on permission
   - **Option B**: Use `config.notifications` structure directly (no endpoint needed)
   - **Option C**: Return available notifications as part of existing `/api/user/settings` response

4. **Migration Strategy**: How to migrate existing notification documents?

   - Need to migrate all existing `notification` collection documents to `user.settings.messages.*.*`
   - Need to handle account-scoped vs user-scoped migration

5. **Backward Compatibility**: Should we maintain the `/api/notification` endpoint during transition?
   - Or remove immediately after migration?

### Recommended Approach (Pending Approval)

Based on the analysis, I recommend:

1. **Account-scoped settings**: Use `user.settings.{account_id}.messages.*.*` structure to maintain current behavior
2. **Permission filtering**: Filter at read time - only return notifications user has permission for
3. **Discovery endpoint**: Create `/api/user/settings/notifications` that returns available notifications based on user's permission level
4. **Migration**: Write migration script to move all notification documents to user.settings structure
5. **Clean removal**: Remove notification model, controller, routes, and all references after migration complete

---

## D: DESIGN - Design detailed solution

<!-- Document the detailed design approach here -->

---

## P: PLAN - Create implementation plan

<!-- Create step-by-step implementation plan here -->

---

## V: REVIEW - Review and validate the implementation plan

<!-- Review notes and validation results here -->

---

## B: BRANCH - Create Git branches for required repos

<!-- Document branch creation and naming here -->

---

## I: IMPLEMENT - Execute the plan

<!-- Track implementation progress here -->

---

## L: LINT - Check and fix linting issues

<!-- Document linting checks and fixes here -->

---

## T: TEST - Run tests

<!-- Document test execution and results here -->

---

## M: DOCUMENT - Document the solution

<!-- Document the final solution here -->

---

## R: PULL REQUEST - Create PRs for all repos

<!-- Document pull request creation and links here -->

---

## Notes

<!-- Additional notes, decisions, or observations -->
