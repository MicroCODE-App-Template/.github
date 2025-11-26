# hotfix/{nnnn}--{hotfix-short-name} (repo: {app-repo-name})

| Issue # | PR #    | Corrected by | Date        | Hotfix # |
| ------- | ------- | ------------ | ----------- | -------- |
| `#nnnn` | `#pppp` | {dev-name}   | {date-time} | `vM.F.H` |

- **Source Branch:** `main`
- **Working Branch:** `hotfix/{nnnn}--{hotfix-short-name}`
- **Target Branch:** `main`, then `develop`, `release/*`
- **Tag:** `vM.F.H`

## Summary

This document describes a Hot Fix applied to `{app-web-name}`. This hotfix is designed to be applied as a patch to a live production Web App on the `main` trunk and back-merged into `develop` and any open `release/*` branches per SSP. After merging to `main` and tagging `vM.F.H`, also merge this hotfix to `develop` (tag `aM.F.H`) and any open `release/*` branches (tag `bM.F.H`) per the workflow in DEVELOPER-GIT.md.

**Problem**: Summary of the problem as reported by the customer or internal testing. Include links to the GitHub Issue / customer ticket if applicable:

{Problem statement}

**Observed Behavior**: Describe in detail the AS-IS state...

{Observations}

- **Before**: the issue as recreated
<p align="left"><img src="./.images/before-nnnn.png" width="720" title="Before correction" style="border: 0.5px solid lightgray;"></p>

**Expected Behavior**: Describe in detail the TO-BE state...

{Expectations}

**Root Cause**: Describe the technical reason for the defect. Use the Five Whys to reach the true cause. Capture the answers briefly:

1. Why #1: ...
2. Why #2: ...
3. Why #3: ...
4. Why #4: ...
5. Why #5: ...

{Root Cause}

- **Cause**: the failing code or configuration...

```
Code causing the issue
```

**Resolution**: Describe the change to code, process, or configuration required to achieve the TO-BE ‘Expected Behavior’. Note any risk or back-out plan here.

{Resolution}

- **After**: the corrected code...

```
Code correcting the issue
```

- **Result**: the issue as corrected
<p align="left"><img src="./.images/after-nnnn.png" width="720" title="After correction" style="border: 0.5px solid lightgray;"></p>

**Verification & Tests**

- Unit Tests: {pass/fail details or links}
- Integration Tests: {details}
- SAT / UAT: {details}
- Logs / Evidence: [Link to logs or markdown]

**Impact & Rollout**

- Production systems / customers affected: {list}
- Deployment date/time: {timestamp}
- Back-out plan executed? {Yes/No}
