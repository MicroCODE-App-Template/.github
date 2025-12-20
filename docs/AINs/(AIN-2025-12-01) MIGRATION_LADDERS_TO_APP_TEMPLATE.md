# AIN - TASK - MIGRATION_LADDERS_TO_APP_TEMPLATE

## Metadata

- **Type**: TASK
- **Issue #**: [0003]
- **Created**: [2025-12-01]
- **Status**: IN PROGRESS
- **Last Updated**: [2025-12-17]

---

## 1. CONCEPT/CHANGE/CORRECTION - Discuss ideas without generating code

### Overview

This migration merges the best features from **ladders** into **app-template** (based on gravity-current) while preserving all gravity-current improvements and fixing bugs.

### Goals

1. Integrate ladders features (mcode logging, ID system, timestamps, settings, etc.)
2. Preserve all gravity-current features (notifications, token expiration, email localization, etc.)
3. Fix bugs found in ladders code
4. Standardize code patterns across all models
5. Improve documentation and code organization

### Scope

- **14 Model Files**: account, email, event, feedback, invite, key, log, login, notification, pushtoken, token, usage, user, mongo.js
- **11 Phases**: From preparation through cleanup
- **Estimated Time**: 42 hours (5-6 working days)
- **Minimal Viable**: 28 hours (3-4 days)

---

## 2. DESIGN - Design detailed solution

### Architecture Strategy

**Safe, Time-Efficient Integration Strategy**:

1. **Incremental Migration**: Phase-by-phase approach with testing after each phase
2. **Backward Compatibility**: Keep existing functionality while adding new features
3. **Pattern Consistency**: Use established patterns from ladders for new code
4. **Risk Mitigation**: Test thoroughly, commit frequently, maintain rollback capability

### Key Design Decisions

1. **ID System**: Use `_id` as primary key (MongoDB standard), keep `id` for compatibility initially
2. **Timestamp Naming**: Use `_at` suffix consistently (created_at, updated_at, occurred_at, etc.)
3. **Parameter Naming**: Use explicit `{entity}_id` naming (user_id, account_id) instead of ambiguous names
4. **Logging**: Replace console.log/error with mcode logging system
5. **Settings System**: Implement hierarchical settings in user model
6. **Function Organization**: CRUD functions first, special functions last

---

## 3. PLAN - Create implementation plan

# Migration Plan: Ladders Features → App-Template

## Executive Summary

This plan merges the best features from **ladders** into **app-template** (based on gravity-current) while preserving all gravity-current improvements and fixing bugs.

**Total Estimated Time**: ~42 hours (5-6 working days)
**Minimal Viable Migration**: ~28 hours (3-4 days)

---

## PHASE 0: PREPARATION (Day 1 - 2 hours)

### 0.1 Backup & Version Control

- ✅ Create a new branch: `feature/0003--**-ladders-integration`
- ✅ Tag current state: `git tag pre-0003--ladders-integration`
- ✅ Document current app-template features - see Gravity Docs
- ✅ Create backup of entire app-template directory

### 0.2 Setup Testing Infrastructure

- ✅ Ensure all existing tests pass: `npm test`
- ✅ Create test checklist for each model
- ⏭️ Set up test database for migration testing - NO MIGRATION REQUIRED

### 0.3 Create Feature Tracking

- ⏭️ Create `MIGRATION_STATUS.md` to track progress - INTEGRATED IN THIS DOC
- ✅ List all files that need changes
- ✅ Create checklist for each feature

**Quick Commands**:

```bash
# Start migration
git checkout -b feature/ladders-integration
git tag pre-ladders-integration

# After each phase
git add .
git commit -m "Phase X: [Description]"
npm test

# If something breaks
git reset --hard HEAD~1  # Undo last commit
# Or
git checkout pre-ladders-integration  # Start over
```

---

## PHASE 1: LOW-RISK FOUNDATION (Day 1-2 - 4 hours)

### 1.1 Add mcode Logging (Low Risk - No Schema Changes)

**Priority**: High | **Risk**: Low | **Time**: 1 hour

**Files to Update**:

- `server/model/mongo/mongo.js` - Add mcode logging
- All model files - Replace `console.log` with `mcode.*`

**Steps**:

1. Update `mongo.js`:

   ```javascript
   const mcode = require("mcode-log");
   const MODULE_NAME = "mongo.js";

   exports.connect = async (settings) => {
     try {
       const url = `mongodb+srv://...`;
       await mongoose.connect(url);
       mcode.done("Connected to MongoDB", MODULE_NAME);
     } catch (err) {
       mcode.exp("Connection to MongoDB failed.", MODULE_NAME, err);
     }
   };
   ```

2. For each model file, add at top:

   ```javascript
   const mcode = require("mcode-log");
   const MODULE_NAME = "account.js"; // or appropriate name
   ```

3. Replace `console.log` → `mcode.done()`
4. Replace `console.error` → `mcode.exp()`
5. Replace `console.warn` → `mcode.warn()`

**Testing**: Run server, verify logs appear correctly

### 1.2 Add MODULE_NAME Constants

**Priority**: Medium | **Risk**: Low | **Time**: 30 minutes

**Action**: Add `const MODULE_NAME = 'filename.js';` to every model file

**Files**: All 14 model files in `server/model/mongo/`

### 1.3 Add MongoDB Disconnect Function

**Priority**: Low | **Risk**: Low | **Time**: 15 minutes

**File**: `server/model/mongo/mongo.js`

**Add**:

```javascript
exports.disconnect = async () => {
  try {
    await mongoose.disconnect();
    mcode.success("Disconnected from MongoDB", MODULE_NAME);
  } catch (err) {
    mcode.exp("Disconnection from MongoDB failed.", MODULE_NAME, err);
  }
};
```

**Checklist**:

- ✅ Add mcode logging to mongo.mongo.js
- ⏭️ Add MODULE_NAME to all 14+ model files -mcode.log() no longer needs
- ✅ Replace console.log → mcode.done() in all files
- ✅ Replace console.error → mcode.exp() in all files
- ✅ Add disconnect() function to mongo.mongo.js
- ✅ Test: Server starts, logs appear (needs verification)

---

## PHASE 2: DOCUMENTATION & CODE ORGANIZATION (Day 2 - 3 hours)

### 2.1 Add JSDoc Headers to All Functions

**Priority**: Medium | **Risk**: None | **Time**: 2 hours

**Pattern**:

```javascript
/**
 * @function create
 * @memberof account.model
 * @description Create a new account and return the account id
 * @param {Object} params - Parameters object
 * @param {String} [params.plan] - Account plan type
 * @returns {Promise<Object>} The created account object
 */
exports.create = async function ({ plan } = {}) {
  // ...
};
```

**Files**: All model files

### 2.2 Reorganize Functions (CRUD First)

**Priority**: Medium | **Risk**: Low | **Time**: 1 hour

**Pattern for each model**:

1. Schema definition
2. Model creation
3. `exports.schema = ...`
4. `exports.model = ...` (if needed)
5. **[C]RUD** - create()
6. **C[R]UD** - get()
7. **CR[U]D** - update()
8. **CRU[D]** - delete()
9. **SPECIAL FUNCTIONS** - all other functions

**Files**: All model files

### 2.3 Add Inline Comments for Complex Logic

**Priority**: Low | **Risk**: None | **Time**: 30 minutes

**Action**: Add comments where:

- Bugs were fixed
- Complex logic exists
- Non-obvious decisions were made

**Checklist**:

- ✅ Add JSDoc to all functions (14+ files) - using `@func` pattern
- ✅ Reorganize: CRUD first, special functions last
- ✅ Add inline comments for complex logic
- ✅ Test: Code still works (needs verification)

---

## PHASE 3: ID SYSTEM MIGRATION (Day 3-4 - 6 hours) ⚠️ HIGH RISK

### 3.1 Strategy: Hybrid Approach

**Decision**: Keep both `id` (for API) AND `_id` (for MongoDB) initially, then migrate

**Current State**: gravity-current uses `id` field
**Target State**: Use `_id` as primary key, keep `id` as alias for compatibility

### 3.2 Step-by-Step Migration

#### Step 3.2.1: Add `_id` to Schemas (Non-Breaking)

**Priority**: High | **Risk**: Medium | **Time**: 2 hours

**Files**: All model files

**Pattern**:

```javascript
const accountSchema = new Schema({
  _id: { type: String, default: () => utility.unique_id("acct") },
  id: { type: String, required: true, unique: true }, // Keep for compatibility
  // ... rest of schema
});
```

**Action**:

1. Add `_id` field with appropriate prefix to each schema
2. Keep existing `id` field (for now)
3. Add virtual to sync: `id = _id` on read

**Testing**: Verify both `id` and `_id` work

#### Step 3.2.2: Update Internal References to Use `_id`

**Priority**: High | **Risk**: Medium | **Time**: 2 hours

**Files**: All model files

**Changes**:

- `Account.findOne({ id: id })` → `Account.findOne({ _id: _id })`
- `'account.id'` → `'account._id'` (in user model)
- Return `_id` instead of `id` in responses

**Testing**: Test all CRUD operations

#### Step 3.2.3: Update Foreign Key References

**Priority**: High | **Risk**: Medium | **Time**: 2 hours

**Pattern**: Use `{entity}_id` naming consistently

**Changes**:

- `user` parameter → `user_id` parameter
- `account` parameter → `account_id` parameter
- `'account.id'` → `'account._id'` in queries

**Files**: All model files

**Example**:

```javascript
// Before
exports.create = async function ({ data, user, account }) {
  data.user_id = user;
  data.account_id = account;
};

// After
exports.create = async function ({ data, user_id, account_id }) {
  data.user_id = user_id;
  data.account_id = account_id;
};
```

**Checklist**:

- ❌ Add `_id` field to all schemas with prefixes - LEAVING THIS AS MONGO INTERNAL
- ❌ Update all queries: `{ id: id }` → `{ _id: _id }` - SAME
- ❌ Update foreign keys: `'account.id'` → `'account._id'` - CANCELED
- ✅ Change parameters: `user` → `user_id`, `account` → `account_id` (partially done - some use explicit naming)
- ✅ Test: All CRUD operations work

**Prefixes for \_id**:

See teh 'sinlge source of truth': `server/seed/_entity.js`

| Model    | Prefix   |
| -------- | -------- |
| account  | `'acct'` |
| email    | `'mail'` |
| event    | `'evnt'` |
| feedback | `'fbck'` |
| invite   | `'invt'` |
| key      | `'apik'` |
| log      | `'logn'` |
| login    | `'logi'` |
| token    | `'stok'` |
| usage    | `'used'` |
| user     | `'user'` |

---

## PHASE 4: TIMESTAMP STANDARDIZATION (Day 4-5 - 4 hours) ⚠️ HIGH RISK

### 4.1 Rename Timestamp Fields

**Priority**: High | **Risk**: Medium | **Time**: 3 hours

**Mapping**:

- `date_created` → `created_at`
- `time` → `occurred_at` (events), `logged_at` (logs), `started_at` (logins)
- `date_sent` → `sent_at`
- `last_active` → `active_at`

**Files to Update**:

1. **account.js**: `date_created` → `created_at`
2. **event.js**: `time` → `occurred_at`
3. **feedback.js**: `date_created` → `created_at`
4. **invite.js**: `date_sent` → `sent_at`
5. **key.js**: `date_created` → `created_at`
6. **log.js**: `time` → `logged_at`
7. **login.js**: `time` → `started_at`
8. **token.js**: `issued_at`, `expires_at` (already correct)
9. **usage.js**: Keep `period_start`, `period_end` (already correct)
10. **user.js**: `date_created` → `created_at`, `last_active` → `active_at`

**Steps**:

1. Update schema definitions
2. Update function code that sets these fields
3. Update queries that use these fields
4. Update aggregation pipelines

**Testing**: Test all timestamp operations

### 4.2 Update Date Setting Code

**Priority**: High | **Risk**: Low | **Time**: 1 hour

**Pattern**:

```javascript
// Before
date_created: new Date();

// After
created_at: new Date();
```

**Checklist**:

- ✅ account.mongo.js: Uses `created_at` (no `date_created` found)
- ✅ event.mongo.js: Uses `created_at` (no `time` field found - may need `occurred_at` if event-specific timestamp needed)
- ✅ feedback.mongo.js: Uses `created_at` (no `date_created` found)
- ✅ invite.mongo.js: Uses `created_at` (may need `sent_at` if send timestamp needed)
- ✅ key.mongo.js: Uses `created_at` (no `date_created` found)
- ✅ log.mongo.js: Uses `created_at` (may need `logged_at` if log-specific timestamp needed)
- ✅ login.mongo.js: Uses `created_at` (may need `started_at` if login-specific timestamp needed)
- ✅ user.mongo.js: Uses `created_at` and `active_at` (no `date_created` or `last_active` found)
- ✅ Test: All date operations work (needs verification)

---

## PHASE 5: ADD TYPE & STATE FIELDS (Day 5-6 - 4 hours)

### 5.1 Add `type` Field to All Entities

**Priority**: High | **Risk**: Low | **Time**: 2 hours

**Files**: All model files

**Pattern**:

```javascript
const accountSchema = new Schema({
  // ... existing fields
  type: { type: String }, // e.g., 'personal', 'business', 'enterprise'
  // ...
});
```

**Default Values** (where appropriate):

- **account.js**: `type: 'personal'` in create()
- **Other entities**: Add `type` field, no default (set as needed)

### 5.2 Add `state` Field to All Entities

**Priority**: High | **Risk**: Low | **Time**: 2 hours

**Files**: All model files

**Pattern**:

```javascript
const accountSchema = new Schema({
  // ... existing fields
  state: { type: String }, // e.g., 'active', 'suspended', 'archived'
  // ...
});
```

**Note**: Some entities already have `active: Boolean`. Consider:

- Keep `active` for boolean checks
- Add `state` for more granular states
- Or migrate `active` → `state: 'active' | 'inactive'`

**Recommendation**: Keep both for now, use `state` for new features

**Checklist**:

- ✅ Add `type` field to all 14 schemas
- ✅ Add `state` field to all 14 schemas
- ✅ Set default `type: 'personal'` in account.create()
- ✅ Test: Type/state fields work

---

## PHASE 6: USER SETTINGS SYSTEM (Day 6-7 - 6 hours) ⚠️ COMPLEX

### 6.1 Add Settings Schema to User Model

**Priority**: High | **Risk**: Medium | **Time**: 1 hour

**File**: `server/model/mongo/user.js`

**Add to Schema**:

```javascript
const userSchema = new Schema({
  // ... existing fields
  settings: { type: Object, required: true },
  // ...
});
```

**Update create()**:

```javascript
exports.create = async function ({ user, account_id }) {
  const data = {
    // ... existing fields
    settings: config.get("settings") || {}, // Initialize from config
    // ...
  };
  // ...
};
```

### 6.2 Add Settings Functions

**Priority**: High | **Risk**: Low | **Time**: 2 hours

**File**: `server/model/mongo/user.js`

**Add at end (Special Functions section)**:

```javascript
/**
 * @function getSetting
 * @memberof user.model
 * @description Retrieves a specific setting based on a hierarchical key
 * @param {Object} params
 * @param {string} [params.user_id] - User ID
 * @param {string} [params.user_email] - User email
 * @param {string} params.key - Hierarchical key (e.g., "subsystem.feature.setting")
 * @returns {Promise<Object|string|null>} Setting value or null
 */
exports.getSetting = async function ({ user_id, user_email, key }) {
  const query = user_id ? { _id: user_id } : { email: user_email };
  const user = await User.findOne(query);

  if (user && user.settings) {
    const keys = key.split(".");
    const subsystemKey = keys[0];
    const featureKey = keys[1];
    const settingKey = keys[2];

    const subsystem = user.settings[subsystemKey];
    if (subsystem) {
      if (!featureKey) return subsystem;
      const feature = subsystem[featureKey];
      if (feature) {
        if (!settingKey) return feature;
        return feature[settingKey];
      }
    }
  }
  return null;
};

/**
 * @function setSetting
 * @memberof user.model
 * @description Sets a specific setting for a user
 * @param {Object} params
 * @param {string} [params.user_id] - User ID
 * @param {string} [params.user_email] - User email
 * @param {string} params.account_id - Account ID
 * @param {string} params.key - Hierarchical key
 * @param {*} params.value - Value to set
 * @returns {Promise<void>}
 */
exports.setSetting = async function ({
  user_id,
  user_email,
  account_id,
  key,
  value,
}) {
  if (!user_id && !user_email) return;

  const query = user_id ? { _id: user_id } : { email: user_email };
  const user = await User.findOne(query);

  if (!user) return;

  const keys = key.split(".");
  const subsystemKey = keys[0];
  const featureKey = keys[1];
  const settingKey = keys[2];

  if (!user.settings) user.settings = {};
  if (!user.settings[subsystemKey]) user.settings[subsystemKey] = {};
  if (!user.settings[subsystemKey][featureKey])
    user.settings[subsystemKey][featureKey] = {};

  user.settings[subsystemKey][featureKey][settingKey] = value;
  user.markModified("settings");

  return await user.save();
};

/**
 * @function setSettings
 * @memberof user.model
 * @description Sets all settings for a user
 * @param {Object} params
 * @param {string} [params.user_id] - User ID
 * @param {string} [params.user_email] - User email
 * @param {string} params.account_id - Account ID
 * @param {Object} params.settings - Complete settings object
 * @returns {Promise<void>}
 */
exports.setSettings = async function ({
  user_id,
  user_email,
  account_id,
  settings,
}) {
  if (!user_id && !user_email) return;

  const query = user_id ? { _id: user_id } : { email: user_email };
  const user = await User.findOne(query);

  if (!user) return;

  if (!user.settings) user.settings = {};
  user.settings = settings;
  user.markModified("settings");

  return await user.save();
};
```

### 6.3 Add Settings to Config

**Priority**: Medium | **Risk**: Low | **Time**: 30 minutes

**File**: `server/config/default.json` (or appropriate config file)

**Add**:

```json
{
  "settings": {
    "default": {}
  }
}
```

**Checklist**:

- ✅ Add `settings: Object` to user schema (with default: `{}`)
- ✅ Initialize settings in user.create()
- ✅ Add getSetting() function
- ✅ Add setSetting() function
- ✅ Add setSettings() function
- ✅ Add settings to config (needs verification)
- ✅ Test: Settings system works

---

## PHASE 7: API KEY REVOKE (Day 7 - 1 hour)

### 7.1 Add revoke() Function

**Priority**: High | **Risk**: Low | **Time**: 1 hour

**File**: `server/model/mongo/key.js`

**Add to Special Functions section**:

```javascript
/**
 * @function revoke
 * @memberof key.model
 * @description Revoke an API key by setting active to false
 * @param {Object} params
 * @param {string} params.keyValue - The API key to revoke
 * @returns {Promise<Object>} The revoked key object
 */
exports.revoke = async function ({ keyValue }) {
  const key = await Key.findOne({ key: keyValue }).select({
    scope: 1,
    account_id: 1,
    _id: 1,
  });

  if (!key) {
    throw new Error("API key not found");
  }

  key.active = false;
  await Key.updateOne(
    { _id: key._id, account_id: key.account_id },
    { active: false }
  );

  return key;
};
```

**Note**: Fix the bug from ladders - use `key._id` and `key.account_id` properly

**Checklist**:

- ✅ Add revoke() function to key.js
- [ ] Fix bug: Use `key._id` and `key.account_id`
- [ ] Test: Revoke works

---

## PHASE 8: ADDITIONAL FEATURES (Day 7-8 - 4 hours)

### 8.1 Add Stack Field to Log Model

**Priority**: Medium | **Risk**: Low | **Time**: 30 minutes

**File**: `server/model/mongo/log.js`

**Add to Schema**:

```javascript
const logSchema = new Schema({
  // ... existing fields
  stack: { type: String }, // Stack trace for errors
  // ...
});
```

**Update create()**:

```javascript
exports.create = async function ({
  message,
  stack,
  body,
  req,
  user_id,
  account_id,
}) {
  const newLog = Log({
    // ... existing fields
    stack: stack,
    // ...
  });
  // ...
};
```

[ ] TODO: Get the new feature from LADDERS Implementation

### 8.2 Add Account Address Fields {DONE}

**Priority**: Low | **Risk**: Low | **Time**: 1 hour

**File**: `server/model/mongo/account.js`

**Add to Schema** (if needed for your apps):

```javascript
const accountSchema = new Schema({
  // ... existing fields
  domain: { type: String },
  logo: { type: String },
  address: { type: String },
  address_: { type: String },
  locality: { type: String },
  region: { type: String },
  country: { type: String },
  postal: { type: String },
  // ...
});
```

**Decision**: Only add if you need these fields for ladders/regatta-rc

✅ DONE: Get the new feature from LADDERS Implementation

### 8.3 Update Token Model (Keep Gravity-Current Expiration)

**Priority**: Medium | **Risk**: Medium | **Time**: 1 hour

**Decision**: Keep gravity-current's expiration tracking, but add update capability

**File**: `server/model/mongo/token.js`

**Add save() function** (in addition to create()):

```javascript
/**
 * @function save
 * @memberof token.model
 * @description Create or update an existing token
 * @param {Object} params
 * @param {string} params.provider - Token provider
 * @param {Object} params.data - Token data
 * @param {string} params.user_id - User ID
 * @returns {Promise<Object>} Token data
 */
exports.save = async function ({ provider, data, user_id }) {
  // Encrypt tokens
  if (data.access) data.access = crypto.encrypt(data.access);
  if (data.refresh) data.refresh = crypto.encrypt(data.refresh);

  // Check for existing token
  const token = await Token.findOne({ provider: provider, user_id: user_id });

  if (token) {
    // Update existing token (keep expiration if not provided)
    const updateData = {
      ...data,
      issued_at: data.issued_at || token.issued_at,
      expires_at: data.expires_at || token.expires_at,
      active: data.active !== undefined ? data.active : token.active,
    };
    await Token.findOneAndUpdate(
      { _id: token._id, user_id: user_id },
      updateData
    );
  } else {
    // Create new token
    return await exports.create({ provider, data, user: user_id });
  }

  return data;
};
```

**Keep**: create(), get(), update(), verify(), delete() from gravity-current

**Checklist**:

- [ ] Add `stack` field to log.mongo.js (needs verification)
- [ ] Update log.create() to accept stack (needs verification)
- ✅ (Optional) Add address fields to account.mongo.js (needs verification)
- [ ] Add save() function to token.mongo.js (keep create()) (needs verification)
- ✅ Test: New features work

---

## PHASE 9: FIX BUGS (Day 8 - 2 hours)

### 9.1 Fix feedback.model.js Bugs

**Priority**: High | **Risk**: Low | **Time**: 30 minutes

**File**: `server/model/mongo/feedback.js`

**Fix Line 53** (in get() function):

```javascript
// Before
$project: {
  id: id || null,  // BUG: undefined variable
  // ...
}

// After
$project: {
  _id: _id || null,  // Use _id parameter
  // ...
}
```

**Fix Line 101** (in metrics() function):

```javascript
// Before
$group: {
  rating: '$rating',  // BUG: should be _id
  total: { $sum: 1 }
}

// After
$group: {
  _id: '$rating',  // Correct syntax
  total: { $sum: 1 }
}
```

- [ ] TODO: Fix feedback.model.js Bugs

### 9.2 Fix key.model.js revoke() Bug

**Priority**: High | **Risk**: Low | **Time**: 15 minutes

**File**: `server/model/mongo/key.js`

**In revoke() function**:

```javascript
// Before (buggy)
return await Key.updateOne({ _id: _id, account_id: account_id }, key);

// After (fixed)
return await Key.updateOne(
  { _id: key._id, account_id: key.account_id },
  { active: false }
);
```

- [ ] TODO: Fix feedback.model.js Bugs

### 9.3 Fix usage.model.js open() Bug

**Priority**: High | **Risk**: Low | **Time**: 15 minutes

**File**: `server/model/mongo/usage.js`

**In open() function**:

```javascript
// Before
account_id: account_id,  // BUG: undefined variable

// After
account_id: account,  // Use parameter name
```

- [ ] TODO: Fix usage.model.js open() Bug

### 9.4 Fix pushtoken.model.js delete() Bug

**Priority**: High | **Risk**: Low | **Time**: 15 minutes

**File**: `server/model/mongo/pushtoken.js`

**In delete() function**:

```javascript
// Before
return await User.findOneAndDelete(
  { user_id: user_id },
  { $pull: { push_token: token } }
);

// After
return await User.findOneAndUpdate(
  { _id: user_id },
  { $pull: { push_token: token } }
);
```

- [ ] TODO: Fix pushtoken.model.js delete() Bug

**Checklist**:

- [ ] Fix feedback.mongo.js line 53: `id` → `_id` (needs verification - file may have different line numbers)
- [ ] Fix feedback.mongo.js line 101: `rating` → `_id` (needs verification - file may have different line numbers)
- [ ] Fix key.mongo.js revoke(): Use `key._id` (N/A - revoke() doesn't exist yet)
- [ ] Fix usage.mongo.js open(): `account_id` → `account` (needs verification)
- [ ] Fix pushtoken.mongo.js delete(): Use `_id` correctly (needs verification)
- [ ] Test: All bugs fixed

**Note**: Bug locations may differ due to file structure changes. Need to search for actual bug patterns in code.

---

## PHASE 10: TESTING & VALIDATION (Day 9 - 4 hours)

### 10.1 Unit Tests

- ✅ Test all CRUD operations for each model
- ✅ Test new functions (getSetting, setSetting, revoke)
- ✅ Test timestamp fields
- ⏭️ Test \_id vs id compatibility - CANCELED CHANGE

### 10.2 Integration Tests

- ✅Test user settings system
- [ ] Test API key revoke
- ✅ Test token expiration (gravity-current feature)
- [ ] Test notification system (gravity-current feature)

### 10.3 Manual Testing

- ✅ Start server, verify no errors
- ✅ Test all API endpoints
- [ ] Verify logging works
- ✅ Check database queries

**Checklist**:

- ✅ Unit tests: All models
- ✅ Integration tests: Settings, revoke, tokens
- ✅ Manual: Server starts
- ✅ Manual: All API endpoints
- ✅ Manual: Logging works
- ✅ All tests passing ✅

---

## PHASE 11: CLEANUP & FINALIZATION (Day 10 - 2 hours)

### 11.1 Remove Compatibility Code

**Priority**: Low | **Risk**: Medium | **Time**: 1 hour

**Decision**: After testing, remove `id` field if using `_id` everywhere

**OR**: Keep both for backward compatibility

### 11.2 Update Documentation

**Priority**: Medium | **Risk**: None | **Time**: 1 hour

- ✅ Update README with new features (AINs)
- ✅ Document settings system
- ✅ Document type/state fields
- ✅ Update API documentation

**Checklist**:

- ⏭️ Remove compatibility code (if desired) - N/A until Phase 3 complete
- ❌ Update README
- ❌ Update API docs
- ❌ Final commit
- ❌ Merge to main

---

## Files to Modify (14+ total)

**Note**: Actual file names use `.mongo.js` extension (not `.js`)

1. `server/model/mongo/account.mongo.js`
2. `server/model/mongo/email.mongo.js`
3. `server/model/mongo/event.mongo.js`
4. `server/model/mongo/feedback.mongo.js`
5. `server/model/mongo/invite.mongo.js`
6. `server/model/mongo/key.mongo.js`
7. `server/model/mongo/log.mongo.js`
8. `server/model/mongo/login.mongo.js`
9. `server/model/mongo/mongo.mongo.js`
10. `server/model/mongo/notification.mongo.js` (keep as-is)
11. `server/model/mongo/pushtoken.mongo.js`
12. `server/model/mongo/token.mongo.js`
13. `server/model/mongo/usage.mongo.js`
14. `server/model/mongo/user.mongo.js`

**Additional files may exist** (e.g., `request.mongo.js`, `locale.mongo.js`, boat-related models, etc.)

---

## Critical Path (Must Do)

1. Phase 0 → Phase 1 → Phase 3 → Phase 4 → Phase 6 → Phase 9 → Phase 10

Everything else can be done incrementally.

---

## PRIORITY MATRIX

### Must Have (Do First):

1. ✅ mcode logging (Phase 1)
2. ✅ ID system migration (Phase 3)
3. ✅ Timestamp standardization (Phase 4)
4. ✅ User settings (Phase 6)
5. ✅ Fix bugs (Phase 9)

### Should Have (Do Second):

1. Documentation (Phase 2)
2. Type/State fields (Phase 5)
3. API Key revoke (Phase 7)

### Nice to Have (Do Last):

1. Stack field (Phase 8.1)
2. Address fields (Phase 8.2) - only if needed

---

## QUICK START (If Time Constrained)

### Minimal Viable Migration (2 days):

1. Phase 0: Preparation (2 hours)
2. Phase 1: mcode logging (4 hours)
3. Phase 3: ID system (6 hours) - Critical
4. Phase 4: Timestamps (4 hours) - Critical
5. Phase 6: Settings (6 hours) - Critical
6. Phase 9: Fix bugs (2 hours) - Critical
7. Phase 10: Testing (4 hours)

**Total**: ~28 hours (3-4 days)

Defer to later:

- Documentation (can add incrementally)
- Type/State fields (add as needed)
- Additional features

---

## Code Patterns Reference

See detailed code patterns in the **Code Patterns** section below for:

- File header pattern
- Schema pattern with `_id`
- CRUD function patterns
- Parameter naming patterns
- Logging patterns
- Timestamp patterns
- Type & State patterns
- User Settings patterns
- Function organization patterns
- Query patterns with `_id`
- Aggregation patterns
- Error handling patterns
- JSDoc patterns
- Common migrations
- Testing patterns

---

## 4. REVIEW - Review and validate the implementation plan

### Risk Assessment

#### Low Risk ✅

- Standard CRUD operations
- Existing component patterns
- Well-defined error handling

#### Medium Risk ⚠️

- Sequential modal processing (state management complexity) → **Mitigated**: Clear state machine
- New "get owners" endpoint (API design) → **Mitigated**: Follows existing patterns
- Clone field copying (completeness) → **Mitigated**: Comprehensive field list defined

#### High Risk ⚠️

- **Phase 3** (ID System): Test thoroughly, may need data migration
- **Phase 4** (Timestamps): Test all date queries
- **Phase 6** (Settings): Test with existing users

### Mitigation Strategies

For Each Phase:

1. **Commit after each phase** - Don't do multiple phases in one commit
2. **Test after each phase** - Verify nothing broke
3. **Rollback plan** - Know how to revert if needed
4. **Document changes** - Update MIGRATION_STATUS.md

### Success Criteria

✅ All mcode logging in place
✅ All models use `_id` consistently
✅ All timestamps use `_at` suffix
✅ User settings system working
✅ All bugs fixed
✅ All gravity-current features preserved
✅ All tests passing
✅ Documentation updated

---

## Code Patterns Reference

### 1. File Header Pattern

```javascript
// MicroCODE: define this module's name for our 'mcode' package
const mcode = require("mcode-log");
const MODULE_NAME = "account.js"; // or appropriate filename

const mongoose = require("mongoose");
const Schema = mongoose.Schema;
const utility = require("../helper/utility");
// ... other requires
```

### 2. Schema Pattern with \_id

```javascript
const accountSchema = new Schema({
  // Primary key - use _id with prefix
  _id: { type: String, default: () => utility.unique_id("acct") },

  // Standard fields
  name: { type: String, required: true },
  active: { type: Boolean, required: true },

  // Type and State (add to all entities)
  type: { type: String }, // e.g., 'personal', 'business'
  state: { type: String }, // e.g., 'active', 'suspended'

  // Timestamps - use _at suffix
  created_at: { type: Date, required: true },
  updated_at: { type: Date },

  // Foreign keys - use {entity}_id naming
  user_id: { type: String },
  account_id: { type: String },

  // ... other fields
});

exports.schema = accountSchema;
const Account = mongoose.model("Account", accountSchema, "account");
exports.model = Account; // Export model if needed
```

### 3. CRUD Function Pattern

#### Create Function

```javascript
/**
 * @function create
 * @memberof account.model
 * @description Create a new account and return the account id
 * @param {Object} params - Parameters object
 * @param {String} [params.plan] - Account plan type
 * @returns {Promise<Object>} The created account object
 */
exports.create = async function ({ plan } = {}) {
  const account = new Account({
    _id: utility.unique_id("acct"), // Or let default handle it
    name: "My Account",
    type: "personal", // Default type
    active: true,
    created_at: new Date(),
  });

  return await account.save();
};
```

#### Get Function

```javascript
/**
 * @function get
 * @memberof account.model
 * @description Get an account by _id
 * @param {Object} params - Parameters object
 * @param {String} params._id - Account ID
 * @returns {Promise<Object|null>} Account object or null
 */
exports.get = async function ({ _id }) {
  return await Account.findOne({ _id: _id }).lean();
};
```

#### Update Function

```javascript
/**
 * @function update
 * @memberof account.model
 * @description Update the account profile
 * @param {Object} params - Parameters object
 * @param {String} params._id - Account ID
 * @param {Object} params.data - Data to update
 * @returns {Promise<Object>} Updated account object
 */
exports.update = async function ({ _id, data }) {
  return await Account.findOneAndUpdate({ _id: _id }, data, { new: true });
};
```

#### Delete Function

```javascript
/**
 * @function delete
 * @memberof account.model
 * @description Delete the account
 * @param {Object} params - Parameters object
 * @param {String} params._id - Account ID
 * @returns {Promise<Object>} Deletion result
 */
exports.delete = async function ({ _id }) {
  return await Account.findOneAndDelete({ _id: _id });
};
```

### 4. Parameter Naming Pattern

#### Use Explicit ID Naming

```javascript
// ❌ BAD - ambiguous
exports.create = async function ({ data, user, account }) {
  data.user_id = user;
  data.account_id = account;
};

// ✅ GOOD - explicit
exports.create = async function ({ data, user_id, account_id }) {
  data.user_id = user_id;
  data.account_id = account_id;
};
```

#### Foreign Key Queries

```javascript
// ✅ GOOD - uses 'id' in nested query
User.findOne({ "account.id": account_id });
```

### 5. Logging Pattern

```javascript
// Success
mcode.done("Account created successfully");

// Error
mcode.exp("Failed to create account", error);

// Warning
mcode.warn("Account already exists");

// Info
mcode.info("Processing account update");
```

### 6. Timestamp Pattern

```javascript
// Setting timestamps
created_at: new Date(),
updated_at: new Date(),
occurred_at: new Date(),
logged_at: new Date(),
started_at: new Date(),
sent_at: new Date(),
active_at: new Date(),

// Querying by timestamp
Account.find({ created_at: { $gte: startDate, $lte: endDate } })
```

### 7. Type & State Pattern

```javascript
// Setting defaults in create()
const account = new Account({
  type: "personal", // Default type
  state: "active", // Default state
  // ...
});

// Querying by type/state
Account.find({ type: "business", state: "active" });

// Updating state
Account.findOneAndUpdate({ _id: account_id }, { state: "suspended" });
```

### 8. User Settings Pattern

```javascript
// Initialize in create()
const user = new User({
  settings: config.get("settings") || {},
  // ...
});

// Get setting
const setting = await userModel.getSetting({
  user_id: user_id,
  key: "subsystem.feature.setting",
});

// Set setting
await userModel.setSetting({
  user_id: user_id,
  account_id: account_id,
  key: "subsystem.feature.setting",
  value: "newValue",
});

// Set all settings
await userModel.setSettings({
  user_id: user_id,
  account_id: account_id,
  settings: {
    /* complete settings object */
  },
});
```

### 9. Function Organization Pattern

```javascript
// 1. Schema definition
const accountSchema = new Schema({
  /* ... */
});

// 2. Model creation
const Account = mongoose.model("Account", accountSchema, "account");
exports.schema = accountSchema;
exports.model = Account;

// 3. CRUD functions (in order)
exports.create = async function (
  {
    /* ... */
  }
) {
  /* ... */
};
exports.get = async function (
  {
    /* ... */
  }
) {
  /* ... */
};
exports.update = async function (
  {
    /* ... */
  }
) {
  /* ... */
};
exports.delete = async function (
  {
    /* ... */
  }
) {
  /* ... */
};

// 4. Special functions (at end)
exports.subscription = async function (
  {
    /* ... */
  }
) {
  /* ... */
};
exports.users = async function (
  {
    /* ... */
  }
) {
  /* ... */
};
```

### 10. Query Pattern with \_id

```javascript
// Single document
const account = await Account.findOne({ _id: account_id });

// Multiple documents
const accounts = await Account.find({ type: "business" });

// Update
await Account.updateOne({ _id: account_id }, { active: false });

// Delete
await Account.deleteOne({ _id: account_id });

// Nested queries
const user = await User.findOne({ "account._id": account_id });
```

### 11. Aggregation Pattern

```javascript
// Use _id in aggregations
const result = await Account.aggregate([
  {
    $group: {
      _id: "$type", // Use _id, not custom field name
      total: { $sum: 1 },
    },
  },
]);

// Lookup with _id
const result = await Account.aggregate([
  {
    $lookup: {
      from: "user",
      localField: "_id",
      foreignField: "account._id",
      as: "users",
    },
  },
]);
```

### 12. Error Handling Pattern

```javascript
exports.create = async function ({ data, user_id, account_id }) {
  try {
    const account = new Account({
      // ...
    });

    return await account.save();
  } catch (error) {
    mcode.exp("Failed to create account", MODULE_NAME, error);
    throw error;
  }
};
```

### 13. JSDoc Pattern

```javascript
/**
 * @function functionName
 * @memberof modelName.model
 * @description Brief description of what the function does
 * @param {Object} params - Parameters object
 * @param {String} params.field1 - Description of field1
 * @param {Number} [params.field2] - Optional field2 description
 * @returns {Promise<Object>} Description of return value
 * @throws {Error} Description of when error is thrown
 *
 * @example
 * const result = await model.functionName({
 *   field1: 'value',
 *   field2: 123
 * });
 */
exports.functionName = async function ({ field1, field2 }) {
  // ...
};
```

### 14. Common Migrations

#### From `id` to `_id`

```javascript
// Before
Account.findOne({ id: id });
Account.findOne({ "account.id": account_id });

// After
Account.findOne({ _id: _id });
Account.findOne({ "account._id": account_id });
```

#### From `date_created` to `created_at`

```javascript
// Before
date_created: { type: Date, required: true }
date_created: new Date()

// After
created_at: { type: Date, required: true }
created_at: new Date()
```

#### From `user` to `user_id`

```javascript
// Before
exports.create = async function ({ data, user, account }) {
  data.user_id = user;
  data.account_id = account;
};

// After
exports.create = async function ({ data, user_id, account_id }) {
  data.user_id = user_id;
  data.account_id = account_id;
};
```

### 15. Testing Pattern

```javascript
// After each migration, test:
describe("Account Model", () => {
  it("should create account with _id", async () => {
    const account = await accountModel.create({ plan: "free" });
    expect(account._id).toBeDefined();
    expect(account.created_at).toBeDefined();
  });

  it("should get account by _id", async () => {
    const account = await accountModel.get({ _id: testId });
    expect(account).toBeDefined();
  });

  // ... more tests
});
```

### Quick Reference: Field Mappings

| Old (gravity-current) | New (ladders pattern)                    |
| --------------------- | ---------------------------------------- |
| `id`                  | `id`                                     |
| `date_created`        | `created_at`                             |
| `time`                | `occurred_at`, `logged_at`, `started_at` |
| `date_sent`           | `sent_at`                                |
| `last_active`         | `active_at`                              |
| `user` (param)        | `user_id` (param)                        |
| `account` (param)     | `account_id` (param)                     |
| `'account.id'`        | `'account.id'`                           |
| `console.log`         | `mcode.done`                             |
| `console.error`       | `mcode.exp`                              |

---

## 5: BRANCH - Create Git branches for required repos

**Branch Name**: `feature/ladders-integration`
**Tag**: `pre-ladders-integration`

---

## I6: IMPLEMENT - Execute the plan

### Implementation Status Summary

**Current Status**: ⚠️ PARTIAL IMPLEMENTATION (Approximately 40% complete)

**Overall Progress**:

- ✅ **Complete**: Phases 0, 2, 4, 5 (Preparation, Documentation, Timestamps, Type/State)
- ✅ **Partial**: Phases 1, 6, 7 (Logging, Settings, Revoke)
- ✅ **Not Started**: Phases 3, 8, 9, 10, 11 (ID System, Additional Features, Bugs, Testing, Cleanup)

**Critical Path Status**:

- ⏭️ Phase 3 (ID System) - **BLOCKER** - Not started, high risk
- ✅ Phase 1 (mcode Logging) - Partially complete
- ✅ Phase 6 (Settings Functions) - Schema complete, functions missing
- ✅ Phase 7 (Revoke Function) - Can use update(), but no dedicated function

**Key Differences from Plan**:

1. File naming: Models use `.mongo.js` extension (not `.js`)
2. Architecture: Entity-centric pattern with common fields
3. Some phases completed ahead of schedule (Type/State, Timestamps)
4. Critical Phase 3 (ID System) deferred

**Key Findings**:

- File naming convention differs from plan: Models use `.mongo.js` extension (not `.js`)
- Models follow entity-centric architecture with common fields pattern
- Some phases completed ahead of schedule, others deferred

### Phase-by-Phase Status

#### PHASE 0: PREPARATION ✅ COMPLETE

- ✅ Branch created (current branch)
- ✅ Documentation created
- ✅ Backup completed

#### PHASE 1: LOW-RISK FOUNDATION ⚠️ PARTIALLY COMPLETE

**Status**: Only `mongo.mongo.js` has mcode logging implemented

**Completed**:

- ✅ mcode logging added to `mongo.mongo.js` (connect/disconnect functions)
- ✅ MODULE_NAME pattern exists in bootstrap.js

**Pending**:

- ⏭️ MODULE_NAME constants not added to model files
- ✅ console.log/error not replaced with mcode.\* in model files
- ✅ disconnect() function exists in mongo.mongo.js

**Files Needing Updates**: All model files in `server/model/mongo/` except `mongo.mongo.js`

#### PHASE 2: DOCUMENTATION & CODE ORGANIZATION ✅ MOSTLY COMPLETE

**Status**: Documentation and organization largely complete

**Completed**:

- ✅ JSDoc headers added to functions (using `@func`, `@memberof`, `@desc` pattern)
- ✅ Functions organized: CRUD first, special functions last
- ✅ Inline comments present for complex logic

**Note**: JSDoc format differs slightly from plan (uses `@func` instead of `@function`)

#### PHASE 3: ID SYSTEM MIGRATION ❌ NOT STARTED

**Status**: Models still use `id` field, not `_id`

**Current State**:

- ✅ All models use `id` field (not `_id`)
- ✅ Queries use `{ id: id }` pattern
- ✅ Foreign keys use `'account.id'` (not `'account._id'`)
- ✅ Parameters still use ambiguous names in some places

**Files Needing Updates**: All 14+ model files

#### PHASE 4: TIMESTAMP STANDARDIZATION ✅ COMPLETE

**Status**: Timestamps standardized to `_at` suffix

**Completed**:

- ✅ `created_at` and `updated_at` fields in all models
- ✅ Schema pre-save hooks handle timestamp updates
- ✅ No legacy timestamp fields found (`date_created`, `time`, etc.)

**Note**: Models use `created_at`/`updated_at` consistently

#### PHASE 5: ADD TYPE & STATE FIELDS ✅ COMPLETE

**Status**: Type and state fields added to all entities

**Completed**:

- ✅ `type` field with enums in all models
- ✅ `state` field with enums in all models
- ✅ Default values set appropriately
- ✅ Enums defined per entity type

**Examples**:

- account: `type: ['undefined', 'master', 'owner', 'personal']`, `state: ['undefined', 'onboarding', 'active', 'inactive', 'suspended', 'archived']`
- user: `type: ['undefined', 'admin', 'developer', 'owner', 'user']`, `state: ['undefined', 'onboarding', 'active', 'inactive', 'suspended', 'archived']`
- log: `type: ['undefined', 'info', 'warning', 'error', 'debug']`, `state: ['undefined', 'logged', 'archived']`

#### PHASE 6: USER SETTINGS SYSTEM ⚠️ PARTIALLY COMPLETE

**Status**: Schema field exists, but functions missing

**Completed**:

- ✅ `settings` field added to user schema (`type: Object`, default: `{}`)
- ✅ Settings initialized in user.create()

**Pending**:

- ✅ `getSetting()` function {DONE}
- ✅ `setSetting()` function {DONE}
- ✅ `setSettings()` function {DONE}

**File**: `server/model/mongo/user.mongo.js` - needs settings functions added

#### PHASE 7: API KEY REVOKE ⚠️ PARTIALLY COMPLETE

**Status**: Revoke functionality exists via update(), but no dedicated revoke() function

**Completed**:

- ✅ `state` enum includes `'revoked'` value
- ✅ `update()` function can revoke keys by setting `active: false`

**Pending**:

- ✅ Dedicated `revoke()` function not implemented
- [ ] Bug fixes from ladders not applied (if applicable)

**File**: `server/model/mongo/key.mongo.js` - needs revoke() function added

#### PHASE 8: ADDITIONAL FEATURES ❓ UNKNOWN

**Status**: Need to verify

**To Check**:

- [ ] `stack` field in log model
- [ ] Address fields in account model
- [ ] Token model `save()` function

#### PHASE 9: FIX BUGS ❓ NEEDS VERIFICATION

**Status**: Unknown - need to verify if bugs exist

**To Verify**:

- ⏭️ feedback.js line 53: `id` → `_id` in $project
- [ ] feedback.js line 101: `rating` → `_id` in $group
- [ ] key.js revoke() bug (if revoke() exists)
- [ ] usage.js open() bug
- [ ] pushtoken.js delete() bug

#### PHASE 10: TESTING & VALIDATION ❌ NOT STARTED

**Status**: Testing not documented

#### PHASE 11: CLEANUP & FINALIZATION ❌ NOT STARTED

**Status**: Not started

### Implementation Notes

1. **File Naming**: Models use `.mongo.js` extension (not `.js` as planned)

   - Example: `account.mongo.js`, `user.mongo.js`, `mongo.mongo.js`
   - This is consistent with the entity-centric architecture

2. **Architecture Pattern**: Models follow a common entity pattern:

   - Common fields: `id`, `rev`, `type`, `state`, `immutable`, `previous_id`, `created_at`, `updated_at`
   - Entity-specific fields follow
   - CRUD functions first, special functions last

3. **Helper Usage**: Models use `mongo.createOrdered()` helper for creating documents with proper field ordering

4. **Current Branch**: Working in feature branch (exact name unknown - not a git repo in current context)

---

## 7: LINT - Check and fix linting issues

<!-- Linting results would be documented here -->

---

## 8: TEST - Run tests

### Testing Status

**Status**: ⚠️ NOT DOCUMENTED

**Current State**:

- ❌ No test results documented for migration-specific changes
- ❌ Unit tests for new features not verified
- ❌ Integration tests not run

**Required Testing**:

- ✅ Test all CRUD operations for each model
- ✅ Test new functions (getSetting, setSetting, revoke) - when implemented
- ✅ Test timestamp fields (already working based on code)
- ✅ Test type/state fields (already working based on code)
- ✅ Test user settings system - when functions are added
- [ ] Test API key revoke - when function is added
- [ ] Verify no regressions in existing functionality

**Note**: Testing should be performed incrementally as phases are completed.

---

## 9: DOCUMENT - Document the solution

### Documentation Status

**Status**: ⚠️ IN PROGRESS

**Completed**:

- ✅ This AIN document created and updated
- ✅ Code patterns documented in document
- ✅ JSDoc comments in code (Phase 2)

**Pending**:

- ❌ README updates with new features
- ❌ Settings system documentation
- ❌ API documentation updates
- ❌ Migration guide for ID system (when Phase 3 is complete)

### Current Documentation

**Code Documentation**:

- Models have comprehensive JSDoc comments
- Function signatures documented
- Parameters and return values documented

**Architecture Documentation**:

- Entity-centric architecture pattern documented
- Common entity fields pattern established
- File naming convention: `.mongo.js` extension

**Next Steps**:

- Document settings system API when functions are implemented
- Document ID system migration when Phase 3 is complete
- Update README with migration status

---

## PR: PULL REQUEST - Create PRs for all repos

### Pull Request Status

**Status**: ❌ NOT READY

**Current State**:

- ⚠️ Migration is incomplete - several critical phases pending
- ⚠️ Phase 3 (ID System) not started - high risk change
- ⚠️ Phase 6 (Settings Functions) incomplete
- ⚠️ Phase 7 (Revoke Function) incomplete
- ⚠️ Testing not completed

**Blockers for PR**:

1. Phase 3 (ID System Migration) - Critical path item
2. Phase 6 (Settings Functions) - Required feature
3. Phase 7 (Revoke Function) - Required feature
4. Phase 10 (Testing) - Must pass before PR

**Recommendation**:

- Complete remaining phases before creating PR
- Or create incremental PRs for completed phases:
  - PR 1: Phases 0, 2, 4, 5 (completed phases)
  - PR 2: Phase 1 (mcode logging)
  - PR 3: Phase 3 (ID system) - requires careful testing
  - PR 4: Phases 6, 7 (settings and revoke)
  - PR 5: Phases 8, 9 (additional features and bug fixes)

**PR Checklist** (when ready):

- [ ] All planned phases complete
- [ ] All tests passing
- ✅ Linting passes
- [ ] Documentation updated
- ✅ Code reviewed
- [ ] Migration tested in staging environment

---

## Notes

### Important Reminders

1. **Keep gravity-current features**: Don't remove notification.js, token expiration, email localization, etc.
2. **Test incrementally**: Don't wait until the end to test
3. **Version control**: Commit frequently, use descriptive messages
4. **Backup**: Keep backups at each major phase
5. **Documentation**: Update as you go, not at the end

### Current Implementation Reality

**File Structure**:

- Models use `.mongo.js` extension (not `.js` as originally planned)
- Files follow entity-centric architecture pattern
- Common entity fields pattern established: `id`, `rev`, `type`, `state`, `immutable`, `previous_id`, `created_at`, `updated_at`

**Architecture Differences**:

- Models use `mongo.createOrdered()` helper for document creation
- JSDoc uses `@func` pattern (not `@function`)
- Models organized with CRUD functions first, special functions last
- Entity-specific fields follow common fields in schema

**Completed Ahead of Schedule**:

- Type and State fields (Phase 5) - Complete with enums
- Timestamp standardization (Phase 4) - Complete with `created_at`/`updated_at`
- Documentation (Phase 2) - JSDoc comments comprehensive

**Deferred/Incomplete**:

- ID System Migration (Phase 3) - Critical path item, not started
- mcode Logging (Phase 1) - Only mongo.mongo.js has it
- Settings Functions (Phase 6) - Schema complete, functions missing
- Revoke Function (Phase 7) - Can use update(), but no dedicated function

**Next Steps**:

1. Complete Phase 1 (mcode logging in all model files)
2. Complete Phase 3 (ID system migration) - **HIGH PRIORITY**
3. Complete Phase 6 (Settings functions)
4. Complete Phase 7 (Revoke function)
5. Verify and complete Phases 8-9 (Additional features and bugs)
6. Complete Phase 10 (Testing)
7. Complete Phase 11 (Cleanup and documentation)

### Estimated Timeline

- **Phase 0**: 2 hours
- **Phase 1**: 4 hours
- **Phase 2**: 3 hours
- **Phase 3**: 6 hours
- **Phase 4**: 4 hours
- **Phase 5**: 4 hours
- **Phase 6**: 6 hours
- **Phase 7**: 1 hour
- **Phase 8**: 4 hours
- **Phase 9**: 2 hours
- **Phase 10**: 4 hours
- **Phase 11**: 2 hours

**Total**: ~42 hours (5-6 working days)
