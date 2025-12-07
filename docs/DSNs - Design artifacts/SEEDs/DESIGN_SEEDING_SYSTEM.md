# Design: Generalized Seeding System

## Executive Summary

This design document outlines a generalized, extensible seeding system that automatically discovers and seeds static data from JSON files in `server/seed/` folders. The system eliminates the need for per-entity JavaScript seeder files, supports both MongoDB and SQL databases, and provides idempotent seeding with configurable matching strategies.

## Current State Analysis

### Existing Structure

```
server/seed/
├── all.js             # Manual orchestrator requiring each entity seeder
├── mfg_company.js     # Per-entity seeder (to be eliminated)
├── plc_family.js      # Per-entity seeder (to be eliminated)
├── plc_mfg.js         # Per-entity seeder (to be eliminated)
├── email/             # Entity folder
│   ├── data.js        # Code that reads locale files (TO BE ELIMINATED)
│   ├── en.js          # Locale-specific data (TO BE RENAMED to data.en.js)
│   └── es.js          # Locale-specific data (TO BE RENAMED to data.es.js)
├── mfg_company/       # Entity folder
│   └── data.js        # JSON data array
├── plc_family/        # Entity folder
│   └── data.js        # JSON data array
├── plc_mfg/           # Entity folder
│   └── data.js        # JSON data array
├── mongo.seed.js      # Simple MongoDB seeder (email only)
└── sql.seed.js        # Simple SQL seeder (email only)
```

### Current Limitations

1. **Manual Entity Registration**: Each new entity requires:

   - Creating `{entity}.js` seeder file
   - Adding `require()` and `await Entity.seed()` to `all.js`
   - Duplicating seeding logic across entities

2. **Inconsistent Patterns**:

   - `mfg_company.js` uses `insertMany()` without idempotency
   - `email` seeding (in `setup.js`) checks for existing records
   - No unified approach

3. **Database-Specific Code**:

   - MongoDB uses Mongoose models
   - SQL uses Knex queries
   - Logic duplicated in multiple places

4. **Limited Extensibility**:
   - Adding new entities requires code changes
   - No configuration for matching strategies
   - No support for custom seeding logic

## Desired State

### Target Structure

```
server/seed/
├── data/
│   ├── {entity}.data.js       # Single-key entity data
│   ├── {entity}.data.en.js    # English locale data (required for locale entities)
│   ├── {entity}.data.fr.js    # French locale data
│   ├── {entity}.data.dt.js    # German locale data
│   ├── {entity}.data.it.js    # Italian locale data
│   └── {entity}.data.{cc}.js  # Any other locale code (2-letter)
├── _entity.js            # Single source of truth for Entity-Prefix mappings
├── all.js                     # Auto-discovery orchestrator (refactored)
├── cache.js
├── discover.js
├── loader.js
├── matcher.js
├── resolver.js
├── seeder.js
└── ...
```

### Key Features

1. **Auto-Discovery**: Automatically detect entities from data files in `server/seed/data/` folder
2. **Zero Configuration**: Works out-of-the-box with sensible defaults
3. **Idempotent & Updating**: Inserts missing records, updates existing records if fields differ, skips unchanged records
4. **Database Agnostic**: Handles both MongoDB and SQL transparently
5. **Extensible**: Supports custom matching strategies and seeding logic
6. **Error Resilient**: Continues seeding other entities if one fails
7. **Simplified Structure**: All data files consolidated in single `data/` folder with consistent naming pattern

## Entity-Prefix Mapping Reference

### Overview

The seeding system enforces a strict mapping between entity names, prefixes, and foreign key field patterns. This mapping ensures consistency across the application and enables automatic foreign key resolution.

**Single Source of Truth**: All prefix mappings are stored in `server/seed/_entity.js`. This file is the authoritative source - all other code loads from this file. The leading underscore ensures it sorts first in directory listings.

**Prefix Format**: All prefixes are **exactly 4 characters** (e.g., `'acct'`, `'mail'`, `'bbrn'`, `'log_'`). Some prefixes use underscore as one of the 4 characters (e.g., `'log_'`, `'plc_'`).

### Authoritative Entity-Prefix Mapping Table

All seed data files **MUST** use prefixes that match this authoritative table. The seeding system validates prefix consistency and uses this mapping for foreign key resolution.

#### Common MicroCODE App: DB ENTITYs

**GENERAL**

| Entity       | Foreign Key Field | Prefix |
| ------------ | ----------------- | ------ |
| account      | account_id        | `acct` |
| user         | user_id           | `user` |
| email        | email_id          | `mail` |
| event        | event_id          | `evnt` |
| feedback     | feedback_id       | `fdbk` |
| invite       | invite_id         | `invt` |
| key          | key_id            | `apik` |
| log          | log_id            | `log_` |
| login        | login_id          | `logn` |
| notification | notification_id   | `notf` |
| token        | token_id          | `tokn` |

#### Regatta-RC: DB ENTITYs

**BOATs**

| Entity       | Foreign Key Field | Prefix |
| ------------ | ----------------- | ------ |
| boat         | boat_id           | `boat` |
| boat_brand   | boat_brand_id     | `bbrn` |
| boat_builder | boat_builder_id   | `bbld` |
| boat_design  | boat_design_id    | `bdsn` |

**RACING**

| Entity  | Foreign Key Field | Prefix |
| ------- | ----------------- | ------ |
| org     | org_id            | `orgn` |
| club    | club_id           | `club` |
| race    | race_id           | `race` |
| regatta | regatta_id        | `rgta` |
| series  | series_id         | `sers` |
| class   | class_id          | `clas` |
| start   | start_id          | `strt` |
| cert    | cert_id           | `cert` |

**BOAT as owned, configured, and raced**

| Entity | Foreign Key Field | Prefix |
| ------ | ----------------- | ------ |
| sv\*   | sv_id             | `svsl` |

\* Sailing Vessel, includes BOAT, USERs - Owner(s) and Crew(s), CERTs

#### LADDERS: DB ENTITYs

**PLCs**

| Entity      | Foreign Key Field | Prefix |
| ----------- | ----------------- | ------ |
| plc         | plc_id            | `plc_` |
| plc_brand   | plc_brand_id      | `plcb` |
| plc_company | plc_company_id    | `plcc` |
| plc_family  | plc_family_id     | `plcf` |
| plc_program | plc_program_id    | `plcp` |

### Prefix Usage Rules

1. **Data File Prefix**: Each seed data file **MUST** export a `prefix` property matching the table above

   ```javascript
   module.exports = {
       prefix: 'acct',  // 4 characters - must match authoritative table
       account: [...]
   };
   ```

2. **Key Field Transformation**: The seeder automatically prepends the prefix to record `key` fields

   - Seed data: `{ key: 'test' }` with prefix `'acct'` (4 characters)
   - Stored in DB: `{ key: 'acct_test' }` (seeder adds underscore separator)

3. **Foreign Key Field Pattern**: Foreign key fields follow the pattern `{prefix}_key` where prefix is exactly 4 characters

   - Example: `acct_key`, `user_key`, `boat_key`, `bbrn_key`, `log__key`, `plc__key`
   - Arrays: `acct_keys: ['acct_test1', 'acct_test2']`

4. **Foreign Key Resolution**: The system automatically resolves `{prefix}_key` → `{prefix}_id` using the prefix-to-entity mapping

### Validation and Enforcement

The seeding system enforces prefix consistency through:

1. **Loader Validation**: Validates that `prefix` property exists in data files
2. **Prefix-to-Entity Mapping**: `cache.js` maintains authoritative mapping table
3. **Foreign Key Resolution**: Uses mapping to resolve foreign key references
4. **Runtime Warnings**: Warns about mismatched prefixes or missing mappings

### Implementation: Single Source of Truth

The prefix-to-entity mapping is stored in `server/seed/_entity.js` as the **single source of truth**. All other code (including `cache.js`, `loader.js`, and any other modules) loads from this file.

**File Structure: `server/seed/_entity.js`**

```javascript
/**
 * Entity-Prefix Mapping Table
 *
 * This is the SINGLE SOURCE OF TRUTH for all prefix-to-entity mappings.
 * All prefixes are exactly 4 characters.
 *
 * Format: { prefix: entityName }
 * - prefix: 4-character identifier (e.g., 'acct', 'mail', 'log_')
 * - entityName: Full entity name matching model file name (e.g., 'account', 'email', 'log')
 *
 * Usage:
 * - Foreign key fields: {prefix}_key (e.g., acct_key, mail_key, log__key, plc__key)
 * - Foreign key IDs: {prefix}_id (e.g., acct_id, mail_id, log__id, plc__id)
 * - Seed data files: prefix: '{prefix}' where prefix is exactly 4 characters (e.g., prefix: 'acct', prefix: 'log_')
 */

module.exports = {
  // GENERAL (Regatta-RC & LADDERS)
  acct: "account", // account_id, acct_key
  user: "user", // user_id, user_key
  mail: "email", // email_id, mail_key
  evnt: "event", // event_id, evnt_key
  fdbk: "feedback", // feedback_id, fdbk_key
  invt: "invite", // invite_id, invt_key
  apik: "key", // key_id, apik_key
  log_: "log", // log_id, log__key (underscore pads to 4 characters)
  logn: "login", // login_id, logn_key
  notf: "notification", // notification_id, notf_key
  tokn: "token", // token_id, tokn_key

  // BOATs (Regatta-RC)
  boat: "boat", // boat_id, boat_key
  bbrn: "boat_brand", // boat_brand_id, bbrn_key
  bbld: "boat_builder", // boat_builder_id, bbld_key
  bdsn: "boat_design", // boat_design_id, bdsn_key

  // RACING (Regatta-RC)
  orgn: "org", // org_id, orgn_key
  club: "club", // club_id, club_key
  race: "race", // race_id, race_key
  rgta: "regatta", // regatta_id, rgta_key
  sers: "series", // series_id, sers_key
  clas: "class", // class_id, clas_key
  strt: "start", // start_id, strt_key
  cert: "cert", // cert_id, cert_key

  // BOAT as owned, configured, and raced (Regatta-RC)
  svsl: "sv", // sv_id, svsl_key (Sailing Vessel)

  // PLCs (LADDERS)
  plc_: "plc", // plc_id, plc__key (underscore pads to 4 characters)
  plcb: "plc_brand", // plc_brand_id, plcb_key
  plcc: "plc_company", // plc_company_id, plcc_key
  plcf: "plc_family", // plc_family_id, plcf_key
  plcp: "plc_program", // plc_program_id, plcp_key
};
```

**Updated `cache.js` Implementation:**

The `buildPrefixToEntityMapping()` function in `seed/cache.js` now loads from the single source of truth:

```javascript
/**
 * @func buildPrefixToEntityMapping
 * @memberof seed.cache
 * @desc Maps prefix abbreviations to full entity names for foreign key resolution.
 * Loads from _entity.js as the single source of truth.
 * @returns {object} Mapping of prefix (4-char) -> entity name
 */
function buildPrefixToEntityMapping() {
  // Load from single source of truth
  return require("./_entity.data");
}
```

**Benefits of Single Source of Truth:**

1. **Centralized Management**: All prefix mappings in one file (`_entity.js`)
2. **Easy Updates**: Add new entities by updating one file
3. **Consistency**: All code uses the same mapping
4. **Validation**: Can validate prefix format (must be 4 characters) in one place
5. **Documentation**: Comments and structure in one location
6. **Reusability**: Other modules can require `_entity.js` directly if needed
7. **File Sorting**: Leading underscore ensures file appears first in directory listings

```javascript
/**
 * @func buildPrefixToEntityMapping
 * @memberof seed.cache
 * @desc Maps prefix abbreviations to full entity names for foreign key resolution.
 * This is the AUTHORITATIVE mapping table - all prefixes MUST match this table.
 * @returns {object} Mapping of prefix (without underscore) -> entity name
 */
function buildPrefixToEntityMapping() {
  return {
    // GENERAL (Regatta-RC & LADDERS)
    acct: "account", // account_id, acct_key
    user: "user", // user_id, user_key
    mail: "email", // email_id, mail_key
    evnt: "event", // event_id, evnt_key
    fdbk: "feedback", // feedback_id, fdbk_key
    invt: "invite", // invite_id, invt_key
    apik: "key", // key_id, apik_key
    log_: "log", // log_id, log__key (underscore pads to 4 characters)
    logn: "login", // login_id, logn_key
    notf: "notification", // notification_id, notf_key
    tokn: "token", // token_id, tokn_key

    // BOATs (Regatta-RC)
    boat: "boat", // boat_id, boat_key
    bbrn: "boat_brand", // boat_brand_id, bbrn_key
    bbld: "boat_builder", // boat_builder_id, bbld_key
    bdsn: "boat_design", // boat_design_id, bdsn_key

    // RACING (Regatta-RC)
    orgn: "org", // org_id, orgn_key
    club: "club", // club_id, club_key
    race: "race", // race_id, race_key
    rgta: "regatta", // regatta_id, rgta_key
    sers: "series", // series_id, sers_key
    clas: "class", // class_id, clas_key
    strt: "start", // start_id, strt_key
    cert: "cert", // cert_id, cert_key

    // BOAT as owned, configured, and raced (Regatta-RC)
    svsl: "sv", // sv_id, svsl_key (Sailing Vessel)

    // PLCs (LADDERS)
    plc_: "plc", // plc_id, plc__key (underscore pads to 4 characters)
    plcb: "plc_brand", // plc_brand_id, plcb_key
    plcc: "plc_company", // plc_company_id, plcc_key
    plcf: "plc_family", // plc_family_id, plcf_key
    plcp: "plc_program", // plc_program_id, plcp_key
  };
}
```

**Important Notes:**

1. **Prefix Format**: All prefixes are **exactly 4 characters**. Underscores are used ONLY to pad to 4 characters when needed:

   - `'acct'` (4 chars, no underscore needed)
   - `'mail'` (4 chars, no underscore needed)
   - `'log_'` (4 chars, underscore is padding)
   - `'plc_'` (4 chars, underscore is padding)

2. **Foreign Key Field Pattern**: Foreign key fields follow the pattern `{prefix}_key`:

   - Prefix `'acct'` → field `acct_key`
   - Prefix `'log_'` → field `log__key` (log\_ + \_key)
   - Prefix `'plc_'` → field `plc__key` (plc\_ + \_key)

3. **Authoritative Source**: This file (`_entity.js`) is the **single source of truth** for prefix-to-entity mapping. Any changes to prefixes must be updated here first.

4. **Validation**: The loader validates that prefixes in data files are exactly 4 characters and match this mapping table exactly.

## Design Architecture

### Core Components

#### 1. Entity Discovery Module (`seed/discover.js`)

**Purpose**: Scan `server/seed/` directory and identify entity folders.

**Responsibilities**:

- Read directory structure
- Filter out non-entity files (`.js` files, `config.js`, etc.)
- Return array of entity names

**Implementation**:

```javascript
/**
 * @func discoverEntities
 * @memberof seed.discover
 * @desc Discover all entity folders in the seed directory.
 * @api private
 * @returns {Promise<Array<string>>} Array of entity folder names
 */
async function discoverEntities() {
  const seedDir = path.join(__dirname);
  const entries = await fs.readdir(seedDir, { withFileTypes: true });

  return entries
    .filter((entry) => entry.isDirectory())
    .map((entry) => entry.name)
    .filter((name) => !name.startsWith("."));
}
```

#### 2. Data Loader Module (`seed/loader.js`)

**Purpose**: Load seed data from `server/seed/data/` folder.

**Responsibilities**:

- Load all files matching pattern `{entity}.data.*.js` from `data/` folder (including `{entity}.data.js`)
- Automatically detect locale-aware entities (presence of `{entity}.data.en.js` or similar)
- Merge data from all matching files
- Validate data structure (must be array)
- Validate prefix against authoritative Entity-Prefix Mapping table
- Return both data and matching strategy hint

**Implementation**:

```javascript
const mcode = require("mcode-log");
const fs = require("fs").promises;
const path = require("path");

/**
 * @func loadEntityData
 * @memberof seed.loader
 * @desc Load seed data for an entity from server/seed/data/ folder. Loads all files matching {entityName}.data.*.js pattern.
 * @api private
 * @param {string} entityName - Entity name
 * @returns {Promise<object>} Object containing { data: Array, isLocaleAware: boolean, prefix: string }
 */
async function loadEntityData(entityName) {
  const dataDir = path.join(__dirname, "data");

  // Read all files in data directory
  const files = await fs.readdir(dataDir);

  // Filter for files matching {entityName}.data.{locale}.js or {entityName}.data.js
  const dataFiles = files.filter((file) => {
    // Match locale-aware: {entityName}.data.{locale}.js
    const localeMatch = file.match(/^(.+)\.data\.([a-z]{2})\.js$/);
    if (localeMatch && localeMatch[1] === entityName) {
      return true;
    }

    // Match simple: {entityName}.data.js
    const simpleMatch = file.match(/^(.+)\.data\.js$/);
    if (simpleMatch && simpleMatch[1] === entityName) {
      return true;
    }

    return false;
  });

  if (dataFiles.length === 0) {
    throw new Error(
      `No data files found for entity: ${entityName} (expected ${entityName}.data.js or ${entityName}.data.{locale}.js in server/seed/data/)`
    );
  }

  // Check if locale-aware (has {entityName}.data.{locale}.js files)
  const localeFiles = dataFiles.filter((file) =>
    file.match(new RegExp(`^${entityName}\\.data\\.[a-z]{2}\\.js$`))
  );
  const isLocaleAware = localeFiles.length > 0;

  // Load and merge all data files
  let allData = [];
  let entityPrefix = null;

  for (const file of dataFiles) {
    const filePath = path.join(dataDir, file);
    const fileData = require(filePath);

    // Handle format: { prefix, data } or { prefix, [entityName] }
    if (
      typeof fileData === "object" &&
      fileData !== null &&
      !Array.isArray(fileData)
    ) {
      // Extract prefix if present
      if (fileData.prefix && !entityPrefix) {
        entityPrefix = fileData.prefix;
      }

      // Extract data array
      let dataArray = fileData.data;
      if (!dataArray) {
        const arrayProps = Object.keys(fileData).filter(
          (key) => Array.isArray(fileData[key]) && key !== "prefix"
        );
        if (arrayProps.length > 0) {
          dataArray = fileData[arrayProps[0]];
        }
      }

      if (!Array.isArray(dataArray)) {
        throw new Error(
          `Entity ${entityName}: ${file} must export an array or object with array property`
        );
      }

      allData = [...allData, ...dataArray];
    } else if (Array.isArray(fileData)) {
      allData = [...allData, ...fileData];
    } else {
      throw new Error(
        `Entity ${entityName}: ${file} must export an array or object with prefix and data`
      );
    }
  }

  if (allData.length === 0) {
    mcode.warn(`Entity ${entityName}: No data found in data files`);
  }

  // Validate: prefix must exist
  if (!entityPrefix) {
    throw new Error(
      `Entity ${entityName}: Data file must export a 'prefix' property`
    );
  }

  // Validate: prefix must match authoritative Entity-Prefix Mapping table
  // Prefixes are exactly 4 characters (e.g., 'acct', 'mail', 'log_')
  const { buildPrefixToEntityMapping } = require("./cache");
  const prefixMapping = buildPrefixToEntityMapping();

  // Validate prefix length (must be exactly 4 characters)
  if (entityPrefix.length !== 4) {
    throw new Error(
      `Entity ${entityName}: Prefix '${entityPrefix}' must be exactly 4 characters. ` +
        `Found ${entityPrefix.length} characters. Valid prefixes are documented in _entity.js.`
    );
  }

  // Prefixes are exactly 4 characters - use as-is for lookup
  const prefixKey = entityPrefix;
  let prefixKey = entityPrefix;
  if (prefixKey.endsWith("__")) {
    prefixKey = prefixKey.slice(0, -2);
  } else if (prefixKey.endsWith("_")) {
    prefixKey = prefixKey.slice(0, -1);
  }

  // Check if prefix maps to this entity name
  const mappedEntity = prefixMapping[prefixKey];
  if (!mappedEntity) {
    throw new Error(
      `Entity ${entityName}: Prefix '${prefixKey}' is not found in authoritative Entity-Prefix Mapping table (_entity.js). ` +
        `Please ensure the prefix matches the mapping in server/seed/_entity.js. ` +
        `Valid prefixes are documented in the Entity-Prefix Mapping Reference section.`
    );
  }

  if (mappedEntity !== entityName) {
    throw new Error(
      `Entity ${entityName}: Prefix '${prefixKey}' maps to entity '${mappedEntity}' in authoritative mapping table, ` +
        `but data file is for entity '${entityName}'. Prefix must match entity name. ` +
        `Expected prefix for '${entityName}' should map to '${entityName}' in _entity.js.`
    );
  }

  // Validate: all records must have a 'key' field
  for (let i = 0; i < allData.length; i++) {
    const record = allData[i];
    if (!record || typeof record !== "object") {
      throw new Error(
        `Entity ${entityName}: Record at index ${i} must be an object`
      );
    }
    if (!record.key || typeof record.key !== "string") {
      throw new Error(
        `Entity ${entityName}: Record at index ${i} must have a 'key' field (string)`
      );
    }
  }

  return {
    data: allData,
    isLocaleAware: isLocaleAware,
    prefix: entityPrefix,
  };
}
```

**Key Features**:

- Loads all `{entity}.data.*.js` files (including `{entity}.data.js`) from `server/seed/data/` folder
- Automatically detects locale-aware entities by presence of `{entity}.data.{locale}.js` files (e.g., `email.data.en.js`, `email.data.es.js`)
- Merges data from all matching files into single array
- Validates prefix against authoritative Entity-Prefix Mapping table
- Returns hint about locale-awareness for matching strategy determination

**File Pattern Matching**:

- Pattern: `{entity}.data.*.js` matches:
  - `{entity}.data.js` (single-key entity - ONLY file for single-key entities)
  - `{entity}.data.en.js` (locale-specific, English)
  - `{entity}.data.es.js` (locale-specific, Spanish)
  - `{entity}.data.fr.js` (locale-specific, French)
  - `{entity}.data.dt.js` (locale-specific, German)
  - `{entity}.data.it.js` (locale-specific, Italian)
  - Any other `{entity}.data.{2-letter-locale}.js` files
- **Single-key entities**: Have ONLY `{entity}.data.js` (no locale files)
- **Dual-key (locale-aware) entities**: Have ONLY `{entity}.data.{locale}.js` files (no `{entity}.data.js`)
- All matching files are loaded and their arrays are merged
- Locale detection: Files matching `{entity}.data.{2-letter-locale}.js` pattern indicate locale-aware entity
- **Important**: An entity should be EITHER single-key (only `{entity}.data.js`) OR dual-key (only `{entity}.data.{locale}.js` files), not both

#### 3. Model Resolver Module (`seed/resolver.js`)

**Purpose**: Resolve entity names to database models.

**Responsibilities**:

- Map entity folder names to model file names
- Handle naming conventions (snake_case → camelCase)
- Support both MongoDB and SQL models
- Detect database type from environment

**Implementation**:

```javascript
/**
 * @func resolveModel
 * @memberof seed.resolver
 * @desc Resolve entity name to database model.
 * @api private
 * @param {string} entityName - Entity folder name (e.g., 'mfg_company')
 * @returns {Promise<object>} Model object with schema/table access
 */
async function resolveModel(entityName) {
  const dbType = process.env.DB_CLIENT || "mongo";

  // Convert entity name to model file name
  // mfg_company -> mfg_company.model.js
  const modelName = entityName;
  const modelPath = path.join(
    __dirname,
    "..",
    "model",
    `${modelName}.model.js`
  );

  try {
    const modelModule = require(modelPath);

    if (dbType === "mongo") {
      // MongoDB: return schema/model
      return {
        type: "mongo",
        schema: modelModule.schema,
        model: modelModule.model || modelModule.schema,
      };
    } else {
      // SQL: return table name and knex instance
      const knex = require("../model/knex")();
      return {
        type: "sql",
        table: modelName,
        knex: knex,
      };
    }
  } catch (err) {
    throw new Error(`Model not found for entity: ${entityName} (${modelPath})`);
  }
}
```

#### 4. Matching Strategy Module (`seed/matcher.js`)

**Purpose**: Determine which records need to be inserted, updated, or skipped.

**Responsibilities**:

- Automatically determine matching strategy based on entity structure
- Define matching strategies (by name, by name+locale, by custom keys)
- Query existing records from database
- Compare seed data with existing records field-by-field
- Return arrays of: missing records (to insert), outdated records (to update), and unchanged records (to skip)

**Automatic Strategy Detection**:

- If entity has locale files (`{entity}.data.en.js`, etc.): Use `['key', 'locale']` as matching keys
- If entity has only `{entity}.data.js`: Use `['key']` as matching key
- Can be overridden via config file if needed

**Update Detection**:

- For each existing record, compare ALL fields (except `_id`, `id`, `created_at`, `updated_at`) with seed data
- If any field differs, mark record for update
- If all fields match, skip record (no action needed)

**Implementation**:

```javascript
/**
 * @func analyzeRecords
 * @memberof seed.matcher
 * @desc Analyze seed data against existing records to determine insert/update/skip actions.
 * @api private
 * @param {object} model - Resolved model object
 * @param {Array} seedData - Seed data array
 * @param {object} strategy - Matching strategy configuration
 * @returns {Promise<object>} Object with { toInsert, toUpdate, toSkip } arrays
 */
async function analyzeRecords(model, seedData, strategy) {
  if (model.type === "mongo") {
    return await analyzeRecordsMongo(model, seedData, strategy);
  } else {
    return await analyzeRecordsSQL(model, seedData, strategy);
  }
}

/**
 * @func analyzeRecordsMongo
 * @memberof seed.matcher
 * @desc Analyze records for MongoDB - find inserts, updates, and skips.
 * @api private
 */
async function analyzeRecordsMongo(model, seedData, strategy) {
  const { keys } = strategy;
  const schema = model.schema;

  // Get all existing records (full documents, not just keys)
  const existing = await schema.find({}).lean();

  // Build map of existing records by key
  const existingMap = new Map();
  existing.forEach((record) => {
    const recordKey = keys.map((key) => record[key]).join(":");
    existingMap.set(recordKey, record);
  });

  // Analyze each seed record
  const toInsert = [];
  const toUpdate = [];
  const toSkip = [];

  for (const seedRecord of seedData) {
    const seedKey = keys.map((key) => seedRecord[key]).join(":");
    const existingRecord = existingMap.get(seedKey);

    if (!existingRecord) {
      // Record doesn't exist - insert
      toInsert.push(seedRecord);
    } else {
      // Record exists - compare fields
      if (recordsDiffer(seedRecord, existingRecord, keys)) {
        // Fields differ - update
        // Include the _id from existing record for update
        toUpdate.push({
          _id: existingRecord._id,
          ...seedRecord,
        });
      } else {
        // Fields match - skip
        toSkip.push(seedRecord);
      }
    }
  }

  return { toInsert, toUpdate, toSkip };
}

/**
 * @func analyzeRecordsSQL
 * @memberof seed.matcher
 * @desc Analyze records for SQL - find inserts, updates, and skips.
 * @api private
 */
async function analyzeRecordsSQL(model, seedData, strategy) {
  const { table, knex } = model;
  const { keys } = strategy;

  // Get all existing records
  const existing = await knex(table).select("*");

  // Build map of existing records by key
  const existingMap = new Map();
  existing.forEach((record) => {
    const recordKey = keys.map((key) => record[key]).join(":");
    existingMap.set(recordKey, record);
  });

  // Analyze each seed record
  const toInsert = [];
  const toUpdate = [];
  const toSkip = [];

  for (const seedRecord of seedData) {
    const seedKey = keys.map((key) => seedRecord[key]).join(":");
    const existingRecord = existingMap.get(seedKey);

    if (!existingRecord) {
      // Record doesn't exist - insert
      toInsert.push(seedRecord);
    } else {
      // Record exists - compare fields
      if (recordsDiffer(seedRecord, existingRecord, keys)) {
        // Fields differ - update
        // Include the id from existing record for update
        toUpdate.push({
          id: existingRecord.id,
          ...seedRecord,
        });
      } else {
        // Fields match - skip
        toSkip.push(seedRecord);
      }
    }
  }

  return { toInsert, toUpdate, toSkip };
}

/**
 * @func recordsDiffer
 * @memberof seed.matcher
 * @desc Compare two records field-by-field to detect differences.
 * @api private
 * @param {object} seedRecord - Record from seed data
 * @param {object} existingRecord - Record from database
 * @param {Array} matchKeys - Keys used for matching (excluded from comparison)
 * @returns {boolean} True if records differ, false if identical
 */
function recordsDiffer(seedRecord, existingRecord, matchKeys) {
  // Fields to exclude from comparison
  const excludeFields = new Set([
    ...matchKeys,
    "_id", // MongoDB internal ID
    "id", // Application ID (may be same, but we compare other fields)
    "created_at", // Timestamp (preserved from original creation)
    "updated_at", // Timestamp (will be updated if record changes)
  ]);

  // Get all unique field names from both records
  const allFields = new Set([
    ...Object.keys(seedRecord),
    ...Object.keys(existingRecord),
  ]);

  // Compare each field (excluding match keys and timestamps)
  for (const field of allFields) {
    if (excludeFields.has(field)) {
      continue; // Skip comparison for these fields
    }

    const seedValue = seedRecord[field];
    const existingValue = existingRecord[field];

    // Deep comparison for objects/arrays
    if (JSON.stringify(seedValue) !== JSON.stringify(existingValue)) {
      return true; // Records differ
    }
  }

  return false; // Records are identical
}
```

#### 5. Seeder Module (`seed/seeder.js`)

**Purpose**: Execute seeding operations for entities.

**Responsibilities**:

- Load entity data
- Resolve model
- Determine matching strategy
- Find missing records
- Insert missing records
- Handle errors gracefully
- Report progress

**Implementation**:

```javascript
/**
 * @func seedEntity
 * @memberof seed.seeder
 * @desc Seed a single entity with idempotent logic.
 * @api private
 * @param {string} entityName - Entity folder name
 * @param {object} config - Optional configuration override
 * @returns {Promise<object>} Seeding result with stats
 */
async function seedEntity(entityName, config = {}) {
  const startTime = Date.now();

  try {
    // Load data (returns { data, isLocaleAware })
    const { data: seedData, isLocaleAware } = await loadEntityData(entityName);

    // Resolve model
    const model = await resolveModel(entityName);

    // Get matching strategy (auto-detects based on locale-awareness)
    const strategy = getMatchingStrategy(entityName, isLocaleAware, config);

    // Analyze records: find inserts, updates, and skips
    const { toInsert, toUpdate, toSkip } = await analyzeRecords(
      model,
      seedData,
      strategy
    );

    // Prepare records for insert (add created_at, no updated_at)
    const recordsToInsert = toInsert.map((record) => ({
      ...record,
      created_at: new Date(),
      // No updated_at on initial creation
    }));

    // Prepare records for update (add updated_at, preserve created_at from existing record)
    const recordsToUpdate = toUpdate.map((record) => {
      // Remove _id/id from update data (used only for query)
      const { _id, id, created_at, ...updateFields } = record;
      return {
        ...updateFields,
        updated_at: new Date(),
        // Note: created_at is NOT included in update - it's preserved from existing record
        // MongoDB will use _id, SQL will use id for the WHERE clause
        _id: _id || undefined, // MongoDB
        id: id || undefined, // SQL
      };
    });

    // Perform inserts
    if (recordsToInsert.length > 0) {
      if (model.type === "mongo") {
        await model.schema.insertMany(recordsToInsert);
      } else {
        await model.knex(model.table).insert(recordsToInsert);
      }
    }

    // Perform updates
    if (recordsToUpdate.length > 0) {
      if (model.type === "mongo") {
        // MongoDB: Update each record individually
        for (const record of recordsToUpdate) {
          const { _id, ...updateData } = record;
          // Update without overwriting created_at (it's preserved automatically)
          await model.schema.findByIdAndUpdate(_id, updateData, {
            new: true,
            runValidators: true,
          });
        }
      } else {
        // SQL: Update each record individually
        for (const record of recordsToUpdate) {
          const { id, ...updateData } = record;
          // Update without overwriting created_at (it's preserved automatically)
          await model.knex(model.table).where({ id }).update(updateData);
        }
      }
    }

    // Return summary
    const totalActions =
      recordsToInsert.length + recordsToUpdate.length + toSkip.length;
    if (totalActions === 0) {
      return {
        entity: entityName,
        status: "skipped",
        total: seedData.length,
        inserted: 0,
        updated: 0,
        skipped: 0,
        duration: Date.now() - startTime,
      };
    }

    return {
      entity: entityName,
      status: "success",
      total: seedData.length,
      inserted: recordsToInsert.length,
      updated: recordsToUpdate.length,
      skipped: toSkip.length,
      duration: Date.now() - startTime,
    };
  } catch (err) {
    return {
      entity: entityName,
      status: "error",
      error: err.message,
      duration: Date.now() - startTime,
    };
  }
}
```

#### 6. Main Orchestrator (`seed/all.js` - Refactored)

**Purpose**: Coordinate the entire seeding process.

**Responsibilities**:

- Discover entities
- Connect to database
- Seed each entity
- Aggregate results
- Report summary
- Handle cleanup

**Implementation**:

```javascript
/**
 * @func seedAll
 * @memberof seed.all
 * @desc Seed all entities discovered in seed directory.
 * @api public
 * @returns {Promise<void>}
 */
async function seedAll() {
  const mcode = require("mcode-log");

  require("dotenv").config();

  let dbConnection = null;

  try {
    // Discover entities
    const entities = await discoverEntities();

    if (entities.length === 0) {
      mcode.warn("No entities found in seed directory");
      return process.exit(0);
    }

    mcode.log(`Discovered ${entities.length} entities: ${entities.join(", ")}`);

    // Connect to database
    const dbType = process.env.DB_CLIENT || "mongo";
    if (dbType === "mongo") {
      const mongo = require("../model/mongo.model");
      await mongo.connect();
      dbConnection = { type: "mongo", instance: mongo };
    } else {
      // SQL connection handled per-query via knex
      dbConnection = { type: "sql" };
    }

    // Seed each entity
    const results = [];
    for (const entity of entities) {
      mcode.log(`Seeding entity: ${entity}`);
      const result = await seedEntity(entity);
      results.push(result);

      // Log result
      if (result.status === "success") {
        const actions = [];
        if (result.inserted > 0) actions.push(`${result.inserted} inserted`);
        if (result.updated > 0) actions.push(`${result.updated} updated`);
        if (result.skipped > 0) actions.push(`${result.skipped} skipped`);
        mcode.log(
          `${entity}: ${actions.join(", ")} (${result.total} total records)`
        );
      } else if (result.status === "skipped") {
        mcode.log(
          `${entity}: All ${result.total} records already exist and are up-to-date, skipped`
        );
      } else {
        mcode.error(`${entity}: Seeding failed - ${result.error}`);
      }
    }

    // Disconnect
    if (dbConnection.type === "mongo") {
      await dbConnection.instance.disconnect();
    }

    // Summary
    const summary = {
      total: results.length,
      success: results.filter((r) => r.status === "success").length,
      skipped: results.filter((r) => r.status === "skipped").length,
      errors: results.filter((r) => r.status === "error").length,
      totalInserted: results.reduce((sum, r) => sum + (r.inserted || 0), 0),
      totalUpdated: results.reduce((sum, r) => sum + (r.updated || 0), 0),
      totalSkipped: results.reduce((sum, r) => sum + (r.skipped || 0), 0),
    };

    const actionSummary = [];
    if (summary.totalInserted > 0)
      actionSummary.push(`${summary.totalInserted} inserted`);
    if (summary.totalUpdated > 0)
      actionSummary.push(`${summary.totalUpdated} updated`);
    if (summary.totalSkipped > 0)
      actionSummary.push(`${summary.totalSkipped} skipped`);

    mcode.log(
      `Seeding complete: ${summary.success} succeeded, ${
        summary.skipped
      } skipped, ${summary.errors} errors. ${actionSummary.join(", ")}.`
    );

    return process.exit(summary.errors > 0 ? 1 : 0);
  } catch (exp) {
    mcode.exp("Seeding failed", exp);

    // Cleanup
    if (dbConnection && dbConnection.type === "mongo") {
      try {
        await dbConnection.instance.disconnect();
      } catch (err) {
        // Ignore cleanup errors
      }
    }

    return process.exit(1);
  }
}

// Execute if run directly
if (require.main === module) {
  seedAll();
}

module.exports = { seedAll };
```

### Configuration System

#### Default Matching Strategies

The system will use sensible defaults based on entity name patterns:

```javascript
/**
 * @func getMatchingStrategy
 * @memberof seed.seeder
 * @desc Get matching strategy for an entity. Automatically detects locale-aware entities.
 * @api private
 * @param {string} entityName - Entity name
 * @param {boolean} isLocaleAware - Whether entity has locale-specific data files
 * @param {object} config - Optional override configuration
 * @returns {object} Matching strategy configuration
 */
function getMatchingStrategy(entityName, isLocaleAware, config = {}) {
  // Check for explicit config override
  if (config.keys) {
    return { keys: config.keys };
  }

  // Check for entity-specific config file
  const configPath = path.join(__dirname, entityName, "config.js");
  try {
    const entityConfig = require(configPath);
    if (entityConfig.matching && entityConfig.matching.keys) {
      return { keys: entityConfig.matching.keys };
    }
  } catch (err) {
    // No config file, use automatic detection
  }

  // Automatic strategy detection based on locale-awareness
  if (isLocaleAware) {
    // Entity has locale files ({entity}.data.en.js, etc.) - match by key + locale
    return { keys: ["key", "locale"] };
  } else {
    // Entity has only {entity}.data.js - match by key
    return { keys: ["key"] };
  }
}
```

#### Optional Configuration File

Entities can optionally provide a `config.js` file for custom behavior:

```javascript
// server/seed/email/config.js
module.exports = {
  matching: {
    keys: ["name", "locale"], // Match by name + locale
  },
  transform: (record) => {
    // Optional: transform record before insertion
    return {
      ...record,
      date_created: new Date(),
    };
  },
};
```

## Implementation Plan

### Phase 0: Email Entity Migration (Prerequisite)

1. **Rename Email Locale Files**

   - Rename `server/seed/email/en.js` → `server/seed/data/email.data.en.js`
   - Rename `server/seed/email/es.js` → `server/seed/data/email.data.es.js`
   - Delete `server/seed/email/data.js` (it's code, not data)

2. **Verify Email Data Structure**
   - Ensure `email.data.en.js` and `email.data.es.js` export arrays directly
   - Each record should have `name` and `locale` fields

### Phase 1: Core Infrastructure

1. **Create Discovery Module** (`seed/discover.js`)

   - Implement `discoverEntities()`
   - Add tests for directory scanning

2. **Create Data Loader Module** (`seed/loader.js`)

   - Implement `loadEntityData()` to load all `data.*.js` files
   - Automatically detect locale-aware entities
   - Merge data from all matching files
   - Add validation

3. **Create Model Resolver Module** (`seed/resolver.js`)
   - Implement `resolveModel()`
   - Support MongoDB and SQL
   - Handle model name mapping

### Phase 2: Matching and Seeding

4. **Create Matching Strategy Module** (`seed/matcher.js`)

   - Implement `analyzeRecords()` to find inserts, updates, and skips
   - Implement `recordsDiffer()` for field-by-field comparison
   - Support MongoDB and SQL queries
   - Add default strategies
   - Handle timestamp fields correctly (exclude from comparison)

5. **Create Seeder Module** (`seed/seeder.js`)

   - Implement `seedEntity()` with insert AND update logic
   - Add `created_at` timestamp to new records
   - Add `updated_at` timestamp to updated records
   - Preserve `created_at` on updates
   - Add error handling
   - Add progress reporting

### Phase 3: Orchestration

6. **Refactor Main Orchestrator** (`seed/all.js`)

   - Replace manual requires with auto-discovery
   - Integrate all modules
   - Add comprehensive logging

7. **Update Package Scripts**
   - Verify `seed:all` script works
   - Update documentation

### Phase 4: Migration and Cleanup

8. **Migrate Existing Seeders**

   - Remove `mfg_company.js`, `plc_family.js`, `plc_mfg.js`
   - Verify all entities work with new system
   - Update `mongo.seed.js` and `sql.seed.js` if needed (may be deprecated)

9. **Add Configuration Files** (Optional)
   - Add `email/config.js` if needed
   - Document configuration options

### Phase 5: Schema Timestamps (Future Refactor)

10. **Add Timestamps to All Schemas**
    - Add `created_at` and `updated_at` fields to all MongoDB schemas
    - Add `created_at` and `updated_at` columns to all SQL tables
    - Update seeding logic to set `created_at` on insert (if not present)
    - Note: This is a separate refactor affecting the entire database layer

### Phase 6: Testing and Documentation

11. **Add Tests**

    - Unit tests for each module
    - Integration tests for full seeding flow
    - Test error scenarios
    - Test locale-aware vs single-key entities
    - Test data file pattern matching

12. **Update Documentation**
    - Update README.md
    - Add seeding guide
    - Document configuration options
    - Document file naming conventions (`{entity}.data.js` vs `{entity}.data.{locale}.js`)

## Error Handling Strategy

### Per-Entity Error Isolation

Each entity is seeded independently. If one entity fails, others continue:

```javascript
for (const entity of entities) {
  try {
    const result = await seedEntity(entity);
    results.push(result);
  } catch (err) {
    results.push({
      entity,
      status: "error",
      error: err.message,
    });
    // Continue with next entity
  }
}
```

### Database Connection Errors

- MongoDB: Attempt reconnection with exponential backoff
- SQL: Handle connection pool errors gracefully
- Both: Log errors but continue with other entities if possible

### Data Validation Errors

- Validate data structure (must be array)
- Validate required fields exist
- Skip invalid records with warning (don't fail entire entity)

## Testing Strategy

### Unit Tests

#### `test/seed/discover.test.js`

```javascript
describe("seed/discover", () => {
  it("should discover entity folders", async () => {
    const entities = await discoverEntities();
    expect(entities).to.include("email");
    expect(entities).to.include("mfg_company");
  });

  it("should exclude .js files", async () => {
    const entities = await discoverEntities();
    expect(entities).to.not.include("all.js");
  });
});
```

#### `test/seed/loader.test.js`

```javascript
describe("seed/loader", () => {
  it("should load entity data", async () => {
    const data = await loadEntityData("email");
    expect(data).to.be.an("array");
    expect(data.length).to.be.greaterThan(0);
  });

  it("should throw if {entity}.data.js not found", async () => {
    await expect(loadEntityData("nonexistent")).to.be.rejected;
  });

  it("should throw if data is not array", async () => {
    // Create test entity with invalid data
    await expect(loadEntityData("invalid")).to.be.rejected;
  });
});
```

#### `test/seed/resolver.test.js`

```javascript
describe("seed/resolver", () => {
  it("should resolve MongoDB model", async () => {
    process.env.DB_CLIENT = "mongo";
    const model = await resolveModel("email");
    expect(model.type).to.equal("mongo");
    expect(model.schema).to.exist;
  });

  it("should resolve SQL model", async () => {
    process.env.DB_CLIENT = "mysql";
    const model = await resolveModel("email");
    expect(model.type).to.equal("sql");
    expect(model.table).to.equal("email");
  });

  it("should throw if model not found", async () => {
    await expect(resolveModel("nonexistent")).to.be.rejected;
  });
});
```

#### `test/seed/matcher.test.js`

```javascript
describe("seed/matcher", () => {
  it("should find missing records for MongoDB", async () => {
    const model = { type: "mongo", schema: EmailSchema };
    const seedData = [{ name: "test", locale: "en" }];
    const strategy = { keys: ["name", "locale"] };

    const missing = await findMissingRecords(model, seedData, strategy);
    expect(missing).to.be.an("array");
  });

  it("should find missing records for SQL", async () => {
    const model = { type: "sql", table: "email", knex: testKnex };
    const seedData = [{ name: "test", locale: "en" }];
    const strategy = { keys: ["name", "locale"] };

    const missing = await findMissingRecords(model, seedData, strategy);
    expect(missing).to.be.an("array");
  });
});
```

#### `test/seed/seeder.test.js`

```javascript
describe("seed/seeder", () => {
  it("should seed entity successfully", async () => {
    const result = await seedEntity("email");
    expect(result.status).to.equal("success");
    expect(result.inserted).to.be.a("number");
  });

  it("should skip if all records exist and are unchanged", async () => {
    // Seed twice with same data
    await seedEntity("email");
    const result = await seedEntity("email");
    expect(result.status).to.equal("skipped");
    expect(result.inserted).to.equal(0);
    expect(result.updated).to.equal(0);
  });

  it("should update records if fields differ", async () => {
    // Seed initial data
    await seedEntity("email");
    // Modify seed data file
    // Seed again
    const result = await seedEntity("email");
    expect(result.status).to.equal("success");
    expect(result.updated).to.be.greaterThan(0);
  });

  it("should handle errors gracefully", async () => {
    const result = await seedEntity("nonexistent");
    expect(result.status).to.equal("error");
    expect(result.error).to.exist;
  });
});
```

### Integration Tests

#### `test/seed/all.test.js`

```javascript
describe("seed/all", () => {
  before(async () => {
    // Setup test database
  });

  after(async () => {
    // Cleanup test database
  });

  it("should seed all entities", async () => {
    await seedAll();
    // Verify all entities were seeded
  });

  it("should handle partial failures", async () => {
    // Create entity with invalid data
    // Verify other entities still seed
  });

  it("should be idempotent", async () => {
    await seedAll();
    const firstRun = await getRecordCounts();

    await seedAll();
    const secondRun = await getRecordCounts();

    expect(firstRun).to.deep.equal(secondRun);
  });

  it("should update records when seed data changes", async () => {
    await seedAll();
    const firstRun = await getRecordCounts();

    // Modify seed data
    // ...

    await seedAll();
    const secondRun = await getRecordCounts();

    // Record counts should be same, but updated_at timestamps should change
    expect(firstRun.total).to.equal(secondRun.total);
    // Verify updated_at changed for modified records
  });
});
```

## Migration Path

### Step 1: Implement New System (Parallel)

- Create new modules alongside existing seeders
- Test with one entity (e.g., `email`)
- Verify functionality

### Step 2: Migrate Entities One-by-One

- Update `all.js` to use new system for `email`
- Keep old seeders for other entities
- Test thoroughly

### Step 3: Complete Migration

- Migrate all entities to new system
- Remove old seeder files (`mfg_company.js`, etc.)
- Update documentation

### Step 4: Cleanup

- Remove unused code
- Update `mongo.seed.js` and `sql.seed.js` if needed
- Final testing

## Edge Cases and Considerations

### 1. Entity Name Mapping

**Issue**: Entity folder name may not match model file name.

**Solution**:

- Default: `{entityName}.model.js`
- Support mapping configuration if needed
- Document naming conventions

### 2. Complex Matching Strategies

**Issue**: Some entities may need complex matching (e.g., multiple fields, computed keys).

**Solution**:

- Support custom matching functions in config
- Allow per-entity configuration files
- Document advanced usage

### 3. Large Datasets

**Issue**: Loading all records into memory may be inefficient.

**Solution**:

- Process in batches if dataset is large (>1000 records)
- Add batching configuration
- Monitor memory usage

### 4. Transaction Support

**Issue**: Should seeding be transactional?

**Solution**:

- Per-entity transactions (if one entity fails, others succeed)
- Optional: Full transaction mode (all-or-nothing)
- Configuration option

### 5. Data Transformation

**Issue**: Some records may need transformation before insertion.

**Solution**:

- Support `transform` function in config
- Apply transformation in seeder module
- Document transformation patterns

### 6. Locale-Specific Data

**Issue**: Some entities have locale-specific files (e.g., `email.data.en.js`, `email.data.es.js`).

**Solution**:

- Loader module automatically detects locale files matching pattern `{entity}.data.{locale}.js`
- All `{entity}.data.*.js` files (including `{entity}.data.js`) are loaded and merged from `server/seed/data/` folder
- Matching strategy automatically uses `['key', 'locale']` for locale-aware entities
- Old `email/data.js` (code file) is eliminated
- Old `email/en.js` and `email/es.js` are renamed to `data/email.data.en.js` and `data/email.data.es.js`

## Performance Considerations

### Database Connection

- MongoDB: Single connection for all entities
- SQL: Connection pool managed by Knex
- Reuse connections across entities

### Query Optimization

- Use projections/selects to minimize data transfer
- Batch inserts when possible
- Index matching fields for faster lookups

### Memory Management

- Process entities sequentially (not in parallel)
- Clear large arrays after processing
- Monitor memory usage in tests

## Security Considerations

### Input Validation

- Validate entity names (prevent path traversal)
- Validate data structure
- Sanitize input before database operations

### Database Access

- Use parameterized queries (SQL)
- Use Mongoose validation (MongoDB)
- Respect account_id scoping if applicable

### Error Messages

- Don't expose internal paths in errors
- Log detailed errors server-side
- Return user-friendly messages

## Documentation Requirements

### README Updates

1. **Seeding Guide Section**

   - How to add new entities
   - Configuration options
   - Matching strategies
   - Troubleshooting

2. **API Documentation**

   - JSDoc for all public functions
   - Configuration file format
   - Example configurations

3. **Migration Guide**
   - How to migrate from old system
   - Breaking changes (if any)
   - Compatibility notes

## Success Criteria

1. ✅ **Auto-Discovery**: System automatically finds all entity folders
2. ✅ **Zero Configuration**: Works with sensible defaults
3. ✅ **Idempotent**: Can run multiple times without duplicates
4. ✅ **Database Agnostic**: Works with MongoDB and SQL
5. ✅ **Error Resilient**: Continues on partial failures
6. ✅ **Extensible**: Easy to add new entities
7. ✅ **Well Tested**: Comprehensive test coverage
8. ✅ **Documented**: Clear documentation for users

## Open Questions

1. **Naming Convention**: Should entity folder names match model file names exactly, or support mapping?

   - **Decision**: Support exact match by default, allow mapping in config if needed

2. **Locale Files**: How should locale-specific files be handled?

   - **Decision**: Files matching `{entity}.data.{locale}.js` pattern are automatically detected. All `{entity}.data.*.js` files (including `{entity}.data.js`) are loaded and merged from `server/seed/data/` folder. Matching strategy automatically uses `['key', 'locale']` for locale-aware entities.

3. **Custom Seeding Logic**: Should we support custom seeding functions?

   - **Decision**: Phase 2 feature - support transform functions first

4. **Batch Processing**: Should large datasets be processed in batches?

   - **Decision**: Monitor performance, add batching if needed

5. **Transaction Mode**: Should we support full transaction mode?

   - **Decision**: Per-entity transactions by default, full transaction as optional config

6. **Schema Timestamps**: When should `created_at` and `updated_at` be added?

   - **Decision**: Seeding system will automatically add `created_at` on insert and `updated_at` on update. If schemas don't have these fields yet, they will be added as part of the database refactor, but seeding will work regardless (fields will be stored if schema allows).

7. **Prefix Validation**: Should the seeding system validate that prefixes in data files match the authoritative Entity-Prefix Mapping table?

   - **Decision**: **YES - STRICT ENFORCEMENT REQUIRED**
     - Loader **MUST** validate prefix against authoritative mapping table
     - Warn if prefix doesn't match expected prefix for entity name
     - Fail if prefix is missing or invalid
     - Cache module maintains authoritative mapping - single source of truth
     - Prevents data corruption from incorrect prefixes
     - Ensures foreign key resolution works correctly

8. **Prefix-to-Entity Mapping Updates**: How should new entities be added to the prefix mapping table?
   - **Decision**:
     - New entities **MUST** be added to `buildPrefixToEntityMapping()` in `cache.js` before seeding
     - Prefix must follow established patterns (typically 4-5 characters with trailing underscore)
     - Entity name must match model file name exactly
     - Foreign key field pattern: `{entity}_id` (e.g., `account_id`, `boat_brand_id`)
     - Foreign key reference pattern: `{prefix}_key` (e.g., `acct_key`, `bbrn_key`)
     - Document new entity in Entity-Prefix Mapping Reference section

---

## Data Layout Simplification Refactoring

### Overview

This refactoring simplifies the seed data file structure by consolidating all data files into a single `data/` folder with a consistent naming pattern. This eliminates the need for per-entity folders and makes the data structure more maintainable and discoverable.

### Current State

**Current Structure:**

```
server/seed/
├── all.js
├── discover.js
├── loader.js
├── matcher.js
├── resolver.js
├── seeder.js
├── cache.js
├── email/
│   ├── data.en.js
│   └── data.es.js
├── boat/
│   └── data.js
├── boat_brand/
│   └── data.js
├── account/
│   └── data.js
├── user/
│   └── data.js
└── sv/
    └── data.js
```

**Current File Patterns:**

- `{entity}/data.js` - Single-key entity data
- `{entity}/data.{locale}.js` - Locale-specific data (e.g., `data.en.js`, `data.es.js`)
- `{entity}/wait.js` - Alternative name for data files (same structure as `data.js`)

**Current Discovery Logic:**

- `discover.js` scans `server/seed/` for directories
- Filters out non-entity files and hidden directories
- Returns array of entity folder names

**Current Loading Logic:**

- `loader.js` receives entity name
- Reads files from `server/seed/{entityName}/` folder
- Filters for files matching `data.js` or `data.*.js` pattern
- Loads and merges all matching files

### Desired State

**Target Structure:**

```
server/seed/
├── all.js
├── discover.js
├── loader.js
├── matcher.js
├── resolver.js
├── seeder.js
├── cache.js
└── data/
    ├── email.data.en.js
    ├── email.data.es.js
    ├── boat.data.js
    ├── boat_brand.data.js
    ├── account.data.js
    ├── user.data.js
    └── sv.data.js
```

**Target File Patterns:**

- `data/{entity}.data.js` - Single-key entity data
- `data/{entity}.data.{locale}.js` - Locale-specific data (e.g., `email.data.en.js`, `email.data.es.js`)
  - Supported locale codes: `en` (English), `es` (Spanish), `fr` (French), `dt` (German), `it` (Italian), or any 2-letter locale code `{cc}`

**Key Changes:**

1. All data files consolidated into single `data/` folder
2. Consistent naming: `{entity}.data.{locale}.js` or `{entity}.data.js`
3. Entity folders eliminated (removed after migration)
4. Discovery logic changed: discover entities from filenames instead of folders
5. Loading logic changed: load from `data/` folder with new naming pattern

### Design Architecture

#### 1. Discovery Module Refactoring (`seed/discover.js`)

**Current Implementation:**

- Scans `server/seed/` for directories
- Returns array of entity folder names

**New Implementation:**

- Scan `server/seed/data/` folder for data files
- Extract entity names from filenames matching pattern `{entity}.data.{locale}.js` or `{entity}.data.js`
- Return unique array of entity names

**Implementation Details:**

```javascript
/**
 * @func discoverEntities
 * @memberof seed.discover
 * @desc Discover all entities by scanning data files in server/seed/data/ folder.
 * Extracts entity names from filenames matching {entity}.data.{locale}.js or {entity}.data.js pattern.
 * @api private
 * @returns {Promise<Array<string>>} Array of unique entity names
 */
async function discoverEntities() {
  const dataDir = path.join(__dirname, "data");

  // Check if data directory exists
  try {
    await fs.access(dataDir);
  } catch (err) {
    throw new Error(
      `Seed data directory not found: ${dataDir}. Please ensure data files are in server/seed/data/`
    );
  }

  // Read all files in data directory
  const files = await fs.readdir(dataDir);

  // Extract entity names from filenames
  // Pattern 1: {entity}.data.{locale}.js (e.g., email.data.en.js)
  // Pattern 2: {entity}.data.js (e.g., boat.data.js)
  const entitySet = new Set();

  for (const file of files) {
    // Skip non-JS files
    if (!file.endsWith(".js")) {
      continue;
    }

    // Match pattern: {entity}.data.{locale}.js
    const localeMatch = file.match(/^(.+)\.data\.([a-z]{2})\.js$/);
    if (localeMatch) {
      const entityName = localeMatch[1];
      entitySet.add(entityName);
      continue;
    }

    // Match pattern: {entity}.data.js
    const simpleMatch = file.match(/^(.+)\.data\.js$/);
    if (simpleMatch) {
      const entityName = simpleMatch[1];
      entitySet.add(entityName);
      continue;
    }

    // Warn about files that don't match expected pattern
    mcode.warn(
      `Seed data file does not match expected pattern: ${file} (expected {entity}.data.js or {entity}.data.{locale}.js)`
    );
  }

  const entities = Array.from(entitySet).sort();
  return entities;
}
```

**Key Features:**

- Scans `server/seed/data/` folder instead of entity folders
- Extracts entity names from filenames using regex patterns
- Handles both locale-aware (`{entity}.data.{locale}.js`) and single-key (`{entity}.data.js`) files
- Returns unique, sorted list of entities
- Warns about files that don't match expected patterns
- Throws error if `data/` folder doesn't exist

#### 2. Data Loader Module Refactoring (`seed/loader.js`)

**Current Implementation:**

- Receives entity name
- Reads files from `server/seed/{entityName}/` folder
- Filters for `data.js` or `data.*.js` files
- Loads and merges data

**New Implementation:**

- Receives entity name
- Reads files from `server/seed/data/` folder
- Filters for files matching `{entityName}.data.{locale}.js` or `{entityName}.data.js`
- Loads and merges data

**Implementation Details:**

```javascript
/**
 * @func loadEntityData
 * @memberof seed.loader
 * @desc Load seed data for an entity from server/seed/data/ folder.
 * Loads all files matching {entityName}.data.{locale}.js or {entityName}.data.js pattern.
 * Automatically detects locale-aware entities by presence of {entityName}.data.{locale}.js files.
 * @api private
 * @param {string} entityName - Entity name
 * @returns {Promise<object>} Object containing { data: Array, isLocaleAware: boolean, prefix: string|null }
 * @throws {Error} If no data files found or if data files don't export arrays or objects with arrays
 */
async function loadEntityData(entityName) {
  const dataDir = path.join(__dirname, "data");

  // Read all files in data directory
  const files = await fs.readdir(dataDir);

  // Filter for files matching {entityName}.data.{locale}.js or {entityName}.data.js
  const dataFiles = files.filter((file) => {
    // Match locale-aware: {entityName}.data.{locale}.js
    const localeMatch = file.match(/^(.+)\.data\.([a-z]{2})\.js$/);
    if (localeMatch && localeMatch[1] === entityName) {
      return true;
    }

    // Match simple: {entityName}.data.js
    const simpleMatch = file.match(/^(.+)\.data\.js$/);
    if (simpleMatch && simpleMatch[1] === entityName) {
      return true;
    }

    return false;
  });

  if (dataFiles.length === 0) {
    throw new Error(
      `No data files found for entity: ${entityName} ` +
        `(expected ${entityName}.data.js or ${entityName}.data.{locale}.js in server/seed/data/)`
    );
  }

  // Check if locale-aware (has {entityName}.data.{locale}.js files)
  const localeFiles = dataFiles.filter((file) =>
    file.match(new RegExp(`^${entityName}\\.data\\.[a-z]{2}\\.js$`))
  );
  const isLocaleAware = localeFiles.length > 0;

  // Load and merge all data files
  let allData = [];
  let entityPrefix = null;

  for (const file of dataFiles) {
    const filePath = path.join(dataDir, file);
    const fileData = require(filePath);

    // Handle format: { prefix, data } or { prefix, [entityName] }
    if (
      typeof fileData === "object" &&
      fileData !== null &&
      !Array.isArray(fileData)
    ) {
      // Extract prefix if present
      if (fileData.prefix && !entityPrefix) {
        entityPrefix = fileData.prefix;
      }

      // Extract data array - could be in 'data' property or entity name property
      let dataArray = fileData.data;
      if (!dataArray) {
        // Try to find array property (entity name like 'boat_brand', 'plc_family', etc.)
        const arrayProps = Object.keys(fileData).filter(
          (key) => Array.isArray(fileData[key]) && key !== "prefix"
        );
        if (arrayProps.length > 0) {
          dataArray = fileData[arrayProps[0]];
        }
      }

      if (!Array.isArray(dataArray)) {
        throw new Error(
          `Entity ${entityName}: ${file} must export an array or object with array property`
        );
      }

      allData = [...allData, ...dataArray];
    }
    // Handle old format: direct array export
    else if (Array.isArray(fileData)) {
      allData = [...allData, ...fileData];
    } else {
      throw new Error(
        `Entity ${entityName}: ${file} must export an array or object with prefix and data`
      );
    }
  }

  if (allData.length === 0) {
    mcode.warn(`Entity ${entityName}: No data found in data files`);
  }

  // Validate: prefix must exist
  if (!entityPrefix) {
    throw new Error(
      `Entity ${entityName}: Data file must export a 'prefix' property`
    );
  }

  // Validate: prefix must match authoritative Entity-Prefix Mapping table
  const { buildPrefixToEntityMapping } = require("./cache");
  const prefixMapping = buildPrefixToEntityMapping();

  // Validate prefix length (must be exactly 4 characters)
  if (entityPrefix.length !== 4) {
    throw new Error(
      `Entity ${entityName}: Prefix '${entityPrefix}' must be exactly 4 characters. ` +
        `Found ${entityPrefix.length} characters. Valid prefixes are documented in _entity.js.`
    );
  }

  // Prefixes are exactly 4 characters - use as-is for lookup
  const prefixKey = entityPrefix;

  // Check if prefix maps to this entity name
  const mappedEntity = prefixMapping[prefixKey];
  if (!mappedEntity) {
    throw new Error(
      `Entity ${entityName}: Prefix '${prefixKey}' is not found in authoritative Entity-Prefix Mapping table (_entity.js). ` +
        `Please ensure the prefix matches the mapping in server/seed/_entity.js. ` +
        `Valid prefixes are documented in the Entity-Prefix Mapping Reference section.`
    );
  }

  if (mappedEntity !== entityName) {
    throw new Error(
      `Entity ${entityName}: Prefix '${prefixKey}' maps to entity '${mappedEntity}' in authoritative mapping table, ` +
        `but data file is for entity '${entityName}'. Prefix must match entity name. ` +
        `Expected prefix for '${entityName}' should map to '${entityName}' in _entity.js.`
    );
  }

  // Validate: all records must have a 'key' field
  for (let i = 0; i < allData.length; i++) {
    const record = allData[i];
    if (!record || typeof record !== "object") {
      throw new Error(
        `Entity ${entityName}: Record at index ${i} must be an object`
      );
    }
    if (!record.key || typeof record.key !== "string") {
      throw new Error(
        `Entity ${entityName}: Record at index ${i} must have a 'key' field (string)`
      );
    }
  }

  return {
    data: allData,
    isLocaleAware: isLocaleAware,
    prefix: entityPrefix,
  };
}
```

**Key Changes:**

- Changed path from `server/seed/{entityName}/` to `server/seed/data/`
- Changed file pattern matching from `data.js`/`data.*.js` to `{entityName}.data.{locale}.js`/`{entityName}.data.js`
- Locale detection now checks for `{entityName}.data.{locale}.js` pattern
- **Prefix validation added**: Validates prefix against authoritative Entity-Prefix Mapping table
- **Strict enforcement**: Throws error if prefix doesn't match expected prefix for entity name
- **Handles underscore padding**: Correctly processes prefixes like `log_` and `plc_` (4 characters with underscore padding)
- Error messages updated to reflect new file locations and provide guidance for fixing prefix issues

#### 3. Migration Script Requirements

**Migration Steps:**

1. **Create `data/` folder** if it doesn't exist
2. **Discover all data files** in entity folders:
   - Scan all entity folders for `data.js` and `data.{locale}.js` files
3. **Rename and move files**:
   - `{entity}/data.js` → `data/{entity}.data.js`
   - `{entity}/data.{locale}.js` → `data/{entity}.data.{locale}.js`
4. **Remove empty entity folders** after migration
5. **Validate migration**:
   - Verify all files moved successfully
   - Verify no data loss
   - Test discovery and loading with new structure

**Migration Script Structure:**

```javascript
/**
 * Migration script to consolidate seed data files into data/ folder
 * Run once before deploying updated seeding system
 */
async function migrateSeedData() {
  const seedDir = path.join(__dirname);
  const dataDir = path.join(seedDir, "data");

  // Create data directory if it doesn't exist
  await fs.mkdir(dataDir, { recursive: true });

  // Discover all entity folders
  const entries = await fs.readdir(seedDir, { withFileTypes: true });
  const entityFolders = entries
    .filter((entry) => entry.isDirectory())
    .map((entry) => entry.name)
    .filter((name) => !name.startsWith("."));

  const migrationLog = [];

  for (const entityFolder of entityFolders) {
    const entityPath = path.join(seedDir, entityFolder);
    const files = await fs.readdir(entityPath);

    // Find data files
    const dataFiles = files.filter(
      (file) =>
        file === "data.js" ||
        file.match(/^data\.[a-z]{2}\.js$/) ||
        file === "wait.js"
    );

    if (dataFiles.length === 0) {
      continue; // No data files in this folder
    }

    // Handle data.js and wait.js conflict
    const hasDataJs = dataFiles.includes("data.js");
    const hasWaitJs = dataFiles.includes("wait.js");

    if (hasDataJs && hasWaitJs) {
      mcode.warn(
        `${entityFolder}: Both data.js and wait.js found, using data.js and ignoring wait.js`
      );
      dataFiles.splice(dataFiles.indexOf("wait.js"), 1);
    }

    // Migrate each file
    for (const file of dataFiles) {
      const sourcePath = path.join(entityPath, file);
      let targetFileName;

      if (file === "wait.js") {
        targetFileName = `${entityFolder}.data.js`;
      } else if (file === "data.js") {
        targetFileName = `${entityFolder}.data.js`;
      } else {
        // data.{locale}.js -> {entity}.data.{locale}.js
        const localeMatch = file.match(/^data\.([a-z]{2})\.js$/);
        if (localeMatch) {
          targetFileName = `${entityFolder}.data.${localeMatch[1]}.js`;
        } else {
          mcode.warn(`${entityFolder}/${file}: Unexpected file name, skipping`);
          continue;
        }
      }

      const targetPath = path.join(dataDir, targetFileName);

      // Check for conflicts
      try {
        await fs.access(targetPath);
        mcode.warn(
          `${entityFolder}/${file}: Target file already exists: ${targetPath}, skipping`
        );
        continue;
      } catch (err) {
        // File doesn't exist, safe to move
      }

      // Move file
      await fs.copyFile(sourcePath, targetPath);
      await fs.unlink(sourcePath);
      migrationLog.push(
        `Moved: ${entityFolder}/${file} → data/${targetFileName}`
      );
    }

    // Remove empty entity folder
    const remainingFiles = await fs.readdir(entityPath);
    if (remainingFiles.length === 0) {
      await fs.rmdir(entityPath);
      migrationLog.push(`Removed empty folder: ${entityFolder}/`);
    }
  }

  mcode.info("Migration complete. Summary:");
  migrationLog.forEach((log) => mcode.info(`  ${log}`));
}
```

### Testing Strategy

#### Unit Tests

**`test/seed/discover.test.js` - Updated Tests:**

```javascript
const { discoverEntities } = require("../../server/seed/discover");
const fs = require("fs").promises;
const path = require("path");

describe("seed/discover", () => {
  const seedDir = path.join(__dirname, "../../server/seed");
  const dataDir = path.join(seedDir, "data");

  beforeEach(async () => {
    // Create data directory if it doesn't exist
    await fs.mkdir(dataDir, { recursive: true });
  });

  afterEach(async () => {
    // Cleanup: remove test data files
    try {
      const files = await fs.readdir(dataDir);
      for (const file of files) {
        if (file.startsWith("test_")) {
          await fs.unlink(path.join(dataDir, file));
        }
      }
    } catch (err) {
      // Ignore cleanup errors
    }
  });

  it("should discover entities from data files", async () => {
    // Create test data files
    await fs.writeFile(
      path.join(dataDir, "test_entity.js"),
      'module.exports = { prefix: "test", test_entity: [] };'
    );
    await fs.writeFile(
      path.join(dataDir, "test_entity2.data.js"),
      'module.exports = { prefix: "test2", test_entity2: [] };'
    );

    const entities = await discoverEntities();

    expect(entities).to.include("test_entity");
    expect(entities).to.include("test_entity2");
  });

  it("should discover locale-aware entities", async () => {
    // Create locale-aware test files
    await fs.writeFile(
      path.join(dataDir, "test_email.data.en.js"),
      'module.exports = { prefix: "mail", emails: [] };'
    );
    await fs.writeFile(
      path.join(dataDir, "test_email.data.es.js"),
      'module.exports = { prefix: "mail", emails: [] };'
    );

    const entities = await discoverEntities();

    expect(entities).to.include("test_email");
    expect(entities.length).to.equal(1); // Should only appear once
  });

  it("should ignore files that do not match pattern", async () => {
    // Create non-matching files
    await fs.writeFile(
      path.join(dataDir, "random_file.js"),
      "module.exports = {};"
    );
    await fs.writeFile(
      path.join(dataDir, "test_entity.js"), // Missing .data.
      "module.exports = {};"
    );

    const entities = await discoverEntities();

    expect(entities).to.not.include("random_file");
    expect(entities).to.not.include("test_entity");
  });

  it("should throw error if data directory does not exist", async () => {
    // Temporarily rename data directory
    const backupDir = path.join(seedDir, "data_backup");
    try {
      await fs.rename(dataDir, backupDir);

      await expect(discoverEntities()).to.be.rejectedWith(
        /Seed data directory not found/
      );
    } finally {
      // Restore data directory
      try {
        await fs.rename(backupDir, dataDir);
      } catch (err) {
        // Ignore restore errors
      }
    }
  });

  it("should return sorted unique entity names", async () => {
    // Create multiple files for same entity
    await fs.writeFile(
      path.join(dataDir, "zebra.data.js"),
      'module.exports = { prefix: "z", zebra: [] };'
    );
    await fs.writeFile(
      path.join(dataDir, "zebra.data.en.js"),
      'module.exports = { prefix: "z", zebra: [] };'
    );
    await fs.writeFile(
      path.join(dataDir, "alpha.data.js"),
      'module.exports = { prefix: "a", alpha: [] };'
    );

    const entities = await discoverEntities();

    expect(entities).to.deep.equal(["alpha", "zebra"]);
    expect(entities.length).to.equal(2); // No duplicates
  });
});
```

**`test/seed/loader.test.js` - Updated Tests:**

```javascript
const { loadEntityData } = require("../../server/seed/loader");
const fs = require("fs").promises;
const path = require("path");

describe("seed/loader", () => {
  const seedDir = path.join(__dirname, "../../server/seed");
  const dataDir = path.join(seedDir, "data");

  beforeEach(async () => {
    await fs.mkdir(dataDir, { recursive: true });
  });

  afterEach(async () => {
    // Cleanup test files
    try {
      const files = await fs.readdir(dataDir);
      for (const file of files) {
        if (file.startsWith("test_")) {
          await fs.unlink(path.join(dataDir, file));
        }
      }
    } catch (err) {
      // Ignore cleanup errors
    }
  });

  it("should load entity data from data folder", async () => {
    await fs.writeFile(
      path.join(dataDir, "test_entity.js"),
      'module.exports = { prefix: "test", test_entity: [{ key: "test1" }, { key: "test2" }] };'
    );

    const result = await loadEntityData("test_entity");

    expect(result.data).to.be.an("array");
    expect(result.data.length).to.equal(2);
    expect(result.prefix).to.equal("test");
    expect(result.isLocaleAware).to.be.false;
  });

  it("should load and merge locale-aware data files", async () => {
    await fs.writeFile(
      path.join(dataDir, "test_email.data.en.js"),
      'module.exports = { prefix: "mail", emails: [{ key: "en1", locale: "en" }] };'
    );
    await fs.writeFile(
      path.join(dataDir, "test_email.data.es.js"),
      'module.exports = { prefix: "mail", emails: [{ key: "es1", locale: "es" }] };'
    );

    const result = await loadEntityData("test_email");

    expect(result.data.length).to.equal(2);
    expect(result.isLocaleAware).to.be.true;
    expect(result.prefix).to.equal("mail");
  });

  it("should throw if no data files found", async () => {
    await expect(loadEntityData("nonexistent")).to.be.rejectedWith(
      /No data files found for entity: nonexistent/
    );
  });

  it("should throw if prefix missing", async () => {
    await fs.writeFile(
      path.join(dataDir, "test_entity.js"),
      'module.exports = { test_entity: [{ key: "test1" }] };'
    );

    await expect(loadEntityData("test_entity")).to.be.rejectedWith(
      /Data file must export a 'prefix' property/
    );
  });

  it("should throw if record missing key field", async () => {
    await fs.writeFile(
      path.join(dataDir, "test_entity.js"),
      'module.exports = { prefix: "test", test_entity: [{ name: "test1" }] };'
    );

    await expect(loadEntityData("test_entity")).to.be.rejectedWith(
      /must have a 'key' field/
    );
  });

  it("should validate prefix against authoritative mapping table", async () => {
    // Test with valid prefix for account entity (4 characters)
    await fs.writeFile(
      path.join(dataDir, "account.data.js"),
      'module.exports = { prefix: "acct", account: [{ key: "test1" }] };'
    );

    const result = await loadEntityData("account");
    expect(result.prefix).to.equal("acct");
  });

  it("should throw if prefix not found in mapping table", async () => {
    await fs.writeFile(
      path.join(dataDir, "test_entity.data.js"),
      'module.exports = { prefix: "inva", test_entity: [{ key: "test1" }] };'
    );

    await expect(loadEntityData("test_entity")).to.be.rejectedWith(
      /is not found in authoritative Entity-Prefix Mapping table/
    );
  });

  it("should throw if prefix maps to different entity", async () => {
    // Try to use 'acct' prefix for 'user' entity (should fail)
    await fs.writeFile(
      path.join(dataDir, "user.data.js"),
      'module.exports = { prefix: "acct", user: [{ key: "test1" }] };'
    );

    await expect(loadEntityData("user")).to.be.rejectedWith(
      /maps to entity 'account' in authoritative mapping table/
    );
  });

  it("should handle prefixes with underscore padding correctly", async () => {
    // Test with log_ prefix (4 chars, underscore is padding)
    await fs.writeFile(
      path.join(dataDir, "log.data.js"),
      'module.exports = { prefix: "log_", log: [{ key: "test1" }] };'
    );

    const result = await loadEntityData("log");
    expect(result.prefix).to.equal("log_");
  });

  it("should handle 4-character prefixes without underscores correctly", async () => {
    // Test with acct prefix (4 chars, no underscore)
    await fs.writeFile(
      path.join(dataDir, "account.data.js"),
      'module.exports = { prefix: "acct", account: [{ key: "test1" }] };'
    );

    const result = await loadEntityData("account");
    expect(result.prefix).to.equal("acct");
  });
});
```

#### Integration Tests

**`test/seed/all.test.js` - Updated Tests:**

```javascript
describe("seed/all", () => {
  before(async () => {
    // Setup: Ensure data directory exists with test files
    const dataDir = path.join(__dirname, "../../server/seed/data");
    await fs.mkdir(dataDir, { recursive: true });
  });

  after(async () => {
    // Cleanup: Remove test data files
    const dataDir = path.join(__dirname, "../../server/seed/data");
    try {
      const files = await fs.readdir(dataDir);
      for (const file of files) {
        if (file.startsWith("test_")) {
          await fs.unlink(path.join(dataDir, file));
        }
      }
    } catch (err) {
      // Ignore cleanup errors
    }
  });

  it("should seed all entities from data folder", async () => {
    // Create test data files
    await fs.writeFile(
      path.join(dataDir, "test_entity1.data.js"),
      'module.exports = { prefix: "test1", test_entity1: [{ key: "key1" }] };'
    );
    await fs.writeFile(
      path.join(dataDir, "test_entity2.data.js"),
      'module.exports = { prefix: "test2", test_entity2: [{ key: "key2" }] };'
    );

    await seedAll();

    // Verify entities were seeded
    // (Implementation depends on test database setup)
  });

  it("should handle locale-aware entities correctly", async () => {
    await fs.writeFile(
      path.join(dataDir, "test_email.data.en.js"),
      'module.exports = { prefix: "mail", emails: [{ key: "en1", locale: "en" }] };'
    );
    await fs.writeFile(
      path.join(dataDir, "test_email.data.es.js"),
      'module.exports = { prefix: "mail", emails: [{ key: "es1", locale: "es" }] };'
    );

    await seedAll();

    // Verify both locales were seeded
  });
});
```

### Migration Plan

#### Phase 1: Preparation

1. Review all existing data files
2. Identify any special cases or conflicts
3. Create backup of current structure
4. Document current file locations

#### Phase 2: Implementation

1. Update `discover.js` to scan `data/` folder
2. Update `loader.js` to load from `data/` folder with new naming
3. Create migration script
4. Run migration script in development environment
5. Verify all entities discovered correctly
6. Verify all data loads correctly

#### Phase 3: Testing

1. Run unit tests for `discover.js` and `loader.js`
2. Run integration tests for full seeding flow
3. Verify backward compatibility (if needed during transition)
4. Test with both MongoDB and SQL databases

#### Phase 4: Cleanup

1. Remove empty entity folders
2. Remove migration script (or archive it)
3. Update documentation
4. Update any CI/CD scripts that reference old structure

### Risk Assessment

**Low Risk Areas:**

- File discovery logic (straightforward pattern matching)
- File loading logic (minimal changes to path resolution)
- Data structure remains unchanged (only file location changes)

**Medium Risk Areas:**

- Migration script must handle edge cases (conflicts, missing files)
- Need to ensure no data loss during migration
- Need to verify all entities are discovered correctly

**Mitigation Strategies:**

1. Create comprehensive backup before migration
2. Test migration script on development environment first
3. Run full test suite after migration
4. Keep migration script available for rollback if needed
5. Document rollback procedure

### Edge Cases and Considerations

1. **File Name Conflicts:**

   - Migration script handles standard `data.js` and `data.{locale}.js` files only
   - No special handling needed for temporary files (all have been renamed to `data.js`)

2. **Missing Data Directory:**

   - Throw clear error message with instructions
   - Provide helpful error message in discovery

3. **Invalid File Names:**

   - Warn about files that don't match expected pattern
   - Don't fail entire discovery, just skip invalid files

4. **Empty Data Directory:**

   - Return empty array (no entities found)
   - Log warning message

5. **Case Sensitivity:**

   - File system case sensitivity may vary
   - Use consistent casing in patterns (lowercase for locale codes)

6. **Special Characters in Entity Names:**
   - Entity names should be valid JavaScript identifiers or follow naming conventions
   - Validate entity names match model file names

### Success Criteria

1. ✅ `_entity.js` created as single source of truth for prefix mappings
2. ✅ All data files successfully moved to `data/` folder
3. ✅ All files renamed to new pattern: `{entity}.data.{locale}.js` or `{entity}.data.js`
4. ✅ Discovery correctly identifies all entities from filenames
5. ✅ Loading correctly loads data from new location
6. ✅ Prefix validation enforces 4-character format
7. ✅ All existing tests pass with new structure
8. ✅ New tests added for discovery, loading, and prefix validation
9. ✅ Empty entity folders removed
10. ✅ No data loss during migration
11. ✅ Documentation updated

### Open Questions

1. **Migration Timing**: Should migration be done automatically on first run, or require manual execution?

   - **Decision**: Manual execution via migration script for safety and control

2. **Backward Compatibility**: Should we support both old and new structures during transition?

   - **Decision**: No - clean break. Migration script handles transition, then old structure is removed.

3. **Wait.js Files**: Should `wait.js` files be treated specially or just renamed to `{entity}.data.js`?

   - **Decision**: All `wait.js` files have been renamed back to `data.js`. Migration script only handles `data.js` and `data.{locale}.js` files. No special handling needed.

4. **Entity Name Validation**: Should we validate entity names match model file names?

   - **Decision**: Yes - add validation in discovery to warn about mismatches, but don't fail (model resolution will fail later if mismatch exists).

5. **Prefix Validation Enforcement**: How strictly should prefix validation be enforced?

   - **Decision**: **STRICT ENFORCEMENT REQUIRED**
     - Loader **MUST** validate prefix against authoritative mapping table
     - **FAIL** if prefix is missing, invalid, or doesn't match entity name
     - **FAIL** if prefix not found in `buildPrefixToEntityMapping()` table
     - Prevents data corruption and ensures foreign key resolution works correctly
     - Clear error messages guide developers to fix prefix issues
     - Single source of truth: `seed/cache.js` `buildPrefixToEntityMapping()` function

6. **Adding New Entities**: What process should be followed when adding new entities with prefixes?
   - **Decision**:
     - **STEP 1**: Add prefix-to-entity mapping to `server/seed/_entity.js` (single source of truth)
     - **STEP 2**: Update Entity-Prefix Mapping Reference table in this design document
     - **STEP 3**: Create seed data file with correct prefix matching the mapping
     - **STEP 4**: Prefix must be exactly 4 characters (e.g., `'acct'`, `'mail'`, `'log_'`)
     - **STEP 5**: Entity name must match model file name exactly
     - **STEP 6**: Foreign key field pattern: `{entity}_id` (e.g., `account_id`, `boat_brand_id`)
     - **STEP 7**: Foreign key reference pattern: `{prefix}_key` (e.g., `acct_key`, `bbrn_key`)
     - **STEP 8**: Prefix must be exactly 4 characters (e.g., `'acct'`, `'mail'`, `'log_'`)
     - Validation will automatically enforce consistency and 4-character format

### Additional Updates Required

**JSDoc Comment Updates:**
The following files contain JSDoc comments that reference "entity folders" and should be updated to reflect the new structure:

1. `server/seed/all.js` - Line 13: Update description from "discovers entity folders" to "discovers entities from data files"
2. `server/seed/loader.js` - Line 8: Update description from "from its folder" to "from data folder"
3. `server/seed/loader.js` - Line 11: Update param description from "Entity folder name" to "Entity name"
4. `server/seed/discover.js` - Line 7: Update description from "Discover all entity folders" to "Discover all entities from data files"
5. `server/seed/discover.js` - Line 9: Update return description from "Array of entity folder names" to "Array of entity names"
6. `server/seed/seeder.js` - Line 42: Update param description from "Entity folder name" to "Entity name"
7. `server/seed/resolver.js` - Line 8: Update param description from "Entity folder name" to "Entity name"

**Error Message Updates:**
Update error messages in `loader.js` to reference new file locations:

- Change references from `server/seed/{entityName}/` to `server/seed/data/`
- Update file pattern examples in error messages

### Implementation Checklist

- [ ] Update `discover.js` to scan `data/` folder
- [ ] Update `loader.js` to load from `data/` folder with new naming
- [ ] Update JSDoc comments in all affected files
- [ ] Update error messages to reference new file locations
- [ ] Create migration script
- [ ] Update unit tests for `discover.js`
- [ ] Update unit tests for `loader.js`
- [ ] Update integration tests
- [ ] Test migration script on development environment
- [ ] Run full test suite
- [ ] Execute migration in development
- [ ] Verify all entities discovered
- [ ] Verify all data loads correctly
- [ ] Remove empty entity folders
- [ ] Update documentation
- [ ] Update CI/CD scripts if needed

---

## Next Steps

1. **Review and Approval**: Get stakeholder approval on design
2. **Implementation**: Begin Phase 1 implementation
3. **Testing**: Continuous testing throughout implementation
4. **Documentation**: Update docs as features are added
5. **Migration**: Plan migration timeline with team

---

**Design Status**: ✅ Complete - Ready for Implementation

**Confidence Level**: 97%

**Updated Based on Clarifications**:

1. ✅ **Prefix Format**: All prefixes are exactly 4 characters (underscores used only for padding to 4 characters)
2. ✅ **Wait.js Files**: Removed all references - files have been renamed to `data.js`
3. ✅ **Single Source of Truth**: `_entity.js` file created as authoritative prefix mapping
4. ✅ **Cache.js**: Updated to load from `_entity.js` instead of hardcoded mapping
5. ✅ **Prefix Validation**: Updated to enforce 4-character format
6. ✅ **Migration Script**: Simplified to handle only `data.js` and `data.{locale}.js` files

**Key Updates Based on Review**:

1. **Email Entity Restructuring**:

   - ✅ Eliminated `email/data.js` (it was code that reads locale files, not data)
   - ✅ Renamed locale files: `en.js` → `data.en.js`, `es.js` → `data.es.js`
   - ✅ Locale files now export arrays directly (no code logic)

2. **Data Loading**:

   - ✅ Loader loads all files matching pattern `{entity}.data.*.js` (including `{entity}.data.js`) from `server/seed/data/` folder
   - ✅ Automatic locale detection via `{entity}.data.{2-letter-locale}.js` pattern (e.g., `email.data.en.js`)
   - ✅ All matching files are merged into single array

3. **Matching Strategy**:

   - ✅ Automatic detection: `['key', 'locale']` for locale-aware entities
   - ✅ Automatic detection: `['key']` for single-key entities
   - ✅ No manual configuration required for common cases

4. **Prefix Format Clarification**:

   - ✅ All prefixes are exactly 4 characters (e.g., `'acct'`, `'mail'`, `'log_'`, `'plc_'`)
   - ✅ Underscores are used ONLY to pad to 4 characters when needed (e.g., `'log_'`, `'plc_'` have underscore as padding)
   - ✅ Single source of truth: `server/seed/_entity.js`
   - ✅ Prefix validation enforces 4-character format

5. **Wait.js Files**:

   - ✅ All `wait.js` files have been renamed back to `data.js`
   - ✅ Migration script only handles `data.js` and `data.{locale}.js` files
   - ✅ No special handling needed for temporary files

6. **Logging**:

   - ✅ Removed `MODULE_NAME` from all mcode logging calls
   - ✅ Using `mcode.log()`, `mcode.warn()`, `mcode.error()`, `mcode.exp()` consistently
   - ✅ Import: `const mcode = require('mcode-log');`

7. **Future Considerations**:
   - ✅ Documented `created_at`/`updated_at` timestamp refactor (separate effort affecting entire DB layer)
   - ✅ Seeding system will support timestamps when schemas are updated

**File Structure Changes Required**:

```
BEFORE:
server/seed/
├── email/
│   ├── data.js  (code that reads locale files - DELETE)
│   ├── en.js    (locale data - MOVE and RENAME)
│   └── es.js    (locale data - MOVE and RENAME)
├── account/
│   └── data.js  (MOVE and RENAME)
└── boat/
    └── data.js  (MOVE and RENAME)

AFTER:
server/seed/
├── _entity.js  (NEW - single source of truth for prefix mappings, leading underscore sorts first)
├── data/
│   ├── email.data.en.js  (exports array directly)
│   ├── email.data.es.js  (exports array directly)
│   ├── account.data.js
│   └── boat.data.js
└── cache.js  (UPDATED - loads from _entity.js)
```

**Key Implementation Changes**:

1. **New File**: `server/seed/_entity.js` - Single source of truth for all prefix mappings (leading underscore ensures it sorts first)
2. **Updated**: `server/seed/cache.js` - Now loads from `_entity.js` instead of hardcoded mapping
3. **Updated**: `server/seed/loader.js` - Validates prefixes are exactly 4 characters
4. **Updated**: `server/seed/discover.js` - Scans `data/` folder instead of entity folders
5. **Migration**: One-time script to move and rename all data files

**Estimated Implementation Time**: 2-3 days

**Risk Level**: Low (well-defined scope, clear patterns, automatic detection reduces configuration, single source of truth simplifies maintenance)
