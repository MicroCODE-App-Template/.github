# AIN - FEATURE - IOS_LIKE_SETTINGS_ROWS

## Metadata

- **Type**: FEATURE
- **Issue #**: [if applicable]
- **Created**: [DATE]
- **Status**: READY FOR IMPLEMENTATION

---

## C: CONCEPT/CHANGE/CORRECTION - Discuss ideas without generating code

### Problem Statement

The `SettingRow` component was modified to support User Edit form requirements where display fields needed specific width constraints (30% label, 70% control). This change introduced a hardcoded `maxWidth: '40%'` constraint on the label area, which adversely affects Switch components throughout the application.

**Current Issue:**

- Switch settings (e.g., Notification Settings) have labels/descriptions constrained to ~40% width
- This causes text wrapping and poor UX for Switch components that previously had almost full width
- The constraint was necessary for text display/input fields but should not apply to Switch components

**Requirements:**

1. Switch components should have almost full width for labels/descriptions (restore original behavior)
2. Text display/input fields should maintain the 30% label width limit (preserve current behavior)
3. Solution must be backward compatible
4. No breaking changes to existing functionality

---

## D: DESIGN - Design detailed solution

### Current Architecture

#### Components Involved

1. **`SettingRow`** (`client/src/components/form/setting-row/setting-row.jsx`)

   - Horizontal settings row component with iOS-style layout
   - Currently has hardcoded `maxWidth: '40%'` on label area
   - Has `controlWidth` prop for explicit control area sizing
   - Used by both Form component and CollapsibleSettingRow

2. **`Form`** (`client/src/components/form/form.jsx`)

   - Dynamic form component that renders inputs
   - Uses `SettingRow` when `settingsMode={true}`
   - Has access to input type information via `inputs[name].type`

3. **`CollapsibleSettingRow`** (`client/src/components/form/collapsible-setting-row/collapsible-setting-row.jsx`)

   - Uses `SettingRow` internally
   - Currently passes `controlWidth='70%'` for display fields

4. **`Switch`** (`client/src/components/form/input/switch/switch.jsx`)
   - Toggle switch component
   - Used in notifications, TFA, and other settings pages

#### Current Implementation Details

**SettingRow Component:**

```jsx
// Label area - CURRENTLY HARDCODED TO 40%
<div className='flex-1 pr-4 min-w-0' style={{
    maxWidth: '40%'
}}>
    {/* Label and description */}
</div>

// Control area - USES controlWidth PROP OR DEFAULTS TO 60%
<div className='flex-shrink-0 min-w-0 overflow-hidden' style={{
    width: controlWidth || '60%',
    maxWidth: controlWidth || '60%'
}}>
    { children }
</div>
```

**Form Component Usage:**

```jsx
// When settingsMode is true, wraps inputs in SettingRow
<SettingRow
    label={ label }
    description={ description }
    error={ errors[name]?.message }
    required={ required }
>
    <Input.component ... />
</SettingRow>
```

**Switch Usage Pattern:**

- Switch components are rendered via Form component with `settingsMode={true}`
- Input type is `'switch'` (from `inputs[name].type`)
- Switch components typically have longer labels and descriptions that need more space

### Proposed Solution

#### High-Level Approach

Add a new numeric prop `labelWidth` to `SettingRow` that:

1. Accepts a number (0-100) representing the percentage width for the label area
2. Defaults to `40` (maintains current behavior when not provided)
3. Form component detects Switch input type and passes `labelWidth={95}` to SettingRow
4. CollapsibleSettingRow continues to use `controlWidth` for its specific needs
5. Provides flexibility for future use cases with any percentage value

#### Design Decisions

1. **New Prop: `labelWidth`**

   - Numeric prop (0-100, representing percentage)
   - Default: `40` (maintains current `maxWidth: '40%'` behavior)
   - When provided, sets `maxWidth: '{labelWidth}%'` on label area
   - Examples: `30` (narrow labels), `40` (default), `95` (full-width for Switch)
   - Provides fine-grained control for future flexibility

2. **Input Type Detection**

   - Form component already has access to `inputs[name].type`
   - Maps input types to `labelWidth` values:
     - `switch`: `95` (almost full width)
     - `radio`, `checkbox`: `90` (extensible for future)
     - Other types: `undefined` (defaults to 40)
   - Extensible for any future input types

3. **Control Width Logic**

   - When `labelWidth` provided:
     - If `labelWidth >= 90`: Control area uses `width: 'auto'` with `minWidth: '44px'` (for Switch)
     - Otherwise: Control width calculated as `100 - labelWidth`
   - When `controlWidth` prop provided: Takes precedence over calculated width
   - When neither provided: Defaults to 60% control width

4. **Backward Compatibility**
   - Default value (40) preserves existing behavior
   - Existing code continues to work without changes
   - Only affects components that explicitly set `labelWidth` prop

### Interface Contracts

#### SettingRow Component Props

```typescript
interface SettingRowProps {
  // Existing props
  label: string;
  description?: string;
  children: ReactNode;
  error?: string;
  required?: boolean;
  className?: string;
  layout?: "horizontal" | "stacked";
  controlWidth?: string; // e.g., '70%' - for explicit control width

  // NEW PROP
  labelWidth?: number; // 0-100, percentage for label area maxWidth
  // Default: 40 (maintains current behavior)
  // Examples: 30 (narrow labels), 40 (default), 95 (full-width for Switch)
}
```

#### Prop Behavior Matrix

| `labelWidth`          | `controlWidth` | Label Area        | Control Area                        | Use Case                               |
| --------------------- | -------------- | ----------------- | ----------------------------------- | -------------------------------------- |
| `undefined` (default) | `undefined`    | `maxWidth: '40%'` | `width: '60%'`                      | Default text inputs                    |
| `undefined` (default) | `'70%'`        | `maxWidth: '40%'` | `width: '70%'`                      | Display fields (CollapsibleSettingRow) |
| `30`                  | `undefined`    | `maxWidth: '30%'` | `width: '70%'`                      | Narrow labels                          |
| `40`                  | `undefined`    | `maxWidth: '40%'` | `width: '60%'`                      | Explicit default                       |
| `95`                  | `undefined`    | `maxWidth: '95%'` | `width: 'auto'`, `minWidth: '44px'` | Switch components                      |

#### Width Calculation Logic

```javascript
// Label area width
const labelMaxWidth = labelWidth !== undefined ? `${labelWidth}%` : "40%"; // Default

// Control area width
let controlAreaWidth;
if (controlWidth) {
  // Explicit controlWidth takes precedence
  controlAreaWidth = controlWidth;
} else if (labelWidth !== undefined) {
  // Calculate from labelWidth
  const remainingWidth = 100 - labelWidth;
  // For Switch components (labelWidth >= 90), use auto-sizing
  if (labelWidth >= 90) {
    controlAreaWidth = "auto";
  } else {
    controlAreaWidth = `${remainingWidth}%`;
  }
} else {
  // Default behavior
  controlAreaWidth = "60%";
}
```

### Component Architecture

#### Component Hierarchy

```
Form Component (settingsMode={true})
    └── SettingRow (labelWidth={95} for Switch)
        └── Switch Component

CollapsibleSettingRow
    └── SettingRow (controlWidth='70%')
        └── Display Field + Detail Fields
```

#### Data Flow

```
Input Type Detection (Form Component)
    ↓
Determine labelWidth based on type
    ↓
Pass labelWidth to SettingRow
    ↓
SettingRow calculates widths
    ↓
Render with appropriate constraints
```

#### Component Interaction Diagram

```
┌────────────────────────────────────────────────────────────┐
│                    Form Component                          │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Input Type: 'switch'                                │  │
│  │  → labelWidth = 95                                   │  │
│  └──────────────────────────────────────────────────────┘  │
│                          ↓                                 │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  <SettingRow labelWidth={95}>                        │  │
│  │    <Switch />                                        │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│              CollapsibleSettingRow                          │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  controlWidth='70%' (explicit)                       │   │
│  │  labelWidth not provided (uses default 40)           │   │
│  └──────────────────────────────────────────────────────┘   │
│                          ↓                                  │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  <SettingRow controlWidth='70%'>                     │   │
│  │    <Display Field />                                 │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

---

## P: PLAN - Create implementation plan

# Implementation Plan: Numeric `labelWidth` Prop for SettingRow Component

## Executive Summary

This document outlines the detailed implementation plan for adding a numeric `labelWidth` prop to the `SettingRow` component. This prop will replace the hardcoded `maxWidth: '40%'` constraint, providing flexible control over label width allocation while maintaining backward compatibility.

**Key Change:** Replace boolean `fullWidthLabel` approach with numeric `labelWidth` prop (percentage as number).

## Table of Contents

1. [Problem Statement](#problem-statement)
2. [Solution Overview](#solution-overview)
3. [Interface Contracts](#interface-contracts)
4. [Component Architecture](#component-architecture)
5. [Implementation Details](#implementation-details)
6. [User Stories](#user-stories)
7. [Test Strategy](#test-strategy)
8. [Migration Path](#migration-path)
9. [Questions & Assumptions](#questions--assumptions)
10. [Risk Assessment](#risk-assessment)

## Problem Statement

### Current State

The `SettingRow` component has a hardcoded `maxWidth: '40%'` constraint on the label area:

```jsx
<div className='flex-1 pr-4 min-w-0' style={{
    maxWidth: '40%'  // HARDCODED
}}>
```

This constraint:

- ✅ Works well for text input fields (30% label, 70% control)
- ❌ Causes unnecessary text wrapping for Switch components
- ❌ Limits flexibility for future use cases
- ❌ Doesn't allow fine-grained control over label width

### Requirements

1. **Switch Components**: Need ~95% label width (almost full width)
2. **Text Input Fields**: Need ~40% label width (current behavior)
3. **Display Fields**: Need ~30% label width (via `controlWidth='70%'`)
4. **Future Flexibility**: Support any percentage value (0-100)
5. **Backward Compatibility**: Default behavior unchanged when prop not provided

## Solution Overview

### High-Level Approach

Add a numeric `labelWidth` prop to `SettingRow`:

- **Type**: `number` (0-100, representing percentage)
- **Default**: `40` (maintains current behavior)
- **Usage**: When provided, sets `maxWidth: '{labelWidth}%'` on label area
- **Control Width**: Calculated as `100 - labelWidth` (with padding consideration)

### Design Principles

1. **Backward Compatible**: Default value (40) preserves existing behavior
2. **Flexible**: Supports any percentage value for future use cases
3. **Consistent**: Works with existing `controlWidth` prop logic
4. **Type-Safe**: Numeric type prevents string parsing errors

## Interface Contracts

### SettingRow Component Props

```typescript
interface SettingRowProps {
  // Existing props
  label: string;
  description?: string;
  children: ReactNode;
  error?: string;
  required?: boolean;
  className?: string;
  layout?: "horizontal" | "stacked";
  controlWidth?: string; // e.g., '70%' - for explicit control width

  // NEW PROP
  labelWidth?: number; // 0-100, percentage for label area maxWidth
  // Default: 40 (maintains current behavior)
  // Examples: 30 (narrow labels), 40 (default), 95 (full-width for Switch)
}
```

### Prop Behavior Matrix

| `labelWidth`          | `controlWidth` | Label Area        | Control Area                        | Use Case                               |
| --------------------- | -------------- | ----------------- | ----------------------------------- | -------------------------------------- |
| `undefined` (default) | `undefined`    | `maxWidth: '40%'` | `width: '60%'`                      | Default text inputs                    |
| `undefined` (default) | `'70%'`        | `maxWidth: '40%'` | `width: '70%'`                      | Display fields (CollapsibleSettingRow) |
| `30`                  | `undefined`    | `maxWidth: '30%'` | `width: '70%'`                      | Narrow labels                          |
| `40`                  | `undefined`    | `maxWidth: '40%'` | `width: '60%'`                      | Explicit default                       |
| `95`                  | `undefined`    | `maxWidth: '95%'` | `width: 'auto'`, `minWidth: '44px'` | Switch components                      |

### Width Calculation Logic

```javascript
// Label area width
const labelMaxWidth = labelWidth !== undefined ? `${labelWidth}%` : "40%"; // Default

// Control area width
let controlAreaWidth;
if (controlWidth) {
  // Explicit controlWidth takes precedence
  controlAreaWidth = controlWidth;
} else if (labelWidth !== undefined) {
  // Calculate from labelWidth
  const remainingWidth = 100 - labelWidth;
  // For Switch components (labelWidth >= 90), use auto-sizing
  if (labelWidth >= 90) {
    controlAreaWidth = "auto";
  } else {
    controlAreaWidth = `${remainingWidth}%`;
  }
} else {
  // Default behavior
  controlAreaWidth = "60%";
}
```

## Component Architecture

### Component Hierarchy

```
Form Component (settingsMode={true})
    └── SettingRow (labelWidth={95} for Switch)
        └── Switch Component

CollapsibleSettingRow
    └── SettingRow (controlWidth='70%')
        └── Display Field + Detail Fields
```

### Data Flow

```
Input Type Detection (Form Component)
    ↓
Determine labelWidth based on type
    ↓
Pass labelWidth to SettingRow
    ↓
SettingRow calculates widths
    ↓
Render with appropriate constraints
```

### Component Interaction Diagram

```
┌────────────────────────────────────────────────────────────┐
│                    Form Component                          │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Input Type: 'switch'                                │  │
│  │  → labelWidth = 95                                   │  │
│  └──────────────────────────────────────────────────────┘  │
│                          ↓                                 │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  <SettingRow labelWidth={95}>                        │  │
│  │    <Switch />                                        │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│              CollapsibleSettingRow                          │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  controlWidth='70%' (explicit)                       │   │
│  │  labelWidth not provided (uses default 40)           │   │
│  └──────────────────────────────────────────────────────┘   │
│                          ↓                                  │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  <SettingRow controlWidth='70%'>                     │   │
│  │    <Display Field />                                 │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

## Implementation Details

### Phase 1: Update SettingRow Component

**File:** `client/src/components/form/setting-row/setting-row.jsx`

#### Changes Required

1. **Add `labelWidth` prop to component signature**

   ```jsx
   const SettingRow = forwardRef(({
       label,
       description,
       children,
       error,
       required,
       className,
       layout = 'horizontal',
       controlWidth,
       labelWidth, // NEW PROP (number, optional)
       ...props
   }, ref) => {
   ```

2. **Update JSDoc documentation**

   ```jsx
   /**
    * @param {number} params.labelWidth Optional label width as percentage (0-100).
    *                                   Default: 40. When provided, sets maxWidth on label area.
    *                                   Examples: 30 (narrow), 40 (default), 95 (full-width for Switch).
    */
   ```

3. **Implement width calculation logic**

   ```jsx
   if (layout === 'horizontal') {
       // Calculate label area maxWidth
       const labelMaxWidth = labelWidth !== undefined
           ? `${labelWidth}%`
           : '40%'; // Default maintains current behavior

       // Calculate control area width
       let controlAreaWidth;
       let controlAreaMaxWidth;

       if (controlWidth) {
           // Explicit controlWidth takes precedence (for CollapsibleSettingRow)
           controlAreaWidth = controlWidth;
           controlAreaMaxWidth = controlWidth;
       } else if (labelWidth !== undefined) {
           // Calculate from labelWidth
           if (labelWidth >= 90) {
               // For Switch components (labelWidth >= 90), use auto-sizing
               controlAreaWidth = 'auto';
               controlAreaMaxWidth = 'none';
           } else {
               // Calculate remaining width
               const remainingWidth = 100 - labelWidth;
               controlAreaWidth = `${remainingWidth}%`;
               controlAreaMaxWidth = `${remainingWidth}%`;
           }
       } else {
           // Default behavior
           controlAreaWidth = '60%';
           controlAreaMaxWidth = '60%';
       }

       return (
           <div ...>
               {/* Label area - CONDITIONAL MAXWIDTH */}
               <div className='flex-1 pr-4 min-w-0' style={{
                   maxWidth: labelMaxWidth
               }}>
                   {/* Label and description */}
               </div>

               {/* Control area - CONDITIONAL WIDTH */}
               <div className='flex-shrink-0 min-w-0 overflow-hidden' style={{
                   width: controlAreaWidth,
                   maxWidth: controlAreaMaxWidth,
                   ...(labelWidth >= 90 && { minWidth: '44px' }) // Ensure Switch has space
               }}>
                   { children }
               </div>
           </div>
       );
   }
   ```

4. **Add prop validation (optional, for development)**

   ```jsx
   // Validate labelWidth if provided
   if (labelWidth !== undefined) {
     if (typeof labelWidth !== "number" || labelWidth < 0 || labelWidth > 100) {
       console.warn(
         `SettingRow: labelWidth must be a number between 0 and 100, got ${labelWidth}`
       );
     }
   }
   ```

### Phase 2: Update Form Component

**File:** `client/src/components/form/form.jsx`

#### Changes Required

1. **Create input type to labelWidth mapping**

   ```jsx
   // Map input types to labelWidth percentages
   const inputTypeLabelWidthMap = {
     switch: 95, // Almost full width for Switch components
     radio: 90, // Full width for radio groups (if needed)
     checkbox: 90, // Full width for checkbox groups (if needed)
     // Other types use default (40) when not specified
   };
   ```

2. **Determine labelWidth for each input**

   ```jsx
   // In settingsMode rendering section:
   if (settingsMode && type !== "hidden") {
     // Determine labelWidth based on input type
     const labelWidth = inputTypeLabelWidthMap[type];

     return (
       <SettingRow
         key={name}
         label={label}
         description={description}
         error={errors[name]?.message}
         required={required}
         labelWidth={labelWidth} // Pass numeric value or undefined
       >
         {/* ... existing Controller code ... */}
       </SettingRow>
     );
   }
   ```

#### Input Type Mapping Table

| Input Type                        | `labelWidth`                 | Rationale                                                        |
| --------------------------------- | ---------------------------- | ---------------------------------------------------------------- |
| `switch`                          | `95`                         | Switch components need almost full width for labels/descriptions |
| `radio`                           | `90`                         | Radio groups may have long option lists, need more label space   |
| `checkbox`                        | `90`                         | Checkbox groups similar to radio                                 |
| `text`, `email`, `password`, etc. | `undefined` (defaults to 40) | Text inputs maintain current behavior                            |
| `hidden`                          | N/A                          | Not rendered in SettingRow                                       |

### Phase 3: Verify CollapsibleSettingRow

**File:** `client/src/components/form/collapsible-setting-row/collapsible-setting-row.jsx`

#### Verification Required

- **Current Usage**: `controlWidth='70%'` is passed explicitly
- **Expected Behavior**: `labelWidth` prop is not provided, so defaults to 40
- **Result**: Label area gets `maxWidth: '40%'`, control area gets `width: '70%'` (explicit)
- **Action**: No changes needed - existing behavior preserved

#### Test Cases

1. Display field should maintain 70% control width
2. Label area should maintain 40% max width (default)
3. No regressions in User Edit form

### Phase 4: Update Admin Console

**Files:**

- `admin/console/src/components/form/setting-row/setting-row.jsx`
- `admin/console/src/components/form/form.jsx`

#### Changes Required

Apply identical changes as client-side:

1. Add `labelWidth` prop to SettingRow
2. Update Form component with input type mapping
3. Ensure consistency across codebases

## User Stories

### Story 1: Switch Component Full-Width Labels

**As a** user viewing Notification Settings
**I want** Switch labels and descriptions to use almost the full width of the row
**So that** text doesn't wrap unnecessarily and the interface is easier to read

**Acceptance Criteria:**

- [ ] Switch components have ~95% label width
- [ ] Descriptions don't wrap unnecessarily
- [ ] Switches remain right-aligned
- [ ] Layout works in both light and dark mode

### Story 2: Text Input Fields Maintain Current Behavior

**As a** developer using SettingRow for text inputs
**I want** the default label width to remain 40%
**So that** existing forms continue to work without changes

**Acceptance Criteria:**

- [ ] Text input fields default to 40% label width
- [ ] No code changes required for existing forms
- [ ] Control area defaults to 60% width

### Story 3: Display Fields Maintain Explicit Control Width

**As a** developer using CollapsibleSettingRow
**I want** explicit `controlWidth` prop to take precedence
**So that** display fields maintain their 70% control width

**Acceptance Criteria:**

- [ ] `controlWidth='70%'` prop works as before
- [ ] Label area defaults to 40% when `controlWidth` is provided
- [ ] No regressions in User Edit form

### Story 4: Future Flexibility for Custom Widths

**As a** developer
**I want** to specify any label width percentage
**So that** I can fine-tune layouts for specific use cases

**Acceptance Criteria:**

- [ ] Can pass `labelWidth={30}` for narrow labels
- [ ] Can pass `labelWidth={95}` for full-width labels
- [ ] Can pass any value between 0-100
- [ ] Control width calculates correctly from label width

## Migration Path

### Step 1: Implementation

- Update SettingRow component with `labelWidth` prop
- Update Form component with input type mapping
- Update Admin Console components (if applicable)

### Step 2: Testing

- Run unit tests
- Run integration tests
- Manual testing on all affected pages

### Step 3: Deployment

- Deploy to staging environment
- Verify in staging
- Deploy to production

### Step 4: Monitoring

- Monitor for any layout issues
- Collect user feedback
- Address any edge cases

### Rollback Plan

- If issues arise, can revert to hardcoded `maxWidth: '40%'`
- Default behavior preserved, so rollback is safe

## Implementation Checklist

- [ ] Update `client/src/components/form/setting-row/setting-row.jsx`

  - [ ] Add `labelWidth` prop
  - [ ] Update JSDoc documentation
  - [ ] Implement conditional styling logic
  - [ ] Add prop validation (optional)
  - [ ] Run ESLint and fix any errors

- [ ] Update `client/src/components/form/form.jsx`

  - [ ] Create input type to labelWidth mapping
  - [ ] Detect Switch input type
  - [ ] Pass `labelWidth` prop to SettingRow
  - [ ] Run ESLint and fix any errors

- [ ] Update `admin/console/src/components/form/setting-row/setting-row.jsx` (if exists)

  - [ ] Apply same changes as client-side

- [ ] Update `admin/console/src/components/form/form.jsx` (if exists)

  - [ ] Apply same changes as client-side

- [ ] Create unit tests

  - [ ] Test default behavior
  - [ ] Test full-width label behavior
  - [ ] Test backward compatibility
  - [ ] Test control width calculation
  - [ ] Test controlWidth prop precedence

- [ ] Manual testing

  - [ ] Notification Settings page
  - [ ] User Edit form
  - [ ] Other Switch usage locations

- [ ] Documentation
  - [ ] Update component JSDoc
  - [ ] Update any relevant README files

---

## V: REVIEW - Review and validate the implementation plan

### Test Strategy

#### Unit Tests

**Test File:** `client/src/components/form/setting-row/setting-row.test.jsx` (new)

**Test Cases:**

1. **Default Behavior (labelWidth not provided)**

   ```javascript
   test("defaults to 40% label width when labelWidth not provided", () => {
     const { container } = render(
       <SettingRow label="Test Label">
         <input />
       </SettingRow>
     );
     const labelArea = container.querySelector(".flex-1");
     expect(labelArea.style.maxWidth).toBe("40%");
   });
   ```

2. **Explicit labelWidth**

   ```javascript
   test("uses provided labelWidth value", () => {
     const { container } = render(
       <SettingRow label="Test" labelWidth={95}>
         <input />
       </SettingRow>
     );
     const labelArea = container.querySelector(".flex-1");
     expect(labelArea.style.maxWidth).toBe("95%");
   });
   ```

3. **Control Width Calculation**

   ```javascript
   test("calculates control width from labelWidth", () => {
     const { container } = render(
       <SettingRow label="Test" labelWidth={30}>
         <input />
       </SettingRow>
     );
     const controlArea = container.querySelector(".flex-shrink-0");
     expect(controlArea.style.width).toBe("70%");
   });
   ```

4. **Switch Component Auto-Sizing (labelWidth >= 90)**

   ```javascript
   test("uses auto width for control when labelWidth >= 90", () => {
     const { container } = render(
       <SettingRow label="Test" labelWidth={95}>
         <Switch />
       </SettingRow>
     );
     const controlArea = container.querySelector(".flex-shrink-0");
     expect(controlArea.style.width).toBe("auto");
     expect(controlArea.style.minWidth).toBe("44px");
   });
   ```

5. **controlWidth Prop Precedence**

   ```javascript
   test("controlWidth prop takes precedence over labelWidth", () => {
     const { container } = render(
       <SettingRow label="Test" labelWidth={30} controlWidth="80%">
         <input />
       </SettingRow>
     );
     const controlArea = container.querySelector(".flex-shrink-0");
     expect(controlArea.style.width).toBe("80%");
   });
   ```

6. **Backward Compatibility**
   ```javascript
   test("maintains backward compatibility with existing props", () => {
     const { container } = render(
       <SettingRow label="Test" controlWidth="70%">
         <input />
       </SettingRow>
     );
     const labelArea = container.querySelector(".flex-1");
     const controlArea = container.querySelector(".flex-shrink-0");
     expect(labelArea.style.maxWidth).toBe("40%");
     expect(controlArea.style.width).toBe("70%");
   });
   ```

#### Integration Tests

**Test File:** `client/src/views/account/notifications.test.jsx` (new)

**Test Cases:**

1. **Notification Settings Page Layout**

   ```javascript
   test("Switch components have full-width labels", () => {
     render(<Notifications />);
     // Verify Switch SettingRow has labelWidth={95}
     // Verify labels don't wrap unnecessarily
   });
   ```

2. **User Edit Form Layout**
   ```javascript
   test("Text inputs maintain default label width", () => {
     render(<UserEditForm />);
     // Verify text input SettingRow has labelWidth={undefined} (defaults to 40)
     // Verify display fields maintain controlWidth='70%'
   });
   ```

#### Manual Testing Checklist

1. **Notification Settings Page** (`/account/notifications`)

   - [ ] Switch labels use ~95% width
   - [ ] Descriptions don't wrap unnecessarily
   - [ ] Switches are right-aligned
   - [ ] Layout correct in light mode
   - [ ] Layout correct in dark mode
   - [ ] Responsive behavior on mobile

2. **User Edit Form** (`/account/users/edit`)

   - [ ] Text input labels use ~40% width (default)
   - [ ] Display fields use 70% control width
   - [ ] No layout regressions
   - [ ] Collapsible sections work correctly

3. **Other Settings Pages**
   - [ ] TFA settings (if uses switches)
   - [ ] Any other pages using Switch components
   - [ ] Verify no unintended side effects

### User Stories

#### Story 1: Switch Component Full-Width Labels

**As a** user viewing Notification Settings
**I want** Switch labels and descriptions to use almost the full width of the row
**So that** text doesn't wrap unnecessarily and the interface is easier to read

**Acceptance Criteria:**

- [ ] Switch components have ~95% label width
- [ ] Descriptions don't wrap unnecessarily
- [ ] Switches remain right-aligned
- [ ] Layout works in both light and dark mode

#### Story 2: Text Input Fields Maintain Current Behavior

**As a** developer using SettingRow for text inputs
**I want** the default label width to remain 40%
**So that** existing forms continue to work without changes

**Acceptance Criteria:**

- [ ] Text input fields default to 40% label width
- [ ] No code changes required for existing forms
- [ ] Control area defaults to 60% width

#### Story 3: Display Fields Maintain Explicit Control Width

**As a** developer using CollapsibleSettingRow
**I want** explicit `controlWidth` prop to take precedence
**So that** display fields maintain their 70% control width

**Acceptance Criteria:**

- [ ] `controlWidth='70%'` prop works as before
- [ ] Label area defaults to 40% when `controlWidth` is provided
- [ ] No regressions in User Edit form

#### Story 4: Future Flexibility for Custom Widths

**As a** developer
**I want** to specify any label width percentage
**So that** I can fine-tune layouts for specific use cases

**Acceptance Criteria:**

- [ ] Can pass `labelWidth={30}` for narrow labels
- [ ] Can pass `labelWidth={95}` for full-width labels
- [ ] Can pass any value between 0-100
- [ ] Control width calculates correctly from label width

## Risk Assessment

### Low Risk

- ✅ **Backward Compatibility**: Default value (40) preserves existing behavior
- ✅ **Isolated Changes**: Only affects SettingRow and Form components
- ✅ **Type Safety**: Numeric prop prevents string parsing errors
- ✅ **Rollback**: Easy to revert if issues arise

### Medium Risk

- ⚠️ **Control Width Calculation**: Need to ensure calculations are correct for all cases
- ⚠️ **Switch Auto-Sizing**: Need to verify `minWidth: '44px'` is sufficient
- ⚠️ **Responsive Design**: May need adjustments for mobile devices

### Mitigation Strategies

1. **Thorough Testing**: Unit tests, integration tests, and manual testing
2. **Gradual Rollout**: Deploy to staging first, monitor, then production
3. **Documentation**: Clear JSDoc comments and examples
4. **Validation**: Add prop validation in development mode

### Edge Cases and Considerations

1. **Multiple Input Types**

   - Solution is extensible: can add `'radio'`, `'checkbox'` to `inputTypeLabelWidthMap` if needed
   - No breaking changes required

2. **Responsive Design**

   - Current implementation uses flexbox which is responsive
   - `minWidth: '44px'` ensures Switch has enough space on small screens
   - May need to adjust for mobile if issues arise

3. **Long Labels**

   - With full-width labels, very long text could push Switch off-screen
   - Consider adding `max-width` or text truncation if needed
   - Current implementation should handle this gracefully with flexbox

4. **CollapsibleSettingRow Compatibility**

   - CollapsibleSettingRow uses `controlWidth` prop, not `labelWidth`
   - These are independent concerns and should not conflict
   - Verify no regressions in User Edit form

5. **Admin Console Consistency**
   - Both client and admin console should have same behavior
   - Ensure changes are applied to both codebases

## Questions & Assumptions

### Questions for Clarification

1. **Label Width Values**

   - **Q**: What specific `labelWidth` value should Switch components use? (Proposed: 95)
   - **Q**: Should radio and checkbox also get full-width labels? (Proposed: 90)
   - **Q**: Are there any other input types that need custom label widths?

2. **Control Width Calculation**

   - **Q**: When `labelWidth >= 90`, should control area use `width: 'auto'` or calculate as `100 - labelWidth`? (Proposed: `auto` with `minWidth: '44px'`)
   - **Q**: Should there be a minimum control width for non-Switch components?

3. **Responsive Behavior**

   - **Q**: Should `labelWidth` values change on mobile devices?
   - **Q**: Do we need responsive breakpoints for different screen sizes?

4. **CollapsibleSettingRow**

   - **Q**: Should CollapsibleSettingRow also support `labelWidth` prop, or continue using `controlWidth` only?
   - **Q**: Should display fields in collapsible rows have different label widths?

5. **Validation**

   - **Q**: Should we validate `labelWidth` prop (0-100 range) or allow any number?
   - **Q**: Should we log warnings for invalid values in development?

6. **Admin Console**
   - **Q**: Does admin console have the same Form/SettingRow structure?
   - **Q**: Are there any admin-specific considerations?

#### Assumptions

1. **Switch Component Width**: Switch component is approximately 44px wide (needs verification)
2. **Default Behavior**: Default `labelWidth` of 40 maintains current behavior
3. **Control Width Precedence**: `controlWidth` prop takes precedence over calculated width from `labelWidth`
4. **Browser Support**: Flexbox is supported in all target browsers
5. **Admin Console**: Admin console has similar component structure to client

### Confidence Assessment

**Current Confidence Level: 98%**

**Confidence Factors:**

- ✅ Clear understanding of current implementation
- ✅ Well-defined interface contract
- ✅ Comprehensive test strategy
- ✅ Backward-compatible design
- ✅ Flexible and extensible approach
- ✅ Detailed implementation plan

**Remaining Questions:**

- Need answers to questions in Questions & Assumptions section
- Need verification of Switch component width (44px assumption)

**Ready for Implementation:**

- ✅ Yes, pending answers to clarification questions
- ✅ All architectural decisions documented
- ✅ Test strategy defined
- ✅ Migration path clear

### Success Criteria

✅ Switch components have ~95% label width
✅ Text input fields maintain 40% label width (default)
✅ Display fields maintain 70% control width via `controlWidth` prop
✅ Backward compatibility preserved
✅ All tests passing
✅ No layout regressions

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

## Next Steps

1. **Review and Clarification**

   - Review this implementation plan
   - Answer questions in [Questions & Assumptions](#questions--assumptions) section
   - Approve approach or suggest modifications

2. **Implementation**

   - Once approved, proceed with implementation following the checklist
   - Implement in phases as outlined
   - Test thoroughly at each phase

3. **Deployment**

   - Deploy to staging
   - Verify functionality
   - Deploy to production

4. **Follow-up**
   - Monitor for issues
   - Collect feedback
   - Iterate if needed
