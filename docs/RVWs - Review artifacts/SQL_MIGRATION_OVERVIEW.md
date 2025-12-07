# SQL Migration Overview: User Settings System (Phase 6)

**Date**: 2025-12-05
**Phase**: Phase 6 - User Settings System
**Status**: Overview Only (Not Implemented)

---

## Overview

This document outlines what would be required to migrate the User Settings System from MongoDB to SQL database. Currently, the system uses MongoDB with a flexible `settings` object field. SQL databases require a different approach due to their structured nature.

---

## Current State

### MongoDB Implementation

- **Settings Storage**: Single `settings` field as a JSON/BSON object
- **Structure**: Hierarchical nested object (`subsystem.feature.setting`)
- **Example**: `{ theme: { sight: { darkMode: true } } }`
- **Flexibility**: Can store any nested structure without schema changes

### SQL Current Schema

The existing SQL migration (`20200729133645_user_table.js`) includes:

- `dark_mode` (boolean) - **DEPRECATED** - Should be migrated to `settings.theme.sight.darkMode`
- `support_enabled` (boolean) - **DEPRECATED** - Should be migrated to `settings.support.remoteHelp.enabled`
- `locale` (string) - **DEPRECATED** - Should be migrated to `settings.theme.sight.language`

---

## Migration Options

### Option 1: JSON Column (Recommended for PostgreSQL/MySQL 5.7+)

**Pros:**

- Closest to MongoDB flexibility
- Single column for all settings
- Easy to query nested values (with JSON functions)
- Minimal schema changes

**Cons:**

- Requires JSON support (PostgreSQL, MySQL 5.7+, MariaDB 10.2+)
- Less type safety
- Indexing can be complex

**Implementation:**

```sql
-- Migration: Add settings JSON column
ALTER TABLE user ADD COLUMN settings JSON DEFAULT '{}';

-- Migration: Migrate existing data
UPDATE user SET settings = JSON_OBJECT(
  'theme', JSON_OBJECT(
    'sight', JSON_OBJECT(
      'darkMode', dark_mode,
      'language', locale
    )
  ),
  'support', JSON_OBJECT(
    'remoteHelp', JSON_OBJECT(
      'enabled', support_enabled
    )
  )
) WHERE settings IS NULL OR settings = '{}';

-- Migration: Drop deprecated columns (optional, after verification)
-- ALTER TABLE user DROP COLUMN dark_mode;
-- ALTER TABLE user DROP COLUMN support_enabled;
-- ALTER TABLE user DROP COLUMN locale;
```

**Model Functions:**

- `settings.get()`: Use JSON extraction functions (`JSON_EXTRACT`, `->`, `->>`)
- `settings.set()`: Use JSON update functions (`JSON_SET`, `JSON_REPLACE`)
- `settings.setAll()`: Simple UPDATE with JSON object

---

### Option 2: Normalized Tables (Traditional SQL)

**Pros:**

- Full relational integrity
- Easy to query specific settings
- Type safety
- Works with all SQL databases

**Cons:**

- Complex schema (multiple tables)
- More complex queries (JOINs)
- Slower for bulk operations
- Requires significant refactoring

**Implementation:**

```sql
-- New table: user_settings
CREATE TABLE user_settings (
  id CHAR(36) PRIMARY KEY,
  user_id CHAR(36) NOT NULL,
  subsystem VARCHAR(50) NOT NULL,
  feature VARCHAR(50) NOT NULL,
  setting VARCHAR(50) NOT NULL,
  value TEXT NOT NULL,
  value_type VARCHAR(20) NOT NULL, -- 'boolean', 'string', 'number', 'object'
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  UNIQUE KEY unique_setting (user_id, subsystem, feature, setting),
  FOREIGN KEY (user_id) REFERENCES user(id) ON DELETE CASCADE
);

-- Migration: Migrate existing data
INSERT INTO user_settings (id, user_id, subsystem, feature, setting, value, value_type)
SELECT
  CONCAT('uset_', UUID()),
  id,
  'theme',
  'sight',
  'darkMode',
  dark_mode,
  'boolean'
FROM user WHERE dark_mode IS NOT NULL;

INSERT INTO user_settings (id, user_id, subsystem, feature, setting, value, value_type)
SELECT
  CONCAT('uset_', UUID()),
  id,
  'support',
  'remoteHelp',
  'enabled',
  support_enabled,
  'boolean'
FROM user WHERE support_enabled IS NOT NULL;

INSERT INTO user_settings (id, user_id, subsystem, feature, setting, value, value_type)
SELECT
  CONCAT('uset_', UUID()),
  id,
  'theme',
  'sight',
  'language',
  locale,
  'string'
FROM user WHERE locale IS NOT NULL;
```

**Model Functions:**

- `settings.get()`: SELECT with WHERE clauses for subsystem/feature/setting
- `settings.set()`: INSERT ... ON DUPLICATE KEY UPDATE
- `settings.setAll()`: DELETE all + INSERT multiple rows (transaction)

---

### Option 3: Hybrid Approach (Settings Table + JSON Column)

**Pros:**

- Best of both worlds
- Common settings in normalized table (queryable)
- Complex/nested settings in JSON column

**Cons:**

- Most complex to maintain
- Requires careful logic to determine which storage to use

**Implementation:**

- Use normalized table for frequently queried settings
- Use JSON column for complex nested structures
- Model functions check both sources

---

## Recommended Approach: Option 1 (JSON Column)

For this codebase, **Option 1 (JSON Column)** is recommended because:

1. **Consistency**: Matches MongoDB implementation pattern
2. **Simplicity**: Minimal code changes required
3. **Flexibility**: Can store any nested structure
4. **Performance**: Single column read/write is fast
5. **Modern SQL**: PostgreSQL and MySQL 5.7+ have excellent JSON support

---

## Required Changes

### 1. Database Migration

**File**: `server/migrations/[timestamp]_add_user_settings.sql.js`

```javascript
exports.up = async function (knex) {
  // Add settings JSON column
  await knex.schema.table("user", (table) => {
    table.json("settings").defaultTo("{}");
  });

  // Migrate existing data from deprecated columns
  const users = await knex("user").select(
    "id",
    "dark_mode",
    "support_enabled",
    "locale"
  );

  for (const user of users) {
    const settings = {
      theme: {
        sight: {
          darkMode: !!user.dark_mode,
          language: user.locale || "en",
        },
      },
      support: {
        remoteHelp: {
          enabled: !!user.support_enabled,
        },
      },
      messages: {
        email: {},
        text: {},
      },
    };

    await knex("user")
      .where("id", user.id)
      .update({ settings: JSON.stringify(settings) });
  }

  // Optional: Drop deprecated columns after verification
  // await knex.schema.table('user', table =>
  // {
  //     table.dropColumn('dark_mode');
  //     table.dropColumn('support_enabled');
  //     table.dropColumn('locale');
  // });
};

exports.down = async function (knex) {
  // Restore deprecated columns if needed
  await knex.schema.table("user", (table) => {
    table.dropColumn("settings");
  });
};
```

---

### 2. SQL Model Functions

**File**: `server/model/sql/user.sql.js`

#### Add `exports.settings = {}` object

#### `settings.get()` Function

```javascript
exports.settings.get = async function ({ id, email, key }) {
  if (!id && !email) return null;
  if (!key) return null;

  const query = id ? { id } : { email };
  const user = await db("user").select("settings").where(query).first();

  if (!user || !user.settings) return null;

  const settings =
    typeof user.settings === "string"
      ? JSON.parse(user.settings)
      : user.settings;

  const keys = key.split(".");
  const subsystemKey = keys[0];
  const featureKey = keys[1];
  const settingKey = keys[2];

  const subsystem = settings[subsystemKey];
  if (!subsystem) return null;

  if (!featureKey) return subsystem;

  const feature = subsystem[featureKey];
  if (!feature) return null;

  if (!settingKey) return feature;

  return feature[settingKey] !== undefined ? feature[settingKey] : null;
};
```

#### `settings.set()` Function

```javascript
exports.settings.set = async function ({ id, email, key, value }) {
  if (!id && !email)
    throw { message: "user.settings.set: id or email required" };
  if (!key) throw { message: "user.settings.set: key required" };
  if (value === undefined)
    throw { message: "user.settings.set: value required" };

  const query = id ? { id } : { email };
  const user = await db("user").select("settings").where(query).first();

  if (!user) throw { message: "user.settings.set: user not found" };

  const keys = key.split(".");
  if (keys.length !== 3)
    throw {
      message:
        'user.settings.set: key must have format "subsystem.feature.setting"',
    };

  const settings =
    typeof user.settings === "string"
      ? JSON.parse(user.settings || "{}")
      : user.settings || {};

  const subsystemKey = keys[0];
  const featureKey = keys[1];
  const settingKey = keys[2];

  // Initialize nested structure
  if (!settings[subsystemKey]) settings[subsystemKey] = {};
  if (!settings[subsystemKey][featureKey])
    settings[subsystemKey][featureKey] = {};

  // Set the value
  settings[subsystemKey][featureKey][settingKey] = value;

  // Update database
  await db("user")
    .where(query)
    .update({
      settings: JSON.stringify(settings),
      updated_at: db.fn.now(),
    });

  return { id: user.id, settings };
};
```

#### `settings.setAll()` Function

```javascript
exports.settings.setAll = async function ({ id, email, settings }) {
  if (!id && !email)
    throw { message: "user.settings.setAll: id or email required" };
  if (!settings || typeof settings !== "object")
    throw { message: "user.settings.setAll: settings object required" };

  const query = id ? { id } : { email };
  const user = await db("user").select("id").where(query).first();

  if (!user) throw { message: "user.settings.setAll: user not found" };

  await db("user")
    .where(query)
    .update({
      settings: JSON.stringify(settings),
      updated_at: db.fn.now(),
    });

  return { id: user.id, settings };
};
```

---

### 3. Update User Creation

**File**: `server/model/sql/user.sql.js` - `exports.create()`

Add settings initialization:

```javascript
const config = require("config");

exports.create = async function ({ user, account }) {
  const data = {
    id: utility.unique_id("user"),
    type: "user",
    state: "active",
    name: escape(user.name),
    email: user.email,
    facebook_id: user.facebook_id,
    twitter_id: user.twitter_id,
    default_account_id: account,
    avatar: user.avatar,
    verified: user.verified,
    settings: JSON.stringify(config.get("settings") || {}), // Initialize from config
  };
  // ... rest of function
};
```

---

### 4. Update User Get Function

**File**: `server/model/sql/user.sql.js` - `exports.get()`

Update column selection to include `settings`:

```javascript
const cols = [
  "id",
  "name",
  "email",
  "created_at",
  "settings", // Add settings
  "active_at",
  "onboarded",
  "facebook_id",
  "twitter_id",
  "disabled",
  "account_id",
  "permission",
  "default_account_id",
  "password",
  "tfa_enabled",
  "verified",
  "avatar",
  "locale",
]; // Keep locale for backward compatibility
```

Parse settings in response:

```javascript
if (data?.length) {
  return data.map((u) => {
    delete u.password;
    // Parse settings JSON if it's a string
    if (u.settings && typeof u.settings === "string") {
      try {
        u.settings = JSON.parse(u.settings);
      } catch (e) {
        u.settings = {};
      }
    }
    return u;
  });
}
```

---

### 5. Database-Specific Considerations

#### PostgreSQL

- Use `JSONB` type instead of `JSON` for better performance
- Use `->` and `->>` operators for JSON queries
- Example: `SELECT settings->'theme'->'sight'->>'darkMode' FROM user`

#### MySQL 5.7+

- Use `JSON` type
- Use `JSON_EXTRACT()` or `->` and `->>` operators
- Example: `SELECT JSON_EXTRACT(settings, '$.theme.sight.darkMode') FROM user`

#### MariaDB 10.2+

- Similar to MySQL 5.7+
- Use `JSON` type and JSON functions

#### SQLite

- Store as `TEXT` with JSON string
- Parse in application code (no native JSON functions in older versions)
- SQLite 3.38+ has JSON functions

---

## Testing Requirements

1. **Migration Test**: Verify data migration from deprecated columns
2. **Settings Get Test**: Test all three key path levels (subsystem, feature, setting)
3. **Settings Set Test**: Test setting creation and updates
4. **Settings SetAll Test**: Test bulk replacement
5. **Backward Compatibility**: Ensure old code still works during transition

---

## Rollback Plan

1. Keep deprecated columns (`dark_mode`, `support_enabled`, `locale`) until fully verified
2. Add feature flag to switch between old and new settings
3. Maintain dual-write during transition period
4. Monitor for errors before removing deprecated columns

---

## Estimated Effort

- **Database Migration**: 2-4 hours
- **Model Functions**: 4-6 hours
- **Testing**: 2-3 hours
- **Documentation**: 1 hour
- **Total**: 9-14 hours

---

## Notes

- This migration is **NOT required** if using MongoDB exclusively
- SQL migration should only be implemented if/when SQL database support is needed
- The MongoDB implementation is complete and production-ready
- Consider using a migration tool like `knex` migrations for version control

---

## References

- Current MongoDB implementation: `server/model/mongo/user.mongo.js`
- Current SQL model: `server/model/sql/user.sql.js`
- Migration file: `server/migrations/20200729133645_user_table.js`
- Settings config: `server/config/default.json`
