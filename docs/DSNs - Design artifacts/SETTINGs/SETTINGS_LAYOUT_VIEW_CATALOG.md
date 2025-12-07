# Settings Layout - View Adoption Catalog

**Last Updated**: December 3, 2025
**Quick Reference**: Which views use (or should use) iOS-style settings layout

---

## ✅ COMPLETED - Using Settings Mode (2 views)

### Client Web App

| View              | File                                         | Status  | Notes                                     |
| ----------------- | -------------------------------------------- | ------- | ----------------------------------------- |
| **Notifications** | `client/src/views/account/notifications.jsx` | ✅ Live | Confirmed: "looks amazing, very iOS-like" |

### React Native App

| View              | File                                 | Status  | Notes                 |
| ----------------- | ------------------------------------ | ------- | --------------------- |
| **Notifications** | `app/views/account/notifications.js` | ✅ Live | Settings mode enabled |

---

## 🎯 RECOMMENDED - High Priority Candidates (4 views)

### Client Web App

| View                    | File                                   | Type    | Rationale                       | Effort  |
| ----------------------- | -------------------------------------- | ------- | ------------------------------- | ------- |
| **Two-Factor Auth**     | `client/src/views/account/tfa.jsx`     | Toggle  | Enable/disable 2FA is a setting | 1-2 hrs |
| **Profile Preferences** | `client/src/views/account/profile.jsx` | Partial | Default account selector only   | 2-3 hrs |

### React Native App

| View                    | File                           | Type    | Rationale                       | Effort  |
| ----------------------- | ------------------------------ | ------- | ------------------------------- | ------- |
| **Two-Factor Auth**     | `app/views/account/tfa.js`     | Toggle  | Enable/disable 2FA is a setting | 1-2 hrs |
| **Profile Preferences** | `app/views/account/profile.js` | Partial | Default account selector only   | 2-3 hrs |

**Total Recommended Effort**: 6-10 hours

---

## 📋 ANALYZED - Keep Traditional Mode (20+ views)

### Authentication & Onboarding (Keep Traditional)

| View Category    | Files                            | Count | Rationale                 |
| ---------------- | -------------------------------- | ----- | ------------------------- |
| **Sign In**      | `client/src/views/auth/signin/*` | 7     | Data entry (credentials)  |
| **Sign Up**      | `client/src/views/auth/signup/*` | 4     | Data entry (registration) |
| **Setup Wizard** | `client/src/views/setup/*`       | 8     | Multi-step data entry     |
| **Onboarding**   | `client/src/views/onboarding/*`  | 1     | Guided flow               |

### Account Management (Keep Traditional)

| View             | File                                            | Rationale                               |
| ---------------- | ----------------------------------------------- | --------------------------------------- |
| **Password**     | `client/src/views/account/password.jsx`         | Password entry (not a toggle)           |
| **Profile Edit** | `client/src/views/account/profile.jsx`          | Name, email, avatar upload (data entry) |
| **Billing Card** | `client/src/views/account/billing/card.jsx`     | Payment card entry                      |
| **Plan Upgrade** | `client/src/views/account/billing/plan.jsx`     | Plan selection & purchase               |
| **API Keys**     | `client/src/views/account/apikey/edit.jsx`      | Key generation (data entry)             |
| **Users**        | `client/src/views/account/users.jsx`            | User invitation (email entry)           |
| **Invoices**     | `client/src/views/account/billing/invoices.jsx` | List view (no form)                     |

---

## 🔍 Admin Console (TBD - Requires Investigation)

| View           | File                        | Analysis Status  |
| -------------- | --------------------------- | ---------------- |
| Admin Settings | `admin/console/src/views/*` | Not yet analyzed |

**Note**: Admin console may have system configuration pages that would benefit from settings mode. Requires separate analysis.

---

## 📐 Decision Matrix

Use this matrix to determine if a view should use settings mode:

| Question                                           | Settings Mode | Traditional Mode |
| -------------------------------------------------- | ------------- | ---------------- |
| **Is the user toggling existing options?**         | ✅ Yes        | ❌ No            |
| **Is the user selecting from predefined choices?** | ✅ Yes        | Maybe            |
| **Is the user entering new text/data?**            | ❌ No         | ✅ Yes           |
| **Are there 3+ text inputs?**                      | ❌ No         | ✅ Yes           |
| **Is there a textarea/multi-line input?**          | ❌ No         | ✅ Yes           |
| **Is it part of a multi-step wizard?**             | ❌ No         | ✅ Yes           |
| **Does it require extensive validation feedback?** | ❌ No         | ✅ Yes           |

**Rule of Thumb**: If the form primarily has **switches, radio buttons, or select dropdowns** for **existing preferences**, use settings mode. If the form has **text inputs for new data**, use traditional mode.

---

## 🚀 Quick Migration Checklist

When migrating a view to settings mode:

- [ ] Confirm view is a settings/preferences page (not data entry)
- [ ] Add `settingsMode={true}` to Form component
- [ ] Add `description` fields to input definitions
- [ ] Update locale files with `{key}_desc` translation keys
- [ ] Test on mobile viewport (ensure responsive)
- [ ] Test dark mode
- [ ] Verify form submission still works
- [ ] Check validation error display

**Time per view**: 30 minutes - 2 hours depending on complexity

---

## 📞 Support

**Questions about whether a view should use settings mode?**

Ask yourself:

1. Is this configuring app behavior or entering new information?
2. Would this look natural in iOS Settings app?
3. Are the inputs primarily toggles/selects vs text entry?

If yes to all three → **Use settings mode**
If no to any → **Keep traditional mode**

---

**Status**: Infrastructure complete. Adoption is ongoing and opt-in. 🎉
