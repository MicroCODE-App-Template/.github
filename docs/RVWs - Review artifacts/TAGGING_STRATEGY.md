# Git Tagging Strategy for Migration

## Tag Naming Convention

### Format

```
migration-phase-{phase-number}-{short-description}
```

### Examples

```
migration-phase-0-prep
migration-phase-1-foundation
migration-phase-3-id-system
migration-phase-4-timestamps
migration-phase-6-settings
migration-phase-9-bug-fixes
migration-phase-10-testing-complete
```

---

## Tagging Points

### 1. Pre-Migration Baseline

**Tag**: `migration-baseline`
**When**: Before starting any migration work
**Command**:

```bash
git tag -a migration-baseline -m "Baseline before ladders integration migration"
git push origin migration-baseline
```

### 2. After Each Phase Completion

**Tag**: `migration-phase-{N}-{description}`
**When**: After completing and testing each phase
**Command**:

```bash
git tag -a migration-phase-1-foundation -m "Phase 1: mcode logging foundation complete"
git push origin migration-phase-1-foundation
```

### 3. Before High-Risk Changes

**Tag**: `migration-checkpoint-{description}`
**When**: Before starting high-risk phases (3, 4, 6)
**Command**:

```bash
git tag -a migration-checkpoint-before-id-migration -m "Checkpoint before ID system migration"
git push origin migration-checkpoint-before-id-migration
```

### 4. After Successful Testing

**Tag**: `migration-tested-{phase-number}`
**When**: After all tests pass for a phase
**Command**:

```bash
git tag -a migration-tested-phase-3 -m "Phase 3 tested and verified"
git push origin migration-tested-phase-3
```

### 5. Final Completion

**Tag**: `migration-complete`
**When**: All phases complete, all tests passing, ready to merge
**Command**:

```bash
git tag -a migration-complete -m "Migration complete - all phases done"
git push origin migration-complete
```

---

## Complete Tag List

### Phase Tags (Create after each phase)

- `migration-phase-0-prep`
- `migration-phase-1-foundation`
- `migration-phase-2-documentation`
- `migration-phase-3-id-system`
- `migration-phase-4-timestamps`
- `migration-phase-5-type-state`
- `migration-phase-6-settings`
- `migration-phase-7-api-revoke`
- `migration-phase-8-additional-features`
- `migration-phase-9-bug-fixes`
- `migration-phase-10-testing`
- `migration-phase-11-cleanup`

### Checkpoint Tags (Create before high-risk phases)

- `migration-checkpoint-before-id-migration`
- `migration-checkpoint-before-timestamps`
- `migration-checkpoint-before-settings`

### Tested Tags (Create after successful testing)

- `migration-tested-phase-3`
- `migration-tested-phase-4`
- `migration-tested-phase-6`
- `migration-tested-phase-9`
- `migration-tested-all`

---

## Tagging Workflow

### Standard Workflow

```bash
# 1. Start a phase
git checkout feature/ladders-integration
# ... do work ...

# 2. Commit work
git add .
git commit -m "Phase X: [Description]"

# 3. Test
npm test
# ... verify manually ...

# 4. If tests pass, create phase tag
git tag -a migration-phase-X-description -m "Phase X complete and tested"
git push origin migration-phase-X-description

# 5. Continue to next phase
```

### High-Risk Phase Workflow

```bash
# 1. Before starting high-risk phase
git tag -a migration-checkpoint-before-phase-X -m "Checkpoint before Phase X"
git push origin migration-checkpoint-before-phase-X

# 2. Do work
# ... make changes ...

# 3. Commit
git add .
git commit -m "Phase X: [Description]"

# 4. Test thoroughly
npm test
# ... extensive testing ...

# 5. If tests pass, create tested tag
git tag -a migration-tested-phase-X -m "Phase X tested and verified"
git push origin migration-tested-phase-X

# 6. Create phase completion tag
git tag -a migration-phase-X-description -m "Phase X complete"
git push origin migration-phase-X-description
```

---

## Rollback Procedures

### View All Tags

```bash
git tag -l "migration-*"
```

### View Tag Details

```bash
git show migration-phase-3-id-system
```

### Rollback to a Tag

#### Option 1: Create New Branch from Tag (Recommended)

```bash
# Create new branch from known good point
git checkout -b feature/ladders-integration-fix migration-phase-3-id-system

# Or create branch from checkpoint
git checkout -b feature/ladders-integration-fix migration-checkpoint-before-id-migration
```

#### Option 2: Reset Current Branch (Destructive)

```bash
# WARNING: This will lose commits after the tag
git reset --hard migration-phase-3-id-system
```

#### Option 3: Revert Specific Commits

```bash
# Find commits to revert
git log migration-phase-3-id-system..HEAD

# Revert specific commit
git revert <commit-hash>
```

### Compare Current State to Tag

```bash
# See what changed since tag
git diff migration-phase-3-id-system

# See commits since tag
git log migration-phase-3-id-system..HEAD
```

---

## Tag Annotation Best Practices

### Good Tag Messages

```bash
git tag -a migration-phase-3-id-system -m "Phase 3: ID system migration complete
- All models use _id as primary key
- Foreign key references updated
- All tests passing
- Ready for Phase 4"
```

### Include in Message

- What was done
- Key changes
- Test status
- Known issues (if any)
- Next steps

---

## Helper Scripts

### Create Phase Tag Script

Save as `.git/hooks/post-phase-tag.sh`:

```bash
#!/bin/bash
# Usage: ./post-phase-tag.sh <phase-number> <description>

PHASE=$1
DESC=$2
TAG_NAME="migration-phase-${PHASE}-${DESC}"

echo "Creating tag: ${TAG_NAME}"
git tag -a "${TAG_NAME}" -m "Phase ${PHASE}: ${DESC} complete and tested"
git push origin "${TAG_NAME}"
echo "Tag created and pushed: ${TAG_NAME}"
```

Make executable:

```bash
chmod +x .git/hooks/post-phase-tag.sh
```

Usage:

```bash
./.git/hooks/post-phase-tag.sh 3 "id-system"
```

### List Migration Tags Script

Save as `scripts/list-migration-tags.sh`:

```bash
#!/bin/bash
echo "=== Migration Tags ==="
git tag -l "migration-*" | sort -V
echo ""
echo "=== Latest Tag ==="
git describe --tags --abbrev=0
```

### Rollback Helper Script

Save as `scripts/rollback-to-tag.sh`:

```bash
#!/bin/bash
# Usage: ./rollback-to-tag.sh <tag-name>

TAG=$1

if [ -z "$TAG" ]; then
  echo "Usage: ./rollback-to-tag.sh <tag-name>"
  echo "Available tags:"
  git tag -l "migration-*"
  exit 1
fi

echo "Creating new branch from tag: ${TAG}"
BRANCH_NAME="rollback-${TAG}-$(date +%Y%m%d-%H%M%S)"
git checkout -b "${BRANCH_NAME}" "${TAG}"
echo "Created branch: ${BRANCH_NAME}"
echo "You can now work from this point or merge back to main branch"
```

---

## Tag Organization

### By Type

```bash
# List all phase tags
git tag -l "migration-phase-*"

# List all checkpoint tags
git tag -l "migration-checkpoint-*"

# List all tested tags
git tag -l "migration-tested-*"
```

### By Phase Number

```bash
# List tags sorted by phase number
git tag -l "migration-phase-*" | sort -V
```

### Latest Tag

```bash
# Get latest migration tag
git describe --tags --abbrev=0 --match "migration-*"
```

---

## Recommended Tagging Schedule

### Minimal (Time-Constrained)

Tag only at:

1. `migration-baseline` - Before starting
2. `migration-checkpoint-before-id-migration` - Before Phase 3
3. `migration-checkpoint-before-timestamps` - Before Phase 4
4. `migration-checkpoint-before-settings` - Before Phase 6
5. `migration-phase-9-bug-fixes` - After bug fixes
6. `migration-complete` - When done

### Standard (Recommended)

Tag at:

1. `migration-baseline` - Before starting
2. After each phase completion (12 tags)
3. Before high-risk phases (3 checkpoint tags)
4. After successful testing of high-risk phases (3 tested tags)
5. `migration-complete` - When done

**Total**: ~20 tags

### Comprehensive (Maximum Safety)

Tag at:

1. `migration-baseline` - Before starting
2. After each phase completion (12 tags)
3. Before AND after each high-risk phase (6 checkpoint tags)
4. After each testing session (11 tested tags)
5. Before each commit (optional, use lightweight tags)
6. `migration-complete` - When done

**Total**: ~30+ tags

---

## Tag Cleanup (After Migration)

### Keep These Tags

- `migration-baseline` - Historical reference
- `migration-complete` - Final state marker
- `migration-phase-3-id-system` - Major milestone
- `migration-phase-6-settings` - Major milestone

### Can Delete (After Merge)

- Intermediate phase tags (if not needed)
- Checkpoint tags (if not needed)
- Tested tags (if not needed)

### Delete Tag

```bash
# Delete local tag
git tag -d migration-phase-1-foundation

# Delete remote tag
git push origin --delete migration-phase-1-foundation
```

---

## Integration with Migration Status

Update `MIGRATION_STATUS.md` with tag information:

```markdown
### Phase 3: ID System Migration

**Status**: ✅ Complete
**Tag**: `migration-phase-3-id-system`
**Tested Tag**: `migration-tested-phase-3`
**Rollback Point**: `migration-checkpoint-before-id-migration`
```

---

## Quick Reference Commands

```bash
# Create annotated tag
git tag -a migration-phase-X-desc -m "Message"

# Push tag to remote
git push origin migration-phase-X-desc

# List all migration tags
git tag -l "migration-*"

# View tag details
git show migration-phase-X-desc

# Checkout tag (creates detached HEAD)
git checkout migration-phase-X-desc

# Create branch from tag
git checkout -b new-branch migration-phase-X-desc

# Compare current to tag
git diff migration-phase-X-desc

# See commits since tag
git log migration-phase-X-desc..HEAD

# Delete local tag
git tag -d migration-phase-X-desc

# Delete remote tag
git push origin --delete migration-phase-X-desc
```

---

## Example: Complete Tagging Session

```bash
# Start Phase 3
git checkout feature/ladders-integration

# Create checkpoint before starting
git tag -a migration-checkpoint-before-id-migration \
  -m "Checkpoint before ID system migration"
git push origin migration-checkpoint-before-id-migration

# Do work...
# ... make changes ...

# Commit
git add .
git commit -m "Phase 3: Migrate ID system to _id"

# Test
npm test
# ... all tests pass ...

# Create tested tag
git tag -a migration-tested-phase-3 \
  -m "Phase 3 tested: All ID migrations working"
git push origin migration-tested-phase-3

# Create phase completion tag
git tag -a migration-phase-3-id-system \
  -m "Phase 3: ID system migration complete
- All models use _id as primary key
- Foreign keys updated to use _id
- All tests passing"
git push origin migration-phase-3-id-system

# Continue to Phase 4...
```

---

## Safety Tips

1. **Always push tags immediately** - Don't lose them locally
2. **Use annotated tags** - Include meaningful messages
3. **Tag after testing** - Only tag when code is verified
4. **Tag before risky changes** - Create checkpoints
5. **Document in status file** - Keep track of which tag = which state
6. **Test rollback procedure** - Know it works before you need it

---

This tagging strategy gives you multiple safe rollback points throughout the migration process.
