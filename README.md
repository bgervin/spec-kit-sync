# Spec Kit Sync Extension

Detect and resolve drift between specs and implementation. AI-assisted resolution with human approval.

## Problem

As codebases evolve through vibe coding, integration testing, and bug fixing, implementation drifts from specifications:
- Specs become documentation fossils
- New developers can't trust specs
- AI agents working from specs produce conflicting code
- The "regenerate from spec" promise breaks

## Solution

Bidirectional sync between specs and code:
- **Analyze**: Detect where specs and code diverge
- **Propose**: AI suggests resolutions (update spec or fix code)
- **Apply**: Apply approved changes with human oversight
- **Conflicts**: Surface inter-spec contradictions
- **Backfill**: Generate specs from unspecced code

## Installation

```bash
# From catalog (once published)
specify extension add sync

# From URL
specify extension add --from https://github.com/bgervin/spec-kit-sync/archive/refs/tags/v0.1.0.zip

# Development mode
specify extension add --dev /path/to/spec-kit-sync
```

## Commands

### `speckit.sync.analyze`

Analyze drift between specs and implementation.

```
/speckit.sync.analyze                    # Full analysis
/speckit.sync.analyze --spec 011         # Single spec
/speckit.sync.analyze --json             # Machine-readable output
```

Output:
- `.specify/sync/drift-report.md` - Human-readable report
- `.specify/sync/drift-report.json` - Machine-readable data

### `speckit.sync.propose`

Generate resolution proposals for detected drift.

```
/speckit.sync.propose                    # Generate proposals
/speckit.sync.propose --interactive      # Review one-by-one
/speckit.sync.propose --strategy backfill-all
```

Output:
- `.specify/sync/proposals.md`
- `.specify/sync/proposals.json`

### `speckit.sync.apply`

Apply approved resolutions.

```
/speckit.sync.apply --dry-run            # Preview changes
/speckit.sync.apply                      # Apply approved proposals
/speckit.sync.apply --auto-commit        # Commit after apply
```

### `speckit.sync.conflicts`

Detect inter-spec and spec-vs-design-doc conflicts.

```
/speckit.sync.conflicts                  # Find conflicts
/speckit.sync.conflicts --include-design-docs
/speckit.sync.conflicts resolve 1 --supersede docs/design.md
```

### `speckit.sync.backfill`

Generate a spec from unspecced code.

```
/speckit.sync.backfill reconciliation    # By feature name
/speckit.sync.backfill src/Services/X.cs # By file path
/speckit.sync.backfill feature --create  # Create spec files
```

## Resolution Strategies

| Strategy | When to Use | Action |
|----------|-------------|--------|
| **Backfill** | Code is right, spec is outdated | Update spec to match code |
| **Align** | Spec is right, code diverged | Generate task to fix code |
| **Supersede** | Newer doc replaces older | Mark old spec as superseded |
| **Human** | Can't determine automatically | Surface for review |

## Ralph Loop Integration

Enable post-iteration drift checking in `sync-config.yml`:

```yaml
ralph:
  post_iteration_check: true
  on_drift: pause  # or: backfill, warn
```

In your Ralph loop script:

```bash
# After each iteration
speckit sync analyze --json > .sync-report.json

if jq -e '.summary.drifted > 0' .sync-report.json; then
  echo "⚠️ Drift detected"
  # Handle based on config
fi
```

## Configuration

Copy the config template to your project:

```bash
mkdir -p .specify/extensions/sync
cp sync-config.template.yml .specify/extensions/sync/sync-config.yml
```

Key settings:
- `proposals.default_strategy` - How to handle drift
- `proposals.min_confidence` - When to require human review
- `ralph.on_drift` - What to do in loops

## Workflow Example

```
# 1. Analyze current state
/speckit.sync.analyze

# Output: "Found 31 drifted requirements, 36 unspecced features"

# 2. Check for conflicts
/speckit.sync.conflicts

# Output: "2 conflicts between specs and design docs"

# 3. Generate proposals
/speckit.sync.propose --interactive

# Review each proposal, approve/reject/modify

# 4. Apply approved changes
/speckit.sync.apply --dry-run
/speckit.sync.apply

# 5. Backfill missing specs
/speckit.sync.backfill reconciliation --create
/speckit.sync.backfill hints --create

# 6. Commit
git add specs/ && git commit -m "sync: resolve drift and backfill specs"
```

## License

MIT
