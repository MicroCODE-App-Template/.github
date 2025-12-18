# AIN - FEATURE - REORGANIZE_ENTITY_MANAGEMENT_DIALOGS

## Metadata

- **Type**: FEATURE
- **Issue #**: [if applicable]
- **Created**: [DATE]
- **Status**: READY FOR IMPLEMENTATION

---

## C: CONCEPT/CHANGE/CORRECTION - Discuss ideas without generating code

<!-- Initial concept discussion would go here - not present in phase-specific files -->

---

## D: DESIGN - Design detailed solution

<!-- Design details would go here - not present in phase-specific files -->

---

## P: PLAN - Create implementation plan

# Implementation Plan: Reorganize Entity Management Dialogs & Replace SV with Boat Management

## Executive Summary

This plan combines two related initiatives:

### Part 1: UI Reorganization

Reorganize the "Manage Orgs, Clubs, and Boats" dialogs with **conditional UI behavior** based on whether the user has entities:

1. **Empty State (No Entities)**: NO CHANGES - UI remains exactly as currently implemented
2. **Non-Empty State (Has Entities)**: Move "[ + Add {Entity} ]" button below the list and add "[ Request New {Entity} ]" button next to it

### Part 2: Boat Management Migration

Migrate from **Sailing Vessel (SV)**-based management to **direct Boat management**, aligning Boat management with the existing Organizations and Clubs patterns:

- **Remove**: SV creation/management UI from user-facing interface
- **Remove**: `GET /api/user/svs` endpoint (SV endpoints remain for internal use)
- **Add**: Direct Boat management UI matching Organizations/Clubs pattern
- **Add**: `boat_ids` field to User schema (alongside existing `sv_ids`)
- **Preserve**: All SV endpoints (`/api/sv/*`) for future internal use
- **Preserve**: `sv_ids` and `sv_keys` in User model schema for internal use
- **Preserve**: SV translation files
- **No Migration**: Clean database import will handle data setup
- **Out of Scope**: Boat editing and image uploads (handled separately, not part of this implementation)

### ⚠️ CRITICAL REQUIREMENTS

**Empty State (No Entities):**

- **ZERO CHANGES** - UI must remain exactly as shown in the reference image
- Help Section → Add Button (above) → Alert Box with Request button → Save Button
- The Alert box and its internal Request button must remain unchanged

**Non-Empty State (Has Entities):**

- Alert box does NOT appear
- Entity list is displayed
- Add button is moved BELOW the list
- New Request button appears NEXT TO Add button (side-by-side)
- Both buttons positioned between list and Save button

## Current State Analysis

### Components Affected

**For UI Reorganization:**
Three React components follow identical patterns:

1. **Organizations** (`client/src/views/account/organizations.jsx`)
2. **Clubs** (`client/src/views/account/clubs.jsx`)
3. **Boats** (`client/src/views/account/boats.jsx`) - Will replace `svs.jsx`

**For Boat Management Migration:**

- **SVs** (`client/src/views/account/svs.jsx`) - To be replaced by `boats.jsx`

### Current UI Structure

**When User Has NO Entities (Empty State) - CURRENT:**

```
┌─────────────────────────────────────────┐
│  Manage {Entity}                        │
├─────────────────────────────────────────┤
│  [About {Entity} Help Section]          │
│                                         │
│  [+ Add {Entity}] ← Currently HERE      │
│                                         │
│  [No {Entity}s Alert Box]               │
│  └─ [Request to Add {Entity}] Button    │ ← Inside Alert
│                                         │
│  [Save Button]                          │
└─────────────────────────────────────────┘
```

**This structure must remain UNCHANGED**

**When User Has Entities (Non-Empty State) - CURRENT:**

```
┌─────────────────────────────────────────┐
│  Manage {Entity}                        │
├─────────────────────────────────────────┤
│  [About {Entity} Help Section]          │
│                                         │
│  [+ Add {Entity}] ← Currently HERE      │
│                                         │
│  [List of User's {Entities}]            │
│  - Entity 1                    [X]      │
│  - Entity 2                    [X]      │
│  - Entity 3                    [X]      │
│                                         │
│  [Save Button]                          │
└─────────────────────────────────────────┘
```

**This structure WILL CHANGE**

## Proposed Changes

### New UI Structure

**When User Has NO Entities (Empty State) - NO CHANGES:**

```
┌─────────────────────────────────────────┐
│  Manage {Entity}                        │
├─────────────────────────────────────────┤
│  [About {Entity} Help Section]          │
│                                         │
│  [+ Add {Entity}] ← STAYS HERE          │
│                                         │
│  [No {Entity}s Alert Box]               │ ← UNCHANGED
│  └─ [Request to Add {Entity}] Button    │ ← UNCHANGED
│                                         │
│  [Save Button]                          │
└─────────────────────────────────────────┘
```

**This remains EXACTLY as-is**

**When User Has Entities (Non-Empty State) - NEW STRUCTURE:**

```
┌────────────────────────────────────────────┐
│  Manage {Entity}                           │
├────────────────────────────────────────────┤
│  [About {Entity} Help Section]             │
│                                            │
│  [List of User's {Entities}]               │
│  - Entity 1                    [X]         │
│  - Entity 2                    [X]         │
│  - Entity 3                    [X]         │
│                                            │
│  [+ Add {Entity}] [Request New {Entity}]   │← NEW location, side-by-side
│                                            │
│  [Save Button]                             │
└────────────────────────────────────────────┘
```

**Alert box does NOT appear when list is non-empty**

### Key Changes

1. **Conditional Rendering**:

   - Empty state: Render Add button ABOVE Alert box (current behavior)
   - Non-empty state: Render Add button BELOW entity list (new behavior)

2. **New Request Button**:

   - Only appears when list is NON-EMPTY
   - Positioned next to Add button (side-by-side)
   - Label: "Request NEW {Entity}" (shorter, clear)

3. **Alert Box Behavior**:

   - Only appears when list is EMPTY
   - Completely hidden when list is NON-EMPTY

4. **Layout**:

   - Non-empty state: Both buttons displayed side-by-side in a flex container
   - Empty state: No changes to layout

5. **Consistency**: Apply same pattern to all three components

## Affected Files

### Primary Files (Direct Changes)

1. **`client/src/views/account/organizations.jsx`**

   - Modify conditional rendering logic for Add button
   - Add conditional Request button (only when `currentOrgs.length > 0`)
   - Ensure Alert box only renders when `currentOrgs.length === 0`
   - Move Add button section conditionally based on list state

2. **`client/src/views/account/clubs.jsx`**

   - Same changes as Organizations component
   - Apply consistent pattern

3. **`client/src/views/account/svs.jsx`**
   - Same changes as Organizations component
   - Apply consistent pattern

### Translation Files (May Need Updates)

4. **`client/src/locales/en/account/en_organizations.json`**

   - Update: `"request": { "button": "Request NEW Organization" }`

5. **`client/src/locales/en/account/en_clubs.json`**

   - Update: `"request": { "button": "Request NEW Club" }`

6. **`client/src/locales/en/account/en_svs.json`**
   - Update: `"request": { "button": "Request NEW Boat" }`

### No Backend Changes Required

- Entity request API (`/api/entity_request`) already exists and works
- No database schema changes needed
- No API endpoint modifications required

## Implementation Steps

### Phase 1: Organizations Component

1. **Read current file** (`organizations.jsx`)
2. **Identify current structure**:

   - Add button section (lines ~390-501)
   - Entity list/Alert conditional (lines ~503-539)
   - Save button (lines ~541-585)

3. **Refactor conditional rendering**:

   - Wrap Add button in conditional: only show ABOVE when `currentOrgs.length === 0`
   - Keep Alert box conditional: only show when `currentOrgs.length === 0`
   - Add new button section: only show when `currentOrgs.length > 0`

4. **Create new button section** (non-empty state):

   - Position after entity list (before Save button)
   - Create flex container: `flex flex-col sm:flex-row gap-2`
   - Add Add button (moved from above)
   - Add Request button next to it

5. **Verify structure**:

   - Empty: Help → Add → Alert → Save
   - Non-empty: Help → List → [Add] [Request] → Save

6. **Test functionality** in both states

### Phase 2: Clubs Component

1. **Apply same changes** as Organizations
2. **Ensure consistency** in styling and behavior
3. **Test functionality**

### Phase 3: Boat Management Migration (Replace SV Component)

1. **Create new `boats.jsx` component** (copy from `clubs.jsx`)
2. **Implement boat-specific functionality**:
   - Search by name, sail_no, sail_cc
   - Display sail info in selection list
   - EntityCard shows boat-specific fields (sail, brand, model, builder)
   - No acronym field (boats don't have acronyms)
3. **Apply UI reorganization changes** (same as Organizations/Clubs)
4. **Update routes** to use `/account/boats` instead of `/account/svs`
5. **Update navigation** menu
6. **Create translation file** (`en_boats.json`)
7. **Test functionality**

### Phase 4: Translation Updates

1. **Review translation files**
2. **Update translation keys**:
   - `account.organizations.request.button`: "Request NEW Organization"
   - `account.clubs.request.button`: "Request NEW Club"
   - `account.svs.request.button`: "Request NEW Boat"
3. **Verify all text is translatable**

### Phase 5: Testing & Validation

1. **Test empty state** - verify NO changes
2. **Test non-empty state** - verify new layout
3. **Test state transitions** - empty ↔ non-empty
4. **Test mobile responsiveness**
5. **Test accessibility** (keyboard navigation, screen readers)

## JSX Structure to Implement

```jsx
<Card>
  {/* Help Section */}
  <div className='mb-6'>...</div>

  {/* Conditional Rendering Based on List State */}
  {currentOrgs.length === 0 ? (
    // EMPTY STATE - NO CHANGES
    <>
      {/* Add Button - ABOVE Alert (current position) */}
      <div className='mb-4'>
        <Popover>
          <PopoverTrigger asChild>
            <Button
              text={t('account.organizations.add.button')}
              icon='plus'
              variant='outline'
            />
          </PopoverTrigger>
          <PopoverContent>...</PopoverContent>
        </Popover>
      </div>

      {/* Alert Box - UNCHANGED */}
      <div className='px-4'>
        <Alert
          variant='info'
          title={t('account.organizations.empty_state.title')}
          description={t('account.organizations.empty_state.description')}
          button={{
            text: t('account.organizations.empty_state.button'),
            action: handleRequestNew,
            size: 'full',
            className: 'w-full'
          }}
        />
      </div>
    </>
  ) : (
    // NON-EMPTY STATE - NEW STRUCTURE
    <>
      {/* Entity List */}
      <div className='space-y-2'>
        {currentOrgs.map(org => (
          <div key={org.id} className='flex items-center justify-between p-3 border...'>
            <div>
              <span className='font-medium'>
                {org.acronym ? `${org.acronym} - ` : ''}{org.name}
              </span>
            </div>
            <Button
              variant='ghost'
              icon='x'
              size='sm'
              action={() => handleRemove(org.id)}
            />
          </div>
        ))}
      </div>

      {/* Action Buttons - NEW LOCATION */}
      <div className='mt-4 flex flex-col sm:flex-row gap-2 transition-all duration-300'>
        {/* Add Button - MOVED HERE */}
        <Popover>
          <PopoverTrigger asChild>
            <Button
              text={t('account.organizations.add.button')}
              icon='plus'
              variant='outline'
              className='flex-1'
            />
          </PopoverTrigger>
          <PopoverContent>...</PopoverContent>
        </Popover>

        {/* Request Button - NEW */}
        <Button
          text={t('account.organizations.request.button')}
          icon='message-square'
          variant='outline'
          onClick={handleRequestNew}
          className='flex-1'
        />
      </div>
    </>
  )}

  {/* Save Button */}
  <div className='mt-6 px-4'>
    <Button text={t('account.organizations.save')} ... />
  </div>
</Card>
```

---

## Boat Management Migration Details

### Current State Analysis

#### User Entity Relationships

```
User
├── org_ids: Array<string>        // Organization IDs user belongs to
├── club_ids: Array<string>       // Club IDs user belongs to
├── sv_ids: Array<string>         // Sailing Vessel IDs user owns/is associated with
└── boat_ids: ❌ NOT PRESENT      // Boat IDs - does not exist
```

#### Target Architecture

```
User
├── org_ids: Array<string>        // Organization IDs user belongs to
├── club_ids: Array<string>       // Club IDs user belongs to
├── boat_ids: Array<string>       // Boat IDs user owns/is associated with ✨ NEW
└── sv_ids: ✅ KEEP                // Preserved for internal use (not user-facing)
```

### Pattern Consistency

The Boat management will follow the **exact same pattern** as Organizations and Clubs:

#### Organizations Pattern

```
1. GET /api/org?search=... → List organizations
2. User selects orgs → Updates user.org_ids via PATCH /api/user
3. EntityCard displays org with logo, contact info
```

#### Clubs Pattern

```
1. GET /api/club?search=... → List clubs
2. User selects clubs → Updates user.club_ids via PATCH /api/user
3. EntityCard displays club with logo, contact info
```

#### Boats Pattern (Target)

```
1. GET /api/boat?search=... → List boats ✅ EXISTS
2. User selects boats → Updates user.boat_ids via PATCH /api/user ✨ NEW
3. EntityCard displays boat with sail info, brand, model, builder ✨ NEW
```

### API Contract Changes

#### 1. User Update Endpoint (MODIFIED)

**Current:**

```typescript
PATCH /api/user
{
    org_ids?: string[];
    club_ids?: string[];
    sv_ids?: string[];  // ❌ TO BE REMOVED from user-facing API
}
```

**Target:**

```typescript
PATCH /api/user
{
    org_ids?: string[];
    club_ids?: string[];
    boat_ids?: string[];  // ✨ NEW
    sv_ids?: string[];    // ✅ KEEP (for internal use, not user-facing)
}
```

#### 2. User Get Endpoint (MODIFIED)

**Target:**

```typescript
GET /api/user
Response: {
    data: {
        id: string;
        name: string;
        org_ids: string[];
        club_ids: string[];
        boat_ids: string[];  // ✨ NEW
        sv_ids: string[];     // ✅ KEEP (for internal use, returned but not used in UI)
    }
}
```

#### 3. Boat List Endpoint (ENHANCED - Already Exists)

**Enhancement:** Populate brand, model (design), builder for EntityCard display

```typescript
GET /api/boat?search=...
Response: {
    data: Array<{
        id: string;
        name: string;
        sail_no: string;
        sail_cc: string;
        hin: string;
        boat_design_id?: string;      // ✨ ENHANCE: Include for EntityCard
        boat_builder_id?: string;     // ✨ ENHANCE: Include for EntityCard
        // Populated fields for EntityCard:
        brand?: { name: string };     // ✨ ENHANCE: Populate brand
        model?: { name: string };     // ✨ ENHANCE: Populate model (design)
        builder?: { name: string };   // ✨ ENHANCE: Populate builder
    }>
}
```

#### 4. SV Endpoints (PRESERVED FOR INTERNAL USE)

**Endpoints to Keep:**

- `POST /api/sv` ✅ KEEP
- `GET /api/sv` ✅ KEEP
- `GET /api/sv/:id` ✅ KEEP
- `GET /api/sv/:id/owners` ✅ KEEP
- `GET /api/sv/by-boat` ✅ KEEP
- `PATCH /api/sv/:id` ✅ KEEP
- `DELETE /api/sv/:id` ✅ KEEP

**Endpoints to Remove:**

- `GET /api/user/svs` ❌ REMOVE (only user-facing SV endpoint)

### Frontend Implementation - Boat Component

#### New Component: `client/src/views/account/boats.jsx`

**Structure:** Copy from `clubs.jsx` and adapt for boats

**Key Differences from Clubs:**

- Search by `name`, `sail_no`, `sail_cc` (already in boat.controller.js)
- Display sail info in selection list
- EntityCard shows boat-specific fields (sail, brand, model, builder)
- No acronym field (boats don't have acronyms)

**Key Similarities:**

- Same Popover-based selection UI
- Same add/remove pattern
- Same "Request New" entity request flow
- Same EntityCard display pattern
- Same UI reorganization (conditional button placement)

**Component Structure:**

```javascript
export function Boats({ t: propT }) {
    // State (same as clubs.jsx)
    const [selectedBoatIds, setSelectedBoatIds] = useState([]);
    const [selectedBoats, setSelectedBoats] = useState([]);
    const [boats, setBoats] = useState([]);
    const [searchTerm, setSearchTerm] = useState('');
    const [loading, setLoading] = useState(false);
    const [connectedBoats, setConnectedBoats] = useState([]);

    // User data
    const user = useAPI('/api/user', 'get', refreshTrigger);
    const currentBoatIds = user?.data?.boat_ids || [];

    // Fetch boats list (GET /api/boat)
    useEffect(() => {
        // Fetch boats with search
    }, [searchTerm]);

    // Fetch connected boats for EntityCard display
    useEffect(() => {
        // Fetch boats where id in currentBoatIds
    }, [currentBoatIds]);

    // Handle add selected boats
    const handleAddSelected = async () => {
        // PATCH /api/user with boat_ids
    };

    // Handle remove boat
    const handleRemove = async (boatId) => {
        // PATCH /api/user with updated boat_ids
    };

    // Handle request new boat
    const handleRequestNew = () => {
        // POST /api/entity_request with entity_type: 'boat'
    };

    // Render (same structure as clubs.jsx with UI reorganization)
    return (
        // Popover with search
        // Conditional rendering: empty state vs non-empty state
        // Selected boats list
        // EntityCard display
    );
}
```

### Backend Implementation

#### 1. User Schema Update

**File:** `server/model/mongo/user.mongo.js`

**Changes:**

```javascript
// Add boat_ids and boat_keys fields:
boat_keys: {
    type: Array,
    required: false
},
boat_ids: {
    type: Array,
    required: false
},
// Keep sv_ids for internal use:
sv_ids: {
    type: Array,
    required: false
    // ✅ KEEP - for internal use (not user-facing)
}
```

#### 2. User Controller Update

**File:** `server/controller/user.controller.js`

**Changes:**

**A. Update Validation Schema:**

```javascript
// In exports.update validation:
boat_keys: joi.array().items(joi.string()).optional(),
boat_ids: joi.array().items(joi.string()).optional(),
sv_ids: joi.array().items(joi.string()).optional(), // ✅ KEEP - for internal use

// Add deduplication logic for boat_ids:
if (data.boat_ids) {
    data.boat_ids = [...new Set(data.boat_ids)]; // Remove duplicates
}
```

**B. Remove User SV Endpoint:**

```javascript
// Remove only the user-facing endpoint:
exports.svs = async function (req, res) {
  // ❌ DELETE THIS FUNCTION (GET /api/user/svs)
};

// Note: All other SV endpoints in sv.controller.js remain intact
```

#### 3. User Route Update

**File:** `server/api/user.route.js`

**Changes:**

```javascript
// Remove:
api.get(
  "/api/user/svs",
  auth.verify("user", "user.read"),
  use(userController.svs)
);
```

#### 4. Boat Controller Enhancement

**File:** `server/controller/boat.controller.js`

**Enhancement:** Populate brand, model (design), builder for EntityCard display

```javascript
exports.list = async function (req, res) {
  // ... existing search logic ...

  // Fetch boats
  const boats = await boat.schema
    .find(query)
    .select("id name sail_no sail_cc hin boat_design_id boat_builder_id logo")
    .lean()
    .sort({ sail_no: 1, name: 1 });

  // ✨ NEW: Populate brand, model, builder for EntityCard
  const boatsWithDetails = await Promise.all(
    boats.map(async (boatData) => {
      // Populate boat design (model) and brand
      if (boatData.boat_design_id) {
        const designData = await boatDesign.schema
          .findOne({ id: boatData.boat_design_id })
          .select("id name boat_brand_id")
          .lean();

        if (designData) {
          boatData.model = { name: designData.name };

          // Populate brand
          if (designData.boat_brand_id) {
            const brandData = await boatBrand.schema
              .findOne({ id: designData.boat_brand_id })
              .select("id name")
              .lean();
            if (brandData) {
              boatData.brand = { name: brandData.name };
            }
          }
        }
      }

      // Populate builder
      if (boatData.boat_builder_id) {
        const builderData = await boatBuilder.schema
          .findOne({ id: boatData.boat_builder_id })
          .select("id name")
          .lean();
        if (builderData) {
          boatData.builder = { name: builderData.name };
        }
      }

      return boatData;
    })
  );

  res.status(200).send({
    data: boatsWithDetails,
  });
};
```

### EntityCard Updates

**File:** `client/src/components/entitycard/entitycard.jsx`

**Changes:**

- Add `'boat'` as valid `entityType`
- Keep `'sailing-vessel'` entityType (for internal use, but not used in new boats.jsx UI)
- Add boat-specific display logic (sail info, brand, model, builder)
- Remove SV-specific logic from user-facing UI (owners display)

**Updated EntityType Handling:**

```javascript
const isBoat = entityType === "boat";
const isSailingVessel = entityType === "sailing-vessel"; // ⚠️ DEPRECATED (internal use only)

// Boat display fields:
if (isBoat) {
  // Show: sail_no, sail_cc, brand, model, builder
  // Don't show: address, phone, email (boats don't have these)
  // Don't show: owners (boats don't have owners array)
}
```

### Routes & Navigation Updates

**File:** `client/src/routes/account.js`

**Changes:**

```javascript
// Remove:
import { SVs } from 'views/account/svs';
{
    path: '/account/svs',
    view: SVs,
    ...
}

// Add:
import { Boats } from 'views/account/boats';
{
    path: '/account/boats',
    view: Boats,
    layout: 'account',
    permission: 'user',
    title: 'account.index.title'
}
```

**File:** `client/src/components/layout/account/account.jsx`

**Changes:**

```javascript
// Remove:
{
    label: t('account.nav.svs'),
    link: '/account/svs',
    icon: 'sailboat',
    permission: 'user'
}

// Add:
{
    label: t('account.nav.boats'),
    link: '/account/boats',
    icon: 'sailboat',
    permission: 'user'
}
```

### Translation Files

**New File:** `client/src/locales/en/account/en_boats.json`

**Structure:** Copy from `en_clubs.json` and adapt:

```json
{
  "title": "Boats",
  "subtitle": "Manage Boats",
  "description": "Manage your boats",
  "button": "Manage Boats",
  "help": {
    "title": "About Boats",
    "description": "Boats represent vessels you own or are associated with. Each boat has sail information, brand, model, and builder details."
  },
  "search": {
    "placeholder": "Search by name, sail number, or country code..."
  },
  "add": {
    "button": "Add Boat",
    "selected": "Add {{count}} Selected",
    "success": "Boat added successfully",
    "error": "Failed to add boat"
  },
  "remove": {
    "success": "Boat removed successfully",
    "error": "Failed to remove boat"
  },
  "save": "Save",
  "save_success": "Boats saved successfully",
  "save_error": "Failed to save boats",
  "empty": "No boats found",
  "request": {
    "title": "Request New Boat",
    "description": "Request a new boat to be added to the system",
    "button": "Request NEW Boat",
    "success": "Boat request submitted successfully",
    "form": {
      "name": {
        "label": "Boat Name",
        "placeholder": "Enter boat name"
      },
      "sail_cc": {
        "label": "Sail Country Code",
        "placeholder": "e.g., US"
      },
      "sail_no": {
        "label": "Sail Number",
        "placeholder": "e.g., 12345"
      },
      "brand": {
        "label": "Brand",
        "placeholder": "Enter brand name"
      },
      "model": {
        "label": "Model",
        "placeholder": "Enter model name"
      },
      "builder": {
        "label": "Builder",
        "placeholder": "Enter builder name"
      },
      "button": "Submit Request"
    }
  },
  "empty_state": {
    "title": "No Boats",
    "description": "You haven't added any boats yet. Search for existing boats or request a new one.",
    "button": "Request New Boat"
  },
  "boat": {
    "sail_no": "Sail No",
    "hin": "HIN"
  }
}
```

**Keep:** `client/src/locales/en/account/en_svs.json` - ✅ KEEP (preserved for internal use)

### Data Migration Strategy

**NO MIGRATION REQUIRED** - Clean database import will handle data setup.

- User will alter data import process to create clean database matching new plan
- No migration scripts needed
- `boat_ids` will be populated directly during data import
- `sv_ids` will remain in User schema for internal use (not user-facing)

### Affected Files Summary

#### Frontend Files

**To Create:**

- ✨ `client/src/views/account/boats.jsx` - New boat management component
- ✨ `client/src/locales/en/account/en_boats.json` - Boat translations

**To Modify:**

- 📝 `client/src/routes/account.js` - Change `/account/svs` to `/account/boats`
- 📝 `client/src/components/layout/account/account.jsx` - Update navigation
- 📝 `client/src/views/account/index.jsx` - Update account index page
- 📝 `client/src/components/entitycard/entitycard.jsx` - Add boat support
- 📝 `client/src/components/entitycard/entitycard.utils.js` - Update entity types

**To Remove/Deprecate:**

- ❌ `client/src/views/account/svs.jsx` - Remove SV management component (replace with boats.jsx)
- ✅ `client/src/locales/en/account/en_svs.json` - KEEP (preserved for internal use)

#### Backend Files

**To Modify:**

- 📝 `server/model/mongo/user.mongo.js` - Add `boat_ids` and `boat_keys` fields
- 📝 `server/controller/user.controller.js` - Add `boat_ids` validation, remove `svs` endpoint
- 📝 `server/api/user.route.js` - Remove `/api/user/svs` route
- 📝 `server/controller/boat.controller.js` - Enhance list to populate brand/model/builder

**To Preserve:**

- ✅ `server/api/sv.route.js` - KEEP (all routes preserved for internal use)
- ✅ `server/controller/sv.controller.js` - KEEP (preserved for internal use)
- ✅ `server/model/mongo/sv.mongo.js` - KEEP (preserved for internal use)

### Implementation Phases

#### Phase 1: Database & Backend Foundation

**Duration:** 2-3 days

1. ✅ Add `boat_ids` and `boat_keys` fields to user schema
2. ✅ Update user controller validation to accept `boat_ids`
3. ✅ Enhance boat controller to populate brand/model/builder
4. ✅ Update user controller to return `boat_ids` in GET response
5. ✅ Remove `GET /api/user/svs` route only

#### Phase 2: Frontend Boat Management UI

**Duration:** 3-4 days

1. ✅ Create `boats.jsx` component (copy from `clubs.jsx`)
2. ✅ Adapt component for boat-specific fields
3. ✅ Apply UI reorganization changes (conditional button placement)
4. ✅ Create `en_boats.json` translation file
5. ✅ Update routes to use `/account/boats`
6. ✅ Update navigation menu
7. ✅ Update account index page
8. ✅ Test boat add/remove functionality

#### Phase 3: EntityCard Updates

**Duration:** 1-2 days

1. ✅ Update EntityCard to support `'boat'` entityType
2. ✅ Add boat-specific display fields (sail, brand, model, builder)
3. ✅ Keep `'sailing-vessel'` entityType (for internal use)
4. ✅ Update EntityCard utils
5. ✅ Test EntityCard display with boats

#### Phase 4: SV UI Removal & Cleanup

**Duration:** 1-2 days

1. ✅ Remove `GET /api/user/svs` endpoint (only user-facing SV endpoint)
2. ✅ Remove SV UI components (`svs.jsx`)
3. ✅ Keep SV translation files (preserved for internal use)
4. ✅ Keep SV routes/controller/model (preserved for internal use)

#### Phase 5: Testing & Validation

**Duration:** 2-3 days

1. ✅ Test boat management flow end-to-end
2. ✅ Verify EntityCard displays correctly
3. ✅ Test entity request flow for boats
4. ✅ Verify SV UI removed but SV endpoints still work
5. ✅ Test UI reorganization (empty vs non-empty states)
6. ✅ Performance testing

### Risk Assessment

#### High Risk

1. **Data Import Issues**

   - **Risk:** Clean database import doesn't populate `boat_ids` correctly
   - **Mitigation:** Validate import process, test with sample data, verify data structure

2. **UI Breaking Changes**

   - **Risk:** Users lose access to SV management UI
   - **Mitigation:** Ensure boat management UI is fully functional before removing SV UI

3. **Performance Impact**
   - **Risk:** Populating brand/model/builder for all boats slows down API
   - **Mitigation:** Test performance, consider caching, optimize queries

#### Medium Risk

4. **UI Inconsistencies**

   - **Risk:** Boat UI doesn't match Clubs/Organizations exactly
   - **Mitigation:** Copy exact pattern from clubs.jsx, code review

5. **Translation Missing**
   - **Risk:** Missing translation keys cause UI errors
   - **Mitigation:** Copy from clubs translations, test all language paths

### Success Criteria

#### Functional Requirements

- ✅ Users can view boats they're associated with
- ✅ Users can add boats to their profile
- ✅ Users can remove boats from their profile
- ✅ Users can request new boats
- ✅ Boat information displays correctly in EntityCard
- ✅ Boat management UI matches Clubs/Organizations pattern exactly
- ✅ UI reorganization works correctly (empty vs non-empty states)
- ✅ No SV creation/management UI remains (SV endpoints preserved for internal use)
- ✅ Clean database import populates `boat_ids` correctly

#### Non-Functional Requirements

- ✅ Performance: Boat list loads in < 2 seconds
- ✅ UI consistency: Boat UI matches Clubs/Organizations exactly
- ✅ Data integrity: Clean database import handles data setup
- ✅ SV endpoints preserved: All `/api/sv/*` endpoints remain for internal use

---

## V: REVIEW - Review and validate the implementation plan

## Confidence Rating: **98%** (UI Reorganization: 97%, Boat Management: 99%)

## All Questions Answered - Plan Finalized

### ✅ Final Confirmed Decisions - UI Reorganization

1. **Animation Approach**: Use CSS transitions (`transition-all duration-300`) on container divs

   - Component already wrapped in `<Animate>` at top level
   - Safest approach, no additional wrappers needed

2. **Add Button Placement**: Conditionally positioned

   - Empty state: Above Alert (preserves current UI)
   - Non-empty state: Below list, next to Request button
   - Single Add button that appears in different positions based on state

3. **Translation Strategy**: Reuse existing `request.button` key

   - Update text: "Request New {Entity}" → "Request NEW {Entity}" (capitalize NEW)
   - Files: `en_organizations.json`, `en_clubs.json`, `en_boats.json` (new)
   - No new keys needed

4. **Button Implementation**:

   - Icon: `icon='message-square'` (kebab-case, matches existing pattern)
   - Styling: `variant='outline'`, `className='flex-1'`
   - Layout: `flex flex-col sm:flex-row gap-2`
   - Order: Add left, Request right
   - Mobile breakpoint: `sm` (640px)

5. **Empty State Preservation**:

   - NO CHANGES to empty state UI
   - Add button stays above Alert
   - Alert box and Request button inside remain unchanged

6. **Non-Empty State Changes**:
   - Entity list displayed
   - Add button moved below list
   - Request button appears next to Add button
   - Alert box does NOT appear

### ✅ Final Confirmed Decisions - Boat Management Migration

1. **Pattern Consistency**: Boat management follows exact same pattern as Organizations/Clubs

   - Same Popover-based selection UI
   - Same add/remove pattern
   - Same "Request New" entity request flow
   - Same EntityCard display pattern
   - Same UI reorganization (conditional button placement)

2. **SV Preservation**: All SV endpoints preserved for internal use

   - `POST /api/sv`, `GET /api/sv`, etc. - ✅ KEEP
   - Only `GET /api/user/svs` removed (user-facing endpoint)
   - SV translation files preserved
   - SV model/controller/routes preserved

3. **User Schema Changes**: Add `boat_ids` and `boat_keys` arrays

   - Follows existing pattern (`org_ids`, `club_ids`, `sv_ids`)
   - `sv_ids` kept for internal use
   - Deduplication logic required for `boat_ids`

4. **Boat Controller Enhancement**: Populate brand/model/builder for EntityCard

   - Follows pattern from `sv.controller.js`
   - Performance acceptable (1 boat per user on average)
   - No pagination or caching needed initially

5. **EntityCard Updates**: Add `'boat'` entityType support

   - Display: sail_no, sail_cc, brand, model, builder
   - Don't display: address, phone, email, owners
   - Keep `'sailing-vessel'` entityType for internal use

6. **Data Migration**: NO MIGRATION REQUIRED

   - Clean database import handles data setup
   - `boat_ids` populated directly during import
   - No migration scripts needed

### Validation Checklist

- [x] Empty state structure understood
- [x] Non-empty state structure understood
- [x] List movement understood (above Add button)
- [x] Alert collapse understood (replaced by button)
- [x] Translation changes understood
- [x] Button implementation details confirmed
- [x] Animation approach confirmed
- [x] All three components will match
- [x] Boat management migration understood
- [x] SV preservation strategy confirmed
- [x] EntityCard boat display format confirmed
- [x] Backend changes understood

### Updated Implementation Details

**Translation Changes**:

- **Files**: `en_organizations.json`, `en_clubs.json`, `en_svs.json`
- **Change**: Update existing `request.button` text:
  - `"Request New Organization"` → `"Request NEW Organization"`
  - `"Request New Club"` → `"Request NEW Club"`
  - `"Request New Boat"` → `"Request NEW Boat"`
- **No new keys** - reuse existing structure

**Button Implementation**:

- **Icon**: `icon='message-square'` (kebab-case, matches existing pattern)
- **Styling**: `variant='outline'`, `className='flex-1'`
- **Layout**: `flex flex-col sm:flex-row gap-2`
- **Animation**: CSS transitions `transition-all duration-300`

**Add Button Placement**:

- **Empty State**: Above Alert (preserves current UI)
- **Non-Empty State**: Below list, next to Request button
- **Conditional Rendering**: Single Add button, conditionally positioned

### Success Criteria

#### UI Reorganization

1. ✅ Empty state UI remains exactly unchanged
2. ✅ Add button appears above Alert when empty
3. ✅ Add button appears below list when non-empty
4. ✅ Request button appears next to Add button (non-empty state only)
5. ✅ Alert box only appears when empty
6. ✅ Buttons stack vertically on mobile (< 640px)
7. ✅ Buttons display side-by-side on desktop (≥ 640px)
8. ✅ Translation keys updated correctly
9. ✅ All three components (Organizations, Clubs, Boats) match exactly
10. ✅ State transitions work smoothly

#### Boat Management Migration

1. ✅ Users can view boats they're associated with
2. ✅ Users can add boats to their profile
3. ✅ Users can remove boats from their profile
4. ✅ Users can request new boats
5. ✅ Boat information displays correctly in EntityCard
6. ✅ Boat management UI matches Clubs/Organizations pattern exactly
7. ✅ UI reorganization works correctly (empty vs non-empty states)
8. ✅ No SV creation/management UI remains (SV endpoints preserved for internal use)
9. ✅ Clean database import populates `boat_ids` correctly
10. ✅ Performance: Boat list loads in < 2 seconds
11. ✅ SV endpoints preserved: All `/api/sv/*` endpoints remain for internal use

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
