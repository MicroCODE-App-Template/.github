# Issue: Expose Full User Schema Editing in Client UI (Expanded)

## 1. Problem Description

The current Client UI (`client/src/views/account/users.jsx`) and the backend API (`server/controller/user.controller.js`) only support editing a limited subset of the User Schema fields. The user wants to:

1.  Edit **all** fields defined in the User Schema (except `type` and `state`).
2.  Expand the schema and UI to support extended fields: `name_`, `email_`, `phone_`, and `address_` (mirroring the structure found in `server/seed/USER_COMPLETE.js`).

## 2. Schema Analysis & Expansion

### 2.1 Model File Structure

**IMPORTANT**: The true source files are in `server/model/mongo/` and `server/model/sql/`. Files in `server/model/` root are **generated copies** created by `server/bin/setup.js` based on `DB_CLIENT` environment variable.

- **MongoDB Source**: `server/model/mongo/user.mongo.js` → copied to `server/model/user.model.js` when `DB_CLIENT=mongo`
- **SQL Source**: `server/model/sql/user.sql.js` → copied to `server/model/user.model.js` when `DB_CLIENT=sql`

**We must modify the SOURCE files** (`server/model/mongo/user.mongo.js` and `server/model/sql/user.sql.js`), not the generated copies.

### 2.2 Current vs. Target Schema

#### MongoDB Schema (`server/model/mongo/user.mongo.js`)

| Field      | Current Type | Target Type | Notes                                                                                                                                                |
| :--------- | :----------- | :---------- | :--------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`     | String       | String      | **Display/Assembled**: Auto-generated from `name_` at edit time. Stored in DB for quick viewing.                                                     |
| `name_`    | Object       | Object      | **Expanded**: `{ prefix, first, middle, last, suffix }` - See USER_COMPLETE.js (15-22)                                                               |
| `email`    | String       | String      | **Primary Login**: Driven by `email_.primary` if present. Display field used for direct email sending (clickable mailto:).                           |
| `email_`   | Object       | Object      | **Expanded**: `{ primary, personal, work }` (may include `other`)                                                                                    |
| `phone`    | Object       | **String**  | **Primary Phone**: Driven by `phone_.primary` if present. Display formatted as +1 (AAA) ZZZ-NNNN (US) or E.164 (international). Clickable tel: link. |
| `phone_`   | **N/A**      | **Object**  | **New**: `{ primary, cell, home, work }`                                                                                                             |
| `address`  | Object       | **String**  | **Display/Assembled**: Auto-generated from `address_` at edit time. Format: `${street}, ${city}, ${state} ${zip}`                                    |
| `address_` | **N/A**      | **Object**  | **New**: `{ company, c_o, street, unit, city, state, zip, country }` - Renamed from current `address`                                                |
| `settings` | Object       | Object      | Flexible JSON                                                                                                                                        |

#### SQL Schema (`server/model/sql/user.sql.js`)

- SQL schema uses table structure, not Mongoose schema.
- Need to check current SQL table structure and add columns for `phone_` and `address_`.
- `phone` column should change from JSON/TEXT to VARCHAR.
- `address` column should remain JSON/TEXT (for `address_`), add new `address` VARCHAR column.

**Read-Only / System Managed (Excluded from Edit):**

- `id`, `key`, `rev`
- `type`, `state`
- `created_at`, `updated_at`, `active_at`
- `password`, `tfa_*`
- `facebook_id`, `twitter_id`, `push_token`
- `account`, `default_account_id` (Managed via other UI)

### 2.3 Data Structure (from USER_COMPLETE.js)

```javascript
// name_ - Full structure
{
    prefix: 'Mr.',
    first: 'Timothy',
    middle: 'J.',
    last: 'McGuire',
    suffix: 'Sr.'
}
// Assembled name: "Mr. Timothy J. McGuire, Sr."

// email_
{
    primary: 'tmcguire@mcode.com',
    personal: 'timothy.mcguire@mcode.com',
    work: 'timothy.mcguire@gm.com'
}

// phone_
{
    primary: '+18104597508',
    cell: '+18104597508',
    home: '+15864160611',
    work: '+12484211010'
}
// Display format: "+1 (810) 459-7508" (US) or "+441234567890" (E.164 for international)

// address_ - Full structure (renamed from address)
{
    company: 'MicroCODE, Inc.',
    c_o: 'Cathy McGuire',
    street: '48882 Beacon Square Dr.',
    city: 'Macomb',
    state: 'MI',
    zip: '48044-5918',
    country: 'USA'
}
// Assembled address: "48882 Beacon Square Dr., Macomb, MI 48044-5918"
```

## 3. Proposed Solution

### 3.1 Backend Changes

#### 3.1.1 Schema Update - MongoDB (`server/model/mongo/user.mongo.js`)

- Change `phone` type from `Object` to `String`.
- Add `phone_` field with type `Object`.
- **Rename `address` to `address_`** (keep as Object).
- **Add new `address` field** as `String` (display/assembled field).

#### 3.1.2 Schema Update - SQL (`server/model/sql/user.sql.js`)

- **Database**: PostgreSQL (assumed)
- **Current State**: SQL user table does NOT currently have `phone`, `address`, `name_`, or `email_` columns (verified in migration `20200729133645_user_table.js`).
- **Clean Slate Approach**: No migrations needed. Schema changes will be handled in model file for new installations.
- **New Columns** (to be added when table is created):
  - `name_` JSONB (nullable) - PostgreSQL JSONB type for better performance
  - `email_` JSONB (nullable)
  - `phone` VARCHAR(255) (nullable) - Display string
  - `phone_` JSONB (nullable)
  - `address` VARCHAR(500) (nullable) - Display string
  - `address_` JSONB (nullable)
- **Model Update**: Update `server/model/sql/user.sql.js` to:
  - Handle new fields in `create` function.
    - PostgreSQL JSONB: Can insert objects directly, Knex handles serialization automatically.
    - For safety, use `JSON.stringify()` when building data object, or let Knex handle it.
  - Add new columns to explicit column list in `get` function.
  - Handle new fields in `update` function.
    - PostgreSQL JSONB: Knex handles JSON serialization automatically, but we can use `JSON.stringify()` for explicit control.

#### 3.1.3 Controller Update (`server/controller/user.controller.js`)

- Update `exports.update` validation (Joi) to accept `name_`, `email_`, `phone_`, `address_`, `settings`.
- **Sync Logic** (Backend):
  - **Name Assembly**: If `name_` is present, assemble `name` from:
    - Extract first letter of `middle`, capitalize, add period: `middle.charAt(0).toUpperCase() + '.'`
    - Handle cases: full name (extract first letter), single letter, or single letter + period.
    - If `middle` is empty, add nothing.
    - If `prefix` is empty or "none", add nothing.
    - If `suffix` is empty or "none", add nothing.
    - Format: `${prefix ? prefix + ' ' : ''}${first} ${middleFormatted}${middleFormatted ? ' ' : ''}${last}${suffix ? ', ' + suffix : ''}`.trim()
  - **Address Assembly**: If `address_` is present, assemble `address` from:
    - Format: `${address_.street}, ${address_.city}, ${address_.state} ${address_.zip}`.trim()
    - Handle empty fields gracefully.
  - If `email_.primary` is present, set `email = email_.primary`.
  - If `phone_.primary` is present, set `phone = phone_.primary` (store as E.164 format: `+1XXXXXXXXXX`).

### 3.2 Frontend Changes

#### 3.2.1 UI Pattern: Accordion-Style Expansion

For each `{property}` / `{property}_` pair:

- **Display Field**: Show assembled/formatted `{property}` value (read-only or display-only).
- **Arrow Icon Button**: Click to expand/collapse detail fields below with smooth animation.
- **Detail Fields**: When expanded, show all `{property}_` sub-fields inline below the display field.
- **Real-time Updates**: All display fields update in real-time as user edits detail fields.
- **Clickable Links**: Email display is clickable `mailto:` link, phone display is clickable `tel:` link.

**Example Layout (Name):**

```
┌─────────────────────────────────────────────────────────┐
│ Name: [ Mr. Timothy J. McGuire, Sr. ]           [ > ]   │ ← Arrow button
├─────────────────────────────────────────────────────────┤
│ (Expanded section appears below when arrow clicked)     │
│ Prefix: [ Mr. ▼ ]  First: [ Timothy          ]          │
│ Middle: [ J              ]                              │
│ Last: [ McGuire          ]  Suffix: [ Sr. ▼ ]           │
└─────────────────────────────────────────────────────────┘
```

**Example Layout (Phone):**

```
┌─────────────────────────────────────────────────────────┐
│ Phone: [ +1 (810) 459-7508 ]                    [ > ]   │ ← Display formatted, clickable tel:
├─────────────────────────────────────────────────────────┤
│ (Expanded section)                                      │
│ Primary: [ +18104597508 ]  (stored as E.164)            │
│ Cell: [ +18104597508 ]                                  │
│ Home: [ +15864160611 ]                                  │
│ Work: [ +12484211010 ]                                  │
└─────────────────────────────────────────────────────────┘
```

**Example Layout (Address):**

```
┌─────────────────────────────────────────────────────────┐
│ Address: [ 48882 Beacon Square Dr., Macomb, MI 48044-5918 ] [ > ] │
├─────────────────────────────────────────────────────────┤
│ (Expanded section)                                      │
│ Company: [ MicroCODE, Inc. ]                            │
│ Care Of: [ Cathy McGuire ]                              │
│ Street: [ 48882 Beacon Square Dr. ]                     │
│ City: [ Macomb ]  State: [ MI ]                         │
│ ZIP: [ 48044-5918 ]  Country: [ USA ]                   │
└─────────────────────────────────────────────────────────┘
```

#### 3.2.2 Component Requirements

1. **Accordion/Collapsible Component**:

   - Use `@radix-ui/react-collapsible` (needs to be added to dependencies).
   - Create reusable `Collapsible` component wrapper following existing Radix UI + Shadcn/UI patterns in codebase:
     - Follow pattern from `dialog.jsx` and `tabs.jsx` (use `forwardRef`, `cn()` utility, Tailwind classes).
     - Export: `Collapsible`, `CollapsibleTrigger`, `CollapsibleContent`.
     - Use Radix primitives: `CollapsiblePrimitive.Root`, `CollapsiblePrimitive.Trigger`, `CollapsiblePrimitive.Content`.
     - Include smooth CSS transitions via Tailwind `data-[state=open]:animate-in` and `data-[state=closed]:animate-out` classes.
     - **Custom Consideration**: May need to integrate with `SettingRow` component for iOS-style settings layout. Discuss integration approach before implementation.

2. **Form Structure** (`client/src/views/account/users.jsx`):

   - Use **iOS-style Settings Layout** (`settingsMode=true` in `Form`).
   - Implement collapsible sections for:
     - **Identity**: `name` (Display) + Collapsible → `prefix`, `first`, `middle`, `last`, `suffix`.
     - **Contact**:
       - `email` (Display - shows `primary` only, clickable `mailto:`) + Collapsible → `primary`, `personal`, `work`.
       - `phone` (Display - formatted as `+1 (AAA) ZZZ-NNNN` for US, E.164 for international, clickable `tel:`) + Collapsible → `primary`, `cell`, `home`, `work`.
     - **Address**: Summary (Display: `${street}, ${city}, ${state} ${zip}`) + Collapsible → `company`, `c_o`, `street`, `city`, `state`, `zip`, `country`.
     - **Settings**: JSON Editor (Textarea).

3. **Dropdown Fields**:

   - `name_.prefix`: Dropdown with options from locales (empty string = "none", Mr., Mrs., Ms., Dr., etc.).
   - `name_.suffix`: Dropdown with options from locales (empty string = "none", Sr., Jr., II, III, etc.).

4. **Real-time Assembly & Formatting**:

   - **Name**: As user edits `name_` fields, update display `name` field in real-time (client-side).
     - Middle field: Accept full name, single letter, or single letter + period. Extract first letter for display.
   - **Phone**: Format phone numbers for display:
     - US numbers (`+1`): Format as `+1 (AAA) ZZZ-NNNN`.
     - International numbers: Display as E.164 format (e.g., `+441234567890`).
     - Store/transmit as E.164 format (`+1XXXXXXXXXX`).
   - **Email**: Display shows `primary` value only, updates in real-time. Clickable `mailto:` link.
   - **Address**: Display summary updates in real-time: `${street}, ${city}, ${state} ${zip}`.
   - On form submit, backend will re-assemble and validate.

5. **Phone Formatting Utility**:

   - Create/use utility function to format E.164 phone numbers:
     - US numbers (`+1XXXXXXXXXX`): Format as `+1 (AAA) ZZZ-NNNN`.
     - International numbers: Display as-is in E.164 format.

#### 3.2.3 Locales Structure

Add to `client/src/locales/en/account/en_users.json`:

```json
{
  "edit": {
    "form": {
      "name": { "label": "Name" },
      "name_": {
        "prefix": {
          "label": "Prefix",
          "options": ["", "Mr.", "Mrs.", "Ms.", "Miss", "Dr.", "Prof."],
          "none": ""
        },
        "first": { "label": "First Name" },
        "middle": { "label": "Middle Name" },
        "last": { "label": "Last Name" },
        "suffix": {
          "label": "Suffix",
          "options": ["", "Sr.", "Jr.", "II", "III", "IV", "Esq."],
          "none": ""
        }
      },
      "email": { "label": "Email" },
      "email_": {
        "primary": { "label": "Primary Email" },
        "personal": { "label": "Personal Email" },
        "work": { "label": "Work Email" }
      },
      "phone": { "label": "Phone" },
      "phone_": {
        "primary": { "label": "Primary Phone" },
        "cell": { "label": "Cell Phone" },
        "home": { "label": "Home Phone" },
        "work": { "label": "Work Phone" }
      },
      "address": { "label": "Address" },
      "address_": {
        "company": { "label": "Company" },
        "c_o": { "label": "Care Of" },
        "street": { "label": "Street Address" },
        "city": { "label": "City" },
        "state": { "label": "State/Province" },
        "zip": { "label": "ZIP/Postal Code" },
        "country": { "label": "Country" }
      }
    }
  }
}
```

## 4. Implementation Plan

### Step 1: Schema Migration - MongoDB

- **File**: `server/model/mongo/user.mongo.js` (SOURCE FILE)
- **Action**:
  - Change `phone` type from `Object` to `String`.
  - Add `phone_` as `Object`.
  - **Rename `address` to `address_`** (keep as Object).
  - **Add new `address` field** as `String`.
- **Note**: After changes, run `server/bin/setup.js` to regenerate `server/model/user.model.js` if needed, or it will be regenerated on next setup.

### Step 2: Schema Update - SQL

- **File**: `server/model/sql/user.sql.js` (SOURCE FILE)
- **Database**: PostgreSQL (assumed)
- **Action**:
  - **NO MIGRATION FILE NEEDED** - Clean slate approach, schema handled in model.
  - Update `create` function to handle new fields (`name_`, `email_`, `phone`, `phone_`, `address`, `address_`).
    - PostgreSQL JSONB: Can insert JavaScript objects directly - Knex/PostgreSQL handles serialization.
    - For explicit control: Use `JSON.stringify()` when building data object, or pass objects directly.
  - Update `get` function to:
    - Add new columns to explicit column list: `'name_', 'email_', 'phone', 'phone_', 'address', 'address_'`
    - PostgreSQL JSONB: Returns as JavaScript objects automatically, no `JSON.parse()` needed.
  - Update `update` function to handle new fields.
    - PostgreSQL JSONB: Can update with JavaScript objects directly, or use `JSON.stringify()` for explicit control.

### Step 3: Backend Controller

- **File**: `server/controller/user.controller.js`
- **Action**:
  - Update Joi validation schema to accept nested objects (`name_`, `email_`, `phone_`, `address_`).
  - Implement name assembly logic:
    - Extract first letter of `middle` (handle full name, single letter, or single letter + period).
    - Capitalize, add period: `middle.charAt(0).toUpperCase() + '.'`
    - Handle empty `middle`, empty/`"none"` prefix/suffix.
    - Format: `${prefix ? prefix + ' ' : ''}${first} ${middleFormatted}${middleFormatted ? ' ' : ''}${last}${suffix ? ', ' + suffix : ''}`.trim()
  - Implement address assembly logic:
    - Format: `${address_.street}, ${address_.city}, ${address_.state} ${address_.zip}`.trim()
    - Handle empty fields gracefully.
  - Sync `email` from `email_.primary` if present.
  - Sync `phone` from `phone_.primary` if present (store as E.164).

### Step 4: Frontend Dependencies

- **File**: `client/package.json`
- **Action**: Add `@radix-ui/react-collapsible` dependency.

### Step 5: Frontend Components

- **File**: `client/src/components/collapsible/collapsible.jsx` (NEW)
- **Action**: Create Collapsible component wrapper using `@radix-ui/react-collapsible`, following Radix UI + Shadcn/UI patterns:
  - Pattern: Follow `dialog.jsx` and `tabs.jsx` structure (use `forwardRef`, `cn()` utility, Tailwind classes).
  - Export: `Collapsible`, `CollapsibleTrigger`, `CollapsibleContent`.
  - Use Radix primitives: `CollapsiblePrimitive.Root`, `CollapsiblePrimitive.Trigger`, `CollapsiblePrimitive.Content`.
  - Include smooth animations via Tailwind `data-[state=open]:animate-in` and `data-[state=closed]:animate-out` classes.
  - **Integration Consideration**: May need custom wrapper to integrate with `SettingRow` for iOS-style layout. Discuss approach before implementation.

### Step 6: Phone Formatting Utility

- **File**: `client/src/utils/phone.js` (NEW) or add to existing utils
- **Action**: Create utility function to format E.164 phone numbers:
  - US numbers (`+1XXXXXXXXXX`): Format as `+1 (AAA) ZZZ-NNNN`.
  - International numbers: Display as-is in E.164 format.

### Step 7: Frontend UI - Client

- **File**: `client/src/views/account/users.jsx`
- **Action**:
  - Refactor `editUser` function to use `settingsMode=true`.
  - Implement collapsible sections for `name_`, `email_`, `phone_`, `address_`.
  - Add real-time name assembly on client-side (extract first letter of middle, handle various formats).
  - Add real-time phone formatting for display (US vs international).
  - Add real-time address assembly for display.
  - Make email display clickable `mailto:` link.
  - Make phone display clickable `tel:` link.
  - Integrate dropdowns for `prefix` and `suffix` (empty string = "none").
  - Ensure all display fields update in real-time.

### Step 8: Frontend Locales

- **File**: `client/src/locales/en/account/en_users.json`
- **Action**: Add all new field labels and dropdown options (including empty string for "none").
- **File**: `client/src/locales/es/account/es_users.json` (if exists)
- **Action**: Add Spanish translations.

### Step 9: Admin Console (if user editing exists)

- **Files**: Deep search `admin/console/src/**/*user*.jsx`, `admin/console/src/**/*users*.jsx`
- **Action**: Apply same UI patterns **only** where user editing/display currently exists. Do not add new capabilities.

### Step 10: App Directory (if user editing exists)

- **Files**: Deep search `app/**/*user*.jsx`, `app/**/*users*.jsx`
- **Action**: Apply same UI patterns **only** where user editing/display currently exists. Do not add new capabilities.

## 5. Files to Modify/Create

### Backend (SOURCE FILES):

- `server/model/mongo/user.mongo.js` (MODIFY - rename address to address*, add address String, change phone to String, add phone* Object)
- `server/migrations/YYYYMMDDHHMMSS_user_expand_fields.js` (CREATE NEW - SQL migration)
- `server/model/sql/user.sql.js` (MODIFY - update SQL schema handling for new fields)
- `server/controller/user.controller.js` (MODIFY)

### Frontend:

- `client/package.json` (MODIFY - add dependency)
- `client/src/components/collapsible/collapsible.jsx` (CREATE)
- `client/src/components/lib.jsx` (MODIFY - export Collapsible)
- `client/src/utils/phone.js` (CREATE - phone formatting utility)
- `client/src/views/account/users.jsx` (MODIFY)
- `client/src/locales/en/account/en_users.json` (MODIFY)
- `client/src/locales/es/account/es_users.json` (MODIFY - if exists)

### Admin Console (if applicable):

- `admin/console/src/views/**/users.jsx` (MODIFY - if exists and has user editing)

### App Directory (if applicable):

- `app/**/users.jsx` (MODIFY - if exists and has user editing)

## 6. Confidence Score

**98%**

## 6.1 Custom Implementation Considerations

### Collapsible + SettingRow Integration

The `Form` component with `settingsMode=true` uses `SettingRow` for iOS-style horizontal layout. To integrate collapsible sections, we have two options:

**Option A: Custom CollapsibleSettingRow Component**

- Create a wrapper component that combines `Collapsible` + `SettingRow`.
- Display field and arrow button in the main `SettingRow`.
- Expanded content uses nested `SettingRow` components for detail fields.
- **Pros**: Clean separation, reusable.
- **Cons**: Requires new component creation.

**Option B: Inline Collapsible in Form (Custom JSX)**

- Modify `users.jsx` to render custom JSX (bypassing Form's `inputs` object) for collapsible sections.
- Use `Collapsible` directly with `SettingRow` components inside.
- Mix custom JSX with Form's `inputs` object for non-collapsible fields.
- **Pros**: More flexible, direct control.
- **Cons**: Bypasses Form's input mapping system, less consistent.

**Option C: Extend Form Component**

- Modify `Form` component to support a special input type (e.g., `type: 'collapsible'`) that renders collapsible sections.
- **Pros**: Fully integrated with Form system.
- **Cons**: Requires modifying core Form component, more complex.

**Recommendation**: Option A - Create `CollapsibleSettingRow` component that wraps `Collapsible` and integrates with `SettingRow` pattern. This maintains consistency with existing form patterns while adding collapsible functionality.

**✅ CONFIRMED**: Option A selected.

**Implementation Approach for Option A**:

- Create `CollapsibleSettingRow` component that:
  - Accepts display field config (label, value, type, etc.)
  - Accepts detail fields config (array of field configs for `_` fields)
  - Accepts `control` prop from `react-hook-form` for form integration
  - Renders display field + arrow button in main `SettingRow` (horizontal layout)
  - When expanded, renders detail fields in nested `SettingRow` components
  - Uses `Controller` from `react-hook-form` for all fields
  - Handles real-time display field updates based on detail field changes
- In `users.jsx`, use `CollapsibleSettingRow` for collapsible fields (name/name*, email/email*, phone/phone*, address/address*)
- Use Form's `inputs` object for non-collapsible fields (id, permission, etc.)
- Mix both approaches: Form handles non-collapsible, CollapsibleSettingRow handles collapsible sections

## 7. Final Questions & Assumptions

### All Questions Answered:

1. ✅ **Phone Formatting Locale**: US format (`+1 (AAA) ZZZ-NNNN`) for `+1` numbers, E.164 for international.
2. ✅ **Middle Name Input**: Accept full names, single letter, or single letter + period. Extract first letter for display.
3. ✅ **Email Display Field Clickable**: Yes, clickable `mailto:` link.
4. ✅ **Phone Display Field Clickable**: Yes, clickable `tel:` link.
5. ✅ **Address Schema**: Renamed `address` → `address_`, added new `address` String field for display.
6. ✅ **Model File Structure**: Understand that `server/model/mongo/` and `server/model/sql/` are source files, `server/model/` root contains generated copies.
7. ✅ **SQL Database**: PostgreSQL assumed.
8. ✅ **SQL Column Types**: Use JSONB type for JSON fields (`name_`, `email_`, `phone_`, `address_`).
9. ✅ **Migrations**: NO MIGRATIONS - Clean slate approach.
10. ✅ **SQL get function**: Add new columns explicitly to the column list.
11. ✅ **UI Framework**: Follow Radix UI + Shadcn/UI patterns (use `forwardRef`, `cn()`, Tailwind classes).
12. ✅ **Collapsible Integration**: Option A confirmed - Create `CollapsibleSettingRow` component.

### Confirmed Assumptions:

- Name assembly: First letter of middle (capitalized + period), handles full name/single letter/single letter+period, empty middle adds nothing, "none" prefix/suffix adds nothing.
- Prefix/Suffix: i18n style (locales-based).
- Admin/App: Deep search, only implement where capabilities exist (do not add new).
- Animation: Smooth expand/collapse transitions.
- Address: Display as `${street}, ${city}, ${state} ${zip}`, assembled from `address_`.
- Email: Show primary only in display field, clickable `mailto:` link.
- Phone: Show primary formatted as `+1 (AAA) ZZZ-NNNN` (US) or E.164 (international) in display, store as E.164, clickable `tel:` link.
- Real-time: All display fields update in real-time as user edits.
- Collapsible default state: Closed by default (cleaner initial UI).
- Icon: Use `chevron-down`/`chevron-up` icons (consistent with existing Select component pattern).
- Validation errors: Display inline with each field via SettingRow's `error` prop (already supported).
- Empty state: Display fields show empty string when `_` fields are empty; clickable links (email/phone) only active when values exist.
- Schema consistency: All `{property}` / `{property}_` pairs follow same pattern (name/name*, email/email*, phone/phone*, address/address*).
- Model files: Source files are in `server/model/mongo/` and `server/model/sql/`, not `server/model/` root.
- Collapsible default state: Closed by default (cleaner initial UI).
- Icon: Use `chevron-down`/`chevron-up` icons (consistent with existing Select component pattern).
- Validation errors: Display inline with each field via SettingRow's `error` prop (already supported).
- Empty state: Display fields show empty string when `_` fields are empty; clickable links (email/phone) only active when values exist.

## 8. Final Review & Confidence Assessment

### Confidence Score: **98%**

### Remaining Minor Implementation Details (Non-Blocking):

1. **Collapsible Icon Position**: Arrow icon should be positioned on the right side of the display field (consistent with iOS settings pattern). Use `chevron-down` when closed, `chevron-up` when open.

2. **Form Integration Pattern**: `CollapsibleSettingRow` will be rendered as custom JSX in `users.jsx` alongside Form's `inputs` object. Both will share the same `react-hook-form` context (`control` prop passed to CollapsibleSettingRow).

3. **Display Field Read-Only**: Display fields (`name`, `email`, `phone`, `address`) should be read-only in the UI - they are auto-generated from `_` fields. They can be styled as read-only inputs or as plain text with clickable links (for email/phone).

4. **Phone Number Parsing**: Phone number formatting utility needs to handle:

   - E.164 format parsing (`+1XXXXXXXXXX`)
   - US number detection (country code `+1`)
   - Formatting to `+1 (AAA) ZZZ-NNNN` for US
   - Preserving E.164 format for international
   - Handling invalid/malformed numbers gracefully

5. **Name Assembly Edge Cases**:

   - Empty `first` or `last`: Should still assemble what's available
   - All fields empty: Display field should be empty string
   - "none" string vs empty string: Treat both as empty for prefix/suffix

6. **Address Assembly**: Only include non-empty fields in display. If `street` is empty, start with `city`. Handle missing commas/spacing gracefully.

7. **Email/Phone Primary Field**: If `email_.primary` or `phone_.primary` is empty, display field should be empty (not show other email/phone values).

8. **SQL Column Names**: PostgreSQL column names will match field names exactly (`name_`, `email_`, `phone_`, `address_`). JSONB columns will store JSON objects.

9. **Backend Sync Logic**: Controller's `update` function should:

   - Always re-assemble `name`, `email`, `phone`, `address` from `_` fields on every update
   - Handle missing `_` fields gracefully (don't error, just set display field to empty)
   - For MongoDB: Detect old format (if `phone` is Object, migrate to new format)
   - For SQL: Knex handles JSONB serialization/deserialization automatically

10. **Locale Keys**: All new UI labels and dropdown options should follow existing locale structure:
    - `client/src/locales/en/account/en_users.json` for user management labels
    - Prefix/suffix options should be arrays in locale files
    - Error messages should use existing error message patterns

### Implementation Readiness:

✅ **Schema Changes**: Fully defined for both MongoDB and SQL
✅ **Backend Logic**: Clear sync logic defined
✅ **Frontend Components**: Component structure and integration approach confirmed
✅ **UI Patterns**: Radix UI + Shadcn/UI patterns understood
✅ **Form Integration**: Approach confirmed (Option A)
✅ **Data Flow**: Real-time updates and backend sync understood
✅ **Edge Cases**: Most edge cases identified and addressed

### Ready for Implementation:

The plan is **98% complete** and ready for implementation. The remaining 2% represents minor implementation details that will be resolved during coding (icon positioning, exact styling, edge case handling in utility functions). All major architectural decisions have been made and confirmed.
