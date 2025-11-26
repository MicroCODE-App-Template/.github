# bugfix/{nnnn}--{bugfix-short-name} (repo: {app-repo-name})

| Issue # | PR #    | Corrected by | Date        | Release # |
| ------- | ------- | ------------ | ----------- | --------- |
| `#nnnn` | `#pppp` | {dev-name}   | {date-time} | `bM.F.0`  |

- **Source Branch:** `feature/{nnnn}--{feature-short-name}`
- **Alternate Source:** `release/bM.F.0`
- **Working Branch:** `bugfix/{nnnn}--{bugfix-short-name}`
- **Target Branch:** `feature/{nnnn}--{feature-short-name}`, then `release/bM.F.0`, `develop`
- **Tag:** `bM.F.H`

## Summary

This document describes a Bug Fix applied to `{app-web-name}` while the release candidate branch `release/bM.F.0` is in Beta testing. Beta bugfixes are created from the affected `feature/*` branch (canonical source) to ensure fixes persist even if the `release/*` branch is abandoned.

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

- Beta systems / customer pilots affected: {list}
- Date/time applied to `release/bM.F.0`: {timestamp}
- Propagation plan after release promotion: {notes}
