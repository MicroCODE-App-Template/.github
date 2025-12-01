# Migration Plan: Ladders Features → App-Template

## Safe, Time-Efficient Integration Strategy

### Overview

This plan merges the best features from **ladders** into **app-template** (based on gravity-current) while preserving all gravity-current improvements and fixing bugs.

---

## PHASE 0: PREPARATION (Day 1 - 2 hours)

**Status**: ✅ **COMPLETE** - Test infrastructure fully functional

**Summary**: All 42 tests passing consistently. Test suite now includes:

- Pre-test cleanup (removes leftover test data before each run)
- Post-test cleanup (removes test data after each run)
- Proper server initialization for test mode
- MongoDB connection handling in test environment
- Robust error handling to prevent test suite crashes

### 0.1 Backup & Version Control

- ✅ Create a new branches: `feature/0003--![1764608058516](image/MIGRATION_PLAN/1764608058516.png)ladders-integration`
- ✅ Tag current state: `git tag -a migration-baseline -m "Baseline before ladders integration"`
- ✅ Push tag: `git push origin migration-baseline`
- ✅ Document current app-template features - see Gravity Docs
- ✅ Create backup of entire app-template directory

**Tagging Strategy**: See `TAGGING_STRATEGY.md` for complete tagging guide. Quick reference:

- Create checkpoint tags before high-risk phases (3, 4, 6)
- Create phase tags after each phase completes and tests pass
- Create tested tags after successful testing of high-risk phases

### 0.2 Setup Testing Infrastructure

- ✅ Ensure all existing tests pass: `npm test` - **COMPLETE: All 42 tests passing**
- ✅ Fixed test infrastructure issues (server export, MongoDB connection, cleanup)
- ✅ Added pre-test cleanup to ensure tests can run repeatedly
- ✅ All tests now pass consistently on repeated runs
- ✅ Create test checklist for each model
- ❌ Set up test database for migration testing - NO MIGRATION REQUIRED

### 0.3 Create Feature Tracking

- ✅ Create `MIGRATION_STATUS.md` to track progress
- ✅ List all files that need changes
- ✅ Create checklist for each feature

---

## PHASE 1: LOW-RISK FOUNDATION (Day 1-2 - 4 hours) ✅

### 1.1 Add mcode Logging (Low Risk - No Schema Changes) ✅

**Priority**: High | **Risk**: Low | **Time**: 1 hour

**Files to Update**:

- `server/model/mongo/mongo.js` - Add mcode logging
- All model files - Replace `console.log|warn|error` with `mcode.log|warn|error`

**Steps**:

1. Update `mongo.js`:

NOTE: The current version of mcode.\* logging does not require MODULE_NAME.
It now uses trace stack for logging source if not supplied.

2. For each model file, add at top:

NOTE: The current version of mcode.\* logging does not require MODULE_NAME.
It now uses trace stack for logging source if not supplied.

3. Replace `console.log` → `mcode.done()`
4. Replace `console.error` → `mcode.exp()`
5. Replace `console.warn` → `mcode.warn()`

**Testing**: Run server, verify logs appear correctly

---

### 1.2 Add MODULE_NAME Constants ✅

NOTE: The current version of mcode.\* logging does not require MODULE_NAME.
It now uses trace stack for logging source if not supplied.

**Priority**: Medium | **Risk**: Low | **Time**: 30 minutes

**Action**: Add `const MODULE_NAME = 'filename.js';` to every model file

**Files**: All 14 model files in `server/model/mongo/`

---

### 1.3 Add MongoDB Disconnect Function ✅

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

---

## PHASE 2: DOCUMENTATION & CODE ORGANIZATION (Day 2 - 3 hours)

### 2.1 Add JSDoc Headers to All Functions ✅

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

**Order**:

1. CRUD functions first (create, get, update, delete)
2. Speciality functions at end

---

### 2.2 Reorganize Functions (CRUD First) ✅

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

---

### 2.3 Add Inline Comments for Complex Logic ✅

**Priority**: Low | **Risk**: None | **Time**: 30 minutes

**Action**: Add comments where:

- Bugs were fixed
- Complex logic exists
- Non-obvious decisions were made

---

## PHASE 3: ID GENERATION MIGRATION (Day 3-4 - 4 hours) ⚠️ MEDIUM RISK

### 3.0 Create Checkpoint Tag (BEFORE STARTING)

**⚠️ CRITICAL**: Create checkpoint tag before starting this phase:

```bash
git tag -a migration-checkpoint-before-id-handling -m "After PHASE 2 100% complete."
git push origin migration-checkpoint-before-id-handling
```

This allows safe rollback if something goes wrong.

### 3.1 Strategy: Replace UUID with unique_id()

**Decision**: Keep MongoDB's `_id` invisible (handled by MongoDB), use `id` as primary key with `utility.unique_id()` generation

**Current State**: All models use `uuidv4()` for `id` generation
**Target State**: All models use `utility.unique_id('prefix')` for `id` generation

**Key Principles**:

1. **MongoDB `_id`**: Keep invisible, let MongoDB handle it automatically (like gravity-current)
2. **App `id`**: Use `utility.unique_id('prefix')` instead of `uuidv4()`
3. **Foreign Keys**: Continue using `{entity}_id` naming (already correct)

### 3.2 Entity Prefix Mapping

See `PHASE3_ID_PREFIXES.md` for complete prefix mapping. Each entity uses a 4-character prefix:

**Database Entities:**

- `account` → `'acct'`
- `user` → `'user'`
- `event` → `'evnt'`
- `feedback` → `'fdbk'`
- `invite` → `'invt'`
- `key` (API key) → `'apik'`
- `log` → `'log_'`
- `login` → `'logn'`
- `notification` → `'notf'`
- `pushtoken` → `'psht'`
- `token` → `'tokn'`
- `usage` → `'used'`
- `email` → `'mail'`

**Non-Database Entities:**

- `job` (queue) → `'jobs'` (Redis/Bull)
- `file` (upload) → `'file'` (S3/File System)

### 3.3 Step-by-Step Migration

#### Step 3.3.1: Update Schema Defaults (MongoDB Models)

**Priority**: High | **Risk**: Low | **Time**: 1 hour

**Files**: All MongoDB model files (`server/model/mongo/*.mongo.js` and `admin/model/mongo/*.mongo.js`)

**Pattern**:

```javascript
// Before
const AccountSchema = new Schema({
  id: {
    type: String,
    required: true,
    unique: true,
  },
  // ...
});

// After
const utility = require("../helper/utility");

const AccountSchema = new Schema({
  id: {
    type: String,
    required: true,
    unique: true,
    default: () => utility.unique_id("acct"),
  },
  // ...
});
```

**Action**:

1. Add `const utility = require('../helper/utility');` to each model file (if not already present)
2. Update `id` field in schema to include `default: () => utility.unique_id('prefix')`
3. Use appropriate prefix for each entity (see prefix mapping)

**Testing**: Verify schemas compile correctly

---

#### Step 3.3.2: Remove Explicit ID Generation from Create Functions (MongoDB Models)

**Priority**: High | **Risk**: Low | **Time**: 1 hour

**Files**: All MongoDB model files

**Pattern**:

```javascript
// Before
exports.create = async function ({ data, account }) {
  data.id = uuidv4(); // Remove this line
  // ...
};

// After
exports.create = async function ({ data, account }) {
  // id will be auto-generated by schema default - no explicit generation needed
  // ...
};
```

**Action**:

1. Remove `const { v4: uuidv4 } = require('uuid');` from files (if only used for id generation)
2. Remove all `data.id = uuidv4();` lines - schema default will handle ID generation automatically
3. Do NOT add explicit generation - rely on schema defaults only

**Note**: Schema defaults will automatically generate IDs when documents are created. No explicit generation needed in create() functions.

**Testing**: Test all create operations - verify IDs are generated correctly by schema defaults

---

#### Step 3.3.3: Update SQL Model Files

**Priority**: High | **Risk**: Low | **Time**: 1 hour

**Files**: All SQL model files (`server/model/sql/*.sql.js` and `admin/model/sql/*.sql.js`)

**Pattern**:

```javascript
// Before
const { v4: uuidv4 } = require("uuid");

exports.create = async function ({ plan } = {}) {
  const data = {
    id: uuidv4(),
    // ...
  };
  // ...
};

// After
const utility = require("../helper/utility");

exports.create = async function ({ plan } = {}) {
  const data = {
    id: utility.unique_id("acct"), // SQL requires explicit generation
    // ...
  };
  // ...
};
```

**Action**:

1. Replace `const { v4: uuidv4 } = require('uuid');` with `const utility = require('../helper/utility');`
2. Replace all `uuidv4()` calls with `utility.unique_id('prefix')`
3. Use appropriate prefix for each entity
4. **Note**: SQL models require explicit generation (no schema defaults like MongoDB)

**Testing**: Test all SQL create operations

---

#### Step 3.3.4: Update Controller Files (if needed)

**Priority**: Medium | **Risk**: Low | **Time**: 30 minutes

**Files**: Any controller files that generate IDs directly

**Action**:

1. Search for `uuidv4()` in controller files
2. Replace with `utility.unique_id('prefix')` if generating entity IDs
3. Use appropriate prefix for each entity

**Testing**: Test affected endpoints

---

#### Step 3.3.5: Verify Foreign Key Naming

**Priority**: Low | **Risk**: None | **Time**: 30 minutes

**Action**: Verify all foreign keys use `{entity}_id` naming (should already be correct)

**Pattern**:

- ✅ `user_id`, `account_id`, `event_id`, etc. (correct)
- ❌ `userId`, `accountId`, etc. (incorrect - should not exist)

**Testing**: Visual inspection, no code changes expected

**After Testing**: If all tests pass, create phase tag:

```bash
git tag -a migration-phase-3-id-generation -m "Phase 3: ID generation migration complete"
git push origin migration-phase-3-id-generation
```

---

## PHASE 4: TIMESTAMP STANDARDIZATION (Day 4-5 - 4 hours)

### 4.0 Create Checkpoint Tag (BEFORE STARTING)

**⚠️ CRITICAL**: Create checkpoint tag before starting this phase:

```bash
git tag -a migration-checkpoint-before-timestamps -m "Checkpoint before timestamp standardization"
git push origin migration-checkpoint-before-timestamps
```

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

**After Testing**: If all tests pass, create phase tag:

```bash
git tag -a migration-phase-4-timestamps -m "Phase 4: Timestamp standardization complete"
git push origin migration-phase-4-timestamps
```

---

### 4.2 Update Date Setting Code

**Priority**: High | **Risk**: Low | **Time**: 1 hour

**Pattern**:

```javascript
// Before
date_created: new Date();

// After
created_at: new Date();
```

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

---

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

---

## PHASE 6: USER SETTINGS SYSTEM (Day 6-7 - 6 hours) ⚠️ COMPLEX

### 6.0 Create Checkpoint Tag (BEFORE STARTING)

**⚠️ CRITICAL**: Create checkpoint tag before starting this phase:

```bash
git tag -a migration-checkpoint-before-settings -m "Checkpoint before user settings system"
git push origin migration-checkpoint-before-settings
```

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

---

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

---

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

**After Testing**: If all tests pass, create phase tag:

```bash
git tag -a migration-phase-6-settings -m "Phase 6: User settings system complete"
git push origin migration-phase-6-settings
```

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

---

## PHASE 8: ADDITIONAL LADDERS FEATURES (Day 7-8 - 4 hours)

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

---

### 8.2 Add Account Address Fields (Optional)

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

---

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

---

## PHASE 9: FIX BUGS FROM LADDERS (Day 8 - 2 hours)

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

---

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

---

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

---

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

---

## PHASE 10: TESTING & VALIDATION (Day 9 - 4 hours)

### 10.1 Unit Tests

- [ ] Test all CRUD operations for each model
- [ ] Test new functions (getSetting, setSetting, revoke)
- [ ] Test timestamp fields
- [ ] Test \_id vs id compatibility

### 10.2 Integration Tests

- [ ] Test user settings system
- [ ] Test API key revoke
- [ ] Test token expiration (gravity-current feature)
- [ ] Test notification system (gravity-current feature)

### 10.3 Manual Testing

- [ ] Start server, verify no errors
- [ ] Test all API endpoints
- [ ] Verify logging works
- [ ] Check database queries

---

## PHASE 11: CLEANUP & FINALIZATION (Day 10 - 2 hours)

### 11.1 Remove Compatibility Code

**Priority**: Low | **Risk**: Medium | **Time**: 1 hour

**Decision**: After testing, remove `id` field if using `_id` everywhere

**OR**: Keep both for backward compatibility

---

### 11.2 Update Documentation

**Priority**: Medium | **Risk**: None | **Time**: 1 hour

- [ ] Update README with new features
- [ ] Document settings system
- [ ] Document type/state fields
- [ ] Update API documentation

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

## RISK MITIGATION

### For Each Phase:

1. **Commit after each phase** - Don't do multiple phases in one commit
2. **Test after each phase** - Verify nothing broke
3. **Rollback plan** - Know how to revert if needed
4. **Document changes** - Update MIGRATION_STATUS.md

### High-Risk Phases:

- **Phase 3** (ID Generation): Test thoroughly, verify ID format matches expectations
- **Phase 4** (Timestamps): Test all date queries
- **Phase 6** (Settings): Test with existing users

---

## ESTIMATED TIMELINE

- **Phase 0**: 2 hours ✅
- **Phase 1**: 4 hours ✅
- **Phase 2**: 3 hours ✅
- **Phase 3**: 4 hours
- **Phase 4**: 4 hours
- **Phase 5**: 4 hours
- **Phase 6**: 6 hours
- **Phase 7**: 1 hour
- **Phase 8**: 4 hours
- **Phase 9**: 2 hours
- **Phase 10**: 4 hours
- **Phase 11**: 2 hours

**Total**: ~42 hours (5-6 working days)

---

## QUICK START (If Time Constrained)

### Minimal Viable Migration (2 days):

1. Phase 0: Preparation (2 hours) ✅
2. Phase 1: mcode logging (4 hours) ✅
3. Phase 3: ID generation (4 hours) - Critical
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

## NOTES

1. **Keep gravity-current features**: Don't remove notification.js, token expiration, email localization, etc.
2. **Test incrementally**: Don't wait until the end to test
3. **Version control**: Commit frequently, use descriptive messages
4. **Backup**: Keep backups at each major phase
5. **Documentation**: Update as you go, not at the end

---

## SUCCESS CRITERIA

✅ All mcode logging in place
✅ All models use `utility.unique_id()` for ID generation
✅ All timestamps use `_at` suffix
✅ User settings system working
✅ All bugs fixed
✅ All gravity-current features preserved
✅ All tests passing
✅ Documentation updated
