---
description: "Propose resolutions for detected drift. AI generates spec updates or code changes."
tools: []
scripts: []
---

# Spec Sync: Propose Resolutions

Generate resolution proposals for each drift item. The AI analyzes each divergence and proposes either a spec update (backfill) or code change (align).

## User Input

$ARGUMENTS

## Prerequisites

Run `speckit.sync.analyze` first to generate the drift report.

## Context

Read the drift report:
- `.specify/sync/drift-report.json`

## Steps

### 1. Load Drift Report

Read the drift report from the previous analysis. If it doesn't exist, instruct the user to run `speckit.sync.analyze` first.

### 2. For Each Drifted Requirement

Analyze the drift and determine resolution direction:

#### Backfill (Code → Spec)

Recommend when:
- Code has working, tested features not in spec
- Design docs exist that supersede the spec
- The code represents intentional evolution
- Tests pass for the current behavior

Generate:
- Updated spec requirement text
- New acceptance scenarios if needed
- Rationale for why code is authoritative

#### Align (Spec → Code)

Recommend when:
- Spec was recently reviewed/approved
- Code appears to be a bug or misunderstanding
- The spec represents agreed architectural decisions
- Changing the spec would violate constraints

Generate:
- Code change description or diff
- Task for implementation
- Rationale for why spec is authoritative

#### Human Decision Required

Flag when:
- Both interpretations seem valid
- Architectural trade-offs involved
- Insufficient context to decide
- Conflicting design documents

Generate:
- Summary of both options
- Questions that would help decide
- Request for human input

### 3. For Each Unspecced Feature

Propose a new spec:
- Suggested spec ID (next available number)
- Draft title
- Draft user stories based on code behavior
- Draft requirements extracted from implementation
- Confidence score

### 4. For Each Not-Implemented Requirement

Determine:
- Is it still needed? (may be obsolete)
- Should it be removed from spec?
- Should it be implemented?

Propose accordingly.

### 5. Generate Proposal Document

Output format:

```markdown
# Drift Resolution Proposals

Generated: [timestamp]
Based on: drift-report from [timestamp]

## Summary

| Resolution Type | Count |
|-----------------|-------|
| Backfill (Code → Spec) | X |
| Align (Spec → Code) | X |
| Human Decision | X |
| New Specs | X |
| Remove from Spec | X |

## Proposals

### Proposal 1: [spec-id]/[requirement]

**Direction**: BACKFILL | ALIGN | HUMAN_DECISION

**Current State**:
- Spec says: "[spec text]"
- Code does: "[actual behavior]"

**Proposed Resolution**:

[For BACKFILL: new spec text]
[For ALIGN: code change description]
[For HUMAN: questions and options]

**Rationale**: [why this direction]

**Confidence**: HIGH | MEDIUM | LOW

**Action**:
- [ ] Approve
- [ ] Reject
- [ ] Modify

---

### Proposal 2: New Spec for [feature]

**Direction**: NEW_SPEC

**Feature**: [feature name]
**Location**: [code path]

**Draft Spec**:

# Feature Specification: [title]

## User Scenarios

### User Story 1 - [story]
[generated from code behavior]

## Requirements

- **FR-001**: [extracted from code]

**Confidence**: MEDIUM

**Action**:
- [ ] Approve and create spec
- [ ] Reject
- [ ] Modify
```

### 6. Save Proposals

Write to:
- `.specify/sync/proposals.md` (human-readable)
- `.specify/sync/proposals.json` (machine-readable)

## Interactive Mode

When run with `--interactive`, present each proposal one at a time and prompt for approval:

```
Proposal 1/15: spec-011/FR-001

Direction: BACKFILL (Code → Spec)

Spec says: "one row per document"
Code does: "supports split transactions with multiple rows"

Proposed update:
  - FR-001: System MUST append one or more rows per extracted document.
            For split transactions, multiple rows share a LinkID.

Confidence: HIGH

[A]pprove / [R]eject / [M]odify / [S]kip / [Q]uit?
```

## Example Usage

```
/speckit.sync.propose

/speckit.sync.propose --interactive

/speckit.sync.propose --strategy backfill-all
```
