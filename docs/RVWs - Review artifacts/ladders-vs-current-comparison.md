# Comparison Report: ladders vs gravity-current

## Directory: server/model/mongo/_ (ladders) vs server/model/mongo/_ (gravity-current)

### Summary

- **Total files in ladders**: 13 model files (using `.model.js` extension)
- **Total files in gravity-current**: 14 files (using `.js` extension in `mongo/` subdirectory)
- **New file in gravity-current**: notification.js (does not exist in ladders)
- **Files with significant differences**: All files have architectural differences

---

## ARCHITECTURAL DIFFERENCES

### 1. ID System

**Ladders**: Uses MongoDB's native `_id` field with custom UUID generation

- `_id: {type: String, default: () => utility.unique_id('prefix')}`
- Uses prefixes like 'acct', 'user', 'evnt', 'fbck', etc.

**Gravity-Current**: Uses separate `id` field with UUID v4

- `id: {type: String, required: true, unique: true}`
- Generated using `uuidv4()` from 'uuid' package

### 2. Field Naming Conventions

**Ladders**: Uses underscore naming with `_at` suffix for timestamps

- `created_at`, `started_at`, `logged_at`, `occurred_at`, `sent_at`, `active_at`

**Gravity-Current**: Uses mixed naming

- `date_created`, `time`, `date_sent`

### 3. Code Style & Logging

**Ladders**:

- Uses MicroCODE logging (`mcode.done()`, `mcode.exp()`, `mcode.success()`)
- Has `MODULE_NAME` constant for logging
- More verbose comments with dates and author initials
- Exports both `schema` and `model`

**Gravity-Current**:

- Uses standard `console.log()`
- Minimal comments
- Exports `schema` only

### 4. MongoDB Connection

**Ladders**:

- Connection string: `mongodb://...@...:27017/...?authSource=admin`
- Has `disconnect()` function
- Uses `mcode` logging

**Gravity-Current**:

- Connection string: `mongodb+srv://...@.../...`
- No disconnect function
- Uses standard console logging

---

## DETAILED FILE COMPARISONS

### 1. account.model.js (ladders) vs account.js (gravity-current)

#### Schema Differences:

**Ladders** has additional fields:

- `_id` (custom UUID) vs `id` (UUID v4)
- `domain`, `logo`, `address`, `address_`, `locality`, `region`, `country`, `postal` (address fields)
- `type` (account type)
- `created_at` vs `date_created`

**Gravity-Current** has:

- `id` field (separate from MongoDB `_id`)
- Simpler schema

#### Function Differences:

1. **create()**:

   - **Ladders**: Sets `type: 'personal'` by default
   - **Gravity-Current**: No type field

2. **get()**:

   - **Ladders**: `exports.get = async function (_id)` - single parameter
   - **Gravity-Current**: `exports.get = async function({ id, stripeCustomer })` - object with optional stripeCustomer
   - **Ladders**: Queries by `_id`, uses `'account._id'` in user lookup
   - **Gravity-Current**: Queries by `id`, uses `'account.id'` in user lookup

3. **update()**:

   - **Ladders**: `exports.update = async function ({_id, data, stripe_customer_id})` - supports stripe_customer_id lookup
   - **Gravity-Current**: `exports.update = async function({ id, data })` - simpler signature
   - **Ladders**: Returns updated document with `{new: true}`

4. **subscription()**:

   - **Ladders**: More robust error handling, checks `account?.plan?.includes('free')`
   - **Ladders**: Returns `status: 'inactive'` by default if account doesn't exist
   - **Gravity-Current**: Returns `false` if account doesn't exist
   - **Ladders**: Uses `stripeModel.subscription()` (separate model)
   - **Gravity-Current**: Uses `stripe.subscription()` (direct require)

5. **users()**:
   - **Ladders**: Uses `'account._id'` and returns `{_id: 1}`
   - **Gravity-Current**: Uses `'account.id'` and returns `{id: 1}`

---

### 2. email.model.js (ladders) vs email.js (gravity-current)

#### Schema Differences:

**Ladders**:

- `_id` (custom UUID with 'emtp' prefix)
- `index: {type: Number, required: true, unique: true}` - numeric index
- No `locale` field

**Gravity-Current**:

- No `id` or `_id` field in schema (uses MongoDB default)
- `locale: {type: String, required: true}` - multi-language support
- No `index` field

#### Function Differences:

1. **create()**:

   - **Ladders**: Requires `index` parameter
   - **Gravity-Current**: Requires `locale` parameter (defaults to 'en')

2. **get()**:
   - **Ladders**: Queries by `name` only
   - **Gravity-Current**: Queries by `name` AND `locale`

**Key Difference**: Ladders uses numeric indexing, Gravity-Current uses locale-based multi-language support

---

### 3. event.model.js (ladders) vs event.js (gravity-current)

#### Schema Differences:

**Ladders**:

- `_id` (custom UUID with 'evnt' prefix)
- `occurred_at` vs `time`
- Uses `account_id` and `user_id` directly

**Gravity-Current**:

- `id` field (UUID v4)
- `time` field
- Same structure otherwise

#### Function Differences:

1. **create()**:
   - **Ladders**: `exports.create = async function ({data, user_id, account_id})` - separate parameters
   - **Gravity-Current**: `exports.create = async function({ data, user, account })` - uses `user` and `account` names
   - **Ladders**: Sets `occurred_at: new Date()`
   - **Gravity-Current**: Sets `time: new Date()`

---

### 4. feedback.model.js (ladders) vs feedback.js (gravity-current)

#### Schema Differences:

**Ladders**:

- `_id` (custom UUID with 'fbck' prefix)
- `comment: {type: String, required: true}` - required
- `created_at` vs `date_created`

**Gravity-Current**:

- `id` field (UUID v4)
- `comment: {type: String}` - optional
- `date_created` field

#### Function Differences:

1. **get()**:

   - **Ladders**: Has bug - uses undefined `id` variable in `$project` (line 53)
   - **Ladders**: Uses `_id` in lookup and return
   - **Gravity-Current**: Uses `id` in lookup and return
   - **Ladders**: Looks up by `foreignField: 'id'` (inconsistent with `_id` usage)
   - **Gravity-Current**: Looks up by `foreignField: 'id'` (consistent)

2. **metrics()**:
   - **Ladders**: Has bug - uses `rating: '$rating'` instead of `_id: '$rating'` in `$group`
   - **Gravity-Current**: Correctly uses `_id: '$rating'`

**Bugs in Ladders**:

- Line 53: `id: id || null` should be `_id: _id || null`
- Line 101: `rating: '$rating'` should be `_id: '$rating'`

---

### 5. invite.model.js (ladders) vs invite.js (gravity-current)

#### Schema Differences:

**Ladders**:

- `_id` (custom UUID with 'invt' prefix)
- `sent_at` vs `date_sent`

**Gravity-Current**:

- `id` field (UUID v4, generated with `randomstring.generate(16)`)
- `date_sent` field

#### Function Differences:

1. **create()**:

   - **Ladders**: Uses `utility.unique_id('invt')` for `_id`
   - **Gravity-Current**: Uses `randomstring.generate(16)` for `id`

2. **get()**:

   - **Ladders**: Uses `_id` in queries
   - **Gravity-Current**: Uses `id` in queries

3. **update()**:
   - **Ladders**: Uses `_id` in update
   - **Gravity-Current**: Uses `id` in update

---

### 6. key.model.js (ladders) vs key.js (gravity-current)

#### Schema Differences:

**Ladders**:

- `_id` (custom UUID with 'apik' prefix)
- `created_at` vs `date_created`

**Gravity-Current**:

- `id` field (UUID v4)
- `date_created` field

#### Function Differences:

1. **create()**:

   - Both similar, but use different ID systems

2. **get()**:

   - **Ladders**: Uses `_id` in queries
   - **Gravity-Current**: Uses `id` in queries

3. **revoke()** (Ladders only):
   - **Ladders**: Has `revoke()` function that sets `active: false`
   - **Gravity-Current**: No `revoke()` function (uses `update()` instead)
   - **Ladders Bug**: Line 128 uses undefined `_id` and `account_id` variables

**Bug in Ladders**: Line 128 should use `key._id` and `key.account_id`

---

### 7. log.model.js (ladders) vs log.js (gravity-current)

#### Schema Differences:

**Ladders**:

- `_id` (custom UUID with 'logn' prefix)
- `stack: {type: String}` - additional field for stack traces
- `logged_at` vs `time`
- Uses `user_id` and `account_id` directly

**Gravity-Current**:

- `id` field (UUID v4)
- No `stack` field
- `time` field
- Uses `user_id` and `account_id` directly

#### Function Differences:

1. **create()**:
   - **Ladders**: `exports.create = async function ({message, stack, body, req, user_id, account_id})` - includes `stack` parameter
   - **Gravity-Current**: `exports.create = async function({ message, body, req, user, account })` - uses `user` and `account` names
   - **Ladders**: Extracts `req?.user_id` or `user_id`
   - **Gravity-Current**: Extracts `req?.user` or `user`

---

### 8. login.model.js (ladders) vs login.js (gravity-current)

#### Schema Differences:

**Ladders**:

- `_id` (custom UUID with 'logi' prefix)
- `started_at` vs `time`

**Gravity-Current**:

- `id` field (UUID v4)
- `time` field

#### Function Differences:

1. **create()**:

   - **Ladders**: Uses manual string parsing for user agent (old method)
   - **Gravity-Current**: Uses `UAParser` library (modern, more accurate)
   - **Gravity-Current**: Handles IPv6 to IPv4 conversion
   - **Ladders**: No IPv6 handling

2. **verify()**:
   - **Ladders**: Uses `login._id` and `id: {$ne: login._id}` in query
   - **Ladders**: Returns `started_at` as formatted string in `riskRecord` object
   - **Gravity-Current**: Uses `current.id` and `id: { $ne: current.id }` in query
   - **Gravity-Current**: Returns `time` as formatted string directly

**Key Difference**: Gravity-Current has better user agent parsing and IP handling

---

### 9. mongo.model.js (ladders) vs mongo.js (gravity-current)

#### Major Differences:

1. **Connection String**:

   - **Ladders**: `mongodb://...:27017/...?authSource=admin` (standard connection)
   - **Gravity-Current**: `mongodb+srv://.../...` (MongoDB Atlas connection)

2. **Functions**:

   - **Ladders**: Has both `connect()` and `disconnect()` functions
   - **Gravity-Current**: Only has `connect()` function

3. **Logging**:

   - **Ladders**: Uses `mcode.done()`, `mcode.exp()`, `mcode.success()`
   - **Gravity-Current**: Uses `console.log()` and `console.error()`

4. **Error Handling**:
   - **Ladders**: Catches exceptions and logs with `mcode.exp()`
   - **Gravity-Current**: Catches exceptions and logs with `console.error()`

---

### 10. pushtoken.model.js (ladders) vs pushtoken.js (gravity-current)

#### Function Differences:

1. **create()**:

   - **Ladders**: Uses `_id: user_id` in query
   - **Gravity-Current**: Uses `id: user` in query

2. **get()**:

   - **Ladders**: Uses `_id: user_id` in query
   - **Gravity-Current**: Uses `id: user` in query

3. **delete()**:
   - **Ladders**: Uses `findOneAndRemove()` with `{user_id: user_id}` (incorrect - should be `_id`)
   - **Gravity-Current**: Uses `findOneAndDelete()` with `{user: user}` (incorrect - should be `id`)
   - **Both have bugs**: Should use the correct ID field name

---

### 11. token.model.js (ladders) vs token.js (gravity-current)

#### Schema Differences:

**Ladders**:

- `_id` (custom UUID with 'stok' prefix)
- `account_id: {type: String}` - additional field
- No expiration tracking (`issued_at`, `expires_at`, `active`)

**Gravity-Current**:

- `id` field (UUID v4)
- `issued_at: {type: Date, required: true}`
- `expires_at: {type: Date, required: true}`
- `active: {type: Boolean, required: true, default: true}`
- No `account_id` field

#### Function Differences:

1. **save() vs create()**:

   - **Ladders**: `exports.save()` - can create or update existing token
   - **Gravity-Current**: `exports.create()` - always creates new token
   - **Ladders**: Checks for existing token and updates if found
   - **Gravity-Current**: No update logic, always creates new

2. **get()**:

   - **Ladders**: Has `skipDecryption` parameter, automatically decrypts tokens
   - **Gravity-Current**: Has `active` parameter, no automatic decryption
   - **Ladders**: Returns decrypted tokens by default
   - **Gravity-Current**: Returns encrypted tokens as-is

3. **update()**:

   - **Ladders**: No `update()` function (uses `save()` for updates)
   - **Gravity-Current**: Has `update()` function for bulk updates

4. **verify()**:
   - **Ladders**: `exports.verify = async function ({provider, user_id})`
   - **Gravity-Current**: `exports.verify = async function({ id, provider, user })` - includes optional `id`

**Key Difference**: Ladders uses update-or-create pattern, Gravity-Current uses create-only with expiration tracking

---

### 12. usage.model.js (ladders) vs usage.js (gravity-current)

#### Schema Differences:

**Ladders**:

- `_id` (custom UUID with 'used' prefix)

**Gravity-Current**:

- `id` field (UUID v4)

#### Function Differences:

1. **open()**:

   - **Ladders**: Has bug - uses undefined `account_id` variable (line 36), should use `account` parameter
   - **Gravity-Current**: Correctly uses `account` parameter

2. **get()**:

   - **Ladders**: Uses `account_id` parameter
   - **Gravity-Current**: Uses `account` parameter
   - **Ladders**: Looks up by `foreignField: 'id'` (inconsistent - should be `_id`)
   - **Gravity-Current**: Looks up by `foreignField: 'id'` (consistent)

3. **get.total()**:

   - **Ladders**: Uses `account_id` parameter
   - **Gravity-Current**: Uses `account` parameter

4. **increment()**:

   - **Ladders**: Uses `account_id` parameter
   - **Gravity-Current**: Uses `account` parameter

5. **close()**:
   - **Ladders**: Uses `_id` in update
   - **Gravity-Current**: Uses `id` in update

**Bug in Ladders**: Line 36 uses `account_id` instead of `account` parameter

---

### 13. user.model.js (ladders) vs user.js (gravity-current)

#### Schema Differences:

**Ladders** has extensive additional fields:

- `_id` (custom UUID with 'user' prefix)
- `settings: {type: Object, required: true}` - user settings object
- `tfa_enabled`, `tfa_secret`, `tfa_backup_code` (vs `2fa_*` in gravity-current)
- `default_account` (vs `default_account_id`)
- `linkedin_ident`, `facebook_ident`, `twitter_ident`, `github_ident` (vs `facebook_id`, `twitter_id`)
- `created_at`, `active_at` (vs `date_created`, `last_active`)
- `permission`, `onboarded`, `has_password` as direct fields (vs computed in gravity-current)
- `account_id` as direct field

**Gravity-Current**:

- `id` field (UUID v4)
- `2fa_enabled`, `2fa_secret`, `2fa_backup_code`
- `default_account_id`
- `facebook_id`, `twitter_id`
- `date_created`, `last_active`
- `support_enabled`, `dark_mode` (not in ladders)

#### Function Differences:

1. **create()**:

   - **Ladders**: Initializes `settings` from config, sets `permission: 'user'`, `onboarded: false`, `has_password: false`
   - **Ladders**: Uses `default_account_id`
   - **Gravity-Current**: Uses `default_account_id`, sets `support_enabled: false`, `2fa_enabled: false`

2. **get()**:

   - **Ladders**: Uses `_id`, `'account._id'` in queries
   - **Gravity-Current**: Uses `id`, `'account.id'` in queries
   - **Ladders**: Uses `${social.provider}_id` for social IDs
   - **Gravity-Current**: Uses `${social.provider}_id` for social IDs (same pattern)

3. **accounts() vs account()**:

   - **Ladders**: `exports.accounts()` - returns array of account objects
   - **Gravity-Current**: `exports.account()` - same functionality
   - **Ladders**: Uses `_id` throughout
   - **Gravity-Current**: Uses `id` throughout

4. **accounts.add() vs account.add()**:

   - **Ladders**: `exports.accounts.add()` - uses `_id` and `account_id`
   - **Gravity-Current**: `exports.account.add()` - uses `id` and `account`

5. **accounts.delete() vs account.delete()**:

   - **Ladders**: `exports.accounts.delete()` - uses `_id` and `account_id`
   - **Gravity-Current**: `exports.account.delete()` - uses `id` and `account`

6. **password.verify()**:

   - **Ladders**: Uses `_id` and `'account._id'`
   - **Gravity-Current**: Uses `id` and `'account.id'`

7. **tfa vs 2fa**:

   - **Ladders**: Uses `tfa` prefix (two-factor authentication)
   - **Gravity-Current**: Uses `2fa` prefix (two-factor authentication)
   - Same functionality, different naming

8. **New Functions in Ladders**:
   - `getSetting()` - retrieves hierarchical settings
   - `setSetting()` - sets hierarchical settings
   - `setSettings()` - sets all settings at once

**Key Difference**: Ladders has comprehensive user settings management system

---

### 14. notification.js (gravity-current only)

**Status**: 🆕 **Does not exist in ladders**

This is a complete notification preferences system that ladders does not have.

---

## SUMMARY OF KEY DIFFERENCES

### 🔴 Critical Architectural Differences:

1. **ID System**:

   - **Ladders**: Uses MongoDB `_id` with custom UUID prefixes
   - **Gravity-Current**: Uses separate `id` field with UUID v4

2. **Field Naming**:

   - **Ladders**: `created_at`, `started_at`, `logged_at`, `occurred_at`, `sent_at`
   - **Gravity-Current**: `date_created`, `time`, `date_sent`

3. **User Settings**:

   - **Ladders**: Has comprehensive `settings` object with hierarchical get/set functions
   - **Gravity-Current**: No settings management system

4. **Token Management**:

   - **Ladders**: Update-or-create pattern, automatic decryption
   - **Gravity-Current**: Create-only pattern, expiration tracking, no auto-decryption

5. **Email System**:
   - **Ladders**: Numeric index-based
   - **Gravity-Current**: Locale-based multi-language

### 🐛 Bugs Found in Ladders:

1. **feedback.model.js** (Line 53): Uses undefined `id` variable
2. **feedback.model.js** (Line 101): Incorrect `$group` syntax
3. **key.model.js** (Line 128): Uses undefined `_id` and `account_id` variables
4. **usage.model.js** (Line 36): Uses undefined `account_id` variable
5. **pushtoken.model.js** (Line 32): Incorrect field name in query

### ✨ Features in Gravity-Current Not in Ladders:

1. **notification.js** - Complete notification preferences system
2. **Token expiration tracking** - `issued_at`, `expires_at`, `active` fields
3. **Email localization** - Multi-language email templates
4. **Better user agent parsing** - Uses UAParser library
5. **IPv6 handling** - Converts IPv6-mapped IPv4 addresses

### ✨ Features in Ladders Not in Gravity-Current:

1. **User settings management** - Hierarchical settings with `getSetting()`, `setSetting()`, `setSettings()`
2. **Account address fields** - Full address support (domain, logo, address, locality, region, country, postal)
3. **Account type field** - Supports different account types
4. **Stack trace logging** - `stack` field in log model
5. **Token update capability** - Can update existing tokens
6. **Key revoke function** - Dedicated function to revoke API keys
7. **MongoDB disconnect** - Proper connection cleanup
8. **MicroCODE logging** - Structured logging system

### 📦 Dependency Differences:

1. **Ladders**: Uses `bcrypt` (native)
2. **Gravity-Current**: Uses `bcryptjs` (pure JS)
3. **Gravity-Current**: Uses `ua-parser-js` for user agent parsing
4. **Gravity-Current**: Uses `lodash.escape` for XSS protection
5. **Ladders**: Uses `mcode-log` for logging

---

## MIGRATION CONSIDERATIONS

### If Migrating from Ladders to Gravity-Current:

1. **ID System Migration**: Need to map `_id` to `id` field
2. **Field Name Changes**: Update all timestamp field names
3. **User Settings**: Need to implement alternative (or migrate to notification system)
4. **Token System**: Change from update-or-create to create-only pattern
5. **Email System**: Migrate from index-based to locale-based
6. **Account Fields**: Remove address fields or migrate to separate model
7. **Fix Bugs**: Address the bugs found in ladders code

### If Migrating from Gravity-Current to Ladders:

1. **ID System Migration**: Need to map `id` to `_id` field with proper prefixes
2. **Field Name Changes**: Update all timestamp field names
3. **Notification System**: Migrate to user settings or implement separately
4. **Token System**: Change from create-only to update-or-create pattern
5. **Email System**: Migrate from locale-based to index-based
6. **Add Features**: Implement user settings, address fields, etc.
7. **Fix Bugs**: Address the bugs that exist in ladders code

---

## RECOMMENDATIONS

1. **Standardize ID System**: Choose one approach (`_id` vs `id`) and use consistently
2. **Fix Bugs**: Address all identified bugs in ladders code
3. **Standardize Field Names**: Use consistent naming convention for timestamps
4. **Merge Features**: Consider combining best features from both:
   - User settings from ladders
   - Notification system from gravity-current
   - Token expiration from gravity-current
   - Better error handling from ladders
5. **Improve Code Quality**:
   - Add proper error handling
   - Standardize logging approach
   - Add input validation
   - Fix inconsistent field usage
