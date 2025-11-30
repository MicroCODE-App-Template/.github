# Comparison Report: gravity-purchased vs gravity-current
## Directory: server/model/mongo/*

### Summary
- **Total files in gravity-purchased**: 13
- **Total files in gravity-current**: 14
- **New file in gravity-current**: notification.js
- **Files with changes**: 7 files

---

## DETAILED CHANGES

### 1. account.js

#### Changes:
1. **Line 3**: Removed unused import
   - **Purchased**: `const utility = require('../helper/utility')`
   - **Current**: Removed (not used)

2. **Line 50**: Enhanced `get()` function signature
   - **Purchased**: `exports.get = async function(id)`
   - **Current**: `exports.get = async function({ id, stripeCustomer })`
   - **Change**: Now accepts object parameter with optional `stripeCustomer` for lookup by Stripe customer ID

3. **Lines 53-58**: Updated query logic
   - **Purchased**: `Account.findOne({ id: id })`
   - **Current**: Uses spread operator to conditionally include `id` or `stripe_customer_id` in query

4. **Lines 88-94**: Improved error handling in `subscription()`
   - **Purchased**: `utility.assert(accountData, \`Account doesn't exist\`)`
   - **Current**:
     ```javascript
     if (!accountData)
       return false;
     ```
   - **Change**: Returns `false` instead of throwing assertion error

5. **Line 145**: Updated deprecated method
   - **Purchased**: `Account.findOneAndRemove({ id: id })`
   - **Current**: `Account.findOneAndDelete({ id: id })`
   - **Change**: Replaced deprecated `findOneAndRemove` with `findOneAndDelete`

---

### 2. email.js

#### Changes:
1. **Schema Changes (Lines 7-15)**:
   - **Purchased**: Has `id: { type: Number, required: true, unique: true }` field
   - **Current**: Removed `id` field, added `locale: { type: String, required: true }` field
   - **Change**: Schema now supports localization instead of numeric ID

2. **Line 25**: Updated `create()` function
   - **Purchased**: `exports.create = async function({ id, name, subject, preheader, body, button })`
   - **Current**: `exports.create = async function({ id, name, subject, preheader, body, button, locale })`
   - **Change**: Added `locale` parameter with default value `'en'` (line 36)

3. **Line 36**: Added locale handling
   - **Current**: `locale: locale || 'en'`

4. **Line 51**: Updated `get()` function
   - **Purchased**: `exports.get = async function({ name })`
   - **Current**: `exports.get = async function({ name, locale })`
   - **Change**: Now queries by both `name` and `locale` (defaults to 'en')

5. **Line 53**: Updated query
   - **Purchased**: `Email.findOne({ name: name })`
   - **Current**: `Email.findOne({ name: name, locale: locale || 'en' })`

6. **Line 75**: Updated deprecated method
   - **Purchased**: `Email.findOneAndRemove({ name: name })`
   - **Current**: `Email.findOneAndDelete({ name: name })`
   - **Change**: Replaced deprecated `findOneAndRemove` with `findOneAndDelete`

---

### 3. event.js
**Status**: ✅ No changes - Files are identical

---

### 4. feedback.js
**Status**: ✅ No changes - Files are identical

---

### 5. invite.js
**Status**: ✅ No changes - Only difference is trailing newline in current version

---

### 6. key.js
**Status**: ✅ No changes - Files are identical

---

### 7. log.js
**Status**: ✅ No changes - Files are identical

---

### 8. login.js

#### Changes:
1. **Line 2**: Changed user agent parsing library
   - **Purchased**: `const randomstring = require('randomstring');` (not used in this file)
   - **Current**: `const UAParser = require('ua-parser-js');`
   - **Change**: Uses proper user agent parsing library instead of manual string manipulation

2. **Lines 27-48**: Completely rewritten user agent parsing logic
   - **Purchased**: Manual string parsing:
     ```javascript
     device = ua.substring(ua.indexOf('(')+1, ua.indexOf(')')).replace(/_/g, '.');
     uarr = ua.split(' ');
     browser = uarr[uarr.length-1];
     ```
   - **Current**: Uses UAParser library:
     ```javascript
     const parser = new UAParser();
     const ua = parser.setUA(req.headers['user-agent']).getResult();
     device = (ua.device.vendor && ua.device.model) ?
       `${ua.device.vendor} ${ua.device.model}` : 'Desktop'
     browser = `${ua.browser.name}`
     ```
   - **Change**: More reliable and accurate device/browser detection

3. **Lines 31-36**: Improved IP address handling
   - **Current**: Added IPv6 to IPv4 conversion:
     ```javascript
     if (ip.startsWith('::ffff:'))
       ip = ip.split(':').pop();
     ```
   - **Change**: Handles IPv6-mapped IPv4 addresses correctly

---

### 9. mongo.js
**Status**: ✅ No changes - Files are identical

---

### 10. pushtoken.js

#### Changes:
1. **Line 33**: Updated deprecated method
   - **Purchased**: `User.findOneAndRemove({ user: user },{ $pull: { push_token: token }})`
   - **Current**: `User.findOneAndDelete({ user: user },{ $pull: { push_token: token }})`
   - **Change**: Replaced deprecated `findOneAndRemove` with `findOneAndDelete`
   - **Note**: The query parameter `{ user: user }` appears incorrect in both versions (should likely be `{ id: user }`)

---

### 11. token.js

#### Major Changes:
1. **Line 2**: Added new imports
   - **Current**:
     ```javascript
     const config = require('config');
     const utility = require('../helper/utility');
     ```
   - **Change**: Added config and utility imports for token management

2. **Schema Changes (Lines 10-22)**:
   - **Purchased**: Basic schema with `id`, `provider`, `jwt`, `access`, `refresh`, `user_id`
   - **Current**: Enhanced schema with additional fields:
     - `issued_at: { type: Date, required: true }`
     - `expires_at: { type: Date, required: true }`
     - `active: { type: Boolean, required: true, default: true }`
   - **Change**: Added token expiration and active status tracking

3. **Line 31**: Function renamed and restructured
   - **Purchased**: `exports.save = async function({ provider, data, user })`
   - **Current**: `exports.create = async function({ id, provider, data, user })`
   - **Change**:
     - Renamed from `save` to `create`
     - Added `id` parameter (optional, generates UUID if not provided)
     - Removed logic for updating existing tokens (now always creates new)

4. **Lines 40-56**: Complete rewrite of token creation
   - **Purchased**: Checked for existing token and updated if found, created new if not
   - **Current**: Always creates new token with:
     - `issued_at` timestamp
     - `expires_at` calculated from config
     - `active: true` by default
   - **Change**: No longer updates existing tokens, always creates new ones

5. **Line 65**: Updated `get()` function
   - **Purchased**: `exports.get = async function({ id, provider, user, skipDecryption })`
   - **Current**: `exports.get = async function({ id, provider, user, active = true })`
   - **Change**:
     - Removed `skipDecryption` parameter
     - Added `active` parameter (defaults to true)
     - Removed decryption logic (tokens are no longer automatically decrypted)

6. **Lines 80-92**: New `update()` function
   - **Current**: Added new function to update token(s) by ID(s)
   - **Change**: Supports updating multiple tokens at once

7. **Line 99**: Updated `verify()` function
   - **Purchased**: `exports.verify = async function({ provider, user })`
   - **Current**: `exports.verify = async function({ id, provider, user })`
   - **Change**: Added optional `id` parameter for verification

8. **Removed**: Automatic token decryption in `get()` function
   - **Purchased**: Automatically decrypted `access` and `refresh` tokens
   - **Current**: Returns encrypted tokens as-is

---

### 12. usage.js
**Status**: ✅ No changes - Files are identical

---

### 13. user.js

#### Changes:
1. **Line 1**: Changed bcrypt library
   - **Purchased**: `const bcrypt = require('bcrypt');`
   - **Current**: `const bcrypt = require('bcryptjs');`
   - **Change**: Switched from `bcrypt` to `bcryptjs` (pure JavaScript implementation)

2. **Line 5**: Added new import
   - **Current**: `const escape = require('lodash.escape');`
   - **Change**: Added XSS protection for user input

3. **Line 47**: Added input sanitization
   - **Purchased**: `name: user.name,`
   - **Current**: `name: escape(user.name),`
   - **Change**: Escapes user name to prevent XSS attacks

4. **Lines 319-320**: Added input sanitization in `update()`
   - **Current**:
     ```javascript
     if (data.name)
       data.name = escape(data.name);
     ```
   - **Change**: Escapes user name when updating profile

---

### 14. notification.js (NEW FILE)

**Status**: 🆕 **New file in gravity-current** - Does not exist in gravity-purchased

#### File Overview:
- **Purpose**: Manages user notification preferences per account
- **Schema**: Stores notification name, active status, account_id, and user_id
- **Functions**:
  1. `create()` - Creates default notification settings for new users based on permissions
  2. `get()` - Retrieves notification settings (single or all for a user)
  3. `update()` - Updates notification settings using bulk write operations

#### Key Features:
- Permission-based notification creation
- Uses config for default notification settings
- Efficient bulk updates for multiple notification changes

---

## SUMMARY OF CHANGES BY CATEGORY

### 🔧 Bug Fixes & Improvements:
1. **account.js**: Better error handling (returns false instead of throwing)
2. **account.js**: Support for Stripe customer ID lookup
3. **login.js**: Improved IP address handling (IPv6 to IPv4 conversion)
4. **login.js**: Better device/browser detection using UAParser
5. **user.js**: XSS protection with input escaping

### 🔄 API/Method Updates:
1. **account.js**: `findOneAndRemove` → `findOneAndDelete`
2. **email.js**: `findOneAndRemove` → `findOneAndDelete`
3. **pushtoken.js**: `findOneAndRemove` → `findOneAndDelete`
4. **token.js**: `save()` → `create()` (major API change)

### ✨ New Features:
1. **email.js**: Multi-language/locale support
2. **token.js**: Token expiration and active status tracking
3. **notification.js**: Complete notification preferences system

### 📦 Dependency Changes:
1. **login.js**: Added `ua-parser-js` dependency
2. **user.js**: Changed from `bcrypt` to `bcryptjs`
3. **user.js**: Added `lodash.escape` dependency

### 🗑️ Removed Features:
1. **email.js**: Removed numeric ID field from schema
2. **token.js**: Removed automatic token decryption
3. **token.js**: Removed token update logic (now always creates new)

---

## FILES REQUIRING ATTENTION

### High Priority:
1. **token.js** - Major API changes, may break existing code
2. **notification.js** - New feature, ensure proper integration
3. **email.js** - Schema change (removed ID field), may require migration

### Medium Priority:
1. **account.js** - API signature change in `get()` function
2. **user.js** - Dependency change (`bcrypt` → `bcryptjs`)

### Low Priority:
1. **login.js** - Improved functionality, should be backward compatible
2. **pushtoken.js** - Method deprecation fix

---

## MIGRATION NOTES

1. **email.js**: If migrating, need to:
   - Remove `id` field from existing documents
   - Add `locale` field (default to 'en')
   - Update queries to include locale

2. **token.js**: If migrating, need to:
   - Update all `token.save()` calls to `token.create()`
   - Handle token decryption manually where needed
   - Add `issued_at`, `expires_at`, and `active` fields to existing tokens

3. **user.js**: Ensure `bcryptjs` and `lodash.escape` are installed

4. **login.js**: Ensure `ua-parser-js` is installed

5. **notification.js**: New feature - implement notification preferences UI/API
