# Migration Checklist - Quick Reference

## Phase 0: Preparation
- [ ] Create branch: `feature/ladders-integration`
- [ ] Tag: `git tag pre-ladders-integration`
- [ ] Backup app-template
- [ ] Run tests: `npm test` (all passing)
- [ ] Create MIGRATION_STATUS.md

## Phase 1: Foundation (4 hours)
- [ ] Add mcode logging to mongo.js
- [ ] Add MODULE_NAME to all 14 model files
- [ ] Replace console.log → mcode.done() in all files
- [ ] Replace console.error → mcode.exp() in all files
- [ ] Add disconnect() function to mongo.js
- [ ] Test: Server starts, logs appear

## Phase 2: Documentation (3 hours)
- [ ] Add JSDoc to all functions (14 files)
- [ ] Reorganize: CRUD first, special functions last
- [ ] Add inline comments for complex logic
- [ ] Test: Code still works

## Phase 3: ID System (6 hours) ⚠️
- [ ] Add `_id` field to all schemas with prefixes
- [ ] Update all queries: `{ id: id }` → `{ _id: _id }`
- [ ] Update foreign keys: `'account.id'` → `'account._id'`
- [ ] Change parameters: `user` → `user_id`, `account` → `account_id`
- [ ] Test: All CRUD operations work

## Phase 4: Timestamps (4 hours) ⚠️
- [ ] account.js: `date_created` → `created_at`
- [ ] event.js: `time` → `occurred_at`
- [ ] feedback.js: `date_created` → `created_at`
- [ ] invite.js: `date_sent` → `sent_at`
- [ ] key.js: `date_created` → `created_at`
- [ ] log.js: `time` → `logged_at`
- [ ] login.js: `time` → `started_at`
- [ ] user.js: `date_created` → `created_at`, `last_active` → `active_at`
- [ ] Test: All date operations work

## Phase 5: Type & State (4 hours)
- [ ] Add `type` field to all 14 schemas
- [ ] Add `state` field to all 14 schemas
- [ ] Set default `type: 'personal'` in account.create()
- [ ] Test: Type/state fields work

## Phase 6: User Settings (6 hours) ⚠️
- [ ] Add `settings: Object` to user schema
- [ ] Initialize settings in user.create()
- [ ] Add getSetting() function
- [ ] Add setSetting() function
- [ ] Add setSettings() function
- [ ] Add settings to config
- [ ] Test: Settings system works

## Phase 7: API Key Revoke (1 hour)
- [ ] Add revoke() function to key.js
- [ ] Fix bug: Use `key._id` and `key.account_id`
- [ ] Test: Revoke works

## Phase 8: Additional Features (4 hours)
- [ ] Add `stack` field to log.js
- [ ] Update log.create() to accept stack
- [ ] (Optional) Add address fields to account.js
- [ ] Add save() function to token.js (keep create())
- [ ] Test: New features work

## Phase 9: Fix Bugs (2 hours)
- [ ] Fix feedback.js line 53: `id` → `_id`
- [ ] Fix feedback.js line 101: `rating` → `_id`
- [ ] Fix key.js revoke(): Use `key._id`
- [ ] Fix usage.js open(): `account_id` → `account`
- [ ] Fix pushtoken.js delete(): Use `_id` correctly
- [ ] Test: All bugs fixed

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

## Files to Modify (14 total)

1. `server/model/mongo/account.js`
2. `server/model/mongo/email.js`
3. `server/model/mongo/event.js`
4. `server/model/mongo/feedback.js`
5. `server/model/mongo/invite.js`
6. `server/model/mongo/key.js`
7. `server/model/mongo/log.js`
8. `server/model/mongo/login.js`
9. `server/model/mongo/mongo.js`
10. `server/model/mongo/notification.js` (keep as-is)
11. `server/model/mongo/pushtoken.js`
12. `server/model/mongo/token.js`
13. `server/model/mongo/usage.js`
14. `server/model/mongo/user.js`

---

## Critical Path (Must Do)

1. Phase 0 → Phase 1 → Phase 3 → Phase 4 → Phase 6 → Phase 9 → Phase 10

Everything else can be done incrementally.

---

## Quick Commands

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
