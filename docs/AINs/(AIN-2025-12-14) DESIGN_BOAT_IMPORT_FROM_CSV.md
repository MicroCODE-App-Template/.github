# Design: Boat Import from CSV - Race Registrations

## Executive Summary

This design document outlines the implementation of `import.boats.js`, a script to intelligently parse `drya_race_registrations_1764697152671.csv` and generate four interconnected seed data files:

- `server/seed/data/boat_brand.data.js`
- `server/seed/data/boat_builder.data.js`
- `server/seed/data/boat_design.data.js`
- `server/seed/data/boat.data.js`

The script will extract boat information from race registration data, normalize brand/design names, parse sail numbers, and link boats to their owners (users) using the `user_{first}_{last}` key pattern.

### Key Requirements

1. **Import boats owned by users** - Link boats to users via user keys
2. **User key format**: `user_{first}_{last}` (all lowercase)
3. **Boat name**: Column `yachtName` - Title Case
4. **Brand extraction**: First word of `design` column, special case "J" → "J/Boats", Title Case
5. **Sail country code**: Extract 3-letter acronym from `sailNumber`, default "USA" if none
6. **Sail number**: Numeric portion of `sailNumber`
7. **Design parsing**: Break `design` into brand (word) and "{name} {number}" format
8. **Mapping structure**: Extensible mapping for special cases
9. **Builder creation**: Create one builder per brand (no CSV builder data)
10. **Output format**: Generate `.data.js` files matching existing model schemas

---

## Table of Contents

1. [Current State Analysis](#current-state-analysis)
2. [Data Structure Analysis](#data-structure-analysis)
3. [Parsing Rules & Logic](#parsing-rules--logic)
4. [Mapping Structure Design](#mapping-structure-design)
5. [File Generation Strategy](#file-generation-strategy)
6. [User-Boat Linking](#user-boat-linking)
7. [Implementation Architecture](#implementation-architecture)
8. [Data Flow](#data-flow)
9. [Edge Cases & Special Handling](#edge-cases--special-handling)
10. [Test Cases](#test-cases)
11. [Questions for Clarification](#questions-for-clarification)
12. [Risk Assessment](#risk-assessment)

---

## Current State Analysis

### Existing Import Pattern (`import.users.js`)

The existing `server/bin/import.users.js` provides a template for CSV import:

**Key Patterns:**

- Reads CSV from `server/seed/external/`
- Parses CSV rows with custom parser (handles quoted fields)
- Appends to existing `.data.js` files
- Formats output as JavaScript (not JSON) with single quotes
- Normalizes line endings to Windows CRLF (`\r\n`)
- Checks for duplicates before adding
- Maintains array structure in data files

**File Structure Pattern:**

```javascript
const prefix = "entity";
const entity = [
  {
    key: "entity_key",
    // ... fields
  },
  // ... more entities
];

module.exports = {
  prefix,
  entity,
};
```

### Existing Data File Structures

#### `boat_brand.data.js`

```javascript
const prefix = "bbrn";
const boat_brand = [
  {
    key: "jboats",
    name: "J/Boats",
    display: "J/Boats",
    description: "Description of J/Boats",
    logo: "data:image/x-icon;base64,",
  },
];
```

#### `boat_builder.data.js`

```javascript
const prefix = "bbld";
const boat_builder = [
  {
    key: "jcomposites",
    bbrn_key: "jboats",
    name: "J Composites",
    description: "Manufacturer of J/Boats in Europe.",
    logo: "data:image/x-icon;base64,",
  },
];
```

#### `boat_design.data.js`

```javascript
const prefix = "bdsn";
const boat_design = [
  {
    key: "j112e",
    bbrn_key: "jboats",
    bbld_key: "jcomposites",
    designer: "Al Johnstone",
    name: "J/112e",
    description: `A 36 ft racer-cruiser by Al Johnstone.`,
    hull: {
      length: 10973, // ✅ CONFIRMED: Length in millimeters (36 ft * 304.8 = 10972.8 mm)
      beam: 11.5,
      draft: 6.5,
      displacement: 11500,
    },
    rig: { P: 100, E: 100, I: 100, J: 100, sail_area: 690 },
  },
];
```

**Note**: `hull.length` stored in millimeters (integer) ✅ **CONFIRMED**

#### `boat.data.js`

```javascript
const prefix = "boat";
const boat = [
  {
    key: "elevation",
    bdsn_key: "j112e",
    bbld_key: "jcomposites",
    name: "Elevation",
    description: "Elevation is a J/112E sailboat.",
    hin: "FRJBC12032L718",
    sail_cc: "USA",
    sail_no: "61232",
  },
];
```

### Model Schema Requirements

#### Boat Model (`boat.mongo.js`)

- `key`: String (unique)
- `bdsn_key`: String (required) - boat_design.key
- `boat_design_id`: String (optional)
- `bbld_key`: String (required) - boat_builder.key
- `boat_builder_id`: String (optional)
- `name`: String (required)
- `display`: String (optional)
- `description`: String (optional)
- `hin`: String (optional)
- `sail_cc`: String (optional)
- `sail_no`: String (optional)

#### Boat Brand Model (`boat_brand.mongo.js`)

- `key`: String (unique)
- `name`: String (required)
- `display`: String (required)
- `description`: String (required)
- `logo`: String (required)

#### Boat Builder Model (`boat_builder.mongo.js`)

- `key`: String (unique)
- `bbrn_key`: String (required) - boat_brand.key
- `boat_brand_id`: String (optional)
- `name`: String (required)
- `title`: String (optional)
- `description`: String (required)
- `logo`: String (required)

#### Boat Design Model (`boat_design.mongo.js`)

- `key`: String (unique)
- `bbrn_key`: String (required) - boat_brand.key
- `boat_brand_id`: String (optional)
- `bbld_key`: String (required) - boat_builder.key
- `boat_builder_id`: String (optional)
- `name`: String (required)
- `display`: String (optional)
- `description`: String (required)
- `designer`: String (optional)
- `hull`: Object (optional)
- `rig`: Object (optional)

### User-Boat Relationship

From `user.mongo.js` schema:

- Users have `boat_keys: Array<string>` and `boat_ids: Array<string>`
- Boats do NOT have direct user references
- Relationship is maintained through user's `boat_keys` array

**Note**: The import script will generate boat data files. User updates to add `boat_keys` will be handled separately or in a follow-up script.

---

## Data Structure Analysis

### CSV File: `drya_race_registrations_1764697152671.csv`

**Columns:**

- `sailNumber`: String - e.g., "USA 35007", "108", "J120 25439"
- `yachtName`: String - Boat name, e.g., "Advantage", "AirForce"
- `owner`: String - Owner name, e.g., "John Vermeulen", "Perrin Fortune"
- `design`: String - Design name, e.g., "Finot First Class 12", "express 27 modified", "Tartan33"
- `length`: String - Length, e.g., "39.3 feet", "27", "40'"
- `class`: String - Racing class, e.g., "PHRF", "ORC & PHRF"
- `status`: String - Registration status, e.g., "paid", "pending", "processing"

**Sample Rows:**

```csv
"USA 35007","Advantage","John Vermeulen","Finot First Class 12","39.3 feet","PHRF","paid"
"108","AirForce","Perrin Fortune","express 27 modified","27","ORC & PHRF","paid"
"Tartan33","Ariel","Mark Aitken","Tartan33","33","Cruising","paid"
"USA15056","Avatar","Andrew Morlan","Santana 35","35","PHRF","paid"
"J120 25439","Hissy Fit","Dave Taylor","J120","40'","ORC & PHRF","paid"
```

**Observations:**

- `sailNumber` varies: "USA 35007", "108", "J120 25439", "usa 32111"
- `design` varies: "Tartan33", "J120", "Nonsuch", "express 27 modified"
- `owner` may be empty: `""`
- `length` varies: "39.3 feet", "27", "40'", "31.58"
- Some designs have numbers embedded: "Tartan33", "J120"
- Some designs are plain text: "Nonsuch", "express 27 modified"

---

## Parsing Rules & Logic

### Rule 1: User Key Generation (for Lookup)

**Input**: `owner` column (e.g., "John Vermeulen", "Perrin Fortune")

**Process** (following `import.users.js` pattern):

1. Split by whitespace
2. First word → `first`
3. Last word → `last`
4. Convert to lowercase
5. Remove special characters: `/[^a-z0-9\s]/g`
6. Replace spaces with underscores: `/\s+/g` → `_`
7. Format: `{first}_{last}` (no "user\_" prefix - matches user.data.js key format)

**Examples**:

- "John Vermeulen" → `john_vermeulen`
- "Perrin Fortune" → `perrin_fortune`
- "Mark Aitken" → `mark_aitken`
- "Donald Maxwell" → `donald_maxwell` (handles double spaces)
- "" (empty) → Skip boat (no owner)

**Edge Cases**:

- Multiple middle names: Use first and last only (matches import.users.js pattern)
- Single name: Use `{name}_unknown` as last name (e.g., "Madonna" → `madonna_unknown`)
- Empty owner: Skip boat
- Double spaces: Normalize to single space before processing

### Rule 2: Boat Name (Title Case)

**Input**: `yachtName` column

**Process**:

1. Trim whitespace
2. Convert to Title Case (first letter of each word uppercase, rest lowercase)
3. Handle special cases (preserve existing capitalization if intentional)

**Examples**:

- "Advantage" → "Advantage"
- "AirForce" → "Airforce" (or preserve? → **QUESTION NEEDED**)
- "ELEVATION" → "Elevation"
- "DOOR PRIZE " → "Door Prize"

### Rule 3: Brand Extraction from Design

**Input**: `design` column (e.g., "Tartan33", "J120", "Nonsuch", "express 27 modified")

**Process**:

1. Extract first word from design
2. Apply special case mappings:
   - "J" → "J/Boats"
   - Other mappings via mapping structure
3. Convert to Title Case
4. Generate key: lowercase, replace spaces/special chars with underscores

**Examples**:

- "Tartan33" → Brand: "Tartan", Key: "tartan"
- "J120" → Brand: "J/Boats", Key: "jboats"
- "J Boat" → Brand: "J/Boats", Key: "jboats"
- "J Boats" → Brand: "J/Boats", Key: "jboats"
- "Nonsuch" → Brand: "Nonsuch", Key: "nonsuch"
- "express 27 modified" → Brand: "Express", Key: "express"
- "Finot First Class 12" → Brand: "Finot", Key: "finot"

**Special Cases**:

- "J" alone → "J/Boats"
- "J Boat" → "J/Boats"
- "J Boats" → "J/Boats"
- "J/111" → "J/Boats"
- "J/112e" → "J/Boats"

### Rule 4: Sail Country Code Extraction

**Input**: `sailNumber` column (e.g., "USA 35007", "108", "J120 25439", "NED 131")

**Process**:

1. Extract all 3-letter sequences (uppercase)
2. If found, use first valid country code
3. If none found, default to "USA"
4. Normalize to uppercase

**Examples**:

- "USA 35007" → "USA"
- "108" → "USA" (default)
- "J120 25439" → "USA" (no 3-letter code)
- "NED 131" → "NED"
- "CAN 099" → "CAN"
- "UAE 74111" → "UAE"
- "usa 32111" → "USA" (normalize case)

**Country Code Detection**:

- Look for sequences of exactly 3 uppercase letters
- Common codes: USA, CAN, NED, UAE, etc.
- If multiple codes found, use first one

### Rule 5: Sail Number Extraction

**Input**: `sailNumber` column

**Process**:

1. Remove all non-numeric characters
2. Extract numeric portion
3. If no numbers found, use empty string

**Examples**:

- "USA 35007" → "35007"
- "108" → "108"
- "J120 25439" → "25439" ✅ **CONFIRMED**: Extract last number sequence (J120 is design, 25439 is sail number)
- "NED 131" → "131"
- "CAN 099" → "099"
- "usa 32111" → "32111"

**Edge Case**: "J120 25439" - Extract "25439" (last number sequence) ✅ **CONFIRMED**

### Rule 6: Design Name Parsing

**Input**: `design` column, `length` column

**Process**:

1. Extract brand (first word) - already done in Rule 3
2. Check if design contains numbers:
   - If yes: Extract brand + number, format as "{Brand} {Number}"
   - If no: Use brand + length (from `length` column) as integer

**Examples**:

- "Tartan33" → Brand: "Tartan", Design: "33" ✅ **CONFIRMED**: Remove brand name from design
- "J120" → Brand: "J/Boats", Design: "120" ✅ **CONFIRMED**: Remove brand name from design
- "Nonsuch" + length "30" → Brand: "Nonsuch", Design: "30" ✅ **CONFIRMED**: Use length if no number in design
- "express 27 modified" → Brand: "Express", Design: "27 modified" ✅ **CONFIRMED**: Remove brand, keep rest
- "Finot First Class 12" → Brand: "Finot", Design: "First Class 12" ✅ **CONFIRMED**: Remove brand name, keep model name

**Number Extraction Logic**:

- Extract brand (first word) and remove from design string
- If design contains numbers after brand removal, keep them in design name
- If no numbers in design, use length column (truncate to integer)
- Format: "{Remaining Design Name}" (without brand prefix)

**Length Column Parsing**:

- Extract numeric value (handle "39.3 feet", "27", "40'", "31.58")
- Truncate to integer
- Use if no number in design name (after brand removal)

**Edge Cases**:

- "express 27 modified" → Brand: "Express", Design: "27 modified"
- "Finot First Class 12" → Brand: "Finot", Design: "First Class 12" ✅ **CONFIRMED**
- "Nonsuch" (no number) + length "30" → Brand: "Nonsuch", Design: "30"

### Rule 7: Design Key Generation

**Input**: Design name (e.g., "Tartan 33", "J/120", "Nonsuch 30")

**Process**:

1. Convert to lowercase
2. Replace spaces with underscores
3. Replace "/" with nothing or underscore? → **QUESTION NEEDED**
4. Remove special characters
5. Ensure uniqueness (append number if duplicate)

**Examples**:

- "Tartan 33" → "tartan_33"
- "J/120" → "j120" or "j_120" → **QUESTION NEEDED**
- "Nonsuch 30" → "nonsuch_30"
- "Express 27" → "express_27"

### Rule 8: Builder Creation

**Input**: Brand key (e.g., "tartan", "jboats", "nonsuch")

**Process**:

1. For each unique brand, create one builder
2. Builder key: `{brand_key}_builder` ✅ **CONFIRMED**: Default pattern when builder unknown
3. Builder name: "{Brand} Manufacturing" ✅ **CONFIRMED**: Default pattern
4. Builder description: "Manufacturer of {Brand} sailboats."
5. Link to brand via `bbrn_key`

**Examples**:

- Brand: "Tartan" → Builder: "Tartan Manufacturing", Key: "tartan_builder"
- Brand: "J/Boats" → Builder: "J Composites", Key: "jcomposites" ✅ **CONFIRMED**: Special case (real-world builder)
- Brand: "Nonsuch" → Builder: "Nonsuch Manufacturing", Key: "nonsuch_builder"

**Special Cases**:

- "J/Boats" → Use "J Composites" (real-world builder) ✅ **CONFIRMED**
- All other brands → Use "{Brand} Manufacturing" pattern

### Rule 9: Boat Key Generation

**Input**: `yachtName` column

**Process**:

1. Convert to lowercase
2. Replace spaces with underscores
3. Remove special characters
4. Ensure uniqueness (append number if duplicate)

**Examples**:

- "Advantage" → "advantage"
- "AirForce" → "airforce"
- "DOOR PRIZE " → "door_prize"
- "Hissy Fit" → "hissy_fit"

---

## Mapping Structure Design

### Brand Mapping Structure

**CONFIRMED**: Flexible mapping structure supporting multiple variations per brand.

```javascript
/**
 * Brand Mapping Structure
 * Maps multiple variations/spellings to a single normalized brand name
 * Format: Array of variations → Normalized brand name
 */
const brandMappings = {
  // Single letter to full name
  J: "J/Boats",

  // Multiple variations mapping to same brand
  "C&C": "C&C Yachts",
  "C & C": "C&C Yachts",
  "C+C": "C&C Yachts",

  // Variations to normalized name
  "J Boat": "J/Boats",
  "J Boats": "J/Boats",
  "J/Boats": "J/Boats",

  // Design-specific mappings (if needed)
  "J/111": "J/Boats",
  "J/112e": "J/Boats",
  "J/120": "J/Boats",
  "J/105": "J/Boats",
  "J/109": "J/Boats",
  "J/130SD": "J/Boats",

  // Complex mappings requiring manual configuration
  // '1D35': 'Nelson-Marek' (design), 'Carrol Marine' (builder)
  // Note: Some mappings may require design-level mapping, not just brand

  // Other special cases (extendable)
  // 'S&S': 'Sparkman & Stephens',
  // 'NM': 'North Marine',
};

/**
 * Helper function to lookup brand from mapping
 * @param {string} designFirstWord - First word from design column
 * @returns {string|null} Normalized brand name or null if not found
 */
function lookupBrand(designFirstWord) {
  const normalized = designFirstWord.trim();
  return brandMappings[normalized] || null;
}
```

**Key Features**:

- ✅ Supports multiple variations per brand (e.g., ['C&C', 'C & C', 'C+C'] → "C&C Yachts")
- ✅ Case-sensitive lookup (normalize input before lookup)
- ✅ Extensible - add new mappings as needed
- ✅ Some complex cases (like "1D35") may require design-level mapping, not just brand

### Design Parsing Rules

```javascript
const designParsingRules = {
  // Patterns that need special handling
  patterns: [
    {
      // Extract number from design name
      regex: /^(\w+)(\d+)/i,
      extract: (match, brand) => {
        return {
          brand: brand,
          designNumber: match[2],
          designName: `${brand} ${match[2]}`,
        };
      },
    },
    {
      // Handle "Brand Model Number" format
      regex: /^(\w+)\s+(\w+)\s+(\d+)/i,
      extract: (match, brand) => {
        return {
          brand: brand,
          designNumber: match[3],
          designName: `${brand} ${match[2]} ${match[3]}`,
        };
      },
    },
  ],

  // Special design name mappings
  specialDesigns: {
    "express 27 modified": { brand: "Express", number: 27, name: "Express 27" },
    "Finot First Class 12": {
      brand: "Finot",
      number: 12,
      name: "Finot First Class 12",
    },
    // Add more as needed
  },
};
```

### Builder Naming Rules

```javascript
const builderNamingRules = {
  // Special builder names for known brands
  jboats: {
    name: "J Composites",
    description: "Manufacturer of J/Boats sailboats.",
  },

  // Default pattern for unknown brands
  default: (brandName) => ({
    name: `${brandName} Yachts`,
    description: `Manufacturer of ${brandName} sailboats.`,
  }),
};
```

---

## File Generation Strategy

### Step 1: Parse CSV

1. Read CSV file
2. Parse rows using existing CSV parser from `import.users.js`
3. Filter out rows with empty `owner` (no owner = skip boat)
4. Extract and normalize all fields

### Step 2: Extract Unique Entities

**Brands:**

- Extract brand from each design
- Apply brand mappings
- Collect unique brands
- Generate brand objects

**Builders:**

- Create one builder per brand
- Apply builder naming rules
- Link to brand via `bbrn_key`

**Designs:**

- Parse design name + length
- Extract brand and number
- Generate design objects
- Link to brand and builder

**Boats:**

- Extract boat information
- Link to design and builder
- Generate boat keys
- Link to user (via user_key reference, not direct field)

### Step 3: Generate Data Files

**For each entity type (brand, builder, design, boat):**

1. Read existing file (if exists)
2. Extract existing keys to avoid duplicates
3. Filter new entities (skip if key exists)
4. Format as JavaScript (not JSON)
5. Append to existing array
6. Write file with CRLF line endings

**File Format:**

```javascript
const prefix = "entity";
const entity = [
  // ... existing entities
  // ... new entities
];

module.exports = {
  prefix,
  entity,
};
```

**CRLF Line Endings**: All files written with Windows CRLF (`\r\n`) line endings, matching `import.users.js` pattern.

### Step 4: User-Boat Linking (Future/Follow-up)

**Note**: This step may be handled separately or in a follow-up script.

1. Read generated `boat.data.js`
2. Read `user.data.js`
3. Match boats to users via `owner` name → `user_key`
4. Update user records to add `boat_keys` array
5. Write updated `user.data.js`

**OR**: Generate a separate mapping file for manual review.

---

## User-Boat Linking

### Current Relationship Model

- **User** has `boat_keys: Array<string>` and `boat_ids: Array<string>`
- **Boat** does NOT have direct user reference
- Relationship maintained through user's `boat_keys` array

### Linking Strategy

**CONFIRMED**: Integrated into import script (Option 1)

- Read `user.data.js` at start of script
- For each boat row processed:
  - Parse owner name: split by space, extract first and last
  - Generate user key: `{first}_{last}` (lowercase, underscore-separated, remove special chars)
  - Look up user in `user.data.js` by `key` field
  - If user found:
    - Initialize `boat_keys` array if it doesn't exist
    - Add boat `key` to `boat_keys` array (avoid duplicates)
  - If user not found: Log warning, continue processing
- Write updated `user.data.js` at end with CRLF line endings

### Owner Name Matching

**Challenge**: CSV has owner names (e.g., "John Vermeulen"), user keys are `user_john_vermeulen`

**Process**:

1. Parse owner name: split by space, extract first + last
2. Generate user key: `{first}_{last}` (lowercase, underscore-separated, remove special chars)
   - Matches pattern from `import.users.js`: `fullName.toLowerCase().replace(/[^a-z0-9\s]/g, '').replace(/\s+/g, '_')`
3. Look up in `user.data.js` by `key` field
4. If found:
   - Initialize `boat_keys` array if it doesn't exist
   - Add boat `key` to `boat_keys` array (avoid duplicates)
   - Preserve existing `boat_keys` entries
5. If not found, log warning (owner not in user data), continue processing

**Edge Cases**:

- Owner name variations: "John Vermeulen" vs "John A. Vermeulen"
- Multiple boats per owner: Add all boat_keys
- Owner not found: Log warning, skip linking

---

## Implementation Architecture

### Script Structure: `server/bin/import.boats.js`

```javascript
/**
 * Script to import boats from CSV file into boat data files
 *
 * Usage: node import.boats.js
 *
 * This script:
 * 1. Reads CSV from server/seed/external/drya_race_registrations_1764697152671.csv
 * 2. Parses each row and extracts boat, brand, builder, design information
 * 3. Applies mapping rules and special cases
 * 4. Generates/updates:
 *    - server/seed/data/boat_brand.data.js
 *    - server/seed/data/boat_builder.data.js
 *    - server/seed/data/boat_design.data.js
 *    - server/seed/data/boat.data.js
 */

// Imports
const fs = require("fs");
const path = require("path");

// Helper functions
// - parseCSVRow (from import.users.js)
// - toTitleCase
// - normalizeToCRLF
// - formatEntityAsJS
// - extractBrand
// - extractSailCC
// - extractSailNo
// - parseDesign
// - generateKeys

// Mapping structures
// - brandMappings
// - designParsingRules
// - builderNamingRules

// Main processing
// 1. Read CSV
// 2. Parse rows
// 3. Extract entities
// 4. Generate files
```

### Key Functions

#### `extractBrand(design: string): { name: string, key: string }`

- Extract first word from design
- Apply brand mappings
- Convert to Title Case
- Generate key

#### `parseDesign(design: string, length: string): { brand: string, name: string, key: string, length_mm: number }`

- Extract brand (first word)
- Remove brand name from design string
- Check for numbers in remaining design
- If number found: Use remaining design name
- If no number: Use length column value as design name
- Convert length to millimeters (imperial → metric)
- Generate design key
- Return length in mm for hull.length storage

#### `extractSailCC(sailNumber: string): string`

- Find 3-letter uppercase sequences
- Return first match or "USA"

#### `extractSailNo(sailNumber: string): string`

- Extract numeric portion
- Return as string

#### `generateUserKey(owner: string): { key: string | null, warning: string | null }`

- Parse owner name
- Generate `{first}_{last}` (no prefix)
- Handle single name: `{name}_unknown`
- Return key and warning message (if single name)
- Return null key if empty owner

#### `convertLengthToMM(lengthStr: string): number`

- Parse imperial length formats:
  - "39.3 feet" → 39.3 feet
  - "27" → 27 feet
  - "40'" → 40 feet
  - "31.58" → 31.58 feet
  - "34'7\"" → 34 feet 7 inches = 34.583 feet
- Convert to millimeters: `(feet * 304.8) + (inches * 25.4)`
- Return integer (rounded)

#### `generateBoatKey(yachtName: string): string`

- Convert to lowercase
- Replace spaces with underscores
- Remove special chars
- Ensure uniqueness

---

## Data Flow

```
CSV File (drya_race_registrations_1764697152671.csv)
    │
    ├─ Parse Rows
    │   ├─ Extract: sailNumber, yachtName, owner, design, length
    │   └─ Filter: Skip rows with empty owner
    │
    ├─ Extract Brands
    │   ├─ Parse design column → first word
    │   ├─ Apply brand mappings (J → J/Boats)
    │   ├─ Generate brand objects
    │   └─ Collect unique brands
    │
    ├─ Extract Builders
    │   ├─ One builder per brand
    │   ├─ Apply builder naming rules
    │   └─ Link to brand (bbrn_key)
    │
    ├─ Extract Designs
    │   ├─ Parse design + length
    │   ├─ Extract brand + number
    │   ├─ Generate design objects
    │   └─ Link to brand + builder
    │
    ├─ Extract Boats
    │   ├─ Parse yachtName, sailNumber
    │   ├─ Extract sail_cc, sail_no
    │   ├─ Generate boat objects
    │   └─ Link to design + builder
    │
    └─ Generate Files
        ├─ Read existing files
        ├─ Check for duplicates
        ├─ Append new entities
        ├─ Format as JavaScript
        └─ Write files (CRLF)
```

---

## Edge Cases & Special Handling

### Edge Case 1: Empty Owner

**Handling**: Skip boat entirely (no owner = no import)

### Edge Case 2: Duplicate Boat Names

**Handling**: Append number to key (e.g., "advantage", "advantage_2")

### Edge Case 3: Design with No Number

**Handling**: Use length column, truncate to integer

### Edge Case 4: Multiple Numbers in Design

**Handling**: Extract last number sequence from sailNumber (e.g., "J120 25439" → "25439") ✅ **CONFIRMED**

- "J120" is design name, "25439" is sail number
- Log if ambiguous: `mcode.warn("Multiple numbers found in sailNumber '{sailNumber}' for boat '{yachtName}', using last sequence '{sailNo}'")`

### Edge Case 5: Brand Variations

**Handling**: Use mapping structure to normalize (e.g., "J", "J Boat", "J Boats" → "J/Boats")

- Check brandMappings for variations
- Log unmapped brands: `mcode.warn("Brand '{brand}' not found in brandMappings for design '{design}', using as-is")`
- Complex cases (like "1D35") may require design-level mapping: `mcode.warn("Design '{design}' requires manual mapping (e.g., 1D35 = Nelson-Marek design, Carrol Marine builder)")`

### Edge Case 6: Sail Number Format Variations

**Handling**: Extract all numbers, use as-is (or parse intelligently → **QUESTION NEEDED**)

### Edge Case 7: Owner Name Variations

**Handling**: Generate user key from first + last, match against user.data.js

- Single name: Use `{name}_unknown`, log: `mcode.warn("Owner only had a single name '{name}', using '{name}_unknown' for key")`
- User not found: Log: `mcode.warn("Owner '{owner}' (key: '{userKey}') not found in user.data.js, boat '{yachtName}' will not be linked")`

### Edge Case 8: Existing Entities in Data Files

**Handling**: Check key before adding, skip duplicates

### Edge Case 9: Special Characters in Names

**Handling**: Remove or replace in keys, preserve in display names

### Edge Case 10: Length Column Format Variations

**Handling**: Extract numeric value, handle "39.3 feet", "40'", "27", "34'7\""

- Parse imperial formats (feet, feet+inches, decimal feet)
- Convert to millimeters for storage
- Log parsing issues: `mcode.warn("Unable to parse length '{lengthStr}' for boat '{yachtName}', using default")`

---

## Test Cases

### Test Case 1: Basic Boat Import

**Input**:

```csv
"USA 35007","Advantage","John Vermeulen","Finot First Class 12","39.3 feet","PHRF","paid"
```

**Expected Output**:

- Brand: `{ key: 'finot', name: 'Finot', display: 'Finot', description: 'Description of Finot', logo: 'data:image/x-icon;base64,' }`
- Builder: `{ key: 'finot_yachts', bbrn_key: 'finot', name: 'Finot Yachts', description: 'Manufacturer of Finot sailboats.', logo: 'data:image/x-icon;base64,' }`
- Design: `{ key: 'finot_12', bbrn_key: 'finot', bbld_key: 'finot_yachts', name: 'Finot 12', description: 'A 39 ft sailboat.' }`
- Boat: `{ key: 'advantage', bdsn_key: 'finot_12', bbld_key: 'finot_yachts', name: 'Advantage', sail_cc: 'USA', sail_no: '35007' }`

### Test Case 2: J/Boats Special Case

**Input**:

```csv
"USA 61232","ELEVATION","Timothy McGuire","J/112e","36.0'","ORC & PHRF","paid"
```

**Expected Output**:

- Brand: `{ key: 'jboats', name: 'J/Boats', ... }`
- Design: `{ key: 'j112e', name: 'J/112e', ... }`

### Test Case 3: Design Without Number

**Input**:

```csv
"USA 102","Boreas","Donald  Maxwell","Nonsuch","30","ORC & PHRF","paid"
```

**Expected Output**:

- Brand: `{ key: 'nonsuch', name: 'Nonsuch', ... }`
- Design: `{ key: 'nonsuch_30', name: 'Nonsuch 30', ... }` (uses length "30")

### Test Case 4: Sail Number Variations

**Input**:

```csv
"108","AirForce","Perrin Fortune","express 27 modified","27","ORC & PHRF","paid"
"NED  131","Northern Spy","John Steigenga","Dufour 45E","45","PHRF","paid"
```

**Expected Output**:

- Row 1: `sail_cc: 'USA'` (default), `sail_no: '108'`
- Row 2: `sail_cc: 'NED'`, `sail_no: '131'`

### Test Case 5: Empty Owner

**Input**:

```csv
"354","Baleia","","Ericson","31.58","PHRF","paid"
```

**Expected Output**: Boat skipped (no owner)

### Test Case 6: Duplicate Boat Names

**Input**:

```csv
"USA 502","Glory","Rob Wood","Nonsuch","30","PHRF","paid"
"244","Hepcat","Tim Sgrazzutti","Nonsuch 30","30","PHRF","paid"
```

**Expected Output**: Both boats imported, different keys based on yachtName

### Test Case 7: Complex Design Names

**Input**:

```csv
"USA 61666","Diablo","","J/111","36.5","ORC","paid"
"J120 25439","Hissy Fit","Dave Taylor","J120","40'","ORC & PHRF","paid"
```

**Expected Output**:

- Both map to J/Boats brand
- Designs: "J/111" and "J/120"

---

## Questions for Clarification

### Q1: User Key Generation - Single Name

**Question**: If owner is a single name (e.g., "Madonna"), should we:

- A) Use as `first`, leave `last` empty → `user_madonna_`
- B) Skip the boat (invalid format)
- C) Use as both first and last → `user_madonna_madonna`

**Recommendation**: Option B (skip) - single names are likely data errors

### Q2: Boat Name Capitalization

**Question**: Should we preserve intentional capitalization (e.g., "AirForce", "ELEVATION") or always normalize to Title Case?

**Recommendation**: Normalize to Title Case for consistency, but preserve if user explicitly requests

### Q3: Sail Number Extraction - Multiple Numbers

**Question**: For "J120 25439", should we extract:

- A) "12025439" (all numbers)
- B) "25439" (last number sequence)
- C) "120" (first number sequence)

**Recommendation**: Option B (last number sequence) - "25439" is likely the sail number, "120" is part of design

### Q4: Design Name - Keep Full or Extract Number?

**Question**: For "Finot First Class 12", should design name be:

- A) "Finot First Class 12" (keep full)
- B) "Finot 12" (extract number only)

**Recommendation**: Option A (keep full) - preserves model name

### Q5: Design Key - Slash Handling

**Question**: For "J/120", should design key be:

- A) "j120" (remove slash)
- B) "j_120" (replace slash with underscore)

**Recommendation**: Option A (remove slash) - cleaner keys

### Q6: Builder Naming Pattern

**Question**: Should builder names be:

- A) "{Brand} Yachts" (e.g., "Tartan Yachts")
- B) "{Brand} Builder" (e.g., "Tartan Builder")
- C) "{Brand} Composites" (for J/Boats style)
- D) Use existing pattern from boat_builder.data.js

**Recommendation**: Option A ("{Brand} Yachts") - consistent with industry naming

### Q7: Builder Key Pattern

**Question**: Should builder keys be:

- A) "{brand_key}\_yachts" (e.g., "tartan_yachts")
- B) "{brand_key}\_builder" (e.g., "tartan_builder")
- C) Custom per brand (like "jcomposites" for J/Boats)

**Recommendation**: Option A ("{brand_key}\_yachts") - consistent pattern

### Q8: User-Boat Linking

**Question**: Should the import script:

- A) Update user.data.js to add boat_keys (integrated)
- B) Generate separate mapping file (deferred)
- C) Create separate link script (modular)

**Recommendation**: Option C (separate script) - keeps import focused, allows review

### Q9: Duplicate Handling

**Question**: If a boat key already exists in boat.data.js, should we:

- A) Skip (don't add duplicate)
- B) Append number (e.g., "advantage_2")
- C) Update existing (merge data)

**Recommendation**: Option A (skip) - avoid duplicates, log warning

### Q10: Description Generation

**Question**: How should we generate descriptions for brands/builders/designs?

- A) Generic template: "Description of {Name}"
- B) More detailed: "A {length} ft {type} sailboat by {designer}"
- C) Leave empty (user fills later)

**Recommendation**: Option B (detailed) - more useful, can be updated later

---

## Risk Assessment

### Risk 1: Owner Name Mismatches

**Risk**: Owner names in CSV may not match user keys exactly
**Impact**: Medium - Boats won't link to users
**Mitigation**: Fuzzy matching, manual review mapping file

### Risk 2: Design Parsing Errors

**Risk**: Complex design names may parse incorrectly
**Impact**: Medium - Wrong brand/design associations
**Mitigation**: Extensive mapping structure, manual review

### Risk 3: Duplicate Keys

**Risk**: Generated keys may conflict with existing data
**Impact**: Low - Script checks for duplicates
**Mitigation**: Uniqueness checks, append numbers if needed

### Risk 4: Data File Corruption

**Risk**: Script may corrupt existing data files
**Impact**: High - Loss of existing data
**Mitigation**: Backup files before import, validate syntax after write

### Risk 5: Special Character Handling

**Risk**: Special characters in names may break key generation
**Impact**: Low - Script handles special chars
**Mitigation**: Sanitize keys, preserve display names

### Risk 6: Performance with Large CSV

**Risk**: Large CSV files may be slow to process
**Impact**: Low - CSV has ~106 rows
**Mitigation**: Efficient parsing, batch processing if needed

---

## Next Steps

1. **Answer Questions**: Resolve all 10 questions above
2. **Review Design**: Confirm parsing rules and mappings
3. **Create Mapping File**: Build comprehensive brand/design mappings
4. **Implement Script**: Write `import.boats.js` following design
5. **Test with Sample Data**: Validate with subset of CSV
6. **Full Import**: Run on complete CSV file
7. **Review Output**: Validate generated data files
8. **User Linking**: Create/run user-boat linking script
9. **Documentation**: Update developer docs

---

## Plan Review & Validation

### ✅ Confirmed Requirements

1. **File Operations**

   - ✅ Append to existing seed data files (not overwrite)
   - ✅ Write all files with CRLF (`\r\n`) line endings (Windows 11)
   - ✅ Follow patterns from `server/bin/import.users.js`

2. **User-Boat Linking**

   - ✅ Integrated into import script (not separate script)
   - ✅ Update `user.data.js` to add `boat_keys` arrays
   - ✅ Match boats to users via owner name → user key lookup
   - ✅ User key format: `{first}_{last}` (lowercase, underscore-separated, matches import.users.js pattern)

3. **Code Patterns**
   - ✅ Use CSV parser from `import.users.js`
   - ✅ Use formatting functions from `import.users.js`
   - ✅ Use CRLF normalization from `import.users.js`
   - ✅ Follow existing data file structure patterns

### ⚠️ Remaining Questions for 95%+ Confidence

The following questions need answers to ensure accurate implementation:

#### Q1: User Key Format Clarification

**Question**: You mentioned "user keys generated will be 'user*{first}*{last}'", but `import.users.js` generates keys as `{first}_{last}` (no "user\_" prefix). Which format should we use for lookup?

- A) `user_john_vermeulen` (with prefix)
- B) `john_vermeulen` (without prefix, matches user.data.js)

**Current Assumption**: Option B (without prefix) - matches existing user.data.js format

#### Q2: Boat Name Capitalization

**Question**: Should boat names be normalized to Title Case, or preserve original capitalization?

- A) Always normalize: "AirForce" → "Airforce", "ELEVATION" → "Elevation"
- B) Preserve original: "AirForce" → "AirForce", "ELEVATION" → "ELEVATION"

**Recommendation**: Option A (normalize) for consistency

#### Q3: Sail Number Extraction - Multiple Numbers

**Question**: For "J120 25439", which number should be extracted as sail_no?

- A) "12025439" (all numbers concatenated)
- B) "25439" (last number sequence - likely sail number)
- C) "120" (first number sequence - part of design name)

**Recommendation**: Option B ("25439") - "120" is part of design, "25439" is sail number

#### Q4: Design Name Parsing

**Question**: For "Finot First Class 12", should design name be:

- A) "Finot First Class 12" (keep full model name)
- B) "Finot 12" (extract brand + number only)

**Recommendation**: Option A (keep full) - preserves model information

#### Q5: Design Key - Slash Handling

**Question**: For "J/120", should design key be:

- A) "j120" (remove slash)
- B) "j_120" (replace slash with underscore)

**Recommendation**: Option A (remove slash) - cleaner, matches existing "j112e" pattern

#### Q6: Builder Naming Pattern

**Question**: Should builder names follow:

- A) "{Brand} Yachts" (e.g., "Tartan Yachts")
- B) "{Brand} Builder" (e.g., "Tartan Builder")
- C) Special case for J/Boats: "J Composites" (existing pattern)
- D) Follow existing boat_builder.data.js patterns exactly

**Recommendation**: Option A with special case C for J/Boats

#### Q7: Builder Key Pattern

**Question**: Should builder keys be:

- A) "{brand_key}\_yachts" (e.g., "tartan_yachts")
- B) "{brand_key}\_builder" (e.g., "tartan_builder")
- C) Custom per brand (e.g., "jcomposites" for J/Boats)

**Recommendation**: Option A with special case C for J/Boats

#### Q8: Duplicate Handling ✅ ANSWERED

**Answer**: Skip duplicates (don't add if key already exists)

#### Q9: Description Generation

**Question**: How should descriptions be generated?

- A) Generic: "Description of {Name}"
- B) Detailed: "A {length} ft sailboat by {brand}"
- C) Empty (user fills later)

**Recommendation**: Option B (detailed) - more useful, can be updated later

#### Q10: Single Name Owners

**Question**: If owner is single name (e.g., "Madonna"), should we:

- A) Skip boat (invalid format)
- B) Use as both first and last: `madonna_madonna`
- C) Use as first only: `madonna_` (empty last)

**Recommendation**: Option A (skip) - likely data error

### 📋 Implementation Readiness Checklist

- [x] CSV parsing pattern confirmed (import.users.js)
- [x] File writing pattern confirmed (CRLF, append)
- [x] User-boat linking approach confirmed (integrated)
- [x] User key generation pattern confirmed
- [x] User key format clarified (no prefix, matches user.data.js)
- [x] Boat name capitalization rule confirmed (Title Case)
- [x] Sail number extraction rule confirmed (last number sequence)
- [x] Design name parsing rule confirmed (remove brand name)
- [x] Design key format confirmed (lowercase, no punctuation)
- [x] Builder naming/key patterns confirmed ({Brand} Manufacturing, {brand}\_builder)
- [x] Description generation template confirmed (detailed from design + length)
- [x] Single name owner handling confirmed ({name}\_unknown)

---

## Confidence Level

**Current Confidence**: **82%** ⬆️ (up from 75%)

**Confidence Breakdown**:

- ✅ CSV parsing: 98% (proven pattern from import.users.js, all edge cases understood)
- ✅ File generation: 98% (proven pattern, CRLF confirmed, append confirmed)
- ✅ User-boat linking: 98% (integrated approach confirmed, pattern from import.users.js, all cases handled, logging added)
- ✅ User key generation: 98% (pattern confirmed, single name handling clarified: `{name}_unknown`, logging added)
- ✅ Brand extraction: 98% (J/Boats special case clear, flexible mapping structure confirmed with multiple variations)
- ✅ Design parsing: 98% (brand removal rule clear: remove brand name from design, length fallback confirmed, metric conversion added)
- ✅ Sail number extraction: 98% (last number sequence rule confirmed: "J120 25439" → "25439", logging added)
- ✅ Builder creation: 98% ({Brand} Manufacturing pattern confirmed, J Composites special case)
- ✅ Description generation: 98% (detailed template confirmed: "{Design}, a {length}' sailboat from {Brand}")
- ✅ Key generation: 98% (all patterns confirmed: lowercase, no punctuation, underscores)
- ✅ Length conversion: 98% (imperial to metric conversion confirmed: feet/inches → millimeters)
- ✅ Logging: 98% (mcode.warn() and mcode.error() patterns confirmed from mongo.reset.js)

**All Questions Answered** ✅:

1. ✅ User key format: `{first}_{last}` (no prefix, matches user.data.js)
2. ✅ Boat name: Normalize to Title Case
3. ✅ Sail number: Extract last number sequence ("J120 25439" → "25439")
4. ✅ Design name: Remove brand name, keep rest ("Finot First Class 12" → "First Class 12")
5. ✅ Design key: Lowercase, remove punctuation ("J/120" → "j120")
6. ✅ Builder naming: "{Brand} Manufacturing" (J Composites exception for J/Boats)
7. ✅ Builder key: "{brand}\_builder" (jcomposites exception for J/Boats)
8. ✅ Description: Detailed from design + length ("Beneteau First 40.7, a 40' sailboat from Beneteau")
9. ✅ Single name: "{name}\_unknown" (e.g., "Madonna" → "madonna_unknown")

**Remaining Minor Considerations** (4% uncertainty - implementation details):

1. Edge case handling for unusual design formats (e.g., "1D35", "NM 43", "S&S Pilot 33")
2. Length column parsing edge cases (e.g., "31.58", "40'", "39.3 feet", "34'7\"")
3. Brand mapping for edge cases (e.g., "S&S", "NM", "C&C", "C & C")
4. Validation of generated keys against existing data files to prevent conflicts

**These are implementation details that can be handled during coding with proper error handling, logging, and testing.**

---

---

## Final Plan Summary

### ✅ All Requirements Confirmed

1. **Logging**: Use `mcode.warn()` and `mcode.error()` for all issues during import

   - Single name owners: `mcode.warn("Owner only had a single name '{name}', using '{name}_unknown' for key")`
   - Empty owners: `mcode.warn("Skipping boat '{yachtName}': owner is empty")`
   - User not found: `mcode.warn("Owner '{owner}' (key: '{userKey}') not found in user.data.js")`
   - Duplicate keys: `mcode.warn("Skipping duplicate {entity} key '{key}'")`
   - Unmapped brands: `mcode.warn("Brand '{brand}' not found in brandMappings")`
   - Complex designs: `mcode.warn("Design '{design}' requires manual mapping")`
   - Length parsing issues: `mcode.warn("Unable to parse length '{lengthStr}'")`

2. **Brand Mapping Structure**: Flexible mapping supporting multiple variations

   - Format: `{ variation: normalizedBrand }`
   - Example: `{ 'C&C': 'C&C Yachts', 'C & C': 'C&C Yachts', 'C+C': 'C&C Yachts' }`
   - Complex cases (like "1D35") logged as warnings for manual mapping

3. **Length Conversion**: Imperial to metric (millimeters)

   - Parse formats: "39.3 feet", "27", "40'", "31.58", "34'7\""
   - Convert: `(feet * 304.8) + (inches * 25.4)` = millimeters
   - Store in `boat_design.hull.length` as integer (mm)

4. **Design Name Parsing**: Remove brand name from design
   - "Finot First Class 12" → Design: "First Class 12" (remove "Finot")
   - "Tartan33" → Design: "33" (remove "Tartan")
   - Use length column if no number in design name

### 📋 Implementation Checklist

- [x] CSV parsing pattern (import.users.js)
- [x] File writing pattern (CRLF, append)
- [x] User-boat linking (integrated)
- [x] User key generation (`{first}_{last}`, single name: `{name}_unknown`)
- [x] Boat name normalization (Title Case)
- [x] Sail number extraction (last number sequence)
- [x] Design name parsing (remove brand name)
- [x] Design key format (lowercase, no punctuation)
- [x] Builder naming/key patterns ({Brand} Manufacturing, {brand}\_builder)
- [x] Description generation (detailed template)
- [x] Length conversion (imperial → millimeters)
- [x] Brand mapping structure (flexible, multiple variations)
- [x] Logging (mcode.warn(), mcode.error())

---

**Status**: ✅ **READY FOR IMPLEMENTATION**

**Next Action**: Proceed with implementation when explicitly approved. All major design decisions confirmed.
