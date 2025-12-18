# AIN - FEATURE - ENTITY_CARDS_AND_SVG_FIELDS

## Metadata

- **Type**: FEATURE
- **Issue #**: [if applicable]
- **Created**: [DATE]
- **Status**: READY FOR IMPLEMENTATION

---

## C: CONCEPT/CHANGE/CORRECTION - Discuss ideas without generating code

This feature involves enabling SVG field support and entity card displays across multiple entities:

1. **Locale Entity**: Add `flag` field to model schema and create new API endpoints
2. **Org and Club Entities**:
   - Enable existing `logo` field in API responses
   - Add additional fields (description, email, phone, address) to API responses for entity cards
3. **Entity Cards**: Create reusable frontend component to display entity information in cards, including logos, contact details, and other relevant information

The feature enables displaying SVG content (flags for locales, logos for orgs/clubs) and detailed entity information in card format within the "Manage {Entity}" dialogs.

---

## D: DESIGN - Design detailed solution

### Overview

The design follows a consistent pattern across entities:

- **Locale**: Add `flag` field to schema and create new API layer (model → controller → route)
- **Org/Club API**:
  - Enable existing `logo` field by adding it to API response `.select()` clauses
  - Add additional fields (description, email, phone, address) to support entity cards
- **Entity Cards Frontend**:
  - Create reusable EntityCard component
  - Display cards to the right of "Manage {Entity}" forms
  - Show logos, contact information, and other entity details
  - Responsive design (two-column on desktop, single column on mobile)

Both use SVG strings stored in the database and follow the same security and performance considerations.

---

## P: PLAN - Create implementation plan

# Implementation Plan: Entity Cards and SVG Fields

## Executive Summary

This document outlines a comprehensive plan to add SVG field support and entity card displays across multiple entities:

1. **Locale Entity**: Add `flag` field to model schema, CRUD operations, and create new API endpoints/controllers following the org/club pattern
2. **Org and Club Entities**:
   - Enable the existing `logo` field in API responses by adding it to `.select()` clauses
   - Add additional fields (description, email, phone, address) to API responses to support entity cards
3. **Entity Cards Frontend**: Create reusable EntityCard component and integrate into "Manage {Entity}" dialogs to display detailed entity information including logos, contact details, and other relevant information

The feature enables displaying SVG content (flags for locales, logos for orgs/clubs) and detailed entity information in card format within account management dialogs.

**Status**: Planning Phase - Validated and Ready for Approval

---

## Part 1: Add `flag` Field to Locale Model and API

### Current State Analysis

#### Existing Field: `flag` in Seed Data ✅

The `locale` seed data file already contains `flag` fields:

- **Location**: `server/seed/app/locale.data.js` (app-level static data)
- **Format**: SVG markup as string (e.g., `<svg xmlns="http://www.w3.org/2000/svg" ...></svg>`)
- **Coverage**: All 45 locale records have `flag` field with placeholder SVG content
- **Current State**:
  - ✅ Present in seed data
  - ❌ NOT in Mongoose schema
  - ❌ NOT handled in model CRUD operations
  - ❌ NOT available via API (no endpoints exist)

#### Model Schema: Missing `flag` Field ❌

**File**: `server/model/mongo/locale.mongo.js` (ONLY file to edit)

**Important**: Files in `server/model/mongo/` are automatically promoted to `server/model/` by `server/bin/setup.js` during setup. We should ONLY edit the mongo version.

**Current Schema Fields** (lines 47-59):

- `name` (String, required)
- `country` (String, required)
- `language` (String, required)
- **Missing**: `flag` field

#### CRUD Operations: Missing `flag` Handling ❌

**File**: `server/model/mongo/locale.mongo.js`

**Current `create()` Function** (lines 81-92):

- Does NOT include `flag` in `localeData` object
- Only handles: `key`, `name`, `country`, `language`
- JSDoc does NOT document `flag` parameter

**Other CRUD Functions**:

- `get()`, `getAll()`, `getByLanguage()` use `.lean()` which returns all schema fields
- `update()` uses spread operator, so it will work once schema is updated
- `delete()` - no changes needed

#### API Endpoints: None Exist ❌

**Current State**: Unlike `org` and `club`, there are **no API routes or controllers** for `locale`:

- ❌ No `server/api/locale.route.js`
- ❌ No `server/controller/locale.controller.js`

**Pattern to Follow**: Based on `org` and `club`:

- Simple `list` endpoint with optional search
- Returns specific fields via `.select()`
- Uses `auth.verify('user')` middleware
- Auto-loaded via `server/api/index.js`

### Requirements & Scope

#### Functional Requirements

1. **FR1**: Add `flag` field to Mongoose schema for `locale`
2. **FR2**: Update `create()` function to handle `flag` field
3. **FR3**: Update JSDoc for `create()` to document `flag` parameter
4. **FR4**: Create `locale.controller.js` with `list` function (following org/club pattern)
5. **FR5**: Create `locale.route.js` with GET `/api/locale` endpoint
6. **FR6**: Include `flag` in API response `.select()` clause
7. **FR7**: Support optional search by name, language, or country
8. **FR8**: Ensure `flag` field is required (must be provided)

#### Non-Functional Requirements

1. **NFR1**: Maintain backward compatibility - existing fields unchanged
2. **NFR2**: Follow same pattern as `logo` field in org/club
3. **NFR3**: Follow same API pattern as org/club endpoints
4. **NFR4**: Performance - SVG strings should not significantly impact response size
5. **NFR5**: Security - SVG content should be safe (seed data is trusted)

### Detailed Implementation Plan

#### Phase 1: Model Schema Updates

**File**: `server/model/mongo/locale.mongo.js` (ONLY file to edit)

**Change Schema To**:

```javascript
// locale specific fields
name: {
    type: String,
    required: true
},
country: {
    type: String,
    required: true
},
language: {
    type: String,
    required: true
},
flag: {
    type: String,
    required: true
}
```

#### Phase 2: CRUD Operations Updates

**File**: `server/model/mongo/locale.mongo.js`

**Update `create()` Function**:

```javascript
exports.create = async function ({ key = "", name, country, language, flag }) {
  const localeData = {
    key: key,
    name: name,
    country: country,
    language: language,
    flag: flag,
  };

  // Instantiate through Mongoose to ensure schema field order
  return await mongo.createOrdered(KEY_PREFIX, Locale, localeData);
};
```

**Update JSDoc**:

```javascript
/**
 * @func create
 * @memberof model.mongo.locale
 * @desc [C]RUD Create a new locale record.
 * @api public
 * @param {object} params Parameters object
 * @param {string} [params.key] Unique locale key (e.g., 'en-US', 'es-ES')
 * @param {string} params.name Locale display name (e.g., 'English (United States)')
 * @param {string} params.country Country code (e.g., 'US', 'ES')
 * @param {string} params.language Language code (e.g., 'en', 'es')
 * @param {string} params.flag Flag SVG markup string
 * @returns {Promise<object>} Created locale object
 */
```

#### Phase 3: Controller Creation

**File**: `server/controller/locale.controller.js` (NEW FILE)

**Implementation**:

```javascript
const joi = require("joi");
const locale = require("../model/locale.model");
const utility = require("../helper/utility");

/* CRUD
 * locale.* - Create, Read, Update, Delete functions
 */

/**
 * @func list
 * @memberof controller.locale
 * @desc C[R]UD List locales with optional search by name, language, or country.
 * @api private
 * @param {object} req - Express request object (requires authentication middleware)
 * @param {object} req.query - Query parameters
 * @param {string} [req.query.search] - Search term to filter by name, language, or country
 * @param {object} res - Express response object
 * @returns {Promise<void>} Sends array of locale objects
 */
exports.list = async function (req, res) {
  const schema = joi.object({
    search: joi.string().allow(""),
  });

  const { error, value } = schema.validate(req.query, {
    abortEarly: false,
    stripUnknown: true,
  });

  if (error) {
    return res.status(400).send({
      message: error.details.map((err) => err.message).join(", "),
    });
  }

  const search = value?.search || "";

  // Build search query
  const query = {};
  if (search) {
    query.$or = [
      { name: { $regex: search, $options: "i" } },
      { language: { $regex: search, $options: "i" } },
      { country: { $regex: search, $options: "i" } },
    ];
  }

  const locales = await locale.schema
    .find(query)
    .select("id key name country language flag")
    .lean()
    .sort({ name: 1 });

  res.status(200).send({
    data: locales,
  });
};
```

#### Phase 4: API Route Creation

**File**: `server/api/locale.route.js` (NEW FILE)

**Implementation**:

```javascript
const express = require("express");
const auth = require("../model/auth.service");
const localeController = require("../controller/locale.controller");
const api = express.Router();
const use = require("../helper/utility").use;

api.get("/api/locale", auth.verify("user"), use(localeController.list));

module.exports = api;
```

### API Implementation Details

#### GET /api/locale

**Request**:

```
GET /api/locale?search=<optional>
Headers: Authorization: Bearer <token>
```

**Response Format**:

```json
{
  "data": [
    {
      "id": "locl_...",
      "key": "en-US",
      "name": "English (United States)",
      "country": "US",
      "language": "en",
      "flag": "<svg xmlns=\"http://www.w3.org/2000/svg\" width=\"64\" height=\"64\" viewBox=\"0 0 24 24\" fill=\"none\"></svg>"
    }
  ]
}
```

---

## Part 2: Enable `logo` Field in Org and Club API Responses

### Current State Analysis

#### Existing Field: `logo` ✅

Both `org` and `club` entities already have a `logo` field that is **fully implemented**:

- **Type**: `String` (optional)
- **Purpose**: Stores SVG markup as string
- **Current Implementation Status**:
  - ✅ **Mongoose schemas**: Defined in `org.mongo.js` (line 84-87) and `club.mongo.js` (line 84-87)
  - ✅ **CRUD operations**: Handled in `create()` functions (line 121 in both files)
  - ✅ **Seed data**: All records have `logo` field with SVG content
  - ❌ **API responses**: NOT currently returned (excluded by `.select()`)

### Key Observations

1. **Models are complete**: `logo` field exists and works correctly
2. **Seed data is ready**: All records have `logo` with SVG content
3. **API limitation**: Controllers use `.select('id acronym name')` which excludes `logo`
4. **Simple fix**: Only need to add `logo` to the `.select()` clause

### Requirements & Scope

#### Functional Requirements

1. **FR1**: Include `logo` field in API responses for `/api/org` endpoint
2. **FR2**: Include `logo` field in API responses for `/api/club` endpoint
3. **FR3**: Ensure `logo` field is optional (may be `null` or `undefined`)

#### Non-Functional Requirements

1. **NFR1**: Maintain backward compatibility - existing fields unchanged
2. **NFR2**: No breaking changes to existing API contracts
3. **NFR3**: Performance - SVG strings should not significantly impact response size
4. **NFR4**: Security - SVG content should be safe (seed data is trusted)

### Detailed Implementation Plan

#### Phase 1: Controller Updates (ONLY CHANGE NEEDED)

**File**: `server/controller/org.controller.js`

**Current Code** (line 50):

```javascript
const orgs = await org.schema
  .find(query)
  .select("id acronym name")
  .lean()
  .sort({ name: 1 });
```

**Change To**:

```javascript
const orgs = await org.schema
  .find(query)
  .select("id acronym name logo")
  .lean()
  .sort({ name: 1 });
```

**File**: `server/controller/club.controller.js`

**Current Code** (line 50):

```javascript
const clubs = await club.schema
  .find(query)
  .select("id acronym name")
  .lean()
  .sort({ name: 1 });
```

**Change To**:

```javascript
const clubs = await club.schema
  .find(query)
  .select("id acronym name logo")
  .lean()
  .sort({ name: 1 });
```

### API Changes

#### Current API Response Format

**GET /api/org**:

```json
{
  "data": [
    {
      "id": "orgn_...",
      "acronym": "DRYA",
      "name": "Detroit Regional Yacht-Racing Association"
    }
  ]
}
```

#### Proposed API Response Format

**GET /api/org**:

```json
{
  "data": [
    {
      "id": "orgn_...",
      "acronym": "DRYA",
      "name": "Detroit Regional Yacht-Racing Association",
      "logo": "<svg xmlns=\"http://www.w3.org/2000/svg\" width=\"64\" height=\"64\" viewBox=\"0 0 24 24\" fill=\"none\"></svg>"
    }
  ]
}
```

**Changes**:

- Additive only - new field added
- Existing fields unchanged
- Field is optional (may be `null` or `undefined` for records without logos)

---

## Part 3: Entity Cards Implementation

### Overview

Add Entity Cards to the right side of "Manage {Entity}" dialogs (Organizations, Clubs, Sailing Vessels). These cards display detailed information about entities that a user is connected to, including their logo, contact information, and other relevant details.

### Current State Analysis

#### Frontend Components

- **Location**: `client/src/views/account/`
- **Files**:
  - `organizations.jsx` - Manages user's organizations
  - `clubs.jsx` - Manages user's clubs
  - `svs.jsx` - Manages user's sailing vessels

#### Current API Response

**Current**: Controllers return limited fields:

- `id`, `acronym`, `name`, `logo` (after Part 2)

**Missing Fields**: `description`, `address`, `phone`, `email`, `phone_`, `email_`, `address_`

#### Schema Fields Available

**Organizations & Clubs**:

- `id`, `key`, `name`, `acronym`
- `description`
- `email`, `email_` (object with multiple emails)
- `phone`, `phone_` (object with multiple phones)
- `address`, `address_` (object with structured address)
- `logo` (SVG string)

**Sailing Vessels**:

- `id`, `key`, `name`, `display`
- `description`
- `boat_key`, `boat_id`
- Note: Sailing Vessels may not have the same contact fields as Organizations/Clubs

### Requirements & Scope

#### Functional Requirements

1. **FR1**: Update org/club controllers to include additional fields in API responses
2. **FR2**: Create reusable EntityCard component
3. **FR3**: Integrate EntityCard into Organizations, Clubs, and Sailing Vessels views
4. **FR4**: Display cards to the right of "Manage {Entity}" forms on desktop
5. **FR5**: Display cards below forms on mobile (responsive design)
6. **FR6**: Display entity logos in upper right corner of cards
7. **FR7**: Format and display contact information (address, phone, email)

#### Non-Functional Requirements

1. **NFR1**: Maintain responsive design (two-column desktop, single-column mobile)
2. **NFR2**: Handle missing data gracefully
3. **NFR3**: Support dark mode
4. **NFR4**: Ensure performance with proper data fetching

### Detailed Implementation Plan

#### Phase 1: Backend API Updates (Additional Fields)

**File**: `server/controller/org.controller.js`

**Current Code** (after Part 2):

```javascript
.select("id acronym name logo")
```

**Change To**:

```javascript
.select("id acronym name description email email_ phone phone_ address address_ logo")
```

**File**: `server/controller/club.controller.js`

**Same change**: Add additional fields to `.select()` clause

**File**: `server/controller/sv.controller.js`

**Investigation Needed**: Determine if SV has similar fields or needs different handling

#### Phase 2: Create Reusable Entity Card Component

**File**: `client/src/components/entitycard/entitycard.jsx` (CREATE)

**Props Interface**:

```typescript
interface EntityCardProps {
  entity: {
    id: string;
    acronym?: string;
    name: string;
    description?: string;
    address?: string;
    address_?: {
      street?: string;
      city?: string;
      state?: string;
      zip?: string;
      country?: string;
    };
    phone?: string;
    phone_?: {
      primary?: string;
      work?: string;
      [key: string]: string;
    };
    email?: string;
    email_?: {
      primary?: string;
      work?: string;
      [key: string]: string;
    };
    logo?: string;
  };
  entityType: "organization" | "club" | "sailing-vessel";
  className?: string;
}
```

**Component Features**:

1. Format and display entity header with type and acronym
2. Render logo in upper right corner (SVG support)
3. Format and display all contact fields
4. Handle missing/null values gracefully
5. Support dark mode
6. Responsive design

**Helper Functions** (Optional):
**File**: `client/src/components/entitycard/entitycard.utils.js` (CREATE)

- `formatAddress(address, address_)` - Format address from string or object
- `formatPhones(phone, phone_)` - Format phone list
- `formatEmails(email, email_)` - Format email list
- `formatEntityHeader(entityType, acronym)` - Format header text

#### Phase 3: Update View Components

**File**: `client/src/views/account/organizations.jsx` (MODIFY)

**Layout Changes**:

- Wrap existing Card in flex container for two-column layout
- Add EntityCard components for connected organizations
- Handle loading and empty states

**New Layout Structure**:

```jsx
<Row width="lg">
  <div className="flex flex-col lg:flex-row gap-6">
    {/* Form Container */}
    <div className="flex-1">
      <Card title={t("account.organizations.subtitle")} loading={loading}>
        {/* Existing form content - NO CHANGES */}
      </Card>
    </div>

    {/* Entity Cards Container */}
    <div className="flex-1 lg:flex lg:flex-wrap lg:gap-4 lg:items-start">
      {currentOrgs.map((org) => (
        <div
          key={org.id}
          className="flex-1 lg:flex-[0_1_calc(50%-0.5rem)] xl:flex-[0_1_calc(33.333%-0.67rem)] min-w-0"
        >
          <EntityCard
            entity={org}
            entityType="organization"
            className="h-full"
          />
        </div>
      ))}
    </div>
  </div>
</Row>
```

**File**: `client/src/views/account/clubs.jsx` (MODIFY)

**Same changes** as Organizations View, replacing "organization" with "club"

**File**: `client/src/views/account/svs.jsx` (MODIFY)

**Investigation Needed**: Adapt to SV-specific fields and structure

#### Phase 4: Data Fetching

**Current Approach**: Fetch entities via search endpoint, filter to connected entities

**Option A** (Recommended for initial implementation):

- Use existing list endpoint with empty search
- Filter to only connected entity IDs
- Pros: Simple, uses existing endpoint
- Cons: Fetches all entities, then filters

**Option B** (Future optimization):

- Create new endpoint: `GET /api/org/bulk?ids=id1,id2,id3`
- Pros: Efficient, only fetches needed entities
- Cons: Requires new endpoint

### Card Layout Design

```
┌──────────────────────────────────────────────────────┐
│                                    ┌──────────────┐  │
│  {Entity}: {Acronym}               │              │  │
│  (Large, Bold White/Black)         │    LOGO      │  │
│                                    │  (Square,    │  │
│  Name: {Entity Name}               │   Aspect     │  │
│  (Smaller Gray)                    │   Preserved) │  │
│                                    │              │  │
│  Description: {Description}        │              │  │
│  (Gray, same size as Name)         └──────────────┘  │
│                                                      │
│  Address: {Formatted Address}                        │
│  (Gray, same size as Name)                           │
│                                                      │
│  Phone(s): {Phone List}                              │
│  (Gray, same size as Name)                           │
│                                                      │
│  Email(s): {Email List}                              │
│  (Gray, same size as Name)                           │
└──────────────────────────────────────────────────────┘
```

### Responsive Design

**Large View (lg - Desktop/Client)**:

- Cards displayed in flex container to the right of Manage form
- Display 2-3 cards across (responsive flex-wrap)
- Layout: Two-column (form left, cards right) using flex

**Small View (sm - Mobile/App)**:

- Cards displayed below Manage form (single column)
- One card per row (full width)
- Cards container scrollable
- Layout: Single column stack (form top, cards below)

### Data Formatting

**Address Formatting**:

- If `address_` object exists: Format as `{street}, {city}, {state} {zip}, {country}`
- If only `address` string exists: Display as-is
- If neither exists: Display "N/A" or omit field

**Phone Formatting**:

- If `phone_` object exists: Display all phone numbers as list
- If only `phone` string exists: Display as-is
- If neither exists: Display "N/A" or omit field

**Email Formatting**:

- If `email_` object exists: Display all emails as list
- If only `email` string exists: Display as-is
- If neither exists: Display "N/A" or omit field

### Questions for Clarification

1. **Sailing Vessel Cards**: What fields should SV cards display? (They don't have contact info like org/club)
2. **Logo Rendering**: Are logos always valid SVG strings? What happens if logo is missing/null?
3. **Empty States**: What should be displayed when there are no connected entities?
4. **Data Fetching**: Confirm Option A (fetch all, filter) is acceptable for initial implementation?

---

## Security Considerations

### SVG Content Security

**Current State**: Seed data contains SVG strings (trusted source)

**Risk Assessment**: **Low Risk**

- Seed data is controlled and trusted
- No user input involved
- SVG content is static
- API endpoints require authentication

**Recommendation**:

- No sanitization needed for seed data
- If user input is added later, implement sanitization at that time

### API Security

- Endpoints require `auth.verify('user')` middleware
- Follows same security pattern as org/club endpoints
- Input validation via Joi schema
- No SQL injection risk (MongoDB)

---

## Testing Strategy

### Unit Tests

1. **Model Tests** (Locale):

   - Test `create()` with `flag` field
   - Test `create()` without `flag` field (backward compatibility)
   - Test `update()` with `flag` field
   - Test schema validation

2. **Controller Tests** (Locale):

   - Test list endpoint returns `flag` field
   - Test list endpoint with search (ensures field included)
   - Test `flag` field is always present (required validation)
   - Test search by name, language, country

3. **Controller Tests** (Org/Club):

   - Test list endpoint returns `logo` field
   - Test list endpoint returns additional fields (description, email, phone, address)
   - Test list endpoint with search (ensures fields included)
   - Test empty `logo` values (null/undefined)

4. **Component Tests** (EntityCard):

   - Test component renders with all fields
   - Test component handles missing fields gracefully
   - Test logo rendering (SVG support)
   - Test data formatting functions

5. **Route Tests**:
   - Test authentication required
   - Test route registration
   - Test error handling

### Integration Tests

1. **API Integration**:

   - Test GET /api/locale returns `flag` field
   - Test GET /api/locale with search parameter
   - Test GET /api/org returns `logo` and additional fields
   - Test GET /api/club returns `logo` and additional fields
   - Test authentication still works
   - Test search functionality works
   - Test records without logo (field should be null/undefined)

2. **Database Integration**:

   - Test records created with `flag` persist correctly
   - Test queries return `flag` field via `.lean()`
   - Test `getAll()`, `get()`, `getByLanguage()` include `flag`

3. **Frontend Integration**:
   - Test EntityCard component displays correctly
   - Test cards appear in correct layout (desktop/mobile)
   - Test data fetching and filtering works correctly

### Manual Testing Checklist

- [ ] Seed data loads without errors
- [ ] Model `create()` accepts `flag` parameter (locale)
- [ ] API endpoint `/api/locale` exists and is accessible
- [ ] API endpoint `/api/locale` returns `flag` field
- [ ] API endpoints `/api/org` and `/api/club` return `logo` field
- [ ] API endpoints `/api/org` and `/api/club` return additional fields
- [ ] Search functionality works (name, language, country for locale)
- [ ] Authentication required (401 without token)
- [ ] Records without `logo` return null/undefined (not error)
- [ ] Response sorted by name
- [ ] No performance degradation
- [ ] Entity cards display correctly in Organizations view
- [ ] Entity cards display correctly in Clubs view
- [ ] Cards layout correctly on desktop (two-column)
- [ ] Cards layout correctly on mobile (single column)
- [ ] Logos render correctly in cards
- [ ] Contact information formats correctly

---

## User Stories

### US1: Display Locale Flag in UI

**As a** frontend developer
**I want to** receive the `flag` field when calling `/api/locale`
**So that** I can display country flags in the UI

**Acceptance Criteria**:

- GET /api/locale returns `flag` field
- Flag contains valid SVG markup
- Flag displays correctly in UI (out of scope for this implementation)

### US2: Display Organization Logo

**As a** user viewing organization information
**I want to** see the organization's SVG logo in the API response
**So that** I can display it in the UI

**Acceptance Criteria**:

- Organization list API returns `logo` field
- Logo contains valid SVG markup
- Logo displays correctly in UI (out of scope for this implementation)

### US3: Display Club Logo

**As a** user viewing club information
**I want to** see the club's SVG logo in the API response
**So that** I can display it in the UI

**Acceptance Criteria**:

- Club list API returns `logo` field
- Logo contains valid SVG markup
- Logo displays correctly in UI (out of scope for this implementation)

### US4: Display Entity Cards

**As a** user viewing my connected entities
**I want to** see detailed information about organizations/clubs/vessels in cards
**So that** I can quickly reference their contact information without leaving the page

**Acceptance Criteria**:

- Entity cards display to the right of "Manage {Entity}" forms on desktop
- Cards display below forms on mobile
- Cards show logos, contact information, and other entity details
- Cards handle missing data gracefully

### US5: Backward Compatibility

**As a** developer using the API
**I want** existing API responses to continue working
**So that** I don't need to update my code immediately

**Acceptance Criteria**:

- All existing fields still present
- New fields are optional (doesn't break if missing)
- No breaking changes to response structure

---

## Interface Contracts

### Locale API Interface

#### GET /api/locale

**Request**:

```
GET /api/locale?search=<optional>
Headers: Authorization: Bearer <token>
```

**Response**:

```typescript
{
  data: Array<{
    id: string;
    key: string;
    name: string;
    country: string;
    language: string;
    flag: string; // Required SVG string
  }>;
}
```

### Org/Club API Interface

#### GET /api/org

**Request**:

```
GET /api/org?search=<optional>
Headers: Authorization: Bearer <token>
```

**Response** (after Part 2 and Part 3):

```typescript
{
  data: Array<{
    id: string;
    acronym: string;
    name: string;
    description?: string;
    email?: string;
    email_?: {
      primary?: string;
      work?: string;
      [key: string]: string;
    };
    phone?: string;
    phone_?: {
      primary?: string;
      work?: string;
      [key: string]: string;
    };
    address?: string;
    address_?: {
      street?: string;
      city?: string;
      state?: string;
      zip?: string;
      country?: string;
      [key: string]: any;
    };
    logo?: string; // Optional SVG string
  }>;
}
```

#### GET /api/club

**Request**:

```
GET /api/club?search=<optional>
Headers: Authorization: Bearer <token>
```

**Response** (after Part 2 and Part 3):

```typescript
{
  data: Array<{
    id: string;
    acronym: string;
    name: string;
    description?: string;
    email?: string;
    email_?: {
      primary?: string;
      work?: string;
      [key: string]: string;
    };
    phone?: string;
    phone_?: {
      primary?: string;
      work?: string;
      [key: string]: string;
    };
    address?: string;
    address_?: {
      street?: string;
      city?: string;
      state?: string;
      zip?: string;
      country?: string;
      [key: string]: any;
    };
    logo?: string; // Optional SVG string
  }>;
}
```

---

## Risk Assessment

### Low Risk ✅

1. **Schema Changes** (Locale): Adding required field is low risk (all seed data has flag)
2. **CRUD Updates** (Locale): Additive changes only
3. **API Creation** (Locale): Follows established pattern exactly
4. **Controller Updates** (Org/Club): Simple field additions to `.select()` clauses
5. **Backward Compatibility**: Existing code will continue to work
6. **Seed Data**: Already contains fields, just needs model/API support

### Medium Risk ⚠️

1. **New Files** (Locale): Creating controller and route files (low risk, follows pattern)
2. **Route Registration** (Locale): Auto-loaded by index.js (should work automatically)
3. **API Response Size**: SVG strings and additional fields could increase response size (mitigated by small current SVGs)
4. **Frontend Component**: EntityCard component development (medium complexity)
5. **Layout Changes**: Modifying view layouts for two-column design (medium complexity)
6. **Sailing Vessel Cards**: May need different approach than org/club (investigation needed)

### High Risk ❌

1. **None identified** - This follows established patterns

### Mitigation Strategies

- **New Files**: Follow exact pattern from org/club files
- **Route Registration**: Verify auto-loading works (index.js handles it)
- **Performance**: Monitor response sizes, implement field selection if needed
- **Frontend Component**: Create reusable component with thorough testing
- **Layout Changes**: Test thoroughly on different screen sizes
- **Sailing Vessel Cards**: Investigate SV structure before implementation
- **Testing**: Comprehensive testing before deployment

---

## Confidence Assessment

### Current Confidence Level: **85-98%** (varies by component)

### High Confidence Areas ✅ (98%)

1. **Schema Changes** (Locale): 100% confident

   - Simple field addition
   - Follows same pattern as `logo` in org/club
   - Only one file to edit (mongo version)

2. **CRUD Updates** (Locale): 100% confident

   - Additive changes only
   - `create()` update is straightforward
   - Other functions work automatically

3. **Controller Creation** (Locale): 100% confident

   - Follows org/club pattern exactly
   - Search logic is clear
   - Field selection is explicit

4. **Route Creation** (Locale): 100% confident

   - Follows org/club pattern exactly
   - Auto-loaded by index.js
   - Simple GET endpoint

5. **Controller Updates** (Org/Club): 100% confident

   - Simple field additions to `.select()` clauses
   - No complex logic involved

6. **Seed Data**: 100% confident

   - Already contains fields
   - All records consistent
   - No changes needed

7. **Backward Compatibility**: 100% confident
   - Additive change only
   - No breaking changes
   - Existing fields unchanged

### Medium Confidence Areas ⚠️ (85-90%)

1. **EntityCard Component**: 85% confident

   - Component structure is clear
   - Data formatting logic is straightforward
   - SVG rendering needs verification
   - Missing data handling needs testing

2. **View Integration**: 88% confident

   - Layout changes are clear
   - Flex layout approach is standard
   - Responsive behavior needs testing

3. **Sailing Vessel Cards**: 75% confident

   - SV structure needs investigation
   - May need different component variant
   - Fields may differ from org/club

### Remaining Uncertainties

1. **Search Fields** (Locale): 1% uncertainty

   - **Question**: Should search include `key` field as well?
   - **Answer**: Following org/club pattern, they search by `acronym` and `name`. For locale, searching by `name`, `language`, and `country` makes sense. `key` could be added if needed.
   - **Mitigation**: Can add `key` to search if requested

2. **Response Format**: 1% uncertainty

   - **Question**: Will MongoDB return `null` or omit field when `logo` is `null`?
   - **Answer**: MongoDB/Mongoose `.lean()` will include `null` values
   - **Mitigation**: Test with records that have `null` logo values

3. **Entity Cards Questions**: 10% uncertainty
   - **Question**: Sailing Vessel card content, logo rendering approach, empty states, data fetching strategy
   - **Mitigation**: Answer clarification questions before implementation

### Questions to Reach 95%+ Confidence

**For Entity Cards**:

1. Sailing Vessel card content (what fields to display?)
2. Logo rendering approach (SVG string handling)
3. Empty state handling (hide cards container or show message?)
4. Data fetching strategy confirmation (Option A acceptable?)

---

## Implementation Checklist

### Pre-Implementation

- [x] Review and validate plan
- [x] Answer clarifying questions (for SVG fields)
- [ ] Answer clarification questions (for Entity Cards)
- [x] Verify file structure (mongo promotion)
- [x] Understand org/club patterns
- [ ] Set up test environment (if needed)
- [ ] Backup database (if applicable)

### Implementation Steps

**Part 1: Locale Entity**:

- [ ] Update `locale.mongo.js` schema (add `flag` field)
- [ ] Update `locale.mongo.js` `create()` function (add `flag` parameter)
- [ ] Update JSDoc for `create()` in `locale.mongo.js`
- [ ] Create `locale.controller.js` with `list` function
- [ ] Create `locale.route.js` with GET endpoint
- [ ] Test seed data loading
- [ ] Test model CRUD operations
- [ ] Test API endpoint
- [ ] Verify route auto-loading
- [ ] Test search functionality

**Part 2: Org and Club Entities (Logo)**:

- [ ] Update `org.controller.js` line 50: Add `logo` to `.select()`
- [ ] Update `club.controller.js` line 50: Add `logo` to `.select()`
- [ ] Test API endpoints locally
- [ ] Verify response includes `logo` field
- [ ] Test with records that have `null` logo values
- [ ] Test search functionality still works

**Part 3: Org and Club Entities (Additional Fields)**:

- [ ] Update `org.controller.js`: Add additional fields to `.select()`
- [ ] Update `club.controller.js`: Add additional fields to `.select()`
- [ ] Test API endpoints return all fields
- [ ] Verify response includes description, email, phone, address fields

**Part 4: Entity Cards Frontend**:

- [ ] Create `entitycard` directory in components
- [ ] Create `entitycard.jsx` component file
- [ ] Create helper utility functions for formatting (optional)
- [ ] Implement logo rendering (SVG support)
- [ ] Implement all field displays with proper formatting
- [ ] Add dark mode support
- [ ] Add responsive styling
- [ ] Test component with sample data
- [ ] Update `organizations.jsx` with entity cards layout
- [ ] Update `clubs.jsx` with entity cards layout
- [ ] Investigate and update `svs.jsx` (if applicable)
- [ ] Test with real data
- [ ] Verify responsive behavior

### Post-Implementation

- [ ] Run seed process
- [ ] Verify API endpoints return all fields
- [ ] Test backward compatibility
- [ ] Performance testing (response times)
- [ ] Test entity cards display correctly
- [ ] Test responsive behavior on different screen sizes
- [ ] ESLint all modified/new files
- [ ] Update documentation (if needed)

---

## File Change Summary

| File                                                   | Changes                                               | Lines Affected        | Risk     |
| ------------------------------------------------------ | ----------------------------------------------------- | --------------------- | -------- |
| `server/model/mongo/locale.mongo.js`                   | Add `flag` to schema, update `create()`               | ~8 lines              | Low      |
| `server/controller/locale.controller.js`               | Create new controller file                            | ~60 lines (new file)  | Low      |
| `server/api/locale.route.js`                           | Create new route file                                 | ~10 lines (new file)  | Low      |
| `server/controller/org.controller.js`                  | Add fields to `.select()` (logo + entity card fields) | 1 line                | Very Low |
| `server/controller/club.controller.js`                 | Add fields to `.select()` (logo + entity card fields) | 1 line                | Very Low |
| `client/src/components/entitycard/entitycard.jsx`      | Create new component file                             | ~200 lines (new file) | Medium   |
| `client/src/components/entitycard/entitycard.utils.js` | Create helper utilities (optional)                    | ~100 lines (new file) | Low      |
| `client/src/views/account/organizations.jsx`           | Add entity cards layout                               | ~30 lines modified    | Medium   |
| `client/src/views/account/clubs.jsx`                   | Add entity cards layout                               | ~30 lines modified    | Medium   |
| `client/src/views/account/svs.jsx`                     | Add entity cards layout (investigation needed)        | TBD                   | Medium   |

**Total Estimated Changes**:

- **Backend Model**: ~8 lines modified
- **Backend Controller**: ~60 lines (new file) + 2 lines modified
- **Backend Route**: ~10 lines (new file)
- **Frontend Component**: ~200-300 lines (new files)
- **Frontend Views**: ~60-90 lines modified

---

## Conclusion

This implementation plan outlines adding SVG field support and entity card displays across multiple entities:

1. **Locale Entity**: Add `flag` field to schema and create new API layer (model → controller → route)
2. **Org and Club Entities**:
   - Enable existing `logo` field by adding to `.select()` clauses
   - Add additional fields (description, email, phone, address) to support entity cards
3. **Entity Cards Frontend**: Create reusable EntityCard component and integrate into "Manage {Entity}" dialogs

**Estimated Complexity**: Medium
**Estimated Time**: 8-12 hours

- Locale API: 2-3 hours
- Org/Club logo field: 15-30 minutes
- Org/Club additional fields: 15-30 minutes
- EntityCard component: 3-4 hours
- View integration: 2-3 hours
- Testing and refinement: 1-2 hours
  **Risk Level**: Low to Medium
  **Confidence Level**: 85-98% (varies by component)

**Key Points**:

- ✅ Seed data already has SVG fields
- ✅ Model updates are straightforward
- ✅ API follows established org/club pattern
- ✅ Only edit `server/model/mongo/` files (promoted by setup.js)
- ✅ Fully backward compatible
- ✅ No breaking changes
- ⚠️ Entity cards require frontend component development
- ⚠️ Sailing Vessel cards may need different approach (investigation needed)

**Next Steps**:

1. Review this validated plan
2. Answer clarification questions (especially for Sailing Vessel cards)
3. Approve implementation
4. Proceed with code changes (can be done in phases)

---

## V: REVIEW - Review and validate the implementation plan

## Final Confidence: **85-98%** (varies by component)

## Plan Status

### ✅ Completed Planning

1. **SVG Fields (Locale)**: 98% confidence - Ready for implementation
2. **SVG Fields (Org/Club)**: 98% confidence - Ready for implementation
3. **Entity Cards Backend**: 95% confidence - Ready for implementation

### ⚠️ Requires Clarification

1. **Entity Cards Frontend**: 85% confidence
   - Sailing Vessel card content needs investigation
   - Logo rendering approach needs confirmation
   - Empty state handling needs decision
   - Data fetching strategy needs confirmation

### Implementation Status

**Status**: ✅ **READY FOR IMPLEMENTATION** (Parts 1-2), ⚠️ **NEEDS CLARIFICATION** (Part 3)

**Confidence**:

- Parts 1-2: **98%**
- Part 3: **85%** (can improve to 95%+ with clarification)

**Recommendation**: Can proceed with Parts 1-2 immediately. Part 3 (Entity Cards) can proceed after answering clarification questions, or can be implemented incrementally.

---

## B: BRANCH - Create Git branches for required repos

<!-- Branch creation would be documented here when ready -->

---

## I: IMPLEMENT - Execute the plan

<!-- Implementation progress would be tracked here -->

---

## L: LINT - Check and fix linting issues

<!-- Linting results would be documented here -->

---

## T: TEST - Run tests

<!-- Test results would be documented here -->

---

## M: DOCUMENT - Document the solution

<!-- Final documentation would be added here -->

---

## R: PULL REQUEST - Create PRs for all repos

<!-- PR creation would be documented here -->

---

## Notes

<!-- Additional notes, decisions, or observations -->
