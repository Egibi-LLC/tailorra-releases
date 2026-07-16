# Architecture Decision Records

Each ADR captures one design decision with its context, the chosen option, and consequences. ADRs are append-only: when a decision changes, write a new ADR that supersedes the old one (do not edit history).

| # | Title | Status |
|---|-------|--------|
| [0001](./0001-storage-sqlite.md) | Storage (SQLite via tauri-plugin-sql) | Accepted |
| [0002](./0002-export-targets.md) | Export targets v1 | Accepted |
| [0003](./0003-listing-ingestion.md) | Job listing ingestion | Accepted |
| [0004](./0004-master-scope.md) | Master scope (single master per install) | Accepted |
| [0005](./0005-cover-letters.md) | Cover letters (v1.5, schema-ready) | Accepted |
| [0006](./0006-submission-tracking.md) | Submission tracking (soft freeze via immutable snapshot) | Accepted |
| [0007](./0007-provenance-ui.md) | Provenance UI (visible until reviewed) | Accepted |
| [0008](./0008-ai-safety-boundaries.md) | AI safety boundaries (enforced architecturally) | Accepted |
| [0009](./0009-app-name-identifier.md) | App name and Tauri identifier | Accepted |
| [0010](./0010-onboarding-api-key.md) | Onboarding API key (contextual prompt) | Accepted |
| [0011](./0011-ai-providers.md) | AI providers (multi-provider via Rust abstraction) | Accepted |
| [0012](./0012-default-template.md) | Default template and extensibility | Accepted |
| [0013](./0013-tailorra-rename-freezes.md) | Tailorra rename completion and frozen legacy identifiers | Accepted |

## Adding a new ADR

1. Copy [`template.md`](./template.md) to `<NNNN>-<slug>.md` with the next number
2. Fill in: status, date, context, decision, consequences
3. Add a row to the table above
4. Reference the ADR from any code or PR that depends on it

## Format

See [`template.md`](./template.md). Keep ADRs concise: context, decision, consequences. Link to longer-form docs if needed (`SPEC.md`, `ARCHITECTURE.md`).
