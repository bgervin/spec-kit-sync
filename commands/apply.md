---
description: "Apply approved drift resolutions to specs and/or code"
tools: []
scripts: []
---

# Spec Sync: Apply Resolutions

Apply approved resolutions from the proposals file. Updates specs or generates implementation tasks.

## User Input

$ARGUMENTS

## Prerequisites

1. Run `speckit.sync.analyze` to generate drift report
2. Run `speckit.sync.propose` to generate proposals
3. Review and mark proposals as approved in `.specify/sync/proposals.json`

## Context

Read:
- `.specify/sync/proposals.json` - proposals with approval status
- Original spec files that need updating

## Steps

### 1. Load Approved Proposals

Read proposals and filter to those marked `approved: true`.

If no proposals are approved:
- In interactive mode: prompt to review proposals first
- In batch mode: exit with "No approved proposals to apply"

### 2. Apply Backfill Proposals

For each approved BACKFILL proposal:

1. Read the original spec file
2. Locate the requirement being updated
3. Replace with proposed text
4. Add any new acceptance scenarios
5. Update the spec's "Status" or "Last Modified" metadata
6. Write the updated spec

Track changes:
```json
{
  "spec": "spec-011",
  "requirement": "FR-001",
  "before": "original text",
  "after": "updated text",
  "applied_at": "timestamp"
}
```

### 3. Apply New Spec Proposals

For each approved NEW_SPEC proposal:

1. Create new spec directory: `specs/{spec-id}/`
2. Write `spec.md` with generated content
3. Create empty `tasks.md` placeholder
4. Add to git (if in a git repo)

### 4. Generate Implementation Tasks for Align Proposals

For ALIGN proposals (spec → code), don't modify code directly. Instead:

1. Generate a task file: `.specify/sync/align-tasks.md`
2. Each task describes the code change needed
3. Optionally create GitHub issues if `--create-issues` is passed

Task format:
```markdown
## Task: Align [spec-id]/[requirement]

**Spec Requirement**: [FR-XXX]
**Current Code**: [description of current behavior]
**Required Change**: [description of needed change]
**Files to Modify**: [list of files]
**Estimated Effort**: [small/medium/large]

### Acceptance Criteria
- [ ] [criterion 1]
- [ ] [criterion 2]
```

### 5. Handle Supersede Resolutions

For SUPERSEDE proposals:

1. Add `superseded_by` field to old spec metadata
2. Add cross-reference in new spec
3. Optionally mark old spec as deprecated

### 6. Generate Apply Report

```markdown
# Sync Apply Report

Applied: [timestamp]

## Changes Made

### Specs Updated
| Spec | Requirement | Change Type |
|------|-------------|-------------|
| spec-011 | FR-001 | Modified |
| spec-011 | FR-015 | Added |

### New Specs Created
- spec-013-reconciliation

### Implementation Tasks Generated
- 3 tasks in `.specify/sync/align-tasks.md`

### Not Applied
| Proposal | Reason |
|----------|--------|
| spec-008/FR-003 | Not approved |

## Next Steps

1. Review updated specs
2. Commit changes: `git add specs/ && git commit -m "sync: apply drift resolutions"`
3. Implement tasks in align-tasks.md
```

### 7. Save Report

Write to:
- `.specify/sync/apply-report.md`
- `.specify/sync/apply-report.json`

## Options

- `--dry-run`: Show what would be applied without making changes
- `--create-issues`: Create GitHub issues for ALIGN tasks
- `--auto-commit`: Commit spec changes automatically
- `--spec <id>`: Apply only proposals for a specific spec

## Example Usage

```
/speckit.sync.apply --dry-run

/speckit.sync.apply

/speckit.sync.apply --auto-commit --create-issues
```

## Safety

- Always creates backups before modifying specs
- Backups stored in `.specify/sync/backups/`
- Use `--dry-run` first to preview changes
- All changes are logged for audit
