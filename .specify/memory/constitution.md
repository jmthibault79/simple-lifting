<!--
Version change: 1.1.0 → 1.2.0 (MINOR: two new principles added)
List of modified principles: none
Added principles: IX. Learning & Knowledge Sharing, X. Evolving with Best Practices
Added sections: none
Removed sections: none
Templates requiring updates: plan-template.md (added learning/discovery gate), spec-template.md (optional: add learning capture requirement)
Follow-up TODOs: Document Spec Kit best practices review SOP by 2026-04-11
-->
# Simple Lifting Constitution

## Core Principles

### I. Library-First
Every feature starts as a standalone library; Libraries must be self-contained, independently testable, documented; Clear purpose required - no organizational-only libraries

### II. CLI Interface
Every library exposes functionality via CLI; Text in/out protocol: stdin/args → stdout, errors → stderr; Support JSON + human-readable formats

### III. Test-First (NON-NEGOTIABLE)
TDD mandatory: Tests written → User approved → Tests fail → Then implement; Red-Green-Refactor cycle strictly enforced

### IV. Integration Testing
Focus areas requiring integration tests: New library contract tests, Contract changes, Inter-service communication, Shared schemas

### V. Observability, VI. Versioning & Breaking Changes, VII. Simplicity
Text I/O ensures debuggability; Structured logging required; Or: MAJOR.MINOR.BUILD format; Or: Start simple, YAGNI principles

### VIII. Data Integrity & Format Preservation
User-provided data MUST be preserved in its original format (recorded unit, timezone, encoding); unit conversions and format transformations occur ONLY as viewing/display functions; source data never modified retroactively

### IX. Learning & Knowledge Sharing
Every feature/task captures learning outcomes: patterns discovered, unexpected gotchas, tool explorations, domain insights; Documentation of decisions (why chosen, alternatives considered) required; Knowledge must be discoverable for team and future-self

### X. Evolving with Best Practices
All development must reference current Spec Kit best practices; Constitution reviewed monthly against latest standards; Changes integrated and documented; Goal: stay current with tooling evolution while maintaining project consistency

## Additional Constraints
Technology stack requirements, compliance standards, deployment policies, etc.

## Development Workflow
Code review requirements, testing gates, deployment approval process, etc.

## Governance
Constitution supersedes all other practices; Amendments require documentation, approval, migration plan.

All PRs/reviews must verify compliance; Complexity must be justified.

**Monthly Review Cycle** (every 4 weeks from ratification date):
- Constitution reviewed against current Spec Kit best practices
- Updates captured in templates and guidance docs
- Changes committed with rationale in commit message and sync report

Use `/.specify/templates/commands/speckit.constitution.md` for runtime development guidance.

**Version**: 1.2.0 | **Ratified**: 2026-03-04 | **Last Amended**: 2026-03-11
