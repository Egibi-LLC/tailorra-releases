# ADR-0001: Storage (SQLite via tauri-plugin-sql)

- **Status**: Accepted
- **Date**: 2026-04-27

## Context

The app needs durable storage for:

- The master resume corpus (multi-table relational data: experiences, bullets, skills, education, projects, certifications, etc.)
- Per-listing tailored versions referencing master items
- Immutable submission snapshots
- Provenance metadata
- AI audit logs

Querying needs go beyond simple CRUD. Examples: "find bullets I haven't used in the last 5 versions", "all skills tagged backend", "items pending review". The schema also has natural integrity rules (e.g., tailored versions can only reference reviewed items, snapshots are immutable) that are best enforced at the storage layer.

## Decision

Use **SQLite** via [`tauri-plugin-sql`](https://docs.rs/tauri-plugin-sql/), with the database file in the OS-standard `app_data_dir`. Schema migrations are managed by the plugin's built-in migration support.

Alternatives considered:

- **JSON files on disk**: trivially simple v1, but querying needs ad-hoc indexing that gets painful as the corpus grows. Loses out on triggers and constraints for invariant enforcement.
- **Hybrid (JSON master + SQLite for history/metadata)**: combines complexity of both with weak benefit.

## Consequences

- **Pro:** Powerful querying for the lifetime of the app
- **Pro:** Single-file backup (copy the `.db`)
- **Pro:** Migration tooling exists; not a green-field problem
- **Pro:** Trigger-based enforcement of invariants (e.g., immutable snapshots)
- **Con:** Schema and migration discipline required from day 1
- **Con:** Slightly higher v1 effort than a JSON dump
