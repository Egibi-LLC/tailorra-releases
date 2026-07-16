# Architecture

A code map of tailorra as it exists today: what runs where, what talks to what, and where to look before changing something. The code is the authority; when this file and the code disagree, fix this file. Decision history lives in [`docs/adr/`](./docs/adr/). `SPEC.md` describes the app as built and carries a decision log of where the implementation diverged from the original design.

tailorra is a Tauri 2 + Angular 20 Windows desktop app for tailoring resumes per job listing. Crate, npm package, and product name are all `tailorra`; the bundle identifier stays `dev.hubbard.tailor` and the keychain service stays `tailor` (both frozen, see ADR-0009).

## Runtime topology

```
┌────────────────────────────────────────────────────────────┐
│  Angular 20 renderer (WebView2)                            │
│  - All UI, all business logic for CRUD                     │
│  - Runs SQL directly via tauri-plugin-sql                  │
│  - Renders resume HTML/Markdown/text/DOCX client-side      │
└──────────────────────┬─────────────────────────────────────┘
                       │ invoke('command', ...) / Tauri events
                       ▼
┌────────────────────────────────────────────────────────────┐
│  Rust host (src-tauri)                                     │
│  - AI provider calls (Anthropic HTTP, Claude CLI child)    │
│  - OS keychain (API keys, capture token)                   │
│  - File parsing (pdf/docx/doc/txt), plain file IO          │
│  - Transactional workspace import (import.rs)              │
│  - PDF printing via a hidden webview (pdf.rs)              │
│  - Capture HTTP server on 127.0.0.1:7341 (capture.rs)      │
│  - Updater plugin, window placement                        │
└────────────────────────────────────────────────────────────┘
```

**Where SQL runs, and why.** The renderer opens the SQLite database itself through `tauri-plugin-sql` (`src/app/services/db.service.ts`) and every data service executes its own statements. This is deliberate: the schema is owned by the migrations in Rust, but query logic lives next to the signals and components that consume it, and no Rust round-trip is needed per query. The cost is that the plugin's pooled connections cannot hold a transaction across invoke calls, so multi-statement flows in the renderer are sequences of autocommit statements.

**The import.rs exception.** Workspace import (restore a `.tailorra` backup) is the one flow where a mid-sequence failure would destroy data, so it does not run in the renderer. `src-tauri/src/import.rs` opens its own `sqlx` connection to the same database file and applies the whole wipe-plus-restore inside a single transaction. Backup content is treated as untrusted: INSERT column names come only from per-table whitelists mirroring the migrations, unknown keys are ignored for forward compatibility, and every row id is regenerated (UUID v4) with child references remapped, so importing the same backup twice cannot collide on primary keys. Any error rolls back and leaves the workspace untouched.

## Directory map

```
src/                          Angular renderer
  app/app.config.ts             bootstrap providers + global lucide icon registry
  app/app.routes.ts             route table (all lazy standalone components)
  app/app.component.*           shell: header, sidebar, boot sequence, update check
  app/services/                 DbService, ProjectsService, ten CRUD data services,
                                workspace IO, AI ops registry, capture, updater,
                                settings, theme, wizard, toasts
  app/master/                   master-corpus pages (personal info, experiences,
                                education, skills, projects, certs, other, import)
  app/tailor/                   listings, versions list, version editor, renderers.ts
  app/designer/                 style designer, 14 CSS skeletons, sample resume
  app/suggestions/              autocomplete-pool admin + file import parsers
  app/settings/                 appearance, AI providers, browser capture,
                                workspaces, about
  app/shared/                   wizard footer, workspace/setup menus, AI panels,
                                month picker, toasts, input formatters
  app/welcome/                  first-run / no-workspace screen
  app/getting-started/          wizard hub
src-tauri/
  src/lib.rs                    entry point: all command registration, migrations,
                                setup hook (state, capture server, window placement)
  src/import.rs                 transactional workspace import (see above)
  src/capture.rs                axum capture server + CaptureBridge event buffer
  src/browser.rs                open a Chromium browser for extension setup
  src/pdf.rs                    hidden-webview PDF printing (tailorpdf:// scheme)
  src/keys.rs                   keyring wrapper (service "tailor")
  src/ai/                       anthropic.rs, claude_cli.rs, cancel.rs,
                                css_sanitize.rs, mod.rs (ProviderId)
  src/parse/                    resume parsing: pdf, docx, legacy doc, url fetch
  migrations/                   001..015 sequential SQL migrations
  capabilities/default.json     main-window grants (sql execute/select, dialog,
                                opener, updater, process)
  tauri.conf.json               CSP, updater endpoint + pubkey, NSIS bundle,
                                extension/ bundled as resource
extension/                      MV3 Chrome extension (popup capture, options page)
data/suggestions/               seed values for the autocomplete pools
docs/adr/                       architecture decision records
docs/RELEASING.md               release flow
scripts/bump-version.mjs        version bump across files + cargo update -p tailorra
.github/workflows/              ci.yml, docs.yml, release.yml
```

## Invoke command surface

All commands are registered in `lib.rs` (`import_workspace` lives in `import.rs`). Grouped by area:

| Area | Commands |
|------|----------|
| Provider keys | `set_provider_key`, `clear_provider_key`, `has_provider_key`, `test_provider_connection` |
| AI operations | `ai_extract_master_items`, `ai_suggest_resume_items`, `ai_refine_text`, `ai_generate_field`, `ai_generate_personal_summary`, `ai_generate_cover_letter`, `ai_polish_resume`, `ai_generate_resume_style`, `ai_generate_resume_style_from_attachment`, `ai_tweak_resume_style`, `cancel_ai_operation` |
| Files and parsing | `extract_resume_text`, `extract_resume_text_from_bytes`, `list_resume_files_in_dir`, `fetch_listing_url`, `read_text_file`, `write_text_file`, `write_binary_file` |
| Workspace | `import_workspace` |
| PDF and documents | `render_resume_pdf_bytes`, `open_pdf_externally`, `convert_doc_to_pdf` |
| Capture and extension | `get_capture_status`, `get_capture_token`, `regenerate_capture_token`, `drain_pending_captures`, `export_browser_extension`, `open_browser_for_extension_setup` |
| Maintenance | `clear_webview_browsing_data` |

Notes: `write_text_file` and `write_binary_file` write via a same-directory temp file plus rename, so an interrupted write cannot corrupt an existing backup. Three commands currently have no renderer caller (`ai_generate_resume_style`, `ai_generate_resume_style_from_attachment`, `convert_doc_to_pdf`); they are working backend paths for the style-mimic-from-example flow, kept registered while that UI is out of the designer.

## AI providers

Two providers work today; there is no provider trait. Dispatch is a `match provider_id` inside each AI command in `lib.rs`, and `ProviderId` (`src/ai/mod.rs`) carries `OpenAI`, `Gemini`, and `OpenAiCompatible` as enum variants whose match arms return "not yet supported" errors. The frontend (`ai-providers.service.ts`) lists them as "Coming soon" and never sends them.

- **Anthropic API** (`src/ai/anthropic.rs`): direct Messages API client over `reqwest`, non-streaming, tool-use for structured output (extraction, suggestions). The API key is read from the OS keychain on every call and never reaches the renderer. `stop_reason` is checked so output truncated at `max_tokens` surfaces as an error instead of a silently half-finished result.
- **Claude Code CLI** (`src/ai/claude_cli.rs`): shells out to the local `claude` binary, prompt piped via stdin, JSON output parsed from stdout. No key is stored; auth is the user's existing `claude login` session, billed against their plan. `has_provider_key` for this provider means "`claude` auth check succeeds". Child processes are spawned with `CREATE_NO_WINDOW`, and cancellation kills the whole process tree (`taskkill /T` on Windows) so a `.cmd` shim cannot orphan the real process. Each call emits `ai-cli-call-start` / `ai-cli-call-end` events that feed the in-app transcript log (in-memory only, capped at 50 entries, never written to disk because prompts contain resume content).

Cross-cutting pieces: `cancel.rs` maps frontend-supplied operation ids to oneshot channels raced against the work with `tokio::select` (backs `cancel_ai_operation`); `css_sanitize.rs` strips content-hiding declarations and `@import` from AI-generated CSS before it reaches the renderer.

## Capture pipeline, end to end

Browser extension to database, hop by hop:

1. **Extension** (`extension/`, MV3, popup-driven): scrapes the current tab with site-specific selectors for LinkedIn, Indeed, Greenhouse, and Lever, falling back to generic body text. Endpoint and bearer token live in `chrome.storage.sync`; the options page has a two-stage connection test. The extension is bundled inside the installer as a resource; `export_browser_extension` copies it to app data for unpacked loading, and `open_browser_for_extension_setup` launches a Chromium browser (extensions-page address goes to the clipboard, since browsers refuse external navigation to `chrome://extensions`).
2. **axum server** (`capture.rs`): binds `127.0.0.1:7341` (override: `TAILORRA_CAPTURE_PORT`; the pre-rename `TAILOR_CAPTURE_PORT` still works). `GET /healthz` is unauthenticated; `POST /capture` requires `Authorization: Bearer <token>`, checked in constant time against a token stored in the keychain (service `tailor`, account `capture-token`) and mirrored in memory. The handler prefers the extension's pre-extracted text, else converts the HTML with `html2text`.
3. **Event or buffer** (`CaptureBridge` in `capture.rs`): if the webview listener is registered, the server emits the `listing-captured` Tauri event. If not (app still booting, page reloading), the capture is buffered, up to 100 entries, oldest dropped first, and the HTTP response marks it `queued`. The frontend calls `drain_pending_captures` right after registering its listener, so nothing is lost across the boot window.
4. **Frontend queue** (`capture.service.ts`): if no workspace is active (every launch starts on the Welcome screen with `activeProjectId === 0`), payloads are held in a pending array with a toast prompting the user to open a workspace; an effect flushes the queue once one is active.
5. **Database**: `JobListingsService.createFromCapture` inserts the `job_listings` row for the active project and a toast links to it. AI extraction is not triggered automatically; the user runs it from the listing.

## PDF export pipeline

The renderer builds the full resume HTML (`src/app/tailor/renderers.ts`: `renderHtml` emits the Classic base CSS first, then the version's custom CSS, so overrides win purely by cascade order) and invokes `render_resume_pdf_bytes`. `pdf.rs` stores the HTML under a one-time token, opens a hidden webview at `tailorpdf://localhost/<token>` (custom scheme, served from that store), waits for page load plus a 200 ms settle, then drives an engine-native print:

- **Windows**: WebView2 CDP `Page.printToPDF` via `with_webview`. Verified, the packaged path.
- **Linux**: WebKitGTK print operation. Code exists, untested.
- **macOS**: unimplemented.

Load and print have hard timeouts (20 s / 30 s) and an RAII guard cleans up the token and window. Bytes return to the renderer as base64; downloads go through the save dialog plus `write_binary_file`, and "open" goes through `open_pdf_externally` (temp file plus the opener plugin). No headless Chrome is involved anywhere; comments referring to it describe the retired predecessor.

Other export formats are rendered entirely client-side in `renderers.ts`: HTML, Markdown, plain text, and DOCX (via the `docx` npm package). `convert_doc_to_pdf` renders Word files to PDF through Word COM (PowerShell); it requires an installed Microsoft Word and exists for the style-mimic input path.

## Database

SQLite via `tauri-plugin-sql`. `db.service.ts` picks the file with Angular's `isDevMode()`: dev uses `sqlite:tailor.db`, packaged builds use `sqlite:tailorra.db`, so test data never leaks into a real install. All 15 migrations are registered in `lib.rs` for both URLs from one closure; the two lists must stay identical.

Multi-workspace: a `projects` table, with every content table scoped by `project_id` and `ON DELETE CASCADE`. The active workspace is an in-memory signal (`0` = none, never persisted across launches). Content-table primary keys are TEXT UUIDs generated in the renderer.

Schema evolution, in order:

| Migration | What it did |
|-----------|-------------|
| 001 | Master corpus: `projects`, `personal_info`, `experiences` + `experience_bullets`, `education`, `skills`, `personal_projects`, `certifications`, `other_items`, with provenance columns (`source`, `reviewed`, `original_text`); seeded a Default project |
| 002 | `suggestions` autocomplete pool (global, scoped by kind) |
| 003 | Tailoring: `job_listings`, `tailored_versions`, `version_items` (soft references to master rows), `version_snapshots` |
| 004 | `tailored_versions.custom_css` per-version style override |
| 005 | `style_templates` (saved designer output) |
| 006 | `application_state` seven-state lifecycle on versions; the legacy `status` column is frozen (its old CHECK rejects the new values, so it stays untouched at `'draft'`) |
| 007 | Per-version cover letter |
| 008 | Drops the seeded Default workspace when untouched |
| 009, 010 | Structured address parts on `personal_info` (street/city/state/zip) and city/state on experiences and education; composed `location` remains what renderers read |
| 011, 012 | Suggestion seed data |
| 013 | `experiences.is_remote` |
| 014 | `app_settings` key/value store (durable settings with a localStorage mirror) |
| 015 | Cleanup: rebuilds `style_templates` with the missing `project_id` FK (purging orphans), drops the never-written `version_snapshots` table and its triggers, replaces the dead `status` index with `(project_id, updated_at DESC)`, normalizes `app_settings` timestamps to ISO-8601 |

Timestamps are ISO-8601 UTC text with a trailing `Z`, lexicographically comparable with `Date.toISOString()`.

**Workspace backup**: Save/Open produce a `.tailorra` file (JSON inside, `tailor_workspace_version` marker; legacy `.json` files still load, validation is by content not extension). Export is renderer-side `SELECT *`; import goes through the transactional `import_workspace` command described above. OS file association for `.tailorra` is deferred to packaging work.

## Updater and distribution

The app checks for updates on launch (`AppComponent` calls `UpdateService.check()`), backed by `tauri-plugin-updater`. The endpoint is `latest.json` on the public releases repo, and the minisign public key is baked into `tauri.conf.json`; installers and updater artifacts are minisign-signed in CI (`createUpdaterArtifacts: true`). Install flow: run the registered wizard save handler to flush drafts, download and install the NSIS package, relaunch.

- Source repo: `Egibi-LLC/tailor` (private).
- Artifacts repo: `Egibi-LLC/tailorra-releases` (public) hosts releases and the GitHub Pages docs site at https://egibi-llc.github.io/tailorra-releases/.
- Stable installer link: https://github.com/Egibi-LLC/tailorra-releases/releases/latest/download/tailorra-x64-setup.exe (valid from the next release onward); all releases: https://github.com/Egibi-LLC/tailorra-releases/releases.
- The installer is minisign-signed for the updater but **not Authenticode-signed**, so Windows SmartScreen shows "Windows protected your PC" on first run; users click "More info" then "Run anyway". This is expected until code signing is bought.

Windows x64 NSIS is the only packaged target today.

## CI

Three workflows in `.github/workflows/`:

- **ci.yml**: on push/PR to `main`, runs `npm run build` (real Angular type check) and `cargo check --locked`, with a concurrency group.
- **docs.yml**: builds TypeDoc and rustdoc plus the ADRs into the public Pages site on the releases repo; rustdoc source pages are stripped so the private repo's source is not published.
- **release.yml**: dispatch-first on `main` (also accepts `v*` tags), guards the tag against `tauri.conf.json`'s version, verifies the lockfile with `cargo metadata --locked`, builds and signs the NSIS installer, publishes to the releases repo as a draft with all assets before flipping it public, and auto-tags on dispatch. See [`docs/RELEASING.md`](./docs/RELEASING.md).

Version bumps: `npm run bump <version>` rewrites `package.json`, `tauri.conf.json`, `Cargo.toml`, and the About page constant, validating all matches before writing any, then runs `cargo update -p tailorra` so `Cargo.lock` stays in step.

## Invariants worth knowing

Real rules, each learned the hard way:

- **Lucide icons must be registered.** Every icon name used in any `<lucide-icon>` must be registered in `src/app/app.config.ts`. An unregistered name throws during render and breaks the page in ways that look like a CSS or change-detection bug ("corrects after navigating").
- **Wizard save-handler protocol.** The guided setup wizard (11 steps, `wizard.service.ts`) navigates via a sticky footer whose Save & Next awaits a page-registered handler. Pages holding draft state call `wizard.registerNextSaveHandler(fn)` in `ngOnInit` and `registerNextSaveHandler(null)` in `ngOnDestroy`. The slot holds exactly one handler; a page that forgets to deregister leaves a stale handler running against a destroyed component. The same handler is also run before workspace switches and updater restarts.
- **Theming is attribute-driven, and CSS order matters.** Both themes are defined as `--tailor-*` custom properties on `:root[data-theme="light"]` / `:root[data-theme="dark"]` in `src/styles.css`, applied pre-boot by an inline script in `index.html` (dark is the default when the user never chose). Component styles must use the variables, never literal palette colors; hardcoded light-theme colors are invisible bugs until dark mode. When a `@media (prefers-color-scheme: dark)` block is ever used, it must come after the light rules, because later same-specificity rules win.
- **Dev and prod databases are different files.** `tailor.db` (dev) vs `tailorra.db` (packaged), selected by `isDevMode()`. Both share the port 7341 capture server and the same keychain token, so running dev and an installed build simultaneously makes them contend for the capture socket.
- **The renderer executes SQL.** `capabilities/default.json` grants `sql:allow-execute`/`sql:allow-select` to the main window. Any comment or doc claiming database access happens only in Rust is wrong; only the import transaction does.
- **`activeProjectId === 0` means no workspace.** It is the state of every launch until the user picks one. Data services do not enforce it; callers must. The capture path queues instead of inserting while it holds.
- **Legacy `tailored_versions.status` is frozen.** Write only `application_state`; the old column's CHECK constraint rejects the new lifecycle values.
- **Frozen identifiers.** Bundle id `dev.hubbard.tailor`, keychain service `tailor`, and the dev database filename predate the rename and must not be "cleaned up"; changing them strands user data and installs.
