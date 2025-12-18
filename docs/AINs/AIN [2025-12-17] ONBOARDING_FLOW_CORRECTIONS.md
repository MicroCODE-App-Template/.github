# AIN - FEATURE - ONBOARDING

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

# Implementation Plan: Onboarding Password Reset UI

## Executive Summary

This plan outlines the implementation to replace the Toast notification with a dedicated "Reset Your Password" screen when externally seeded users (with `verified: false` and `state: "onboarding"`) attempt to sign in. The new screen will mirror the existing email verification UI structure but adapted for password reset functionality.

## Current State Analysis

### Backend Flow (`server/controller/auth.controller.js`)

- **Location**: Lines 76-106
- **Behavior**: When a user with `state === 'onboarding'` and `verified === false` signs in:
  1. Generates a password reset token (5-minute expiration)
  2. Sends password reset email using `password_reset` template
  3. Returns `{ message: res.__('user.password.check_email') }` with status 200
  4. Does NOT authenticate the user

### Frontend Flow (`client/src/views/auth/signin/index.jsx`)

- **Location**: Lines 212-219
- **Current Behavior**:
  - Detects response with `message` but no `token` or `tfa_required`
  - Navigates to `/signup/verify` (email verification screen)
  - This is incorrect - should show password reset screen instead

### Password Reset Completion (`server/controller/user.controller.js`)

- **Location**: Lines 1269-1276
- **Behavior**: When password reset completes:
  - Sets `verified: true` if user was unverified
  - Calls `authController.signin()` to authenticate user
  - This properly completes onboarding

### Existing Email Verification Screen (`client/src/views/auth/signup/verify.jsx`)

- **Structure**:
  - Title: "Verify Your Email Address"
  - Alert component with info variant
  - "Please Check Your Email" message
  - "Re-Send Verification Email" button
  - Footer link to account management
- **API Endpoint**: `/api/user/verify/request` for resending

## Requirements

### Functional Requirements

1. **New Screen**: Create `/signin/resetpassword-request` route
2. **UI Structure**: Mirror email verification screen layout
3. **Email Access**: Access email from Form callback `data.email` (preferred) or URL params
4. **Resend Functionality**: Call `/api/auth/password/reset/request` endpoint
5. **Navigation**: Update signin flow to route to new screen instead of verify screen
6. **Backward Compatibility**: Keep all existing code, add new route only

### Files to Create

**Web Client**:

1. `client/src/views/auth/signin/resetpassword-request.jsx` - New component
2. `client/src/locales/en/auth/signin/en_resetpassword-request.json` - English translations
3. `client/src/locales/es/auth/signin/es_resetpassword-request.json` - Spanish translations

**Mobile App**:

1. `app/views/auth/resetpassword-request.js` - New component
2. `app/locales/en/auth/signin/en_resetpassword-request.json` - English translations
3. `app/locales/es/auth/signin/es_resetpassword-request.json` - Spanish translations

### Files to Modify

**Web Client**:

1. `client/src/routes/auth.js` - Add route definition
2. `client/src/views/auth/signin/index.jsx` - Update navigation logic (line 217)

**Mobile App**:

1. `app/routes/auth.js` - Add route definition
2. `app/views/auth/signin/index.js` - Update navigation logic

---

## V: REVIEW - Review and validate the implementation plan

## Confidence Rating: **95%**

## All Questions Answered - Plan Finalized

### ✅ Fully Ready (95%+ Confidence)

1. **Backend**: No changes needed - already handles onboarding correctly
2. **Web Client Component**: Clear pattern from `SignupVerification` component
3. **Web Client Routing**: Standard route addition pattern understood
4. **Form Integration**: Form callback pattern confirmed - can access `data.email`
5. **API Endpoint**: `/api/auth/password/reset/request` exists and works
6. **Email Template**: `password_reset` template exists in DB
7. **Translation Files**: Locations confirmed for both web and mobile
8. **Mobile App Routes**: Route registration pattern confirmed

### ✅ Key Findings from Codebase Investigation

#### 1. Form Component Callback Signature

**Finding**: Form component callback receives `(res, data)` where `data` contains all form values.

**Location**: `client/src/components/form/form.jsx:203`

```javascript
callback?.(res, data);
```

**Impact**: ✅ Can access email via `data.email` in signin callback - **NO ISSUE**

#### 2. Email Access Strategy

**Decision**: Use Form callback `data.email` - cleaner, no URL exposure

#### 3. Translation Patterns

**Server-side**: ✅ Confirmed - `server/locales/en/user.en.json` contains `user.password.check_email`
**Client-side**: Uses `useTranslation()` hook from `react-i18next`

**Pattern Found**:

- Client components use `t('auth.signin.index.title')` pattern
- Server uses `res.__('user.password.check_email')` pattern
- Client has separate translation files

### ✅ Implementation Readiness

**Status**: ✅ **READY FOR IMPLEMENTATION**

**Confidence**: **95%**

**All critical questions answered through codebase investigation.**

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
