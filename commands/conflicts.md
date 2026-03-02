---
description: "Detect and surface conflicts between specs or between specs and design docs"
tools: []
scripts: []
---

# Spec Sync: Detect Conflicts

Find contradictions between specs, or between specs and design documents. Surface them for human resolution.

## User Input

$ARGUMENTS

## Context

Read all spec files and design documents:
- `specs/*/spec.md`
- `docs/plans/*.md` or `docs/design/*.md`
- Any files with "design" or "plan" in the name

## Steps

### 1. Extract Requirements from All Sources

For each spec, extract:
- All FR-* requirements with full text
- All SC-* success criteria
- Key constraints and assumptions
- Out of scope items

For design docs, extract:
- Design decisions
- Constraints
- Behavioral specifications
- Any "MUST", "SHALL", "SHOULD" statements

### 2. Build Requirement Index

Create a semantic index of all requirements, keyed by:
- Feature area (extraction, routing, register, etc.)
- Entity type (document, transaction, row, etc.)
- Behavior type (create, update, validate, etc.)

### 3. Detect Direct Conflicts

Look for contradictions:

**Type 1: Same Feature, Different Behavior**
- Spec A says "one row per document"
- Spec B says "split transactions produce multiple rows"
- Conflict: Row cardinality

**Type 2: Obsolete Constraints**
- Spec A says "5 extraction fields: Date, Vendor, Total, Tax, Currency"
- Design doc says "type-aware fields, 4-8 per document type"
- Conflict: Field count and composition

**Type 3: Scope Overlap**
- Spec A includes feature X
- Spec B also includes feature X
- Conflict: Unclear ownership

**Type 4: Implicit Conflicts**
- Spec A assumes auth is always valid
- Spec B adds offline mode
- Conflict: Assumption violated

### 4. Determine Resolution Path

For each conflict:

**SUPERSEDE**: Newer document replaces older
- Check dates, version numbers
- Design docs often supersede original specs
- Later specs refine earlier specs

**MERGE**: Combine into single authoritative source
- When both have valid parts
- Create unified spec

**DEPRECATE**: Mark older as obsolete
- When requirement is no longer relevant
- Document why it was removed

**HUMAN_REQUIRED**: Can't determine automatically
- Architectural decisions
- Trade-offs involved
- Missing context

### 5. Generate Conflict Report

```markdown
# Spec Conflict Report

Generated: [timestamp]

## Summary

| Conflict Type | Count |
|---------------|-------|
| Same Feature, Different Behavior | X |
| Obsolete Constraints | X |
| Scope Overlap | X |
| Implicit Conflicts | X |

## Conflicts

### Conflict 1: Extraction Field Count

**Sources**:
- `specs/008-field-extraction/spec.md` (FR-003)
- `docs/plans/2026-02-19-type-aware-extraction-design.md`

**Description**:
Spec 008 defines 5 fixed extraction fields for all document types.
Design doc introduces type-aware extraction with 4-8 fields per type.

**Evidence**:

From spec-008:
> FR-003: System MUST extract the following fields: Date, Vendor, Total, Tax, Currency

From design doc:
> Document Types & Extraction Fields
> ### Receipt
> Fields: Date, Vendor, Total, Tax, Currency, Category
> ### Bill
> Fields: Date, Vendor, Description, Total, Due Date, Currency, Category

**Suggested Resolution**: SUPERSEDE

The design doc (dated 2026-02-19) supersedes spec-008 for extraction fields.
Recommend updating spec-008 to reference the design doc or incorporating
the type-aware fields directly.

**Action Required**:
- [ ] Mark spec-008/FR-003 as superseded by design doc
- [ ] Update spec-008 with type-aware field definitions
- [ ] Or create new spec-013 for type-aware extraction

---

### Conflict 2: Register Row Cardinality

[... additional conflicts ...]

## Resolution Tracking

Track resolution decisions:

| Conflict | Resolution | Decided By | Date |
|----------|------------|------------|------|
| Conflict 1 | SUPERSEDE | pending | - |

## Recommendations

1. Schedule a 30-min review session to resolve conflicts
2. Consider consolidating specs 008 and design doc
3. Add explicit "supersedes" metadata to specs
```

### 6. Save Report

Write to:
- `.specify/sync/conflicts.md`
- `.specify/sync/conflicts.json`

## Conflict Resolution Commands

After reviewing conflicts, mark resolutions:

```
/speckit.sync.conflicts resolve 1 --supersede docs/plans/2026-02-19-type-aware-extraction-design.md

/speckit.sync.conflicts resolve 2 --merge spec-011
```

## Example Usage

```
/speckit.sync.conflicts

/speckit.sync.conflicts --include-design-docs

/speckit.sync.conflicts --spec 008
```
