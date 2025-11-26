# release/bM.F.0 (repo: {app-repo-name})

| Release # | PR #    | Release Owner   | Date        | Scope           | Tag    |
| --------- | ------- | --------------- | ----------- | --------------- | ------ |
| `vM.F.0`  | `#pppp` | {release-owner} | {date-time} | Major / Feature | bM.F.0 |

- **Source Branch:** `main`
- **Working Branch:** `release/bM.F.0`
- **Target Branch:** `main`
- **Tag:** `bM.F.0`

## Summary

Describe the goal of this release, key business outcomes, and high-level scope. Note whether this is the first feature release under a new Major version or an incremental feature bundle into an existing Major release.

{Release summary}

## Contents

### Features Included

Links to all the Features included in this Release:

- [feature/{nnnn}--{feature-short-name}](../feature/feature-nnnn--feature-short-name.md)
- [feature/{nnnn}--{feature-short-name}](../feature/feature-nnnn--feature-short-name.md)
- [feature/{nnnn}--{feature-short-name}](../feature/feature-nnnn--feature-short-name.md)
- ...

### Hotfixes Included

Links to all the Hotfixes included in this Release:

- [hotfix/{nnnn}--{hotfix-short-name}](../hotfix/hotfix-nnnn--hotfix-short-name.md)
- [hotfix/{nnnn}--{hotfix-short-name}](../hotfix/hotfix-nnnn--hotfix-short-name.md)
- [hotfix/{nnnn}--{hotfix-short-name}](../hotfix/hotfix-nnnn--hotfix-short-name.md)
- ...

### Breaking Changes / Migration Notes

Document any changes requiring customer action, data migrations, API contract updates, or configuration adjustments.

{Breaking changes}

## Testing & Validation

Summarize the testing performed in `release/bM.F.0` across all Circles. Attach logs or reports where applicable.

- **Regression / Unit Coverage:** {details}
- **Integration Tests:** {details}
- **System Acceptance Testing (SAT):** {details}
- **User Acceptance Testing (UAT):** {details / customer sites}
- **Performance / Security:** {details}

## Deployment Plan

Outline how this release will move from Beta to Production.

- **Target Production Date:** {date}
- **Deployment Steps:** {ordered list or reference to runbook}
- **Rollback Plan:** {steps}
- **Monitoring / Verification:** {metrics, dashboards}
- **Notified Stakeholders:** {list}

## Post-Release Checklist

- [ ] Merge `release/bM.F.0` → `main`
- [ ] Tag `release/bM.F.0` as `vM.F.0` on `main`
- [ ] Merge `release/bM.F.0` → `develop`
- [ ] Tag `release/bM.F.0` as `aM.F.0` on `develop`
- [ ] Update CHANGELOG / release notes
- [ ] Announce release (internal + customer)

## Appendix

### Linked Issues / PRs

Provide a table or list mapping GitHub Issues to PRs for traceability.

| Issue # | Type    | PR #    | Notes  |
| ------- | ------- | ------- | ------ |
| `#nnnn` | Feature | `#pppp` | {note} |
| `#nnnn` | Hotfix  | `#pppp` | {note} |

### Supporting Documents

- SSP references
- Test evidence
- Rollout approval
