# Migration Checklist - Quick Reference

## Phase 0: Preparation ✅ COMPLETE

**Status**: Test infrastructure fully functional - All 42 tests passing consistently

- ✅ Create branch: `feature/0003--ladders-integration`
- ✅ Tag: `git tag -a migration-baseline -m "Baseline before ladders integration"`
- ✅ Push tag: `git push origin migration-baseline`
- ✅ Backup app-template
- ✅ Run tests: `npm test` - **All 42 tests passing**
- ✅ Fixed test infrastructure issues (server export, MongoDB connection, cleanup)
- ✅ Added pre-test cleanup to ensure tests can run repeatedly
- ✅ All tests now pass consistently on repeated runs
- ✅ Create test checklist for each model
- ✅ Create MIGRATION_STATUS.md (if needed)

## Phase 1: Foundation (4 hours)

- ✅ Add mcode logging to `mongo.mongo.js`
- ❌ Add MODULE_NAME to all 14 model files - no longer needed by latest mcode.\* package
- ✅ Replace console.log → mcode.done() in all files
- ✅ Replace console.error → mcode.exp() in all files
- ✅ Replace console.warn → mcode.warn() in all files
- ✅ Add disconnect() function to `mongo.mongo.js`
- ✅ Test: Server starts, logs appear correctly

## Phase 2: Documentation (3 hours)

- ✅ Add JSDoc to all functions (14+14++ files)
- ✅ Reorganize: CRUD first, special functions last
- ✅ Add inline comments for complex logic
- ✅ Test: Code still works

## Phase 3: ID System (6 hours) ⚠️ HIGH RISK

**⚠️ Create checkpoint tag BEFORE starting this phase**

### 3.1 Strategy: Hybrid Approach

- [ ] Keep both `id` (for API) AND `_id` (for MongoDB) initially
- [ ] Use `_id` as primary key, keep `id` as alias for compatibility

### 3.2 Step-by-Step Migration

#### Step 3.2.1: Add `_id` to Schemas (Non-Breaking)

- [ ] Add `_id` field with appropriate prefix to each schema
- [ ] Keep existing `id` field (for now)
- [ ] Add virtual to sync: `id = _id` on read
- [ ] Test: Verify both `id` and `_id` work

#### Step 3.2.2: Update Internal References to Use `_id`

- [ ] Update queries: `Account.findOne({ id: id })` → `Account.findOne({ _id: _id })`
- [ ] Update references: `'account.id'` → `'account._id'` (in user model)
- [ ] Return `_id` instead of `id` in responses
- [ ] Test: All CRUD operations work
- [ ] **After Testing**: Create phase tag if all tests pass

#### Step 3.2.3: Update Foreign Key References

- [ ] Change parameters: `user` → `user_id`, `account` → `account_id`
- [ ] Update all function signatures to use `{entity}_id` naming
- [ ] Update queries: `'account.id'` → `'account._id'`
- [ ] Test: All operations work with new parameter names

## Phase 4: Timestamps (4 hours) ⚠️ HIGH RISK

**⚠️ Create checkpoint tag BEFORE starting this phase**

### 4.1 Rename Timestamp Fields

**Mapping**:

- `date_created` → `created_at`
- `time` → `occurred_at` (events), `logged_at` (logs), `started_at` (logins)
- `date_sent` → `sent_at`
- `last_active` → `active_at`

**Files to Update**:

- [ ] `account.mongo.js`: `date_created` → `created_at`
- [ ] `event.mongo.js`: `time` → `occurred_at`
- [ ] `feedback.mongo.js`: `date_created` → `created_at`
- [ ] `invite.mongo.js`: `date_sent` → `sent_at`
- [ ] `key.mongo.js`: `date_created` → `created_at`
- [ ] `log.mongo.js`: `time` → `logged_at`
- [ ] `login.mongo.js`: `time` → `started_at`
- [ ] `token.mongo.js`: `issued_at`, `expires_at` (already correct - verify)
- [ ] `usage.mongo.js`: Keep `period_start`, `period_end` (already correct - verify)
- [ ] `user.mongo.js`: `date_created` → `created_at`, `last_active` → `active_at`

**Steps**:

- [ ] Update schema definitions
- [ ] Update function code that sets these fields
- [ ] Update queries that use these fields
- [ ] Update aggregation pipelines
- [ ] Test: All timestamp operations work
- [ ] **After Testing**: Create phase tag if all tests pass

## Phase 5: Type & State (4 hours)

- [ ] Add `type` field to all 14 schemas
- [ ] Add `state` field to all 14 schemas
- [ ] Set default `type: 'personal'` in account.create()
- [ ] Test: Type/state fields work

## Phase 6: User Settings (6 hours) ⚠️ COMPLEX

**⚠️ Create checkpoint tag BEFORE starting this phase**

### 6.1 Add Settings Schema to User Model

- [ ] Add `settings: { type: Object, required: true }` to `user.mongo.js` schema
- [ ] Update user.create() to initialize default settings
- [ ] Test: User creation with settings works

### 6.2 Add Settings Management Functions

- [ ] Add getSetting() function
- [ ] Add setSetting() function
- [ ] Add setSettings() function (bulk update)
- [ ] Add settings to config (default settings structure)
- [ ] Test: All settings functions work

### 6.3 Update Controllers/API

- [ ] Update user controller to use settings functions
- [ ] Add API endpoints for settings (if needed)
- [ ] Test: Settings API works
- [ ] **After Testing**: Create phase tag if all tests pass

## Phase 7: API Key Revoke (1 hour)

- [ ] Add revoke() function to `key.mongo.js`
- [ ] Fix bug: Use `key._id` and `key.account_id` (not undefined variables)
- [ ] Test: Revoke works correctly

## Phase 8: Additional Features (4 hours)

- [ ] Add `stack` field to `log.mongo.js` schema
- [ ] Update log.create() to accept stack parameter
- [ ] (Optional) Add address fields to `account.mongo.js`
- [ ] Add save() function to `token.mongo.js` (keep create())
- [ ] Test: New features work correctly

## Phase 9: Fix Bugs (2 hours)

- [ ] Fix `feedback.mongo.js` line 53: `id` → `_id` (undefined variable)
- [ ] Fix `feedback.mongo.js` line 101: `rating` → `_id` (incorrect $group syntax)
- [ ] Fix `key.mongo.js` revoke(): Use `key._id` and `key.account_id` (not undefined)
- [ ] Fix `usage.mongo.js` open(): `account_id` → `account` (undefined variable)
- [ ] Fix `pushtoken.mongo.js` delete(): Use correct field name in query
- [ ] Test: All bugs fixed, tests pass

## Phase 10: Testing (4 hours)

- [ ] Unit tests: All models
- [ ] Integration tests: Settings, revoke, tokens
- [ ] Manual: Server starts
- [ ] Manual: All API endpoints
- [ ] Manual: Logging works
- [ ] All tests passing ✅

## Phase 11: Cleanup (2 hours)

- [ ] Remove compatibility code (if desired)
- [ ] Update README
- [ ] Update API docs
- [ ] Final commit
- [ ] Merge to main

---

## Files to Modify (14=14++ total)

1. `server/model/mongo/account.mongo.js`
2. `server/model/mongo/email.mongo.js`
3. `server/model/mongo/event.mongo.js`
4. `server/model/mongo/feedback.mongo.js`
5. `server/model/mongo/invite.mongo.js`
6. `server/model/mongo/key.mongo.js`
7. `server/model/mongo/log.mongo.js`
8. `server/model/mongo/login.mongo.js`
9. `server/model/mongo/mongo.mongo.js`
10. `server/model/mongo/notification.mongo.js`
11. `server/model/mongo/pushtoken.mongo.js`
12. `server/model/mongo/token.mongo.js`
13. `server/model/mongo/usage.mongo.js`
14. `server/model/mongo/user.mongo.js`

- SQL Files
- CONTROLLERS

---

## Critical Path (Must Do)

1. Phase 0 → Phase 1 → Phase 3 → Phase 4 → Phase 6 → Phase 9 → Phase 10

Everything else can be done incrementally.

---

## Quick Commands

```bash
# Start migration
git checkout -b feature/0003--ladders-integration
git tag -a migration-baseline -m "Baseline before ladders integration"
git push origin migration-baseline

# After each phase
git add .
git commit -m "Phase X: [Description]"
npm test

# Create checkpoint tags before high-risk phases (3, 4, 6)
git tag -a migration-checkpoint-before-phase-X -m "Checkpoint before Phase X"
git push origin migration-checkpoint-before-phase-X

# After phase completes and tests pass
git tag -a migration-phase-X-complete -m "Phase X: [Description] complete"
git push origin migration-phase-X-complete

# If something breaks
git reset --hard HEAD~1  # Undo last commit
# Or
git checkout migration-baseline  # Start over
# Or
git checkout migration-checkpoint-before-phase-X  # Rollback to checkpoint
```

**Tagging Strategy**: See `TAGGING_STRATEGY.md` for complete guide.
