# Tagging Quick Reference Card

## Tag Naming Format
```
migration-phase-{N}-{description}
migration-checkpoint-{description}
migration-tested-phase-{N}
```

## Essential Tags to Create

### 1. Before Starting
```bash
git tag -a migration-baseline -m "Baseline before migration"
git push origin migration-baseline
```

### 2. After Each Phase (When Tests Pass)
```bash
git tag -a migration-phase-1-foundation -m "Phase 1 complete"
git push origin migration-phase-1-foundation
```

### 3. Before High-Risk Phases (3, 4, 6)
```bash
git tag -a migration-checkpoint-before-id-migration -m "Checkpoint"
git push origin migration-checkpoint-before-id-migration
```

### 4. After Testing High-Risk Phases
```bash
git tag -a migration-tested-phase-3 -m "Phase 3 tested"
git push origin migration-tested-phase-3
```

### 5. When Complete
```bash
git tag -a migration-complete -m "Migration complete"
git push origin migration-complete
```

## Rollback Commands

### View Available Tags
```bash
git tag -l "migration-*"
```

### Create Branch from Tag (Safe)
```bash
git checkout -b fix-branch migration-phase-3-id-system
```

### Reset to Tag (Destructive - Use Carefully)
```bash
git reset --hard migration-phase-3-id-system
```

### See What Changed Since Tag
```bash
git diff migration-phase-3-id-system
git log migration-phase-3-id-system..HEAD
```

## Quick Workflow

```bash
# 1. Before risky change
git tag -a migration-checkpoint-before-X -m "Checkpoint"
git push origin migration-checkpoint-before-X

# 2. Do work and commit
git add . && git commit -m "Phase X: Changes"

# 3. Test
npm test

# 4. If tests pass, tag
git tag -a migration-phase-X-desc -m "Phase X complete"
git push origin migration-phase-X-desc
```

## Minimal Tagging (Time-Constrained)

Only tag these critical points:
1. `migration-baseline` - Before starting
2. `migration-checkpoint-before-id-migration` - Before Phase 3
3. `migration-checkpoint-before-timestamps` - Before Phase 4
4. `migration-checkpoint-before-settings` - Before Phase 6
5. `migration-phase-9-bug-fixes` - After bug fixes
6. `migration-complete` - When done

## Standard Tagging (Recommended)

Tag at:
- Baseline
- After each phase (12 tags)
- Before high-risk phases (3 checkpoints)
- After testing high-risk phases (3 tested tags)
- Complete

**Total: ~20 tags**
