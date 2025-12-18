# AIN - FEATURE - SV_DUPLICATE_HANDLING

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

# Implementation Plan: SV Duplicate Handling with User Choice Modal

## Executive Summary

When a user attempts to create a Sailing Vessel (SV) for a boat that already has existing SV(s), display a modal dialog allowing the user to choose between:

1. **Create New SV** (sole owner) - Creates new SV, deletes existing non-immutable SV
2. **Clone Existing SV** (joint owner) - Clones selected existing SV and adds user as owner
3. **Cancel** - Aborts the operation

## Architecture Overview

### Current Flow

```
User selects boats → Click "Add Selected"
→ POST /api/sv { boat_id } (parallel for all)
→ If duplicate: Return existing SV with duplicate: true
→ Show toast notification
→ Refresh data
```

### New Flow

```
User selects boats → Click "Add Selected"
→ POST /api/sv { boat_id } (parallel for all)
→ Process results:
  ├─ Success: Add to success list
  ├─ Duplicate:
  │   ├─ Fetch ALL existing SVs for boat_id
  │   ├─ Show modal with list of existing SVs
  │   ├─ User selects action + existing_sv_id
  │   └─ POST /api/sv { boat_id, action, existing_sv_id }
  └─ Failed: Add to failed list
→ Show notifications
→ Refresh data
```

### Data Flow Diagrams

#### Sequence Diagram: Duplicate SV Creation Flow

```
User          Frontend          Backend          Database
 │               │                 │                 │
 │──Select Boats─>                 │                 │
 │               │                 │                 │
 │──Click Add───>                 │                 │
 │               │                 │                 │
 │               │──POST /api/sv──>                 │
 │               │  {boat_id}      │                 │
 │               │                 │──Query SV──────>│
 │               │                 │<─Existing SV────│
 │               │<─200 duplicate──│                 │
 │               │  {existingSv}   │                 │
 │               │                 │                 │
 │               │──GET /api/sv/by-boat?boat_id───>  │
 │               │                 │──Query ALL SVs─>│
 │               │<─200 {svs[]}───│<─Multiple SVs───│
 │               │                 │                 │
 │<──Show Modal──│                 │                 │
 │  (List SVs)   │                 │                 │
 │               │                 │                 │
 │──Select:      │                 │                 │
 │  Clone + SV──>│                 │                 │
 │               │                 │                 │
 │               │──POST /api/sv──>│                 │
 │               │  {boat_id,      │                 │
 │               │   action:       │                 │
 │               │   'clone',      │                 │
 │               │   existing_     │                 │
 │               │   sv_id}        │                 │
 │               │                 │──Query SV───────>│
 │               │                 │<─Existing SV────│
 │               │                 │──Clone & Insert>│
 │               │                 │<─New SV─────────│
 │               │<─201 Created───│                 │
 │               │                 │                 │
 │<──Success─────│                 │                 │
 │  Notification │                 │                 │
```

#### State Machine: Modal Flow

```
[Initial State]
    │
    ├─> [Processing Duplicates]
    │       │
    │       ├─> [Show Modal]
    │       │       │
    │       │       ├─> User selects "New" ──> [Create New SV]
    │       │       │                              │
    │       │       │                              └─> [Success]
    │       │       │
    │       │       ├─> User selects "Clone" + SV ──> [Clone SV]
    │       │       │                                  │
    │       │       │                                  ├─> Success ──> [Success]
    │       │       │                                  └─> Fail ──> [Fallback to New]
    │       │       │
    │       │       └─> User selects "Cancel" ──> [Skip]
    │       │
    │       └─> [Next Duplicate?]
    │               │
    │               ├─> Yes ──> [Show Modal] (loop)
    │               └─> No ──> [Complete]
    │
    └─> [All Complete]
            │
            └─> [Show Final Notifications]
```

## API Contract Changes

### 1. POST /api/sv (Modified)

#### Request Body (Enhanced)

```typescript
{
  boat_id: string;           // Required - Boat ID
  description?: string;      // Optional - SV description
  action?: 'new' | 'clone'; // Optional - Action when duplicate exists
  existing_sv_id?: string;  // Required if action='clone' - Existing SV ID to clone
}
```

#### Response Codes

**201 Created** - SV created successfully

```json
{
  "message": "sv.create.success",
  "data": {
    "id": "svsl_...",
    "name": "SV Boat Name",
    "boat_id": "boat_...",
    "user_ids": {
      "owners": ["user_..."]
    },
    ...
  }
}
```

**200 OK** - Duplicate detected (no action specified)

```json
{
  "message": "sv.create.duplicate",
  "data": {
    "id": "svsl_...",
    "name": "SV Boat Name",
    ...
  },
  "duplicate": true,
  "existing_svs": [  // NEW: Array of all existing SVs for this boat
    {
      "id": "svsl_...",
      "name": "SV Boat Name",
      "immutable": false,
      "user_ids": {
        "owners": ["user_1", "user_2"]
      },
      "boat": {
        "name": "Boat Name",
        "sail_cc": "US",
        "sail_no": "12345"
      }
    }
  ]
}
```

**201 Created** - SV cloned successfully

```json
{
  "message": "sv.create.clone_success",
  "data": {
    "id": "svsl_new_...",
    "name": "SV Boat Name",
    "previous_id": "svsl_original_...",
    "user_ids": {
      "owners": ["user_original_1", "user_original_2", "user_current"]
    },
    ...
  },
  "cloned": true
}
```

**400 Bad Request** - Invalid action or missing existing_sv_id

```json
{
  "message": "sv.create.invalid_action"
}
```

### 2. GET /api/sv/by-boat (New Endpoint)

#### Query Parameters

```
?boat_id=<boat_id>
```

#### Response

```json
{
  "data": [
    {
      "id": "svsl_...",
      "name": "SV Boat Name",
      "immutable": false,
      "user_ids": {
        "owners": ["user_1", "user_2"],
        "crew": ["user_3"]
      },
      "boat": {
        "id": "boat_...",
        "name": "Boat Name",
        "sail_cc": "US",
        "sail_no": "12345"
      },
      "owners": [
        // NEW: Populated owner names
        {
          "id": "user_1",
          "name": "John Doe"
        },
        {
          "id": "user_2",
          "name": "Jane Smith"
        }
      ]
    }
  ]
}
```

### 3. GET /api/sv/:id/owners (New Endpoint)

**Purpose**: Get owners for an SV

**Response**:

```json
{
  "owners": [{ "id": "user_456", "name": "First Last" }]
}
```

## Frontend Implementation

### File: `client/src/views/account/svs.jsx`

#### Changes Required

**1. Add State for Modal Management**

```javascript
const [pendingDuplicates, setPendingDuplicates] = useState([]); // Queue: [{ boatId, boatData, existingSvs }]
const [currentDuplicateIndex, setCurrentDuplicateIndex] = useState(0); // Current index in queue
const [isModalOpen, setIsModalOpen] = useState(false);
const [duplicateProgress, setDuplicateProgress] = useState({
  current: 0,
  total: 0,
}); // For progress bar
```

**2. Modify `handleCreateSVs` Function**

**Current Structure:**

```javascript
handleCreateSVs() {
  → Create SVs in parallel
  → Process results (successful, duplicates, failed)
  → Show notifications
}
```

**New Structure:**

```javascript
handleCreateSVs() {
  → Create SVs in parallel (first attempt)
  → Process results:
    → Successful: Add to success list
    → Duplicates:
      → Fetch all existing SVs for each boat (GET /api/sv/by-boat)
      → Queue duplicates for sequential processing
      → Process first duplicate (show modal)
    → Failed: Add to failed list
  → After modal closes:
    → Based on user choice:
      → 'new': POST /api/sv { boat_id, action: 'new' }
      → 'clone': POST /api/sv { boat_id, action: 'clone', existing_sv_id }
      → 'cancel': Skip
    → Process next duplicate (if any)
  → Show final notifications
  → Refresh data
}
```

**3. New Function: `handleDuplicateChoice`**

```javascript
handleDuplicateChoice(boatId, action, existingSvId) {
  → POST /api/sv { boat_id: boatId, action, existing_sv_id: existingSvId }
  → Handle response:
    → Success: Add to success list
    → Error:
      → If clone failed: Fallback to 'new' with warning toast
      → Add to failed list
  → Process next duplicate
}
```

**4. New Function: `showDuplicateModal`**

```javascript
showDuplicateModal(boatId, boatData, existingSvs, currentIndex, totalCount) {
  → Format existing SVs for display:
    → Boat name: boatData.name
    → Sail info: `${boatData.sail_cc} ${boatData.sail_no}`.trim()
    → Owner names: Format as "First Last" from owners array
    → Filter out SVs where user is already owner (frontend validation)
  → Show dialog with:
    → Progress indicator (if totalCount > 1): "Processing {currentIndex + 1} of {totalCount}"
    → Boat and sail information display
    → List of existing SVs with owners
    → Radio buttons for action selection
    → Conditional dropdown for existing_sv_id (only when action='clone')
    → Continue/Cancel buttons
  → On X button close: Cancel all remaining duplicates
  → On submit: Call handleDuplicateChoice
  → On cancel: Skip this duplicate, process next
}
```

**Frontend Validation**:

- Filter existingSvs to exclude SVs where user is already in owners array
- Validate existing_sv_id belongs to boat_id before API call
- Prevent cloning SV user already owns

**5. Modal Component Structure**

```javascript
viewContext.dialog.open({
  title: t("account.svs.duplicate.modal.title"),
  description: t("account.svs.duplicate.modal.description", {
    boatName: boat.name,
    sailInfo: `${boat.sail_cc} ${boat.sail_no}`.trim(),
  }),
  form: {
    inputs: {
      action: {
        type: "radio",
        label: t("account.svs.duplicate.modal.action.label"),
        options: [
          {
            value: "new",
            label: t("account.svs.duplicate.modal.action.new.label"),
            description: t(
              "account.svs.duplicate.modal.action.new.description"
            ),
          },
          {
            value: "clone",
            label: t("account.svs.duplicate.modal.action.clone.label"),
            description: t(
              "account.svs.duplicate.modal.action.clone.description"
            ),
          },
          {
            value: "cancel",
            label: t("account.svs.duplicate.modal.action.cancel.label"),
            description: t(
              "account.svs.duplicate.modal.action.cancel.description"
            ),
          },
        ],
        defaultValue: "new",
        required: true,
      },
      existing_sv_id: {
        type: "radio", // Radio buttons for SV selection
        label: t("account.svs.duplicate.modal.existing_sv.label"),
        options: existingSvs.map((sv) => ({
          value: sv.id,
          label: formatSvDisplay(sv), // "SV Name - Owner1, Owner2 [Immutable]"
        })),
        required: true, // Only required when action='clone'
        conditional: (formData) => formData.action === "clone",
      },
    },
    buttonText: t("account.svs.duplicate.modal.button.continue"),
    callback: async (res, formData) => {
      if (formData.action === "cancel") {
        // Skip this duplicate
        return;
      }
      await handleDuplicateChoice(
        boatId,
        formData.action,
        formData.existing_sv_id
      );
    },
  },
});
```

**6. Helper Function: `formatSvDisplay`**

```javascript
formatSvDisplay(sv) {
  // Format owner names as "First Last, First Last"
  const ownerNames = sv.owners
    ?.map(o => {
      // Name is already formatted as "First Last" from backend
      return o.name || 'Unknown';
    })
    .join(', ') || 'Unknown';

  const immutableBadge = sv.immutable ? ' [Immutable]' : '';
  return `${sv.name} - ${ownerNames}${immutableBadge}`;
}
```

**7. Helper Function: `fetchExistingSVsForBoat`**

```javascript
async fetchExistingSVsForBoat(boatId) {
  → GET /api/sv/by-boat?boat_id=${boatId}
  → Get array of existing SVs
  → For each SV: GET /api/sv/${sv.id}/owners to populate owner names
  → Return array of SVs with populated owners
}
```

**Note**: Frontend calls owners endpoint for each SV to get owner names. Could be optimized with Promise.all for parallel calls.

### Progress Bar Implementation (All Operations, Bottom of Modal)

```javascript
// Progress bar shows ALL operations (successful + duplicates + failed)
// Location: Bottom of modal, above buttons
{
  totalOperations > 1 && (
    <div className="mt-6 pt-4 border-t border-slate-200 dark:border-slate-700">
      <div className="text-sm text-slate-600 dark:text-slate-400 mb-2">
        Processing {completedOperations + 1} of {totalOperations}
      </div>
      <div className="w-full bg-slate-200 dark:bg-slate-800 rounded-full h-2">
        <div
          className="bg-primary h-2 rounded-full transition-all"
          style={{
            width: `${((completedOperations + 1) / totalOperations) * 100}%`,
          }}
        />
      </div>
    </div>
  );
}
```

**Progress Calculation**:

- `totalOperations` = successful + duplicates + failed (all boats selected)
- `completedOperations` = successful + processed duplicates + failed
- Update after each operation completes

## Backend Implementation

### File: `server/controller/sv.controller.js`

#### Changes Required

**1. Modify `exports.create` Function**

**Current Logic:**

```javascript
exports.create() {
  → Validate boat_id
  → Check for duplicate (user is owner)
  → If duplicate: Return existing SV
  → Create new SV
}
```

**New Logic:**

```javascript
exports.create() {
  → Validate boat_id, action, existing_sv_id
  → If action provided:
    → Handle action-specific logic
  → Else:
    → Check for duplicate (user is owner)
    → If duplicate:
      → Fetch ALL existing SVs for boat_id
      → Populate owner names
      → Return with existing_svs array
    → Create new SV
}
```

**2. New Function: `handleNewSvAction`**

```javascript
async handleNewSvAction(req, res, boatData, data) {
  → Find ALL existing SVs for boat_id where user is owner
  → For each existing SV:
    → If immutable: DO NOT TOUCH (leave as-is, immutable records are protected)
    → If not immutable: Delete SV entirely (sv.delete())
  → Create new SV with only current user as owner
  → Return created SV
}
```

**Important**:

- **DO NOT modify immutable SVs** - They are protected and used in race results
- Delete ALL mutable SVs where user is owner (user selected "sole ownership")
- User will own both immutable SV (if exists) and new SV after this operation

**3. New Function: `handleCloneSvAction`**

```javascript
async handleCloneSvAction(req, res, boatData, data, existingSvId) {
  → Validate existing_sv_id exists and belongs to boat_id
  → Validate user is NOT already an owner (prevent duplicate)
  → Fetch existing SV
  → Clone SV:
    → Copy ALL fields including:
      → name (same)
      → display (copy)
      → description (copy)
      → certs (deep copy)
      → user_keys (copy array as-is)
      → user_ids (copy structure, add current user to owners)
      → boat_key, boat_id (copy)
    → Set new fields:
      → id (new unique ID)
      → key (omit - don't set)
      → user_ids.owners (add current user to existing owners array)
      → previous_id (set to existing SV id)
      → immutable (always set to false)
      → created_at (new timestamp)
      → updated_at (null)
      → rev (0)
  → Create cloned SV
  → Return cloned SV
}
```

**Deep Copy for certs**:

- Use JSON.parse(JSON.stringify(existingSv.certs)) or structuredClone()
- Ensure nested objects are fully copied

**4. Helper Function: `getAllSVsForBoat`**

```javascript
async getAllSVsForBoat(boatId) {
  → Query: sv.schema.find({ boat_id: boatId })
  → Populate boat data
  → Populate owner names (user.get() for each owner ID)
  → Return array with owners array populated
}
```

**5. New Endpoint: `GET /api/sv/:id/owners` (Get Owners for SV)**

```javascript
exports.getOwners = async function(req, res) {
  → Validate sv_id parameter
  → Fetch SV by id
  → Extract owner IDs from user_ids.owners
  → Fetch user data for each owner ID (batch lookup)
  → Format names as "First Last"
  → Return: { owners: [{ id: "user_...", name: "John Doe" }, ...] }
}
```

**Note**: This endpoint will be called from frontend for each SV to populate owner names. Backend handles name formatting.

**Name Formatting Logic**:

```javascript
function formatUserName(user) {
  if (!user || !user.name) return "Unknown";

  // user.name might be "John Doe" or structured object
  // If string: split and take first + last
  // If object: use name.first + name.last
  if (typeof user.name === "string") {
    const parts = user.name.trim().split(/\s+/);
    if (parts.length >= 2) {
      return `${parts[0]} ${parts[parts.length - 1]}`;
    }
    return parts[0] || "Unknown";
  }

  // If structured name object
  if (user.name.first && user.name.last) {
    return `${user.name.first} ${user.name.last}`;
  }

  return "Unknown";
}
```

### File: `server/api/sv.route.js`

#### New Routes

```javascript
api.get("/api/sv/by-boat", auth.verify("user"), use(svController.getByBoat));
api.get("/api/sv/:id/owners", auth.verify("user"), use(svController.getOwners));
```

### File: `server/controller/sv.controller.js`

#### New Function: `exports.getByBoat`

```javascript
exports.getByBoat = async function(req, res) {
  → Validate boat_id query parameter
  → Fetch all SVs for boat_id
  → Populate boat data
  → Populate owner names
  → Return array of SVs
}
```

## User Interface Design

### Modal Layout

```
┌─────────────────────────────────────────────┐
│  Sailing Vessel Already Exists        [X]   │
├─────────────────────────────────────────────┤
│                                             │
│  A sailing vessel already exists for:       │
│                                             │
│  Boat: [Boat Name]                          │
│  Sail: [US 12345]                          │
│                                             │
│  Existing Sailing Vessel(s):                │
│  ┌─────────────────────────────────────┐   │
│  │ • SV Boat Name                      │   │
│  │   Owners: John Doe, Jane Smith      │   │
│  │   [Immutable]                       │   │
│  └─────────────────────────────────────┘   │
│                                             │
│  Note: Only Boat Name and Sail info shown  │
│  (No description, certs, dates)            │
│                                             │
│  What would you like to do?                 │
│                                             │
│  ○ Create New SV (I am the sole owner)     │
│    Create a new sailing vessel with only   │
│    you as the owner.                       │
│                                             │
│  ○ Clone Existing SV (Joint ownership)      │
│    Create a copy of an existing SV and     │
│    add yourself as a joint owner.          │
│                                             │
│    Select SV to clone:                      │
│    [Dropdown: SV Boat Name - John, Jane ▼]  │
│                                             │
│  ○ Cancel (Selection mistake)               │
│    Do not create a sailing vessel for      │
│    this boat.                              │
│                                             │
│                    [Cancel]  [Continue]     │
└─────────────────────────────────────────────┘
```

### Conditional Display Logic

- **Action = "new"**: Hide existing_sv_id dropdown
- **Action = "clone"**: Show existing_sv_id dropdown (required, filtered to exclude SVs user already owns)
- **Action = "cancel"**: Hide existing_sv_id dropdown
- **Multiple Existing SVs**: Show dropdown/select list of all available SVs
- **Single Existing SV**: Pre-select in dropdown (if clone selected and user doesn't own it)
- **Immutable SV**: Show "[Immutable]" badge/indicator
- **Progress Bar**: Show if totalCount > 1 (e.g., "Processing 2 of 5")
- **X Button**: Cancel all remaining duplicates, close modal, show cancellation notification

## State Management

### State Variables

```javascript
// Existing
const [selectedBoatIds, setSelectedBoatIds] = useState([]);
const [creating, setCreating] = useState(false);

// New
const [pendingDuplicates, setPendingDuplicates] = useState([]); // Queue: [{ boatId, boatData, existingSvs }]
const [currentDuplicateIndex, setCurrentDuplicateIndex] = useState(0);
const [duplicateResults, setDuplicateResults] = useState({
  successful: [],
  cloned: [],
  failed: [],
  cancelled: [],
});
```

### State Flow

```
Initial State
  → User clicks "Add Selected"
  → creating = true
  → Create SVs in parallel (first attempt)
  → Process results:
    → Successful → Add to success list, show success toast
    → Duplicates →
      → Fetch all existing SVs for boat (GET /api/sv/by-boat)
      → Filter out SVs where user is already owner (frontend validation)
      → Add to pendingDuplicates queue: [{ boatId, boatData, existingSvs }]
    → Failed → Add to failed list, show error toast
  → If pendingDuplicates.length > 0:
    → Set duplicateProgress: { current: 0, total: pendingDuplicates.length }
    → Show modal for pendingDuplicates[0]
    → Wait for user choice:
      → User selects action + existing_sv_id (if clone)
      → User clicks Continue:
        → Validate existing_sv_id belongs to boat_id (frontend)
        → POST /api/sv with action
        → Process response:
          → Success → Add to appropriate list
          → Clone failed → Fallback to 'new' with warning toast
      → User clicks Cancel OR X button:
        → Cancel all remaining duplicates
        → Clear pendingDuplicates queue
        → Close modal
        → Show cancellation notification
    → If Continue clicked:
      → Remove current from queue
      → Increment currentDuplicateIndex
      → Update duplicateProgress
      → If more in queue → Show next modal
      → If queue empty → Complete
  → creating = false
  → Show final summary notifications
  → Refresh data (triggerRefresh, triggerSvsRefresh)
```

### Cancel All Behavior

When user clicks X button or Cancel:

```javascript
→ Count cancelled operations: pendingDuplicates.length
→ Clear pendingDuplicates queue
→ Set currentDuplicateIndex to 0
→ Close modal
→ Show notification: "Cancelled creating {count} sailing vessel(s)"
  → Example: "Cancelled creating 3 sailing vessel(s)"
→ Continue with already successful SVs
→ Refresh data
```

**Notification Format**:

- Show count only: "Cancelled creating X sailing vessel(s)"
- No list of boat names
- No additional details

## Error Handling

### Error Scenarios

**1. Clone Fails (Validation/Other Error)**

- **Action**: Fallback to 'new' action automatically
- **User Feedback**: Warning toast: "Clone failed, created new SV instead"
- **Backend Response**: Return success with flag `{ cloned: false, fallback: true }`
- **Logging**: Log fallback for debugging

**2. Clone Fails (SV Not Found)**

- **Action**: Return 404 error
- **User Feedback**: Error toast with message
- **Logging**: Log error

**3. Clone Fails (Network Error)**

- **Action**: Show error, allow retry
- **User Feedback**: Error toast with retry option
- **Logging**: Log error

**4. Delete Fails (Non-Immutable SV)**

- **Action**: Continue with creation, log error
- **User Feedback**: Error toast: "Failed to delete existing SV, but new SV was created"
- **Logging**: Log error for debugging/admin review

**5. Invalid Action Parameter**

- **Action**: Return 400 Bad Request
- **User Feedback**: Error toast
- **Logging**: Log validation error

**6. Missing existing_sv_id for Clone**

- **Action**: Return 400 Bad Request
- **User Feedback**: Error toast
- **Logging**: Log validation error

### Error Handling Code Structure

```javascript
try {
  if (action === 'clone') {
    try {
      return await handleCloneSvAction(...);
    } catch (cloneError) {
      // Fallback to new
      mcode.warn('SV clone failed, falling back to new:', cloneError);
      // Show warning toast (handled in frontend)
      return await handleNewSvAction(...);
    }
  }
} catch (error) {
  // Handle other errors
}
```

## Security Considerations

### 1. Authorization Checks

**Before Clone:**

- Verify existing_sv_id belongs to the specified boat_id
- Verify user has permission to clone (anyone can clone, but validate boat_id match)

**Before Delete (New Action):**

- Verify user is owner of existing SV
- Verify SV is not immutable
- Verify SV belongs to specified boat_id

**Before Create:**

- Verify boat_id exists and is valid
- Verify user has permission to create SVs

### 2. Input Validation

```javascript
// Validate action
if (data.action && !["new", "clone"].includes(data.action)) {
  throw { message: res.__("sv.create.invalid_action") };
}

// Validate existing_sv_id when cloning
if (data.action === "clone" && !data.existing_sv_id) {
  throw { message: res.__("sv.create.missing_existing_sv_id") };
}

// Validate existing_sv_id exists and belongs to boat
if (data.action === "clone") {
  const existingSv = await sv.schema
    .findOne({
      id: data.existing_sv_id,
      boat_id: data.boat_id,
    })
    .lean();

  if (!existingSv) {
    throw { message: res.__("sv.create.invalid_existing_sv") };
  }
}
```

### 3. Data Sanitization

- Sanitize boat_id, existing_sv_id (prevent injection)
- Validate boat_id format
- Validate existing_sv_id format

### 4. Rate Limiting

- Existing rate limits apply
- Consider additional limits for clone operations

## Testing Strategy

### Unit Tests

**Backend (`server/controller/sv.controller.js`)**

- [ ] Test create with no duplicate (normal flow)
- [ ] Test create with duplicate, no action (returns existing_svs)
- [ ] Test create with action='new', non-immutable SV exists (deletes old, creates new)
- [ ] Test create with action='new', immutable SV exists (keeps old, creates new)
- [ ] Test create with action='clone' (creates clone with user added)
- [ ] Test create with action='clone', invalid existing_sv_id (returns error)
- [ ] Test create with action='clone', existing_sv_id for different boat (returns error)
- [ ] Test clone includes all fields correctly
- [ ] Test clone sets previous_id correctly
- [ ] Test clone sets immutable to false
- [ ] Test populateOwnerNames function
- [ ] Test getAllSVsForBoat function

**Frontend (`client/src/views/account/svs.jsx`)**

- [ ] Test handleCreateSVs with no duplicates
- [ ] Test handleCreateSVs with single duplicate
- [ ] Test handleCreateSVs with multiple duplicates (sequential)
- [ ] Test modal shows correct boat/SV information
- [ ] Test modal radio button selection
- [ ] Test modal SV dropdown (when clone selected)
- [ ] Test handleDuplicateChoice with action='new'
- [ ] Test handleDuplicateChoice with action='clone'
- [ ] Test handleDuplicateChoice with action='cancel'
- [ ] Test clone failure fallback to 'new'
- [ ] Test formatSvDisplay function

### Integration Tests

- [ ] End-to-end: Select boat → Create SV → Duplicate detected → Modal → Clone → Success
- [ ] End-to-end: Select boat → Create SV → Duplicate detected → Modal → New → Success
- [ ] End-to-end: Select boat → Create SV → Duplicate detected → Modal → Cancel → Skipped
- [ ] Multiple boats selected, some duplicates → Process non-duplicates first
- [ ] Clone fails → Fallback to new → Success

### Edge Cases

- [ ] Multiple SVs for same boat (show all in modal)
- [ ] SV with many owners (display truncation)
- [ ] SV with no owners (shouldn't happen, but handle gracefully)
- [ ] Boat with no sail info (display gracefully)
- [ ] Network error during clone → Retry mechanism
- [ ] User closes modal without selecting → Treat as cancel
- [ ] Immutable SV in list (show indicator, disable delete)

## Translation Keys

### Client-Side (`client/src/locales/en.json`)

```json
{
  "account": {
    "svs": {
      "duplicate": {
        "modal": {
          "title": "Sailing Vessel Already Exists",
          "description": "A sailing vessel already exists for this boat. Please choose how you would like to proceed.",
          "boat_label": "Boat",
          "sail_label": "Sail",
          "owners_label": "Owner(s)",
          "existing_svs_label": "Existing Sailing Vessel(s)",
          "action": {
            "label": "What would you like to do?",
            "new": {
              "label": "Create New SV (I am the sole owner)",
              "description": "Create a new sailing vessel configuration with only you as the owner. If a non-immutable SV exists where you are an owner, it will be deleted."
            },
            "clone": {
              "label": "Clone Existing SV (Joint ownership)",
              "description": "Create a copy of an existing SV and add yourself as a joint owner. All configuration, certificates, and other owners will be copied."
            },
            "cancel": {
              "label": "Cancel (Selection mistake)",
              "description": "Do not create a sailing vessel for this boat."
            }
          },
          "existing_sv": {
            "label": "Select SV to clone",
            "placeholder": "Choose an existing sailing vessel"
          },
          "button": {
            "continue": "Continue",
            "cancel": "Cancel"
          }
        },
        "clone_fallback_warning": "Failed to clone SV. Creating new SV instead.",
        "clone_success": "Sailing vessel cloned successfully: {name}",
        "new_success": "New sailing vessel created successfully: {name}",
        "cancelled": "Skipped creating sailing vessel for {boatName}"
      }
    }
  }
}
```

### Server-Side (`server/locales/en.json`)

```json
{
  "sv": {
    "create": {
      "success": "Sailing vessel created successfully",
      "clone_success": "Sailing vessel cloned successfully",
      "clone_error": "Failed to clone sailing vessel",
      "new_success": "New sailing vessel created successfully",
      "invalid_action": "Invalid action. Must be 'new' or 'clone'.",
      "missing_existing_sv_id": "existing_sv_id is required when action is 'clone'.",
      "invalid_existing_sv": "The specified existing SV does not exist or does not belong to this boat.",
      "duplicate": "A sailing vessel already exists for this boat."
    }
  }
}
```

## Migration Considerations

### Database Changes

- **None required** - All changes are application-level

### Data Integrity

- Existing SVs remain unchanged
- New SVs follow same schema
- `previous_id` field already exists for tracking clones

### Backward Compatibility

- API remains backward compatible:
  - If `action` not provided → Returns duplicate response (existing behavior)
  - If `action` provided → New behavior
- Frontend gracefully handles missing `existing_svs` array in duplicate response

## Key Design Decisions (Confirmed)

### 1. Owner Name Display

- ✅ Format: "First Last" only (e.g., "John Doe")
- ✅ Fallback: "Unknown" if name not available
- ✅ No truncation needed (not common occurrence)

### 2. Modal Behavior

- ✅ X button = Cancel ALL remaining duplicates
- ✅ Progress bar: Show "Processing X of Y" if total > 1
- ✅ No going back to previous duplicates

### 3. Clone Details

- ✅ Copy EVERYTHING: name, display, description, certs (deep copy), user_keys, user_ids structure
- ✅ Add current user to owners array (don't replace)
- ✅ Set previous_id to original SV id
- ✅ Set immutable to false (clones are always mutable)

### 4. Delete Behavior (New Action)

- ✅ Immutable SV: **DO NOT TOUCH** - Leave completely unchanged
- ✅ Mutable SV: Delete ALL mutable SVs where user is owner
- ✅ User's choice ("sole ownership" = new) guides deletion of mutable SVs
- ✅ User's choice ("joint ownership" = clone) means no deletions
- ✅ No notification to other owners

### 5. Performance

- ✅ No pagination initially
- ✅ No caching initially
- ✅ Can optimize later if needed

### 6. Validation

- ✅ Frontend: Validate existing_sv_id belongs to boat_id
- ✅ Frontend: Filter out SVs where user is already owner
- ✅ Backend: Re-validate all checks

### 7. Error Handling

- ✅ Clone fails → Fallback to 'new' with warning toast
- ✅ Delete fails → Continue with creation, log warning
- ✅ Remove from owners fails → Continue with creation, log warning

### 8. Processing Order

- ✅ Process non-duplicates first (show success toasts)
- ✅ Then process duplicates sequentially (one modal at a time)
- ✅ Show progress bar if multiple duplicates

## File Change Summary

### Files to Modify

1. **`server/controller/sv.controller.js`**

   - Modify `exports.create` function
   - Add `handleNewSvAction` function
   - Add `handleCloneSvAction` function
   - Add `getAllSVsForBoat` helper function
   - Add `populateOwnerNames` helper function
   - Add `exports.getByBoat` function
   - Add `exports.getOwners` function

2. **`server/api/sv.route.js`**

   - Add GET route for `/api/sv/by-boat`
   - Add GET route for `/api/sv/:id/owners`

3. **`client/src/views/account/svs.jsx`**

   - Modify `handleCreateSVs` function
   - Add `showDuplicateModal` function
   - Add `handleDuplicateChoice` function
   - Add `fetchExistingSVsForBoat` function
   - Add `formatSvDisplay` helper function
   - Add state management for duplicate queue

4. **Translation Files**
   - `client/src/locales/en.json` (and other locales)
   - `server/locales/en.json` (and other locales)

### Files to Create

- None (all functionality fits in existing files)

### Files to Review (No Changes Expected)

- `server/model/sv.model.js` - Verify clone logic compatibility
- `server/model/user.model.js` - Verify user lookup for owner names
- `client/src/components/dialog/dialog.jsx` - Verify modal supports our needs
- `client/src/components/form/form.jsx` - Verify form supports conditional fields

---

## V: REVIEW - Review and validate the implementation plan

## Final Confidence: **96%** ✅ (All questions answered)

**Previous Confidence**: 94%
**Confidence Increase**: +2% from detailed clarifications
**Current Confidence**: 96%

## Confidence Assessment

### Strengths (High Confidence Areas)

1. **Backend API Structure** ✅ **95%**

   - Clear understanding of `sv.create()`, `sv.delete()`, `sv.schema` patterns
   - User lookup via `user.get({ id })` is well-established pattern
   - Route structure (`server/api/sv.route.js`) follows existing patterns
   - Error handling patterns match existing codebase style

2. **Frontend State Management** ✅ **95%**

   - `useAPI` hook pattern is well-understood
   - State management follows existing patterns (`organizations.jsx`, `clubs.jsx`)
   - Modal dialog pattern (`viewContext.dialog.open`) is established
   - Sequential processing queue pattern is clear

3. **Data Flow** ✅ **94%**

   - Duplicate detection logic is clear
   - Modal → API → Response flow is straightforward
   - Error handling and fallback logic is well-defined

4. **UI Components** ✅ **93%**
   - Radio buttons, dropdowns, progress bar are standard components
   - Modal layout follows existing dialog patterns
   - Translation keys follow established structure

## All Questions Answered - Plan Finalized

### ✅ Confirmed Answers Summary

1. **Get Owners Endpoint**:

   - **Q1a**: Path: `GET /api/sv/:id/owners` ✅ (RESTful, follows existing pattern)
   - **Q1b**: Response: `{ owners: [{ id, name: "First Last" }] }` ✅
   - **Q1c**: Backend: Populates names ✅ (simpler frontend, consistent with other endpoints)
   - **Q1d**: Frontend: Uses Promise.all for batch calls ✅ (start with single SV lookup, can add batch later if needed)
   - **Impact**: Medium - Affects API design
   - **Confidence Impact**: +1.5% (answered)

2. **Progress Bar**:

   - **Q2a**: Show immediately when "Add Selected" clicked ✅
   - **Q2b**: Update after each Promise.allSettled result processes ✅
   - Display: "Processing 0 of 5" → "Processing 2 of 5" (as parallel completes) → "Processing 3 of 5" (duplicates start)
   - Track all operations (parallel + sequential)
   - Update incrementally after each operation
   - **Impact**: Low-Medium - Affects UX polish
   - **Confidence Impact**: +0.5% (answered)

3. **Immutable SVs**:

   - Show in modal list
   - Allow "new" action
   - User can own both immutable + new SV

4. **Clone Fallback**:

   - **Q4a**: Backend returns `{ cloned: false, fallback: true }` ✅
   - **Q4b**: Frontend shows warning toast ✅
   - Translation key: `account.svs.duplicate.clone_fallback_warning`
   - Message: "Clone failed, created new SV instead"
   - Response format:
     ```json
     {
       "message": "sv.create.new_success",
       "data": {
         /* new SV data */
       },
       "cloned": false,
       "fallback": true
     }
     ```
   - **Impact**: Low - Affects error communication polish
   - **Confidence Impact**: +0.5% (answered)

5. **Radio Buttons**:

   - **Q5a**: Only show when action='clone' ✅ (conditional display)
   - **Q5b**: Immutable SVs selectable (with badge) ✅ (user can clone immutable SV)
   - **Q5c**: Show even if only one SV (pre-selected) ✅ (user can still see which SV they're cloning)
   - **Impact**: Low - Affects UI polish
   - **Confidence Impact**: +0.5% (answered)

6. **Delete Failure**:

   - **Q6a**: Log error + show error toast, but continue ✅ (don't block new SV creation)
   - **Q6b**: Translation key: `account.svs.create.delete_failed` ✅
   - Message: "Failed to delete existing SV, but new SV was created"
   - **Impact**: Low - Edge case handling
   - **Confidence Impact**: +0.5% (answered)

7. **Boat Data**:
   - Handle missing sail_cc/sail_no gracefully ✅

### Detailed Question Clarifications

#### Critical Questions (Answered)

**Q1: New "Get Owners" Endpoint Details**

- **Q1a**: Endpoint path: `GET /api/sv/:id/owners` ✅
  - RESTful, follows existing pattern
  - Alternative considered: `GET /api/sv/owners/:sv_id` (rejected)
  - Alternative considered: Query param `?sv_id=...` (rejected)
- **Q1b**: Response format: `{ owners: [{ id, name: "First Last" }] }` ✅
  - Includes user ID and formatted name
  - Backend populates names (not just IDs)
- **Q1c**: Backend populates names ✅
  - Recommendation: Backend populates names (simpler frontend, consistent with other endpoints)
  - Frontend doesn't need to do additional lookups
- **Q1d**: Batch lookup: Single SV lookup, frontend uses Promise.all ✅
  - Start with single SV lookup
  - Frontend can use `Promise.all()` to fetch owners for multiple SVs in parallel
  - Can add batch endpoint later if needed

**Impact**: Medium - Affects API design
**Confidence Impact**: +1.5% (answered)

#### Important Questions (Answered)

**Q2: Progress Bar Visibility**

- **Q2a**: Show progress bar immediately when "Add Selected" clicked ✅
  - Display: "Processing 0 of 5" → "Processing 2 of 5" (as parallel completes) → "Processing 3 of 5" (duplicates start)
- **Q2b**: Update after each Promise.allSettled result processes ✅
  - Show: "Processing X of Y" where X increments as each result is processed
  - Update incrementally after each operation completes

**Impact**: Low-Medium - Affects UX polish
**Confidence Impact**: +0.5% (answered)

**Q4: Clone Fallback Response Flag**

- **Q4a**: Backend returns explicit flag `{ cloned: false, fallback: true }` ✅
  - Response format includes flag so frontend can show appropriate toast
- **Q4b**: Toast message: Use translation key `account.svs.duplicate.clone_fallback_warning` ✅
  - Message: "Clone failed, created new SV instead"

**Impact**: Low - Affects error communication polish
**Confidence Impact**: +0.5% (answered)

**Q5: Radio Button Display Logic**

- **Q5a**: Radio buttons only shown when action='clone' ✅
  - Conditional display
  - Hide when action='new' or action='cancel'
- **Q5b**: Immutable SVs are selectable ✅
  - User can clone immutable SV
  - Show visual indicator (badge) but allow selection
- **Q5c**: Show radio buttons even if only one SV exists ✅
  - Pre-select the single SV
  - User can still see which SV they're cloning

**Impact**: Low - Affects UI polish
**Confidence Impact**: +0.5% (answered)

**Q6: Delete Failure Error Toast**

- **Q6a**: Error toast warns but continues ✅
  - Don't block new SV creation
  - Just warn but continue
- **Q6b**: Error message: Use translation key `account.svs.create.delete_failed` ✅
  - Message: "Failed to delete existing SV, but new SV was created"

**Impact**: Low - Edge case handling
**Confidence Impact**: +0.5% (answered)

### Questions Summary

**Questions Answered ✅**:

- Q1a-Q1d: Get Owners Endpoint Details → All answered ✅
- Q2a-Q2b: Progress Bar Visibility → All answered ✅
- Q3: Immutable-only SVs → Show in modal, allow "new" ✅
- Q4a-Q4b: Clone Fallback Response Flag → All answered ✅
- Q5a-Q5c: Radio Button Display Logic → All answered ✅
- Q6a-Q6b: Delete Failure Error Toast → All answered ✅
- Q7: Boat Data → Handle missing gracefully ✅

**Total Confidence Impact**: +3.5% (from detailed clarifications)

### Recommended Defaults (All Applied)

1. **Q1**: Use `GET /api/sv/:id/owners`, backend populates names, frontend uses Promise.all for batch ✅
2. **Q2**: Show progress bar immediately, update after each operation completes ✅
3. **Q4**: Backend returns `{ cloned: false, fallback: true }`, frontend shows warning toast ✅
4. **Q5**: Radio buttons only when action='clone', immutable SVs selectable, show even if one SV ✅
5. **Q6**: Error toast warns but continues, use translation key ✅

## Implementation Readiness Checklist

### Backend ✅ Ready (96%)

- [x] API endpoint structure understood
- [x] Database operations pattern clear
- [x] Error handling approach defined
- [x] Clone validation logic clear
- [x] Delete logic for mutable SVs clear
- [x] Immutable SV handling clear (don't touch)
- [x] New "get owners" endpoint: `GET /api/sv/:id/owners` ✅
- [x] Response format: `{ owners: [{ id, name }] }` ✅
- [x] Backend populates names ✅

### Frontend ✅ Ready (96%)

- [x] State management pattern clear
- [x] Modal dialog pattern understood
- [x] Sequential processing queue logic clear
- [x] Progress bar tracking approach confirmed ✅
- [x] Error handling approach defined
- [x] Translation key structure clear
- [x] Radio button display approach confirmed ✅
- [x] Clone fallback toast confirmed ✅
- [x] Delete failure toast confirmed ✅

### Edge Cases ✅ Ready (96%)

- [x] Clone validation failure → auto-fallback ✅
- [x] Immutable-only SVs scenario
- [x] Clone fallback user communication ✅
- [x] Delete failure handling ✅
- [x] Missing boat data fields

## Risk Assessment

### Low Risk ✅

- Standard CRUD operations
- Existing component patterns
- Well-defined error handling

### Medium Risk ⚠️

- Sequential modal processing (state management complexity)
- Batch user lookups (performance consideration)
- Clone field copying (completeness verification needed)

### Mitigation Strategies

- Comprehensive unit tests for clone logic
- Batch user lookups with Promise.all (or schema.find with $in)
- Clear state machine for modal processing
- Extensive error logging for debugging

## Implementation Status

**Status**: ✅ **READY FOR IMPLEMENTATION**

**Confidence**: **96%** ✅ (All questions answered)

All questions answered and confirmed. The plan is **fully implementable** with all design decisions and technical details resolved.

**Key Points**:

- All major design decisions are resolved
- All critical questions have been answered (Q1a-Q1d)
- All important questions have been answered (Q2a-Q2b, Q4a-Q4b, Q5a-Q5c, Q6a-Q6b)
- All edge cases addressed (Q3, Q7)
- Implementation follows established patterns
- Risk mitigation strategies are in place
- Detailed clarifications provide clear implementation guidance

## Success Criteria

### Functional Requirements ✅

- [x] User can create new SV when duplicate exists
- [x] User can clone existing SV when duplicate exists
- [x] User can cancel SV creation when duplicate exists
- [x] Non-immutable SV is deleted when user chooses "new"
- [x] Immutable SV is preserved when user chooses "new"
- [x] Cloned SV includes all configuration from original
- [x] Cloned SV has user added to owners list
- [x] Multiple duplicates are handled sequentially
- [x] Non-duplicates are processed before duplicates

### Technical Requirements ✅

- [x] New API endpoint for getting owners
- [x] Progress bar tracks all operations
- [x] Radio buttons for SV selection
- [x] Warning toast for clone fallback
- [x] Error toast for delete failure
- [x] Graceful handling of missing boat data

## Implementation Checklist

### Phase 1: Backend API Enhancement

- [ ] Add validation for `action` and `existing_sv_id` parameters
- [ ] Implement `handleNewSvAction` function
- [ ] Implement `handleCloneSvAction` function
- [ ] Implement `getAllSVsForBoat` function
- [ ] Implement `populateOwnerNames` helper function
- [ ] Modify `exports.create` to handle actions
- [ ] Add `exports.getByBoat` endpoint
- [ ] Add `exports.getOwners` endpoint
- [ ] Add route for GET /api/sv/by-boat
- [ ] Add route for GET /api/sv/:id/owners
- [ ] Add server-side translation keys
- [ ] Add error handling and fallback logic
- [ ] Write unit tests for backend functions

### Phase 2: Frontend Modal Implementation

- [ ] Add state management for duplicate queue
- [ ] Modify `handleCreateSVs` to queue duplicates
- [ ] Implement `fetchExistingSVsForBoat` function
- [ ] Implement `showDuplicateModal` function
- [ ] Implement `handleDuplicateChoice` function
- [ ] Implement `formatSvDisplay` helper function
- [ ] Add modal dialog with radio buttons
- [ ] Add conditional SV dropdown for clone action
- [ ] Add client-side translation keys
- [ ] Implement sequential modal processing
- [ ] Add loading states during processing
- [ ] Implement progress bar (all operations, bottom of modal)
- [ ] Write unit tests for frontend functions

### Phase 3: Error Handling & Edge Cases

- [ ] Implement clone failure fallback
- [ ] Handle network errors gracefully
- [ ] Handle invalid existing_sv_id
- [ ] Handle immutable SV deletion attempt
- [ ] Add error toast notifications
- [ ] Add warning toast for fallback scenarios
- [ ] Test all error scenarios

### Phase 4: UI/UX Polish

- [ ] Format boat/SV display in modal
- [ ] Format owner names list
- [ ] Show immutable badge/indicator
- [ ] Handle long owner lists (truncation)
- [ ] Handle missing sail info gracefully
- [ ] Add loading indicators
- [ ] Test responsive design
- [ ] Test accessibility (keyboard navigation, screen readers)

### Phase 5: Testing & Documentation

- [ ] Write comprehensive unit tests
- [ ] Write integration tests
- [ ] Test all user flows
- [ ] Test error scenarios
- [ ] Update API documentation
- [ ] Update user documentation (if applicable)

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
