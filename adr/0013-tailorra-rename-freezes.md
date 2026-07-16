# ADR-0013: Tailorra rename completion and frozen legacy identifiers

- **Status**: Accepted
- **Date**: 2026-07-15
- **Supersedes**: the *name* half of [ADR-0009](./0009-app-name-identifier.md); the identifier half stands.

## Context

The app was renamed from `tailor` to `tailorra` after real users had installed it. By the time the rename completed (2026-07-14, commit `2d35ae7`), everything user-visible already said tailorra: productName, window title, installer names, install dir, uninstall registry key, the `.tailorra` workspace extension, and the updater channel (`Egibi-LLC/tailorra-releases`).

A handful of internal identifiers still say `tailor`. Each one is load-bearing for existing installs: renaming it would silently orphan user data or credentials on the next auto-update. They are invisible to users, so renaming them has zero user benefit and real harm.

## Decision

Renamed to `tailorra` (done): Rust crate + lib (`tailorra`, `tailorra_lib`, so the installed exe is `tailorra.exe`; NSIS migrates shortcuts), npm package, Angular project + dist path, extension branding, capture env var (`TAILORRA_CAPTURE_PORT`, old name still honored as fallback), docs.

**Frozen forever at their legacy values. Do not "clean these up":**

| Identifier | Value | What breaks if renamed |
| --- | --- | --- |
| Bundle identifier (`tauri.conf.json`) | `dev.hubbard.tailor` | Owns `%APPDATA%\dev.hubbard.tailor\` (production SQLite DB) and the WebView2 profile. Rename = every installed app boots with an empty database after update. |
| OS keyring service (`keys.rs`, `capture.rs`) | `tailor` | AI provider keys and the capture pairing token vanish from the app's view; users must re-enter keys and re-pair the extension. |
| Settings/localStorage key prefix (`settings.service.ts`) | `tailor:*` / `tailor.*` | Settings silently reset; one-time migrations re-fire. |
| Workspace backup marker (`.tailorra` files) | `tailor_workspace_version` | Previously exported workspace backups stop loading. |
| Dev database filename | `tailor.db` | Dev-machine data only; renamed for no benefit. |

Also intentionally kept (internal, zero user visibility, high churn to change): the `--tailor-*` CSS custom properties, `tailor-*` component selectors, and the `tailorpdf` internal URI scheme used by the PDF export engine.

## Consequences

- **Pro:** Existing installs update seamlessly; data, keys, settings, and old backups all survive.
- **Con:** The codebase permanently contains both names. Anyone tempted to unify them must read this ADR first: the split is deliberate.
- If a migration is ever truly needed (e.g. an identifier change forced by a platform), it requires explicit first-run migration code (copy the old app-data dir, re-read old keyring entries) before the old values are dropped.
