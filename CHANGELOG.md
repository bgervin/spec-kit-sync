# Changelog

All notable changes to spec-kit-sync will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.1.0] - 2026-03-01

### Added

- **`speckit.sync.analyze`** — Detect drift between specs and implementation
  - Scans all specs and compares requirements against code
  - Outputs drift report in markdown and JSON formats
  - Supports single-spec analysis with `--spec` flag

- **`speckit.sync.propose`** — AI-assisted resolution proposals
  - Four resolution strategies: Backfill, Align, Supersede, Human Required
  - Interactive mode for one-by-one review
  - Configurable confidence thresholds

- **`speckit.sync.apply`** — Apply approved resolutions
  - Dry-run mode for preview
  - Auto-commit option
  - Safety checks before modification

- **`speckit.sync.conflicts`** — Detect inter-spec contradictions
  - Finds conflicts between specs
  - Supports design doc inclusion with `--include-design-docs`
  - Resolution via supersede declarations

- **`speckit.sync.backfill`** — Generate specs from unspecced code
  - Creates complete spec structure: `spec.md`, `plan.md`, `quickstart.md`, `tasks.md`
  - Extracts requirements from code, tests, and documentation
  - Preview mode by default, `--create` to write files

- Configuration template (`sync-config.template.yml`)
- Ralph loop integration support
- Comprehensive README with real-world example

[Unreleased]: https://github.com/bgervin/spec-kit-sync/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/bgervin/spec-kit-sync/releases/tag/v0.1.0
