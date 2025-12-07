# Club Data Collection - Interface Contracts & User Stories

## Interface Contracts

### 1. Data File Interface

#### File: `server/seed/data/club.data.js`

**Structure Contract**:

```javascript
const prefix = 'club';
const club = [
    {
        // Required fields (must exist)
        key: String,        // Unique identifier (e.g., 'byc')
        name: String,       // Full club name (e.g., 'Bayview Yacht Club')
        display: String,    // Display abbreviation (e.g., 'BYC')

        // Optional contact fields
        email?: String,     // Primary email address
        email_?: {
            primary?: String,
            office?: String,
            work?: String
        },
        phone?: String,     // Primary phone number (string format)
        phone_?: {
            primary?: String,
            office?: String,    // May include extension: "number;extension"
            gate?: String
        },

        // Optional location fields
        address?: {
            street?: String,
            city?: String,
            state?: String,     // US state code or Canadian province code
            zip?: String,       // US ZIP or Canadian postal code
            country?: String    // 'USA' or 'CAN'
        },

        // Optional descriptive fields
        description?: String,   // Comprehensive description (100+ words recommended)
        logo?: String          // Base64 data URL: 'data:image/x-icon;base64,{data}'
    },
    // ... more club objects
];

module.exports = {
    prefix,
    club
};
```

**Invariants**:

- `prefix` must always be `'club'`
- `club` must be an array
- Each club object must have `key`, `name`, and `display`
- Club `key` values must be unique within the array
- All optional fields follow the BYC template structure when present

**Preconditions**:

- File exists and is valid JavaScript
- Existing complete entries (BYC, DYC, GLYC) remain unchanged
- File structure maintains consistency

**Postconditions**:

- All incomplete club entries have been updated with collected data
- All entries follow the BYC template structure
- File syntax is valid
- No breaking changes introduced

---

### 2. Schema Interface Contract

#### Schema: `server/model/mongo/club.mongo.js`

**Field Type Contract**:

```typescript
interface ClubSchema {
  // Auto-generated fields
  id: string; // Required, unique, auto-generated
  rev: number; // Auto-incremented on update

  // Common entity fields
  key: string; // Optional, unique
  type: "club"; // Required, default: 'club'
  state: "active" | "inactive" | "archived"; // Required, default: 'active'
  created_at: Date; // Auto-set on create
  updated_at: Date; // Auto-set on update

  // Club-specific fields
  name: string; // Required
  display?: string;
  email?: string;
  email_?: {
    primary?: string;
    office?: string;
    work?: string;
    [key: string]: string | undefined;
  };
  phone?: Object; // Schema says Object, but data uses String
  phone_?: {
    primary?: string;
    office?: string;
    gate?: string;
    [key: string]: string | undefined;
  };
  address?: {
    street?: string;
    city?: string;
    state?: string;
    zip?: string;
    country?: string;
    [key: string]: string | undefined;
  };
  description?: string;
  logo?: string;
}
```

**Validation Rules**:

- `name` is required
- `key` must be unique if provided
- `type` must be 'club'
- `state` must be one of: 'active', 'inactive', 'archived'
- `phone` field type mismatch: Schema says Object, but data uses String (follow data pattern)

**Data Format Rules**:

- Email: Valid email format (contains @ and domain)
- Phone: String format, no dashes/spaces (e.g., '13138221853')
- Phone extension: Format as 'number;extension' (e.g., '13138221853;105')
- Address state: 2-letter code (US: 'MI', 'IL', etc. | Canada: 'ON', 'BC', etc.)
- Address zip: US ZIP (5 digits) or Canadian postal code (A1A 1A1 format)
- Address country: 'USA' or 'CAN'
- Logo: Base64 data URL format: 'data:image/x-icon;base64,{data}'

---

### 3. Seeder Interface Contract

#### Seeder: `server/seed/seeder.js`

**Usage Contract**:

- Seeder expects `club.data.js` to export `prefix` and `club` array
- Seeder matches records by `key` field
- Seeder transforms `key` values by prepending `prefix` (e.g., 'byc' → 'club_byc')
- Seeder handles idempotent operations (insert new, update existing, skip unchanged)

**Data Transformation Rules**:

- Keys are prefixed: `{prefix}_{key}` → `club_byc`
- Timestamps are auto-managed: `created_at` on insert, `updated_at` on update
- Revision counter: `rev` is auto-incremented on update

**Expected Behavior**:

- New clubs are inserted with `created_at` timestamp
- Existing clubs are updated if fields differ, with `updated_at` timestamp
- Unchanged clubs are skipped
- All operations are logged with statistics

---

## User Stories

### Story 1: Complete Club Data Collection

**As a** developer
**I want** all yacht clubs in the seed data file to have complete information
**So that** the application has accurate and comprehensive club data for all entries

**Acceptance Criteria**:

- [ ] All 21 clubs have complete data entries (or best available)
- [ ] All entries follow the BYC template structure
- [ ] All data validates against schema requirements
- [ ] File maintains proper JavaScript syntax
- [ ] No breaking changes to existing complete entries

**Definition of Done**:

- All incomplete clubs have been researched and updated
- Data has been validated for format and completeness
- File has been updated and syntax verified
- Documentation has been updated

---

### Story 2: Consistent Data Formatting

**As a** developer
**I want** all club data to follow consistent formatting rules
**So that** the data is uniform and easy to work with programmatically

**Acceptance Criteria**:

- [ ] All email addresses follow consistent format
- [ ] All phone numbers use consistent format (no dashes/spaces)
- [ ] All addresses follow country-specific formatting
- [ ] All descriptions follow similar style and length
- [ ] All optional fields use consistent structure

**Definition of Done**:

- Format validation rules applied to all entries
- Inconsistencies identified and corrected
- Format documentation updated

---

### Story 3: Accurate Contact Information

**As a** user
**I want** accurate contact information for yacht clubs
**So that** I can reach out to clubs when needed

**Acceptance Criteria**:

- [ ] Email addresses are valid and verified
- [ ] Phone numbers are in correct format
- [ ] Contact information is from official sources
- [ ] Multiple contact methods available when possible

**Definition of Done**:

- All contact information verified from official sources
- Email domains validated
- Phone numbers formatted correctly
- Contact information cross-referenced

---

### Story 4: Complete Location Data

**As a** user
**I want** accurate location information for yacht clubs
**So that** I can find and visit clubs

**Acceptance Criteria**:

- [ ] All addresses are complete (street, city, state, zip, country)
- [ ] Addresses are verified on maps
- [ ] Addresses follow country-specific formatting
- [ ] Location data is accurate

**Definition of Done**:

- All addresses verified with mapping services
- Address formatting follows country standards
- Missing location data identified and noted

---

### Story 5: Comprehensive Descriptions

**As a** user
**I want** detailed descriptions of yacht clubs
**So that** I can learn about their history and features

**Acceptance Criteria**:

- [ ] All descriptions are at least 100 words
- [ ] Descriptions include founding year
- [ ] Descriptions include location details
- [ ] Descriptions include key features/amenities
- [ ] Descriptions include historical context
- [ ] Descriptions maintain professional tone

**Definition of Done**:

- All descriptions meet minimum length requirement
- Historical information included where available
- Feature lists comprehensive
- Professional tone maintained throughout

---

### Story 6: Logo Assets

**As a** developer
**I want** club logos available as base64 data URLs
**So that** logos can be displayed in the application

**Acceptance Criteria**:

- [ ] Logos are converted to base64 format when found
- [ ] Logo format follows data URL structure
- [ ] Empty placeholder used when logo unavailable
- [ ] Logo quality is appropriate for display

**Definition of Done**:

- Logos located and converted (where available)
- Base64 encoding completed
- Placeholders used for missing logos
- Logo format validated

---

## Technical Requirements

### Data Collection Requirements

1. **Research Phase**:

   - Use web search to find official club websites
   - Extract contact information from official sources
   - Verify addresses with mapping services
   - Research historical information from reliable sources

2. **Data Extraction**:

   - Extract email addresses from contact pages
   - Extract phone numbers from contact pages
   - Extract addresses from location pages or maps
   - Extract historical information from about pages or Wikipedia

3. **Data Formatting**:

   - Standardize email formats
   - Standardize phone number formats
   - Format addresses according to country
   - Write descriptions in consistent style

4. **Data Validation**:
   - Validate email formats
   - Validate phone number formats
   - Verify addresses on maps
   - Check description completeness

### File Update Requirements

1. **Structure Preservation**:

   - Maintain existing file structure
   - Preserve existing complete entries
   - Follow BYC template format
   - Maintain proper indentation

2. **Syntax Requirements**:

   - Valid JavaScript syntax
   - Proper object structure
   - Consistent formatting
   - No syntax errors

3. **Data Integrity**:
   - No breaking changes
   - All existing data preserved
   - New data properly formatted
   - Schema compliance maintained

---

## Error Handling

### Missing Information

**Scenario**: Club information not publicly available
**Handling**:

- Leave optional fields undefined (not empty strings)
- Use best available information
- Document missing information in notes

### Incomplete Information

**Scenario**: Partial information available
**Handling**:

- Use available information
- Leave unavailable fields undefined
- Document completeness status

### Format Inconsistencies

**Scenario**: Information in different formats
**Handling**:

- Standardize to BYC template format
- Convert phone numbers to consistent format
- Format addresses according to country standards

### Data Validation Failures

**Scenario**: Collected data doesn't validate
**Handling**:

- Re-verify from source
- Correct format issues
- Document validation failures

---

## Testing Requirements

### Unit Tests

- [ ] File syntax validation
- [ ] Schema compliance check
- [ ] Format validation for each field type
- [ ] Template structure validation

### Integration Tests

- [ ] Seeder can process updated file
- [ ] Data inserts correctly into database
- [ ] Data updates correctly in database
- [ ] No breaking changes to existing functionality

### Manual Verification

- [ ] All clubs reviewed for completeness
- [ ] Sample entries verified for accuracy
- [ ] Contact information tested (if possible)
- [ ] Addresses verified on maps

---

## Documentation Requirements

### Required Documentation

1. **Implementation Plan** ✓ (Created)
2. **Data Structure Diagram** ✓ (Created)
3. **Collection Template** ✓ (Created)
4. **Interface Contracts** ✓ (This document)
5. **Data Collection Log** (To be created during implementation)
6. **Validation Report** (To be created after implementation)

### Documentation Updates

- Update README if club data structure changes
- Document any deviations from standard format
- Document any missing information and reasons
- Document data sources used

---

**Document Version**: 1.0
**Created**: 2024
**Status**: Planning Phase - Awaiting Approval
