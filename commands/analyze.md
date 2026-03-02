---
description: "Analyze drift between specs and implementation. Compares requirements against code to find divergence."
tools: []
scripts: []
---

# Spec Sync: Analyze Drift

Analyze drift between specifications and implementation. This command compares your spec requirements (FR-*, SC-*, acceptance scenarios) against the actual codebase to identify where they've diverged.

## User Input

$ARGUMENTS

## Context

Read the project structure to understand the codebase:
- Specs directory: Look for `specs/*/spec.md` files
- Implementation: Look for source files matching the project type

## Steps

### 1. Discover Specs

Find all spec files in the project:

```bash
find specs -name "spec.md" -type f 2>/dev/null | sort
```

For each spec, extract:
- Spec ID (directory name)
- Title (first heading)
- Functional requirements (FR-001, FR-002, etc.)
- Success criteria (SC-001, SC-002, etc.)
- Key acceptance scenarios

### 2. Analyze Implementation

For each spec, determine:
- Which requirements have corresponding implementation
- Which requirements appear unimplemented
- Which code features exist without spec coverage

Use these heuristics:
- CLI commands mentioned in spec → Check for corresponding Command classes
- Services mentioned in spec → Check for corresponding Service classes
- Entities/models mentioned → Check for corresponding entity files
- Test coverage → Check for corresponding test files

### 3. Detect Unspecced Code

Find code that doesn't match any spec:
- Commands not referenced in any spec
- Services not referenced in any spec
- Features that evolved beyond their spec

### 4. Generate Drift Report

Output a structured report in this format:

```markdown
# Spec Drift Report

Generated: [timestamp]
Project: [project name]

## Summary

| Category | Count |
|----------|-------|
| Specs Analyzed | X |
| Requirements Checked | X |
| ✓ Aligned | X (Y%) |
| ⚠️ Drifted | X (Y%) |
| ✗ Not Implemented | X (Y%) |
| 🆕 Unspecced Code | X |

## Detailed Findings

### Spec: [spec-id] - [title]

#### Aligned ✓
- FR-001: [description] → [implementation location]

#### Drifted ⚠️
- FR-002: Spec says "[spec text]" but code does "[actual behavior]"
  - Location: [file:line]
  - Severity: [minor|moderate|major]

#### Not Implemented ✗
- FR-003: [description]

### Unspecced Code 🆕

| Feature | Location | Lines | Suggested Spec |
|---------|----------|-------|----------------|
| [feature] | [path] | [count] | [spec-XXX] |

## Inter-Spec Conflicts

[List any contradictions between specs]

## Recommendations

1. [First recommendation]
2. [Second recommendation]
```

### 5. Write Report

Save the report to:
- `.specify/sync/drift-report.md` (human-readable)
- `.specify/sync/drift-report.json` (machine-readable)

## Output Format

The JSON report should follow this schema:

```json
{
  "generated": "ISO-8601 timestamp",
  "project": "project name",
  "summary": {
    "specs_analyzed": 0,
    "requirements_checked": 0,
    "aligned": 0,
    "drifted": 0,
    "not_implemented": 0,
    "unspecced_features": 0
  },
  "specs": [
    {
      "id": "spec-id",
      "title": "Spec Title",
      "aligned": ["FR-001", "FR-002"],
      "drifted": [
        {
          "requirement": "FR-003",
          "spec_text": "what spec says",
          "actual": "what code does",
          "location": "path/to/file.cs:123",
          "severity": "moderate"
        }
      ],
      "not_implemented": ["FR-004"]
    }
  ],
  "unspecced": [
    {
      "feature": "FeatureName",
      "location": "path/to/file",
      "lines": 150,
      "suggested_spec": "spec-013"
    }
  ],
  "conflicts": []
}
```

## Example Usage

```
/speckit.sync.analyze

/speckit.sync.analyze --spec 011-register-write

/speckit.sync.analyze --json
```

## Notes

- This is a read-only analysis - no files are modified
- Large codebases may take time to analyze
- Use `--spec <id>` to analyze a single spec
- Use `--json` for machine-readable output only
