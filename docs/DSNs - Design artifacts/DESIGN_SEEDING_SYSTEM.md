# Design: Generalized Seeding System

## Executive Summary

This design document outlines a generalized, extensible seeding system that automatically discovers and seeds static data from JSON files in `server/seed/` folders. The system eliminates the need for per-entity JavaScript seeder files, supports both MongoDB and SQL databases, and provides idempotent seeding with configurable matching strategies.

## Current State Analysis

### Existing Structure

```
server/seed/
├── all.js                    # Manual orchestrator requiring each entity seeder
├── mfg_company.js            # Per-entity seeder (to be eliminated)
├── plc_family.js            # Per-entity seeder (to be eliminated)
├── plc_mfg.js               # Per-entity seeder (to be eliminated)
├── email/                   # Entity folder
│   ├── data.js             # Code that reads locale files (TO BE ELIMINATED)
│   ├── en.js               # Locale-specific data (TO BE RENAMED to data.en.js)
│   └── es.js               # Locale-specific data (TO BE RENAMED to data.es.js)
├── mfg_company/             # Entity folder
│   └── data.js             # JSON data array
├── plc_family/              # Entity folder
│   └── data.js             # JSON data array
├── plc_mfg/                 # Entity folder
│   └── data.js             # JSON data array
├── mongo.seed.js            # Simple MongoDB seeder (email only)
└── sql.seed.js              # Simple SQL seeder (email only)
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
├── all.js                  # Auto-discovery orchestrator (refactored)
├── email/                  # Entity folder (locale-aware)
│   ├── data.en.js          # English locale data (required for locale entities)
│   └── data.es.js          # Spanish locale data (optional)
├── mfg_company/            # Entity folder (single-key)
│   └── data.js             # JSON data array
├── plc_family/             # Entity folder (single-key)
│   └── data.js             # JSON data array
├── plc_mfg/                # Entity folder (single-key)
│   └── data.js             # JSON data array
├── {entity}/               # New entity (example, SINGLE-key)
│   └── data.js             # JSON data array
└── {entity}/               # New entity (example, DUAL-key)
    ├── data.en.js          # English locale data (required for locale entities)
    ├── data.fr.js          # any other language...
    ├── data.dt.js
    ├── data.it.js
    └── data.es.js
```

### Key Features

1. **Auto-Discovery**: Automatically detect entity folders in `server/seed/`
2. **Zero Configuration**: Works out-of-the-box with sensible defaults
3. **Idempotent & Updating**: Inserts missing records, updates existing records if fields differ, skips unchanged records
4. **Database Agnostic**: Handles both MongoDB and SQL transparently
5. **Extensible**: Supports custom matching strategies and seeding logic
6. **Error Resilient**: Continues seeding other entities if one fails

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

**Purpose**: Load seed data from entity folders.

**Responsibilities**:

- Load all files matching pattern `data.*.js` from entity folder (including `data.js`)
- Automatically detect locale-aware entities (presence of `data.en.js` or similar)
- Merge data from all matching files
- Validate data structure (must be array)
- Return both data and matching strategy hint

**Implementation**:

```javascript
const mcode = require("mcode-log");
const fs = require("fs").promises;
const path = require("path");

/**
 * @func loadEntityData
 * @memberof seed.loader
 * @desc Load seed data for an entity from its folder. Loads all files matching data.*.js pattern.
 * @api private
 * @param {string} entityName - Entity folder name
 * @returns {Promise<object>} Object containing { data: Array, isLocaleAware: boolean }
 */
async function loadEntityData(entityName) {
  const entityPath = path.join(__dirname, entityName);

  // Read all files in entity folder
  const files = await fs.readdir(entityPath);

  // Filter for data.*.js files (including data.js)
  const dataFiles = files.filter(
    (file) =>
      file === "data.js" || (file.startsWith("data.") && file.endsWith(".js"))
  );

  if (dataFiles.length === 0) {
    throw new Error(
      `No data files found for entity: ${entityName} (expected data.js or data.*.js)`
    );
  }

  // Check if locale-aware (has data.en.js, data.es.js, etc.)
  const localeFiles = dataFiles.filter(
    (file) => file.match(/^data\.[a-z]{2}\.js$/) // Matches data.en.js, data.es.js, etc.
  );
  const isLocaleAware = localeFiles.length > 0;

  // Load and merge all data files
  let allData = [];
  for (const file of dataFiles) {
    const filePath = path.join(entityPath, file);
    const fileData = require(filePath);

    if (!Array.isArray(fileData)) {
      throw new Error(`Entity ${entityName}: ${file} must export an array`);
    }

    allData = [...allData, ...fileData];
  }

  if (allData.length === 0) {
    mcode.warn(`Entity ${entityName}: No data found in data files`);
  }

  return {
    data: allData,
    isLocaleAware: isLocaleAware,
  };
}
```

**Key Features**:

- Loads all `data.*.js` files (including `data.js`)
- Automatically detects locale-aware entities by presence of `data.{locale}.js` files (e.g., `data.en.js`, `data.es.js`)
- Merges data from all matching files into single array
- Returns hint about locale-awareness for matching strategy determination

**File Pattern Matching**:

- Pattern: `data.*.js` matches:
  - `data.js` (single-key entity - ONLY file for single-key entities)
  - `data.en.js` (locale-specific, English)
  - `data.es.js` (locale-specific, Spanish)
  - `data.fr.js` (locale-specific, French)
  - `data.dt.js` (locale-specific, German)
  - `data.it.js` (locale-specific, Italian)
  - Any other `data.{2-letter-locale}.js` files
- **Single-key entities**: Have ONLY `data.js` (no locale files)
- **Dual-key (locale-aware) entities**: Have ONLY `data.{locale}.js` files (no `data.js`)
- All matching files are loaded and their arrays are merged
- Locale detection: Files matching `data.{2-letter-locale}.js` pattern indicate locale-aware entity
- **Important**: An entity should be EITHER single-key (only `data.js`) OR dual-key (only `data.{locale}.js` files), not both

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

- If entity has locale files (`data.en.js`, etc.): Use `['name', 'locale']` as matching keys
- If entity has only `data.js`: Use `['name']` as matching key (or first unique field)
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
    // Entity has locale files (data.en.js, etc.) - match by name + locale
    return { keys: ["name", "locale"] };
  } else {
    // Entity has only data.js - match by name (or first unique field)
    return { keys: ["name"] };
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

   - Rename `server/seed/email/en.js` → `server/seed/email/data.en.js`
   - Rename `server/seed/email/es.js` → `server/seed/email/data.es.js`
   - Delete `server/seed/email/data.js` (it's code, not data)

2. **Verify Email Data Structure**
   - Ensure `data.en.js` and `data.es.js` export arrays directly
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
    - Document file naming conventions (`data.js` vs `data.{locale}.js`)

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

  it("should throw if data.js not found", async () => {
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

**Issue**: Some entities have locale-specific files (e.g., `email/data.en.js`, `email/data.es.js`).

**Solution**:

- Loader module automatically detects locale files matching pattern `data.{locale}.js`
- All `data.*.js` files (including `data.js`) are loaded and merged
- Matching strategy automatically uses `['name', 'locale']` for locale-aware entities
- Old `email/data.js` (code file) is eliminated
- Old `email/en.js` and `email/es.js` are renamed to `email/data.en.js` and `email/data.es.js`

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

   - **Decision**: Files matching `data.{locale}.js` pattern are automatically detected. All `data.*.js` files (including `data.js`) are loaded and merged. Matching strategy automatically uses `['name', 'locale']` for locale-aware entities.

3. **Custom Seeding Logic**: Should we support custom seeding functions?

   - **Decision**: Phase 2 feature - support transform functions first

4. **Batch Processing**: Should large datasets be processed in batches?

   - **Decision**: Monitor performance, add batching if needed

5. **Transaction Mode**: Should we support full transaction mode?

   - **Decision**: Per-entity transactions by default, full transaction as optional config

6. **Schema Timestamps**: When should `created_at` and `updated_at` be added?
   - **Decision**: Seeding system will automatically add `created_at` on insert and `updated_at` on update. If schemas don't have these fields yet, they will be added as part of the database refactor, but seeding will work regardless (fields will be stored if schema allows).

## Next Steps

1. **Review and Approval**: Get stakeholder approval on design
2. **Implementation**: Begin Phase 1 implementation
3. **Testing**: Continuous testing throughout implementation
4. **Documentation**: Update docs as features are added
5. **Migration**: Plan migration timeline with team

---

**Design Status**: ✅ Complete - Ready for Review

**Confidence Level**: 98%+

**Key Updates Based on Review**:

1. **Email Entity Restructuring**:

   - ✅ Eliminated `email/data.js` (it was code that reads locale files, not data)
   - ✅ Renamed locale files: `en.js` → `data.en.js`, `es.js` → `data.es.js`
   - ✅ Locale files now export arrays directly (no code logic)

2. **Data Loading**:

   - ✅ Loader loads all files matching pattern `data.*.js` (including `data.js`)
   - ✅ Automatic locale detection via `data.{2-letter-locale}.js` pattern (e.g., `data.en.js`)
   - ✅ All matching files are merged into single array

3. **Matching Strategy**:

   - ✅ Automatic detection: `['name', 'locale']` for locale-aware entities
   - ✅ Automatic detection: `['name']` for single-key entities
   - ✅ No manual configuration required for common cases

4. **Logging**:

   - ✅ Removed `MODULE_NAME` from all mcode logging calls
   - ✅ Using `mcode.log()`, `mcode.warn()`, `mcode.error()`, `mcode.exp()` consistently
   - ✅ Import: `const mcode = require('mcode-log');`

5. **Future Considerations**:
   - ✅ Documented `created_at`/`updated_at` timestamp refactor (separate effort affecting entire DB layer)
   - ✅ Seeding system will support timestamps when schemas are updated

**File Structure Changes Required**:

```
BEFORE:
server/seed/email/
├── data.js  (code that reads locale files - DELETE)
├── en.js    (locale data - RENAME to data.en.js)
└── es.js    (locale data - RENAME to data.es.js)

AFTER:
server/seed/email/
├── data.en.js  (exports array directly)
└── data.es.js  (exports array directly)
```

**Estimated Implementation Time**: 2-3 days

**Risk Level**: Low (well-defined scope, clear patterns, automatic detection reduces configuration)
