# ADR-0006: Submission tracking (soft freeze via immutable snapshot)

- **Status**: Accepted
- **Date**: 2026-04-27

## Context

When a user actually sends a tailored version to a company, what happens? Options:

- **Hard freeze**: editing forks into a new version
- **Soft freeze (snapshot)**: immutable snapshot tied to listing+date; live version stays editable
- **Tag-only**: mark "submitted" as metadata, allow continued editing (loses what was sent)
- **No tracking**: list of versions only

Months later, an interview might land. The user needs to know exactly what was sent: typos, claims, formatting. Tag-only and no-tracking lose this. Hard freeze is too rigid (a typo right after sending shouldn't force a fork).

## Decision

**Soft freeze via immutable snapshot.** A `version_snapshots` row is written when the user marks a version as submitted, capturing the full serialized state, target company, target role, and timestamp. The live `tailored_versions` row stays mutable.

Snapshots are protected from modification at the storage layer: `BEFORE UPDATE` and `BEFORE DELETE` SQLite triggers, with app-layer assertion as a backup.

## Consequences

- **Pro:** Always know exactly what was sent
- **Pro:** Multi-submission supported; re-applying later writes a new snapshot
- **Pro:** Live drafting workflow not disrupted by submission events
- **Con:** Storage grows with each submission (acceptable; resumes are small)
- **Con:** Need trigger maintenance if SQLite version changes (low risk)
