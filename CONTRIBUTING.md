# Contributing to tailorra

tailorra is a Tauri 2 + Angular 20 desktop app for tailoring resumes per job listing. The source repo (`Egibi-LLC/tailor`) is private; built installers and the docs site live in the public `Egibi-LLC/tailorra-releases` repo. Windows x64 is the only packaged target today.

## Dev environment

- **Node.js 20+**. CI builds with Node 20, so treat that as the floor. Local Node 24 works.
- **Rust stable** via rustup (`rustup default stable`).
- **Tauri 2 Windows prerequisites**: Microsoft C++ Build Tools (Desktop development with C++) and the WebView2 runtime, which ships with Windows 10/11. See <https://tauri.app/start/prerequisites/> for details and for Linux packages if you build there.

## Setup

```sh
git clone https://github.com/Egibi-LLC/tailor.git
cd tailor
npm install
npm run tauri dev
```

The first `tauri dev` compiles the Rust side from scratch and takes a while. Later runs are incremental.

## Dev loop

`npm run tauri dev` starts the Angular dev server on port 1420 (`npm run start` via `beforeDevCommand`) and opens the app window against it. Frontend changes live-reload; Rust changes trigger a recompile and app restart.

Dev and packaged builds use separate SQLite files in the same per-user app data directory (under the bundle identifier `dev.hubbard.tailor`): dev uses `tailor.db`, production uses `tailorra.db`. The frontend picks the file with Angular's `isDevMode()` in `db.service.ts`, so dev test workspaces never leak into a real install.

## Code layout

See [`ARCHITECTURE.md`](./ARCHITECTURE.md) for the code map. The short version:

- `src/`: Angular frontend. The renderer executes SQL directly through `tauri-plugin-sql`; that is deliberate, not a shortcut. AI calls, keychain access, file parsing, and PDF export go through `tauri::invoke` commands.
- `src-tauri/`: Rust backend. `lib.rs` registers every command; AI dispatch is per-command match arms over the provider enum (there is no provider trait). Migrations (15 as of this writing) are registered in `lib.rs` for both database files.
- `docs/adr/`: Architecture Decision Records. Structural changes get an ADR: copy `template.md`, take the next number, fill in context, decision, consequences.
- `docs/RELEASING.md`: the release process, end to end.

## Conventions

### Commits

- Message format: `YYYY_MM_DD_HHmm: <description>`, 24-hour Central time, no Conventional Commits prefixes. Get the real time right before committing: `date '+%Y_%m_%d_%H%M'` in Git Bash.
- Subject line only, roughly under 80 characters. No multi-paragraph bodies.
- No AI attribution trailers (`Co-Authored-By: Claude` or similar), ever.
- Push immediately after every commit.

### Code and prose

- Comments explain *why*, not *what*. Skip comments that restate the code. Public Rust functions get `///` rustdoc; public TypeScript exports get JSDoc (both feed the published API docs).
- No em-dashes or double-hyphens in prose: docs, comments, commit messages, UI copy. Restructure the sentence instead. Code syntax containing `--` (CLI flags, CSS custom properties) is fine.

### Repo-specific gotchas

- **Lucide icons must be registered.** Every `<lucide-icon name="...">` needs its icon imported and added to the `icons` map in `src/app/app.config.ts`. A missing registration throws at render time and looks like a CSS or change-detection bug (the page "corrects itself after navigating").
- **Wizard Save & Next protocol.** Wizard pages with draft state opt in by calling `wizard.registerNextSaveHandler(fn)` in `ngOnInit` and `registerNextSaveHandler(null)` in `ngOnDestroy`, so the footer's Next button saves their draft and stale handlers do not survive navigation. See `src/app/services/wizard.service.ts`.
- **Dark-mode CSS ordering.** `@media (prefers-color-scheme: dark)` blocks must come after the light-mode rules they override. Later same-specificity light rules silently win otherwise.

## CI

Three workflows in `.github/workflows/`:

- **`ci.yml`** runs on every push and PR to `main`: full Angular AOT build (`npm run build`, which includes template type-checking) plus `cargo check --locked` in `src-tauri`. This is what catches TS errors, Rust errors, and a stale `Cargo.lock` before release time.
- **`docs.yml`** runs on push and PR to `main`: builds TypeDoc and rustdoc and collects the ADRs. On `main` pushes and manual runs it publishes the result to GitHub Pages on the public releases repo, at <https://egibi-llc.github.io/tailorra-releases/>. rustdoc's source listings are stripped before publishing because this repo is private. You can build the same docs locally with `npm run docs` (or `docs:ts` / `docs:rust`).
- **`release.yml`** never runs on ordinary pushes. See Releases below.

## Releases

Full instructions are in [`docs/RELEASING.md`](./docs/RELEASING.md). The short path:

1. `npm run bump <version>` (keeps `package.json`, `tauri.conf.json`, `Cargo.toml`, `Cargo.lock`, and the About page fallback in sync).
2. Commit and push to `main`.
3. GitHub -> Actions -> **Release** -> Run workflow on `main`.

The workflow builds and minisign-signs the NSIS installer on a Windows runner, publishes it (draft first, then promoted) with `latest.json` to `Egibi-LLC/tailorra-releases`, and pushes the `v<version>` tag itself. Installed apps pick up the update through the in-app updater. Keys, secrets, the manual fallback, and troubleshooting are all in `docs/RELEASING.md`.

Note the installer is minisign-signed for the updater but not Authenticode-signed, so first-time downloads hit the Windows SmartScreen warning. That is known and deferred; do not spend time on it in a PR.

## Testing

- **Rust**: `cargo test` from `src-tauri/`. Unit tests exist where logic is self-contained, currently `ai/css_sanitize.rs`, `browser.rs`, and `parse/docx.rs`. CI does not run tests yet (build checks only), so run them locally before pushing backend changes.
- **TypeScript**: no JS test suite yet.
- **UI changes**: manual verification against the dev app (`npm run tauri dev`). Exercise the changed screen in both light and dark themes; dark is the default.

## Questions

Open an issue. Check [`ARCHITECTURE.md`](./ARCHITECTURE.md) and [`docs/adr/`](./docs/adr/) first; most design answers are there.
