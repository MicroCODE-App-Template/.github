# Migration Code Patterns Reference

Quick reference for consistent code patterns during migration.

---

## 1. File Header Pattern

```javascript
// MicroCODE: define this module's name for our 'mcode' package
const mcode = require('mcode-log');
const MODULE_NAME = 'account.js'; // or appropriate filename

const mongoose = require('mongoose');
const Schema = mongoose.Schema;
const utility = require('../helper/utility');
// ... other requires
```

---

## 2. Schema Pattern with _id

```javascript
const accountSchema = new Schema({
  // Primary key - use _id with prefix
  _id: { type: String, default: () => utility.unique_id('acct') },

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
const Account = mongoose.model('Account', accountSchema, 'account');
exports.model = Account; // Export model if needed
```

---

## 3. CRUD Function Pattern

### Create Function

```javascript
/**
 * @function create
 * @memberof account.model
 * @description Create a new account and return the account id
 * @param {Object} params - Parameters object
 * @param {String} [params.plan] - Account plan type
 * @returns {Promise<Object>} The created account object
 */
exports.create = async function({ plan } = {}) {
  const account = new Account({
    _id: utility.unique_id('acct'), // Or let default handle it
    name: 'My Account',
    type: 'personal', // Default type
    active: true,
    created_at: new Date(),
  });

  return await account.save();
};
```

### Get Function

```javascript
/**
 * @function get
 * @memberof account.model
 * @description Get an account by _id
 * @param {Object} params - Parameters object
 * @param {String} params._id - Account ID
 * @returns {Promise<Object|null>} Account object or null
 */
exports.get = async function({ _id }) {
  return await Account.findOne({ _id: _id }).lean();
};
```

### Update Function

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
exports.update = async function({ _id, data }) {
  return await Account.findOneAndUpdate(
    { _id: _id },
    data,
    { new: true }
  );
};
```

### Delete Function

```javascript
/**
 * @function delete
 * @memberof account.model
 * @description Delete the account
 * @param {Object} params - Parameters object
 * @param {String} params._id - Account ID
 * @returns {Promise<Object>} Deletion result
 */
exports.delete = async function({ _id }) {
  return await Account.findOneAndDelete({ _id: _id });
};
```

---

## 4. Parameter Naming Pattern

### Use Explicit ID Naming

```javascript
// ❌ BAD - ambiguous
exports.create = async function({ data, user, account }) {
  data.user_id = user;
  data.account_id = account;
};

// ✅ GOOD - explicit
exports.create = async function({ data, user_id, account_id }) {
  data.user_id = user_id;
  data.account_id = account_id;
};
```

### Foreign Key Queries

```javascript
// ❌ BAD - uses 'id' in nested query
User.findOne({ 'account.id': account_id })

// ✅ GOOD - uses '_id' in nested query
User.findOne({ 'account._id': account_id })
```

---

## 5. Logging Pattern

```javascript
// Success
mcode.done('Account created successfully', MODULE_NAME);

// Error
mcode.exp('Failed to create account', MODULE_NAME, error);

// Warning
mcode.warn('Account already exists', MODULE_NAME);

// Info
mcode.info('Processing account update', MODULE_NAME);
```

---

## 6. Timestamp Pattern

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

---

## 7. Type & State Pattern

```javascript
// Setting defaults in create()
const account = new Account({
  type: 'personal', // Default type
  state: 'active', // Default state
  // ...
});

// Querying by type/state
Account.find({ type: 'business', state: 'active' })

// Updating state
Account.findOneAndUpdate(
  { _id: account_id },
  { state: 'suspended' }
);
```

---

## 8. User Settings Pattern

```javascript
// Initialize in create()
const user = new User({
  settings: config.get('settings') || {},
  // ...
});

// Get setting
const setting = await userModel.getSetting({
  user_id: user_id,
  key: 'subsystem.feature.setting'
});

// Set setting
await userModel.setSetting({
  user_id: user_id,
  account_id: account_id,
  key: 'subsystem.feature.setting',
  value: 'newValue'
});

// Set all settings
await userModel.setSettings({
  user_id: user_id,
  account_id: account_id,
  settings: { /* complete settings object */ }
});
```

---

## 9. Function Organization Pattern

```javascript
// 1. Schema definition
const accountSchema = new Schema({ /* ... */ });

// 2. Model creation
const Account = mongoose.model('Account', accountSchema, 'account');
exports.schema = accountSchema;
exports.model = Account;

// 3. CRUD functions (in order)
exports.create = async function({ /* ... */ }) { /* ... */ };
exports.get = async function({ /* ... */ }) { /* ... */ };
exports.update = async function({ /* ... */ }) { /* ... */ };
exports.delete = async function({ /* ... */ }) { /* ... */ };

// 4. Special functions (at end)
exports.subscription = async function({ /* ... */ }) { /* ... */ };
exports.users = async function({ /* ... */ }) { /* ... */ };
```

---

## 10. Query Pattern with _id

```javascript
// Single document
const account = await Account.findOne({ _id: account_id });

// Multiple documents
const accounts = await Account.find({ type: 'business' });

// Update
await Account.updateOne({ _id: account_id }, { active: false });

// Delete
await Account.deleteOne({ _id: account_id });

// Nested queries
const user = await User.findOne({ 'account._id': account_id });
```

---

## 11. Aggregation Pattern

```javascript
// Use _id in aggregations
const result = await Account.aggregate([
  {
    $group: {
      _id: '$type', // Use _id, not custom field name
      total: { $sum: 1 }
    }
  }
]);

// Lookup with _id
const result = await Account.aggregate([
  {
    $lookup: {
      from: 'user',
      localField: '_id',
      foreignField: 'account._id',
      as: 'users'
    }
  }
]);
```

---

## 12. Error Handling Pattern

```javascript
exports.create = async function({ data, user_id, account_id }) {
  try {
    const account = new Account({
      // ...
    });

    return await account.save();
  } catch (error) {
    mcode.exp('Failed to create account', MODULE_NAME, error);
    throw error;
  }
};
```

---

## 13. JSDoc Pattern

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
exports.functionName = async function({ field1, field2 }) {
  // ...
};
```

---

## 14. Common Migrations

### From `id` to `_id`

```javascript
// Before
Account.findOne({ id: id })
Account.findOne({ 'account.id': account_id })

// After
Account.findOne({ _id: _id })
Account.findOne({ 'account._id': account_id })
```

### From `date_created` to `created_at`

```javascript
// Before
date_created: { type: Date, required: true }
date_created: new Date()

// After
created_at: { type: Date, required: true }
created_at: new Date()
```

### From `user` to `user_id`

```javascript
// Before
exports.create = async function({ data, user, account }) {
  data.user_id = user;
  data.account_id = account;
};

// After
exports.create = async function({ data, user_id, account_id }) {
  data.user_id = user_id;
  data.account_id = account_id;
};
```

---

## 15. Testing Pattern

```javascript
// After each migration, test:
describe('Account Model', () => {
  it('should create account with _id', async () => {
    const account = await accountModel.create({ plan: 'free' });
    expect(account._id).toBeDefined();
    expect(account.created_at).toBeDefined();
  });

  it('should get account by _id', async () => {
    const account = await accountModel.get({ _id: testId });
    expect(account).toBeDefined();
  });

  // ... more tests
});
```

---

## Quick Reference: Field Mappings

| Old (gravity-current) | New (ladders pattern) |
|----------------------|----------------------|
| `id` | `_id` |
| `date_created` | `created_at` |
| `time` | `occurred_at`, `logged_at`, `started_at` |
| `date_sent` | `sent_at` |
| `last_active` | `active_at` |
| `user` (param) | `user_id` (param) |
| `account` (param) | `account_id` (param) |
| `'account.id'` | `'account._id'` |
| `console.log` | `mcode.done` |
| `console.error` | `mcode.exp` |

---

## Quick Reference: Prefixes for _id

| Model | Prefix |
|-------|--------|
| account | `'acct'` |
| email | `'emtp'` |
| event | `'evnt'` |
| feedback | `'fbck'` |
| invite | `'invt'` |
| key | `'apik'` |
| log | `'logn'` |
| login | `'logi'` |
| token | `'stok'` |
| usage | `'used'` |
| user | `'user'` |

---

Use these patterns consistently throughout the migration for maintainable, bug-free code.
