# AIN - FEATURE - EXPAND_USER_PROFILE

## Metadata

- **Type**: FEATURE
- **Issue #**: [if applicable]
- **Created**: [2025-12-17]
- **Status**: READY FOR IMPLEMENTATION

---

## C: CONCEPT/CHANGE/CORRECTION - Discuss ideas without generating code

<!-- Initial concept discussion would go here - not present in phase-specific files -->

---

## D: DESIGN - Design detailed solution

# Issue: Expand Edit Your Profile Dialog with Accordion Fields and Organizations/Clubs/SVs Management

## 1. Problem Description

The current "Edit Your Profile" dialog (`client/src/views/account/profile.jsx`) uses simple text input fields for user information. We need to:

1. **Convert text inputs to read-only display fields with accordion behavior** - Similar to the "Edit User" dialog, convert `name`, `email`, `phone`, and `address` fields to read-only display fields that expand into detail fields (`name_`, `email_`, `phone_`, `address_`).

2. **Add clear separation between user info and account info** - Currently, user fields (name, email) and account fields (account_name) are intermingled. We need a clear visual separation.

3. **Add Organizations management** - Display and edit the list of organizations (`org_ids`) the user belongs to.

4. **Add Clubs management** - Display and edit the list of clubs (`club_ids`) the user belongs to.

5. **Add SVs (Boats) management** - Display and edit the list of SVs (`sv_ids`) the user owns. Note: An "SV" is a Boat + Owners + Handicapping Certs.

6. **Structure Organizations, Clubs, and SVs like Notifications** - Each should be a separate page/section with its own menu item, help card, and Card/Row/Form layout pattern.

7. **Add Theme Settings page** - Create a new Theme Settings page for managing user theme preferences (dark mode, base color, language) with immediate update functionality.

## 2. Current State Analysis

### 2.1 Current Profile Page Structure

**File**: `client/src/views/account/profile.jsx`

- Uses simple `Form` component with `inputs` object
- Fields: `name` (text), `email` (email), `avatar` (file), `account_name` (text, owner only), `default_account_id` (select, multi-account only)
- No accordion behavior
- No separation between user and account fields
- No Organizations, Clubs, or SVs management

### 2.2 Edit User Dialog Reference (Completed)

**File**: `client/src/views/account/user-edit-form.jsx`

- Uses `CollapsibleSettingRow` component for accordion behavior
- Implements collapsible sections for `name/name_`, `email/email_`, `phone/phone_`, `address/address_`
- Uses `react-hook-form` with `Controller` for form management
- Real-time display field updates based on detail field changes
- Uses `CollapsibleSettingRow` component from `client/src/components/form/collapsible-setting-row/collapsible-setting-row.jsx`

### 2.3 Notifications Pattern Reference

**File**: `client/src/views/account/notifications.jsx`

- Uses `Card`, `Row`, `Form` layout pattern
- Uses `settingsMode={true}` for iOS-style settings layout
- Has its own route: `/account/notifications`
- Has its own menu item in account subnav
- Dynamically builds form inputs from settings structure
- Uses custom `handleSubmit` callback (no URL, callback-only mode)

### 2.4 User Schema

**File**: `server/model/mongo/user.mongo.js`

- `name` (String) - Display/assembled field
- `name_` (Object) - `{ prefix, first, middle, last, suffix }`
- `email` (String) - Primary login email
- `email_` (Object) - `{ primary, personal, work }`
- `phone` (String) - Display/assembled field
- `phone_` (Object) - `{ primary, cell, home, work }`
- `address` (String) - Display/assembled field
- `address_` (Object) - `{ company, c_o, street, unit, city, state, zip, country }`
- `org_ids` (Array) - Array of organization IDs
- `club_ids` (Array) - Array of club IDs
- `sv_ids` (Array) - Array of SV (boat) IDs
- `avatar` (String) - Profile picture URL
- `account` (Array) - Array of account objects
- `default_account_id` (String) - Default account ID

### 2.5 Related Models

**Organizations**: `server/model/mongo/org.mongo.js`

- Has `id`, `key`, `name`, `description`, etc.

**Clubs**: `server/model/mongo/club.mongo.js`

- Has `id`, `key`, `name`, `description`, etc.

**SVs**: `server/model/mongo/sv.mongo.js`

- Has `id`, `key`, `boat_key`, `boat_id`, `name`, `display`, `description`, `user_keys`, `user_ids`, `certs`, etc.

### 2.6 Current Routes and Navigation

**File**: `client/src/routes/account.js`

- `/account/profile` - Profile page
- `/account/notifications` - Notifications page

**File**: `client/src/components/layout/account/account.jsx`

- Subnav includes: Profile, Password, TFA, Billing, Notifications, API Keys, Users
- Uses `permission` field to control visibility

**File**: `client/src/views/account/index.jsx`

- Grid of cards linking to account pages
- Includes Profile and Notifications cards

## 3. Proposed Solution

### 3.1 Profile Page Refactoring

#### 3.1.1 Convert to Accordion Pattern

**File**: `client/src/views/account/profile.jsx`

**Changes**:

1. Replace `Form` component's `inputs` object with custom form using `react-hook-form`
2. Use `CollapsibleSettingRow` for `name`, `email`, `phone`, `address` fields
3. Keep `avatar` and account fields (`account_name`, `default_account_id`) as regular form fields
4. Add visual separation between user info section and account info section

#### 3.1.2 Form Implementation Details

- Use `useForm` from `react-hook-form` (similar to `UserEditForm`)
- Parse `name_`, `email_`, `phone_`, `address_` from existing user data (same logic as `UserEditForm`)
- Handle form submission: PATCH `/api/user` with assembled data
- Update `authContext` on successful save (name, avatar, account_name)

#### 3.1.3 Display Field Styling

- **Label Position**: Label appears **above** the display field (not to the left), allowing display field to be full width
- **Display Field**: Full width read-only display fields with `bg-slate-200/75 dark:bg-black/30` background
- **Clickable to Expand**: Clicking on display field (or anywhere in the row) expands accordion
- **Chevron Icon**: Chevron icon (chevron-down/chevron-up) on right side
- **Email/Phone Click Behavior**: Clicking email or phone display fields opens the accordion (does NOT trigger `mailto:` or `tel:` links) - user is editing their own profile, not contacting themselves
- **Detail Fields**: Detail fields (`name_`, `email_`, `phone_`, `address_`) use exact same styling as "Edit User" dialog (horizontal layout with label on left)

### 3.2 Organizations Management Page

#### 3.2.1 New File Structure

**File**: `client/src/views/account/organizations.jsx` (CREATE)

**Pattern**: Follow `notifications.jsx` structure

**Features**:

- Display list of organizations user belongs to (from `user.org_ids`)
- Add/remove organizations
- Search/filter organizations
- Display organization details (name, description, etc.)

#### 3.2.2 API Integration

- **GET** `/api/user` - Get current user with `org_ids`
- **GET** `/api/org` - Get available organizations (for adding)
- **PATCH** `/api/user` - Update `org_ids` array

#### 3.2.3 Form Fields

- Display list of organizations user belongs to (from `user.org_ids`)
- **Add Button**: "+" button presents a dropdown/pull-down menu of available organizations from database
- User selects organization from dropdown to add it to their list
- Remove button/action for each organization in the list
- Same UI/UX pattern applies to Clubs and SVs pages

### 3.3 Clubs Management Page

#### 3.3.1 New File Structure

**File**: `client/src/views/account/clubs.jsx` (CREATE)

**Pattern**: Same as Organizations, but for clubs

**Features**:

- Display list of clubs user belongs to (from `user.club_ids`)
- Add/remove clubs
- Search/filter clubs
- Display club details

#### 3.3.2 API Integration

- **GET** `/api/user` - Get current user with `club_ids`
- **GET** `/api/club` - Get available clubs (for adding)
- **PATCH** `/api/user` - Update `club_ids` array

### 3.4 SVs (Boats) Management Page

#### 3.4.1 New File Structure

**File**: `client/src/views/account/svs.jsx` (CREATE)

**Pattern**: Same as Organizations/Clubs, but for SVs

**Features**:

- Display list of SVs user owns (from `user.sv_ids`)
- Add/remove SVs via "+" button with dropdown of available boats
- Display SV details (boat name, owners, handicapping certs)

**Important Entity Relationship**:

- **BOAT**: The physical boat entity (defined once, persists over time)
- **SV (Sailing Vessel)**: A relationship entity linking USER (Owner) + BOAT + CERT(s)
- **USER claims ownership of a BOAT**: When user selects a boat, an SV is created referencing:
  - The USER (as Owner)
  - The BOAT (the physical boat)
- **SV requires CERT(s)**: Each SV needs handicapping certificates added to complete its definition
- **BOAT can have multiple Owners**: Multiple users can own the same boat (each creates their own SV)
- **BOAT can change ownership**: Over time, boat ownership changes, but the BOAT definition remains the same
- **CERT changes with ownership**: When ownership changes, the SV's CERT(s) would also change

**Workflow**:

1. User clicks "+" button
2. Dropdown shows available BOATs from database
3. User selects a BOAT
4. System creates an SV linking USER + BOAT
5. SV is added to `user.sv_ids` array
6. User can later add CERT(s) to the SV (future enhancement)

#### 3.4.2 API Integration

- **GET** `/api/user` - Get current user with `sv_ids`
- **GET** `/api/sv` - Get available SVs (for adding)
- **PATCH** `/api/user` - Update `sv_ids` array

### 3.5 Routes and Navigation Updates

#### 3.5.1 Routes

**File**: `client/src/routes/account.js` (MODIFY)

**Add**:

```javascript
{
    path: '/account/organizations',
    view: Organizations,
    layout: 'account',
    permission: 'user',
    title: 'account.index.title'
},
{
    path: '/account/clubs',
    view: Clubs,
    layout: 'account',
    permission: 'user',
    title: 'account.index.title'
},
{
    path: '/account/svs',
    view: SVs,
    layout: 'account',
    permission: 'user',
    title: 'account.index.title'
}
```

#### 3.5.2 Navigation

**File**: `client/src/components/layout/account/account.jsx` (MODIFY)

**Add to `subnav` array**:

```javascript
{
    label: t('account.nav.organizations'),
    link: '/account/organizations',
    icon: 'building',
    permission: 'user'
},
{
    label: t('account.nav.clubs'),
    link: '/account/clubs',
    icon: 'users',
    permission: 'user'
},
{
    label: t('account.nav.svs'),
    link: '/account/svs',
    icon: 'ship',
    permission: 'user'
}
```

#### 3.5.3 Account Index Cards

**File**: `client/src/views/account/index.jsx` (MODIFY)

**Add cards** (similar to Notifications card):

```jsx
<Card>
    <Icon name='building' size={iconSize}/>
    <h1>{t('account.organizations.title')}</h1>
    <span>{t('account.organizations.description')}</span>
    <Button
        size='xs'
        variant='outline'
        url='/account/organizations'
        text={t('account.organizations.button')}
    />
</Card>

<Card>
    <Icon name='users' size={iconSize}/>
    <h1>{t('account.clubs.title')}</h1>
    <span>{t('account.clubs.description')}</span>
    <Button
        size='xs'
        variant='outline'
        url='/account/clubs'
        text={t('account.clubs.button')}
    />
</Card>

<Card>
    <Icon name='ship' size={iconSize}/>
    <h1>{t('account.svs.title')}</h1>
    <span>{t('account.svs.description')}</span>
    <Button
        size='xs'
        variant='outline'
        url='/account/svs'
        text={t('account.svs.button')}
    />
</Card>
```

### 3.6 Backend Controller Updates

#### 3.6.1 User Controller

**File**: `server/controller/user.controller.js` (MODIFY)

**Update `exports.update` validation**:

- Already accepts `org_ids`, `club_ids`, `sv_ids` (verify)
- Ensure validation allows array of IDs
- Ensure update logic handles these arrays correctly

**Verify**:

- Joi validation schema includes these fields
- Update logic preserves existing arrays if not provided
- Update logic replaces arrays if provided

### 3.7 Theme Settings Page

#### 3.7.1 New File Structure

**File**: `client/src/views/account/theme.jsx` (CREATE)

**Pattern**: Follow `notifications.jsx` structure

**Features**:

- Sight Control section:
  - Dark Mode switch (immediate update)
  - Base Color select (Black, Purple, Blue) (immediate update)
  - Language select (from `/api/locale`, filtered to en/es) (immediate update)
- Sound Control section:
  - Mute All switch
  - Sound Effects switch
  - Voice Assistance switch
- Uses `settingsMode={true}` for iOS-style settings layout
- All theme changes apply immediately (darkMode, baseColor, language)

#### 3.7.2 Immediate Update Pattern

**For ALL theme changes (darkMode, baseColor, language)**:

1. Update `authContext` immediately
2. Apply to DOM immediately (for darkMode: add/remove 'dark' class)
3. Save to backend via API
4. Revert on error if save fails

**Dark Mode Implementation**:

```javascript
// Update authContext immediately
authContext.update({ settings: updatedSettings });

// Update DOM immediately
newDarkMode
  ? document.getElementById("app").classList.add("dark")
  : document.getElementById("app").classList.remove("dark");

// Persist to backend
await setSettingValue("theme.sight.darkMode", newDarkMode);
```

**Language Implementation**:

```javascript
// Update authContext immediately
authContext.update({ settings: updatedSettings });

// Update i18n immediately
Axios.defaults.headers.common["Accept-Language"] = newLanguage;
i18n.changeLanguage(newLanguage);

// Persist to backend
await setSettingValue("theme.sight.language", newLanguage);
```

**Base Color Implementation**:

```javascript
// Update authContext immediately
authContext.update({ settings: updatedSettings });

// Apply CSS variable or class immediately (if applicable)

// Persist to backend
await setSettingValue("theme.sight.baseColor", newBaseColor);
```

#### 3.7.3 Locale API Integration

**Endpoint**: `GET /api/locale`
**Auth**: Requires `auth.verify('user')`
**Response Format**:

```javascript
{
  data: [
    {
      id: string,
      key: string,        // e.g., 'en-US', 'es-ES'
      name: string,       // e.g., 'English (United States)'
      country: string,    // e.g., 'US', 'ES'
      language: string,   // e.g., 'en', 'es'
      flag: string        // SVG string
    },
    ...
  ]
}
```

**Implementation Notes**:

- Filter response to only locales where `language === 'en'` or `language === 'es'`
- For select dropdown, use `language` field as value (since settings store `language: "en"`)
- Display: Show simplified names like "English" and "Spanish"
- Value: Use `'en'` and `'es'` (language codes, not full locale keys)

#### 3.7.4 Default Values

From `server/config/default.json`:

```javascript
{
  theme: {
    sight: {
      darkMode: true,      // Default: true
      baseColor: "purple", // Default: purple
      language: "en"       // Default: en
    },
    sound: {
      muteAll: true,        // Default: true
      useEffects: false,   // Default: false
      useAssist: false     // Default: false
    }
  }
}
```

#### 3.7.5 Form Input Configuration

**Base Color Select**:

```javascript
baseColor: {
  type: 'select',
  options: [
    { value: 'black', label: 'Black' },
    { value: 'purple', label: 'Purple' },
    { value: 'blue', label: 'Blue' }
  ],
  defaultValue: sightSettings?.baseColor ?? 'purple'
}
```

**Language Select**:

```javascript
language: {
  type: 'select',
  options: localesFromAPI
    .filter(locale => locale.language === 'en' || locale.language === 'es')
    .map(locale => ({
      value: locale.language,  // Use 'en' or 'es', not full key
      label: locale.name       // Or simplified: 'English' / 'Spanish'
    })),
  defaultValue: sightSettings?.language ?? 'en'
}
```

#### 3.7.6 Routes and Navigation

**File**: `client/src/routes/account.js` (MODIFY)

**Add**:

```javascript
{
    path: '/account/theme',
    view: Theme,
    layout: 'account',
    permission: 'user',
    title: 'account.index.title'
}
```

**File**: `client/src/components/layout/account/account.jsx` (MODIFY)

**Add to `subnav` array** (between Billing and Notifications):

```javascript
{
    label: t('account.nav.theme'),
    link: '/account/theme',
    icon: 'palette',
    permission: 'user'
}
```

**File**: `client/src/views/account/index.jsx` (MODIFY)

**Add card**:

```jsx
<Card>
  <Icon name="palette" size={iconSize} />
  <h1>{t("account.theme.title")}</h1>
  <span>{t("account.theme.description")}</span>
  <Button
    size="xs"
    variant="outline"
    url="/account/theme"
    text={t("account.theme.button")}
  />
</Card>
```

#### 3.7.7 Locales for Theme

**File**: `client/src/locales/en/account/en_theme.json` (CREATE)

```json
{
  "title": "Theme",
  "description": "Customize your app appearance and language",
  "button": "Manage Theme",
  "subtitle": "Theme Settings",
  "sections": {
    "sight": "Sight Control",
    "sound": "Sound Control"
  },
  "form": {
    "darkMode": {
      "label": "Dark Mode"
    },
    "baseColor": {
      "label": "Base Color"
    },
    "language": {
      "label": "Language"
    },
    "muteAll": {
      "label": "Mute All"
    },
    "useEffects": {
      "label": "Sound Effects"
    },
    "useAssist": {
      "label": "Voice Assistance"
    },
    "button": "Save Changes"
  },
  "nav": {
    "theme": "Theme"
  }
}
```

**File**: `client/src/locales/es/account/es_theme.json` (CREATE)

Similar structure with Spanish translations.

**File**: `client/src/locales/en/account/en_nav.json` (MODIFY)

**Add** (if not exists):

```json
{
  "account": {
    "nav": {
      "theme": "Theme"
    }
  }
}
```

### 3.8 Locales Updates

#### 3.7.1 Profile Locales

**File**: `client/src/locales/en/account/en_profile.json` (MODIFY)

**Add**:

```json
{
  "sections": {
    "user_info": "Your Information",
    "account_info": "Account Information"
  },
  "form": {
    "name": {
      "label": "Your Name",
      "error": "Name is required"
    },
    "email": {
      "label": "Email address",
      "error": "Email is required"
    },
    "phone": {
      "label": "Phone"
    },
    "address": {
      "label": "Address",
      "error": "Address is required"
    },
    "avatar": {
      "label": "Profile Picture"
    },
    "account_name": {
      "label": "Account Name"
    },
    "default_account_id": {
      "label": "Default Account"
    },
    "button": "Save"
  }
}
```

**Note**: Reuse existing locale keys from `en_users.json` for `name_`, `email_`, `phone_`, `address_` detail fields.

#### 3.7.2 Organizations Locales

**File**: `client/src/locales/en/account/en_organizations.json` (CREATE)

```json
{
  "title": "Organizations",
  "description": "Manage the organizations you belong to",
  "button": "Manage Organizations",
  "subtitle": "Your Organizations",
  "help": {
    "description": "Organizations represent groups or companies you are associated with. Add organizations to connect with other members."
  },
  "form": {
    "button": "Save Changes"
  },
  "nav": {
    "organizations": "Organizations"
  }
}
```

#### 3.7.3 Clubs Locales

**File**: `client/src/locales/en/account/en_clubs.json` (CREATE)

```json
{
  "title": "Clubs",
  "description": "Manage the clubs you belong to",
  "button": "Manage Clubs",
  "subtitle": "Your Clubs",
  "help": {
    "description": "Clubs represent sailing or boating organizations you are a member of. Add clubs to connect with other members."
  },
  "form": {
    "button": "Save Changes"
  },
  "nav": {
    "clubs": "Clubs"
  }
}
```

#### 3.7.4 SVs Locales

**File**: `client/src/locales/en/account/en_svs.json` (CREATE)

```json
{
  "title": "SVs (Boats)",
  "description": "Manage the boats you own",
  "button": "Manage Boats",
  "subtitle": "Your Boats",
  "help": {
    "description": "SVs (Sailing Vessels) represent boats you own, including boat details, owners, and handicapping certificates."
  },
  "form": {
    "button": "Save Changes"
  },
  "nav": {
    "svs": "SVs (Boats)"
  }
}
```

#### 3.7.5 Navigation Locales

**File**: `client/src/locales/en/en_nav.json` (MODIFY)

**Add** (if not exists):

```json
{
  "account": {
    "nav": {
      "organizations": "Organizations",
      "clubs": "Clubs",
      "svs": "SVs (Boats)"
    }
  }
}
```

## 4. Test Cases

### 4.1 Profile Page Tests

**Test Case 1: Accordion Expansion**

- **Given**: User is on Profile page
- **When**: User clicks on Name display field
- **Then**: Name accordion expands showing prefix, first, middle, last, suffix fields
- **And**: Other accordions (email, phone, address) remain closed

**Test Case 2: Real-time Display Updates**

- **Given**: Name accordion is expanded
- **When**: User types in First Name field
- **Then**: Display field updates in real-time showing assembled name

**Test Case 3: Form Submission**

- **Given**: User has edited name, email, phone, address fields
- **When**: User clicks Save button
- **Then**: Form submits PATCH `/api/user` with assembled data
- **And**: `authContext` updates with new name, avatar, account_name
- **And**: Success notification appears

**Test Case 4: Visual Separation**

- **Given**: User is on Profile page
- **When**: Page loads
- **Then**: User info section (name, email, phone, address, avatar) appears at top
- **And**: Account info section (account_name, default_account_id) appears below with visual separator

**Test Case 5: Read-only Display Fields**

- **Given**: User is on Profile page
- **When**: Page loads
- **Then**: Name, email, phone, address fields appear as read-only display fields
- **And**: Display fields have accordion chevron icon
- **And**: Email display is clickable `mailto:` link
- **And**: Phone display is clickable `tel:` link

### 4.2 Organizations Page Tests

**Test Case 1: Display Organizations**

- **Given**: User belongs to organizations (has `org_ids`)
- **When**: User navigates to Organizations page
- **Then**: List of organizations displays
- **And**: Each organization shows name and description

**Test Case 2: Add Organization**

- **Given**: User is on Organizations page
- **When**: User searches for organization and clicks Add
- **Then**: Organization is added to `org_ids` array
- **And**: Form submits PATCH `/api/user` with updated `org_ids`
- **And**: Organization appears in list

**Test Case 3: Remove Organization**

- **Given**: User belongs to organizations
- **When**: User clicks Remove on an organization
- **Then**: Organization is removed from `org_ids` array
- **And**: Form submits PATCH `/api/user` with updated `org_ids`
- **And**: Organization disappears from list

### 4.3 Clubs Page Tests

**Test Cases**: Same as Organizations, but for clubs (`club_ids`)

### 4.4 SVs Page Tests

**Test Cases**: Same as Organizations, but for SVs (`sv_ids`)

### 4.5 Theme Settings Page Tests

**Test Case 1: Display Theme Settings**

- **Given**: User is on Theme Settings page
- **When**: Page loads
- **Then**: Sight Control section displays (darkMode, baseColor, language)
- **And**: Sound Control section displays (muteAll, useEffects, useAssist)
- **And**: Current settings values are pre-populated

**Test Case 2: Dark Mode Immediate Update**

- **Given**: User is on Theme Settings page
- **When**: User toggles Dark Mode switch
- **Then**: Dark mode applies immediately to DOM
- **And**: `authContext` updates immediately
- **And**: Setting saves to backend
- **And**: Setting persists after page reload

**Test Case 3: Language Immediate Update**

- **Given**: User is on Theme Settings page
- **When**: User selects different language
- **Then**: i18n updates immediately
- **And**: `authContext` updates immediately
- **And**: Setting saves to backend
- **And**: Setting persists after page reload

**Test Case 4: Base Color Immediate Update**

- **Given**: User is on Theme Settings page
- **When**: User selects different base color
- **Then**: `authContext` updates immediately
- **And**: Setting saves to backend
- **And**: Setting persists after page reload

**Test Case 5: Locale API Integration**

- **Given**: User is on Theme Settings page
- **When**: Page loads
- **Then**: Language dropdown fetches locales from `/api/locale`
- **And**: Only English and Spanish locales are shown
- **And**: Locale names display correctly

**Test Case 6: Default Values**

- **Given**: User has no theme settings
- **When**: Theme Settings page loads
- **Then**: Default values from config are used
- **And**: Form displays default values correctly

### 4.6 Navigation Tests

**Test Case 1: Navigation Menu**

- **Given**: User is on any account page
- **When**: User views side navigation
- **Then**: Organizations, Clubs, SVs, Theme menu items appear
- **And**: Clicking menu items navigates to respective pages

**Test Case 2: Account Index Cards**

- **Given**: User is on Account index page
- **When**: Page loads
- **Then**: Organizations, Clubs, SVs, Theme cards appear
- **And**: Clicking cards navigates to respective pages

---

## P: PLAN - Create implementation plan

<!-- Detailed implementation plan from IMPLEMENTATION PLAN file -->

[Note: The full implementation plan is extensive. Key points include:]

### Key Changes Summary

1. **Profile Page Refactor**: Convert text inputs to full-width read-only display fields with accordion behavior
2. **New Component**: Extend `CollapsibleSettingRow` component with `labelPosition` prop for label-above, full-width display fields
3. **Organizations Page**: New page for managing `org_ids` with "+" dropdown
4. **Clubs Page**: New page for managing `club_ids` with "+" dropdown
5. **SVs Page**: New page for managing `sv_ids` with "+" dropdown (creates SV from BOAT selection)
6. **Theme Settings Page**: New page for managing theme preferences (darkMode, baseColor, language) with immediate updates
7. **Backend Updates**: Add validation for `org_ids`, `club_ids`, `sv_ids` arrays

### Implementation Order

1. Update `createOrdered()` helper (immutable, previous_id)
2. Update all entity schemas (immutable, previous_id)
3. Rename `display` → `acronym` in org/club schemas
4. Create org/club/boat/sv controllers
5. Create org/club/boat/sv routes
6. Implement search logic
7. Implement SV CRUD with duplicate check
8. Implement SV immutable clone logic
9. Update user controller validation
10. Extend CollapsibleSettingRow component
11. Refactor Profile page
12. Create Organizations page
13. Create Clubs page
14. Create SVs page
15. Add routes and navigation
16. Update locales

### Files to Modify/Create

**Frontend**:

- `client/src/components/form/collapsible-setting-row/collapsible-setting-row.jsx` (MODIFY)
- `client/src/views/account/profile.jsx` (MODIFY)
- `client/src/views/account/organizations.jsx` (CREATE)
- `client/src/views/account/clubs.jsx` (CREATE)
- `client/src/views/account/svs.jsx` (CREATE)
- `client/src/views/account/theme.jsx` (CREATE)
- `client/src/routes/account.js` (MODIFY)
- `client/src/components/layout/account/account.jsx` (MODIFY)
- `client/src/views/account/index.jsx` (MODIFY)
- Multiple locale files (MODIFY/CREATE)

**Backend**:

- `server/helper/mongo.js` (MODIFY)
- `server/model/mongo/sv.mongo.js` (MODIFY)
- `server/model/mongo/org.mongo.js` (MODIFY)
- `server/model/mongo/club.mongo.js` (MODIFY)
- `server/model/mongo/boat.mongo.js` (MODIFY)
- All other entity models (MODIFY)
- `server/controller/sv.controller.js` (CREATE)
- `server/controller/org.controller.js` (CREATE)
- `server/controller/club.controller.js` (CREATE)
- `server/controller/boat.controller.js` (CREATE)
- `server/controller/user.controller.js` (MODIFY)
- `server/api/sv.route.js` (CREATE)
- `server/api/org.route.js` (CREATE)
- `server/api/club.route.js` (CREATE)
- `server/api/boat.route.js` (CREATE)
- `server/api/user.route.js` (MODIFY)

---

## V: REVIEW - Review and validate the implementation plan

## Confidence Rating: **98-99%** (varies by component)

### Component Confidence Breakdown

- **Profile Page Refactor**: 99% - Clear patterns, well-understood
- **Organizations/Clubs/SVs Pages**: 99% - Follows established Notifications pattern
- **Theme Settings Page**: 98% - Minor display preferences can be adjusted
- **Backend Updates**: 99% - Straightforward validation and API updates

## All Answers Integrated - Plan Finalized

Based on comprehensive answers, all decisions are finalized and the plan is ready for implementation:

### ✅ Final Confirmed Decisions

1. **SV Description**: Optional field (not required)

2. **SV user_ids Structure**: Extensible object with multiple arrays

   - Structure: `{ owners: [string], crew: [string], sailmakers: [string], mechanics: [string], ... }`
   - Future-proof for additional role types
   - When creating SV, add current user to `user_ids.owners` array

3. **SV Duplicate Prevention**:

   - Unique SV = `boat_id` + `user_ids.owners` array combination
   - Can have multiple SVs for one physical BOAT with different owner combinations
   - Check: `boat_id` matches AND `user_ids.owners` contains current user
   - If duplicate: Return existing SV (don't create)

4. **Multiple SV Creation**:

   - Create all SVs in parallel (Promise.all)
   - Continue on failure (don't stop)
   - Simple toast alert for each result (success/failure)
   - Toast system handles stacking automatically

5. **SV Owner Removal**:

   - Handle in user controller when updating `user.sv_ids`
   - Remove user from `SV.user_ids.owners` array

6. **SV Immutable Flag**:

   - **CRITICAL**: Add `immutable` field to ALL entities (not just SV)
   - Add in `server/helper/mongo.js` `createOrdered()` function
   - Set default to `false` in helper
   - Add to all schema models right after `state` field
   - When season starts: Set via admin/console "Start New Season" feature (future)
   - When cloning immutable SV:
     - Copy all fields
     - Set `created_at` to now
     - Set `immutable` to `false`
     - Set `updated_at` to `null`
     - Set `previous_id` to original SV's ID

7. **previous_id Field**:

   - Add to ALL entities
   - Add in `server/helper/mongo.js` `createOrdered()` function
   - Add to all schema models right before `created_at` field
   - Links to previous version when cloning

8. **PATCH Immutable Record**:

   - Should NOT fail
   - Create NEW clone of object
   - Return status indicating clone was created
   - Clone gets new ID, `previous_id` links to original

9. **Search Implementation**:

   - **Boat**: Search by Name AND Sail No (case-insensitive)
   - **Org**: Search by Acronym (renamed from display) AND full Name (case-insensitive)
   - **Club**: Search by Acronym (renamed from display) AND full Name (case-insensitive)

10. **Schema Updates**:

    - Rename `display` → `acronym` in org and club models
    - **No migration needed** (clean slate - import code updated separately)
    - Add `immutable` field to all schemas (after `state`)
    - Add `previous_id` field to all schemas (before `created_at`)

11. **Empty State Behavior**:

    - Show "Request to add {Entity}" button
    - Opens dialog with form fields
    - Collect as much information as possible
    - Submit request to admin (via API endpoint)
    - Admin reviews/edits/approves/rejects in admin/console (future)
    - User gets message on status (future)

12. **Multiple Selection UI**:

    - Checkboxes in Popover dropdown for selection
    - UI indication of multiple selections (visual feedback)
    - "Add Selected" [+] button to add selected items to user account
    - Button can be inside or outside Popover (design decision)

13. **Toast Alerts**:
    - Simple toast per result
    - Toast system handles stacking automatically
    - No need for manual queuing or delays

### Updated Helper Function: createOrdered

**File**: `server/helper/mongo.js` (MODIFY)

**Updated Function**: Also sets `immutable` and `previous_id`

```javascript
async function createOrdered(idKey, Model, data) {
  // ... existing validation ...

  // Apply automatic values in schema order
  Object.keys(schemaPaths).forEach((key) => {
    // ... existing logic ...

    // Auto immutable (new)
    if (key === "immutable") {
      doc.immutable = false; // Default to false
      return;
    }

    // Auto previous_id (new)
    if (key === "previous_id") {
      doc.previous_id = null; // Default to null
      return;
    }

    // ... rest of existing logic ...
  });

  // ... rest of function ...
}
```

### Updated Schema Structure (All Entities)

**Standard Schema Order** (for all entities):

```javascript
{
    // Common entity fields
    id: String,
    key: String,
    rev: Number,
    type: String,
    state: String,
    immutable: Boolean,      // NEW - right after state
    previous_id: String,     // NEW - right before created_at
    created_at: Date,
    updated_at: Date,

    // Entity-specific fields
    ...
}
```

### Updated SV Schema

**File**: `server/model/mongo/sv.mongo.js` (MODIFY)

**user_ids Structure**:

```javascript
user_ids: {
    type: Object,
    required: false,
    // Structure: {
    //   owners: [user_id1, user_id2, ...],
    //   crew: [user_id3, user_id4, ...],
    //   sailmakers: [user_id5, ...],
    //   mechanics: [user_id6, ...],
    //   ... (extensible for future roles)
    // }
}
```

**Add Fields**:

```javascript
immutable: {
    type: Boolean,
    required: true,
    default: false
},
previous_id: {
    type: String,
    required: false
}
```

### Updated API Specifications

#### POST /api/sv

**Request**:

```json
{
  "boat_id": "boat_123",
  "boat_key": "boat_key_value", // IMPORT ONLY, not used internally
  "description": "SV description" // Optional
}
```

**Implementation Logic**:

1. Verify boat exists
2. Check for duplicate SV: `boat_id` matches AND `user_ids.owners` contains current user
3. If duplicate: Return existing SV (status 200, don't create)
4. Generate name: `"SV ${boat.name}"`
5. Create SV with:
   - `user_ids: { owners: [req.user] }`
   - `immutable: false` (set by createOrdered)
   - `previous_id: null` (set by createOrdered)
6. Return created SV (status 201)

#### PATCH /api/sv/:id

**Important**: Check `immutable` flag before allowing updates

**If `immutable === false`**:

- Allow normal update
- Return updated SV

**If `immutable === true`**:

- **DO NOT FAIL** - Create clone instead
- Clone all fields from original SV
- Set `created_at` to now
- Set `immutable` to `false`
- Set `updated_at` to `null`
- Set `previous_id` to original SV's ID
- Generate new `id` for clone
- Return cloned SV with status indicating clone was created

#### GET /api/user/svs

**Purpose**: Get user's SVs with populated boat data
**Response**: Array of `{ sv: SV, boat: BOAT }` objects

### Updated Search Specifications

- **Boat**: Search in `name` and `sail_no` fields (case-insensitive)
- **Org**: Search in `acronym` (renamed from `display`) and `name` fields (case-insensitive)
- **Club**: Search in `acronym` (renamed from `display`) and `name` fields (case-insensitive)

### Remaining Questions (All Answered)

1. ✅ Entity Request: Stub for now (new entity like feedback)
2. ✅ SV user_ids: Stay flexible with JSON object
3. ✅ previous_id: Expose in API (for undo/compare UI)
4. ✅ immutable: Expose in API (for padlock icon UI)
5. ✅ Checkbox: Row or checkbox click works, no select all
6. ✅ Request Fields: BOAT (Brand, Model, Builder), ORG (Acronym, Name, Address), CLUB (Acronym, Name, Address)

### Success Criteria

1. ✅ Profile page uses accordion with label-above, full-width display
2. ✅ Email/phone clicks open accordion (not mailto/tel)
3. ✅ Address field required with error message
4. ✅ Organizations page allows add/remove via searchable Popover dropdown
5. ✅ Clubs page allows add/remove via searchable Popover dropdown
6. ✅ SVs page allows claiming boat ownership (creates SV)
7. ✅ Boat dropdown shows "{sail_no} - {boat.name}" format
8. ✅ Search works: Boat (name, sail_no), Org/Club (acronym, name)
9. ✅ SV display shows boat name, hull number, sail number
10. ✅ SV removal removes from array AND removes owner from SV
11. ✅ SV duplicate prevention (boat_id + owners = unique)
12. ✅ Multiple SV creation in parallel with toast alerts
13. ✅ SV immutable flag added to schema (and all entities)
14. ✅ previous_id field added to all entities
15. ✅ PATCH immutable record creates clone (doesn't fail)
16. ✅ createOrdered helper sets immutable and previous_id
17. ✅ Backend validates IDs exist, removes invalid ones
18. ✅ Loading spinners show in buttons
19. ✅ Empty states show "Request to add {Entity}" dialog
20. ✅ Multiple selection UI with checkboxes and "Add Selected" button
21. ✅ CollapsibleSettingRow backward compatibility maintained
22. ✅ Schema updated: `display` → `acronym` for org/club
23. ✅ Navigation works (cards and menu items)
24. ✅ Theme Settings page displays correctly
25. ✅ Theme navigation item appears between Billing and Notifications
26. ✅ Dark mode changes apply immediately (DOM + authContext + backend)
27. ✅ Language changes apply immediately (i18n + authContext + backend)
28. ✅ Base color changes apply immediately (authContext + backend)
29. ✅ Locales fetched from `/api/locale` and filtered to en/es
30. ✅ Settings save successfully via API
31. ✅ Settings persist after page reload
32. ✅ Other settings (notifications, etc.) are preserved
33. ✅ UI matches design reference (iOS-style horizontal layout)
34. ✅ Translations work correctly (en and es)

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

### Theme Settings Code Pattern References

**Immediate Dark Mode Update** (from `user.jsx`):

```javascript
const toggleDarkMode = useCallback(async () => {
  const newDarkMode = !darkMode;

  // Update local state immediately
  const updatedSettings = {
    ...authContext.user.settings,
    theme: {
      ...authContext.user.settings?.theme,
      sight: {
        ...authContext.user.settings?.theme?.sight,
        darkMode: newDarkMode,
      },
    },
  };
  authContext.update({ settings: updatedSettings });

  // Update DOM immediately
  newDarkMode
    ? document.getElementById("app").classList.add("dark")
    : document.getElementById("app").classList.remove("dark");

  // Persist to backend
  try {
    await setSettingValue("theme.sight.darkMode", newDarkMode);
  } catch (err) {
    // Revert on error
    authContext.update({ settings: authContext.user.settings });
    newDarkMode
      ? document.getElementById("app").classList.remove("dark")
      : document.getElementById("app").classList.add("dark");
  }
}, [darkMode, authContext]);
```

**Locale Change** (from `user.jsx`):

```javascript
const changeLocale = useCallback(
  (locale) => {
    // Update Accept-Language header immediately
    Axios.defaults.headers.common["Accept-Language"] = locale;
    i18n.changeLanguage(locale);
    authContext.update({ locale });
  },
  [authContext]
);
```

**Form Submit Pattern** (from `notifications.jsx`):

```javascript
const handleSubmit = async (res, data) => {
    setLoading(true);
    try {
        const currentSettings = authContext.user?.settings || {};
        const updatedSettings = {
            ...currentSettings,
            theme: {
                ...currentSettings.theme,
                sight: { ... },
                sound: { ... }
            }
        };

        const response = await Axios({
            method: 'PUT',
            url: '/api/user/settings/all',
            data: { settings: updatedSettings }
        });

        if (response.data?.data?.settings) {
            authContext.update({ settings: response.data.data.settings });
        }

        viewContext.notification({
            description: 'Settings saved successfully',
            variant: 'success'
        });
    } catch (err) {
        viewContext.notification({
            description: err.response?.data?.message || 'Failed to save settings',
            variant: 'error'
        });
    } finally {
        setLoading(false);
    }
};
```

<!-- Additional notes, decisions, or observations -->
