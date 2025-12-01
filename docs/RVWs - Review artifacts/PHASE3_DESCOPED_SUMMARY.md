# Phase 3: Descoped ID Generation Migration - Summary

## Overview

Phase 3 has been **descoped** from a complex `_id` migration to a simpler ID generation update.

## Key Changes

### What Changed

1. **MongoDB `_id`**: Keep invisible, let MongoDB handle it automatically (like gravity-current)

   - **No changes needed** - MongoDB continues to manage `_id` in the background
   - We don't touch `_id` at all

2. **App `id` Field**: Replace `uuidv4()` with `utility.unique_id('prefix')`

   - **Change**: Update all ID generation from `uuidv4()` to `utility.unique_id('prefix')`
   - **Benefit**: Human-readable entity prefixes, sortable by timestamp, avoids conflicts with `_id`

3. **Foreign Keys**: Continue using `{entity}_id` naming
   - **No changes needed** - Already correct throughout codebase
   - Pattern: `user_id`, `account_id`, `event_id`, etc. (not `userId`, `accountId`)

### What Was Removed

- ❌ Adding `_id` field to schemas
- ❌ Creating virtuals to sync `id = _id`
- ❌ Updating queries to use `_id` instead of `id`
- ❌ Changing parameter names from `user` to `user_id` (already done)
- ❌ Complex migration strategy

### What Remains

- ✅ Replace `uuidv4()` with `utility.unique_id('prefix')` in:
  - MongoDB model schemas (add default)
  - MongoDB model create functions
  - SQL model create functions
  - Controller files (if generating IDs)

## Impact Assessment

### Reduced Complexity

- **Before**: 6 hours, HIGH RISK, complex migration with `_id` handling
- **After**: 4 hours, MEDIUM RISK, simple find-and-replace operation

### Reduced Risk

- **Before**: Risk of breaking queries, need to handle both `id` and `_id`, complex testing
- **After**: Lower risk - just changing ID generation method, no query changes needed

### Files Affected

**MongoDB Models** (14 server + 7 admin = 21 files):

- Update schema `id` field to include `default: () => utility.unique_id('prefix')`
- Remove all `uuidv4()` calls in create functions - rely on schema defaults only

**SQL Models** (14 server + 7 admin = 21 files):

- Replace `uuidv4()` with `utility.unique_id('prefix')` in create functions

**Controllers** (if any generate IDs directly):

- Replace `uuidv4()` with `utility.unique_id('prefix')`

**Total**: ~42-45 files (much simpler than original plan)

## Entity Prefixes

See `PHASE3_ID_PREFIXES.md` for complete mapping. Examples:

- `account` → `'acct'` → `acct_l8k2j3m4n5o6p7q8r9s0t1u2v3w4x5y6z7`
- `user` → `'user'` → `user_l8k2j3m4n5o6p7q8r9s0t1u2v3w4x5y6z7`
- `key` (API key) → `'apik'` → `apik_l8k2j3m4n5o6p7q8r9s0t1u2v3w4x5y6z7`
- `usage` → `'used'` → `used_l8k2j3m4n5o6p7q8r9s0t1u2v3w4x5y6z7`
- `email` → `'mail'` → `mail_l8k2j3m4n5o6p7q8r9s0t1u2v3w4x5y6z7`

## Testing Strategy

1. **Schema Tests**: Verify schemas compile with new defaults
2. **Create Tests**: Verify all create operations generate correct ID format
3. **Query Tests**: Verify existing queries still work (no changes to query logic)
4. **Format Tests**: Verify ID format matches pattern: `prefix_timestamp+random`

## Timeline Impact

- **Original Estimate**: 6 hours
- **New Estimate**: 4 hours
- **Time Saved**: 2 hours
- **Risk Reduction**: HIGH → MEDIUM

## Next Steps

1. Review this summary and prefix mapping
2. Confirm entity prefixes are acceptable
3. Proceed with implementation when ready
4. Test thoroughly before moving to Phase 4
