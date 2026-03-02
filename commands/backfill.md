---
description: "Generate a new spec from unspecced code feature (backfill)"
tools: []
scripts: []
---

# Spec Sync: Backfill Spec

Generate a complete spec from an existing, unspecced code feature. Extracts intent from implementation, tests, and documentation.

## User Input

$ARGUMENTS

The argument should be one of:
- A feature name (e.g., "reconciliation", "hints")
- A file path (e.g., "src/Fina.Cli/Commands/Register/RegisterReconcileCommand.cs")
- An identifier from the drift report (e.g., "unspecced-3")

## Context

If a feature name is provided, search for relevant files:
- Commands with that name
- Services with that name
- Tests for that feature
- Any existing documentation

## Steps

### 1. Discover Feature Scope

Find all files related to the feature:

```bash
# Commands
find src -name "*[Feature]*Command.cs" -o -name "*[Feature]*.cs"

# Services
find src -name "*[Feature]*Service.cs"

# Tests
find tests -name "*[Feature]*Tests.cs"

# Docs
find docs -name "*[feature]*"
```

### 2. Analyze Implementation

Read the discovered files and extract:

**From Commands:**
- Command name and description
- Options and arguments
- Validation rules
- Error handling

**From Services:**
- Public methods (these become requirements)
- Business logic rules
- Dependencies on other services
- Edge case handling

**From Tests:**
- Test scenarios (become acceptance scenarios)
- Expected behaviors
- Edge cases tested

**From Existing Docs:**
- Any informal documentation
- README sections
- Comments explaining "why"

### 3. Infer User Stories

From the command/service analysis, generate user stories:

```markdown
### User Story 1 - [Primary Use Case] (Priority: P1)

As a user, I want to [action from main command]
so that [benefit inferred from feature purpose].

**Acceptance Scenarios**:

1. **Given** [precondition from tests], **When** [action], **Then** [expected result].
```

### 4. Extract Requirements

Convert implementation behaviors to requirements:

**From method signatures:**
```csharp
public async Task<ReconcileResult> ReconcileAsync(string profileName, ...)
```
→ `FR-001: System MUST support reconciliation per profile.`

**From validation:**
```csharp
if (!_authService.IsAuthenticated) throw ...
```
→ `FR-002: System MUST validate authentication before reconciliation.`

**From business logic:**
```csharp
var matches = _fuzzyMatcher.FindMatches(transactions, registryRows, threshold: 0.85);
```
→ `FR-003: System MUST use fuzzy matching with configurable threshold to match transactions.`

### 5. Generate Spec Structure

Follow the standard spec template:

```markdown
# Feature Specification: [Feature Name]

**Feature Branch**: `[spec-id]`
**Created**: [today's date]
**Status**: Backfilled
**Source**: Generated from existing implementation

## Backfill Notice

> ⚠️ This spec was generated from existing code via `speckit.sync.backfill`.
> It documents current behavior, not original intent.
> Review carefully and update to reflect desired behavior.

## User Scenarios & Testing

### User Story 1 - [Story] (Priority: P1)
[generated user story]

## Requirements

### Functional Requirements

- **FR-001**: [extracted requirement]
- **FR-002**: [extracted requirement]

### CLI Commands

- **FR-0XX**: System MUST provide `fina [command]` command...

## Key Entities

- **[Entity]**: [description from code]

## Dependencies

- **Spec [XXX]**: [dependency identified from imports/services]

## Success Criteria

- **SC-001**: [derived from tests or obvious metrics]

## Implementation Notes

> These notes capture current implementation details that may or may not
> belong in the spec long-term.

- [implementation detail 1]
- [implementation detail 2]
```

### 6. Determine Spec ID

Find the next available spec number:

```bash
ls specs/ | grep -E '^[0-9]+' | sort -n | tail -1
```

Suggest: `{next_number}-{feature-name}`

### 7. Generate Plan

Create `plan.md` documenting the implementation architecture:

```markdown
# Implementation Plan: [Feature Name]

**Branch**: `[spec-id]` | **Date**: [today] | **Spec**: [spec.md](spec.md)
**Context**: Backfilled from existing implementation. Documents current architecture.

## Summary

[Brief description of what the feature does]

## Technical Context

**Language/Version**: [detected from project]
**Primary Dependencies**:
- [Service 1] — [purpose]
- [Service 2] — [purpose]

**Performance Goals**: [if identifiable from code/tests]

## Architecture

### Service Layer

```
[Main Service]
├── [Dependency 1]
├── [Dependency 2]
└── [Dependency 3]
```

### Flow

1. [Step 1]
2. [Step 2]
3. [Step 3]

### Key Design Decisions

1. **[Decision 1]**: [rationale from code/comments]
2. **[Decision 2]**: [rationale]

## Project Structure

```text
src/[paths discovered]
tests/[paths discovered]
```

## Dependencies

- **Spec [XXX]**: [dependency]

## Testing

[Test file locations]

## Future Considerations

[Any TODOs or comments suggesting future work]
```

### 8. Generate Quickstart

For user-facing features (CLI commands), create `quickstart.md`:

```markdown
# Quickstart: [Feature Name]

[One-line description of what users can do]

## Prerequisites

1. [Prerequisite 1]
2. [Prerequisite 2]

## Basic Usage

```bash
# [Primary use case]
fina [command] [args]
```

## Options

| Option | Description |
|--------|-------------|
| `--option1` | [description] |
| `--option2` | [description] |

## Examples

### [Use case 1]
```bash
fina [command] [example]
```

### [Use case 2]
```bash
fina [command] [example]
```

## Tips

- [Tip 1]
- [Tip 2]
```

Skip `quickstart.md` for internal/non-user-facing features.

### 9. Output Options

**Preview Mode (default):**
Display generated spec, plan, quickstart, and tasks for review.

**Create Mode (`--create`):**
1. Create `specs/{id}/spec.md`
2. Create `specs/{id}/plan.md`
3. Create `specs/{id}/quickstart.md` (if user-facing)
4. Create `specs/{id}/tasks.md` (with review task)
5. Report location

### 10. Generate Review Task

Add a task to review the backfilled spec:

```markdown
# Tasks

## Review Backfilled Spec

- [ ] Review generated user stories for accuracy
- [ ] Verify requirements match intended behavior (not just current behavior)
- [ ] Remove implementation notes that don't belong in spec
- [ ] Add any missing requirements
- [ ] Mark spec status as "Draft" or "Approved"
```

## Example Usage

```
/speckit.sync.backfill reconciliation

/speckit.sync.backfill src/Fina.App/Services/Reconciliation/ReconciliationService.cs

/speckit.sync.backfill reconciliation --create

/speckit.sync.backfill hints --spec-id 014
```

## Notes

- Backfilled specs should always be reviewed by a human
- The spec documents current behavior, which may include bugs
- Use this as a starting point, not a final spec
- Consider whether the feature should be split into multiple specs
- `plan.md` documents architecture decisions; useful for onboarding and future changes
- `quickstart.md` is user documentation; skip for internal-only features
