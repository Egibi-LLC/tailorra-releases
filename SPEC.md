# Specification

tailorra is a Windows desktop app for customizing your resume to fit specific job listings. Tauri 2 shell, Angular 20 frontend, SQLite storage, pre-1.0.

You keep one master corpus with every job, project, skill, and accomplishment from your career. When you apply somewhere, the app helps you produce a version focused on that listing: selecting the relevant items, ordering them sensibly, and using AI to draft, refine, and style the result. The AI helps with the writing but is never allowed to invent facts, and you review every AI-generated change before it lands in something you send out.

This document describes the app as implemented. Work that is designed but not built lives in the Roadmap section at the end. The original design decisions are recorded in `docs/adr/`; the decision log at the bottom of this file notes where the implementation diverged from them.

## Core principles

1. **The AI doesn't get to invent things.** It can pick from what you've told it, reorder it, and rewrite phrasing, but it must not add jobs, change dates, invent metrics, or make up technologies. Today this is enforced through strict system prompts, structured outputs (tool use on the Anthropic path), and mandatory user review; AI-generated CSS additionally passes through a Rust-side sanitizer that strips content-hiding declarations. An automated verbatim fact check on AI output is designed but not built (see Roadmap).
2. **One big master, many small versions.** The master corpus is everything you've ever done. Each tailored version is a slice of it for one specific job, never adding new content, just selecting and rephrasing.
3. **You see every AI change before it lands.** Items the AI extracts from an uploaded resume enter the master flagged unreviewed, with visible badges, until you accept or edit them. AI item suggestions for a version show up in a review panel with a rationale before you apply them, and applying is undoable. Refined or generated text appears in the editor for you to keep or discard.
4. **Application tracking, not snapshots.** Each tailored version carries an application lifecycle (draft, applied, interviewing, offer, accepted, rejected, archived) with timestamps for when you applied and when the state last changed. The original design called for immutable submission snapshots; that mechanism was never wired up and the table was dropped in migration 015 in favor of this lifecycle (see decision log, row 6).

## Stack

- **Frontend**: Angular 20, SCSS, lucide-angular (every icon must be registered in `app.config.ts`)
- **Desktop shell**: Tauri 2 (Rust backend)
- **Storage**: SQLite via `tauri-plugin-sql`, in the OS `app_data_dir` for the frozen identifier `dev.hubbard.tailor`. Dev builds use `tailor.db`, packaged builds `tailorra.db`. The Angular renderer executes SQL directly through the plugin; this is deliberate architecture, not an accident. The Rust side runs no queries except during workspace import (see below).
- **AI**: two working providers, dispatched per command in `src-tauri/src/lib.rs` (there is no provider trait; see decision log, row 11). API keys never touch the renderer.
- **Key storage**: OS keychain via the `keyring` crate (Windows Credential Manager), service `tailor`, one entry per provider.
- **Naming**: crate, npm package, and product are all `tailorra`. The bundle identifier stays `dev.hubbard.tailor` because it determines the app data directory and updater channel (ADR-0009).
- **Dev port**: 1420 (Tauri default)

## AI providers

### Anthropic API

Direct Messages API client in Rust (`src-tauri/src/ai/anthropic.rs`). The key is stored in the OS keychain and read per request. Structured operations (master extraction, item suggestion) use tool use with JSON schemas; text operations return prose. Non-streaming.

### Claude Code CLI

Passthrough to a locally installed `claude` binary (`src-tauri/src/ai/claude_cli.rs`). No API key is stored; authentication is the user's existing `claude login` session, so a Claude Pro or Max subscription covers usage. The app discovers the binary, checks auth, pipes prompts over stdin, and parses JSON output. Every CLI call emits start and end events that feed the in-app AI log.

### Not implemented

OpenAI, Google Gemini, and OpenAI-compatible endpoints exist only as enum variants; every command returns a "not yet supported" error for them. See Roadmap.

### Operations

All AI work goes through Tauri commands: master extraction from resume text, item suggestion for a version against a listing, style generation (from a text description or from an uploaded example rendered to an image), style tweaking, text refine, field generation, personal summary, cover letter generation, and final HTML polish. Every operation takes a frontend-supplied operation id and can be cancelled mid-flight through a shared cancellation registry.

### AI log

A panel available from any screen shows the prompt and response of every AI call in the session, so you can always see exactly what was sent to the model and what came back. Behavior (such as auto-open on AI actions) is configurable in Settings.

## Data model

Fifteen sequential SQL migrations, embedded in the binary and registered for both the dev and prod database files. Schema evolution is additive: `tauri-plugin-sql` wraps each migration in a transaction, so columns are added alongside legacy ones rather than recreating tables (migration 006 documents why).

```
projects                 : workspaces; every content table is scoped by project_id
personal_info            : name, contact, address parts, links, summary (one per workspace)
experiences              : jobs, roles, military service (structured + prose, remote flag)
experience_bullets       : bullets per experience, with provenance + reviewed flag
education                : institutions, degrees, dates, location
skills                   : name + category + optional proficiency
personal_projects        : side projects, with description prose
certifications           : name, issuer, dates, IDs
other_items              : awards, publications, volunteer work, languages
job_listings             : raw listing text from paste, URL fetch, or browser capture
tailored_versions        : per-listing versions; application_state lifecycle,
                           cover_letter text, custom_css styling
version_items            : items selected into a version (soft references to master rows)
style_templates          : named CSS templates saved from the style designer,
                           per workspace (FK added in migration 015)
suggestions              : global autocomplete pools (skill categories, titles,
                           degrees, issuers), seeded from data/suggestions/
app_settings             : key/value store for durable app settings
```

Dropped: `version_snapshots` (created in migration 003 for the original submission-snapshot design, never written by any code, removed in migration 015).

Conventions: TEXT primary keys are UUIDs generated in the renderer; timestamps are ISO-8601 UTC text comparable lexicographically; provenance columns (`source`, `reviewed`) mark AI-extracted rows until the user accepts them. `project_id` scoping makes every feature multi-workspace.

## Workspaces

The app supports multiple workspaces (the `projects` table). Each launch starts on a Welcome screen where you pick or create one; all master data, listings, versions, and style templates are scoped to the active workspace.

**Backup and restore**: a workspace exports to a single `.tailorra` file (JSON inside; legacy `.json` files still load, validated by content, not extension). Import runs through a dedicated Rust command (`src-tauri/src/import.rs`) that opens its own connection to the database and applies the wipe plus restore inside one transaction, so a failed import rolls back and leaves the workspace untouched. Backup content is treated as untrusted: insert columns come only from per-table whitelists derived from the migrations, unknown keys are ignored for forward compatibility, and every row id is regenerated with child references remapped, so the same backup can be imported into two workspaces of one database without collisions.

## Guided setup wizard

An optional guided mode walks a new user through eleven steps: connect an AI provider, add personal info, at least one experience, education, skills, projects, certifications, other items, a job listing, a tailored version, and a style. Step status is computed live from the actual data, so the wizard picks up wherever the data already is. Pages with draft state register a Save & Next handler (`wizard.registerNextSaveHandler`) so advancing saves the page first. Guided mode can be turned off and resumed at any time.

## Master corpus

Seven editor pages (personal info, experiences with bullets, education, skills, projects, certifications, other) with full manual CRUD, autocomplete backed by the suggestions pools, and inline AI assists for refining or generating prose fields.

**Resume import**: point the app at a file (PDF, DOCX, legacy DOC, TXT, MD), a folder, or drag and drop. Text extraction is pure Rust (pdf-extract, a quick-xml DOCX walker, a hand-rolled reader for legacy .doc). The AI then extracts structured items, which are bulk-inserted flagged `reviewed = 0` with natural-key dedup against existing rows. Unreviewed AI items carry badges until accepted or edited; a cleanup action removes any AI-extracted items you never accepted.

## Job listings

Three ways in:

- **Paste**: manual entry of listing text.
- **URL fetch**: the backend fetches the page and converts it to text.
- **Browser capture**: the app runs a local HTTP server on `127.0.0.1:7341` and bundles a Chrome (MV3) extension you can export from Settings and load unpacked. The extension scrapes the listing (site-specific extractors for LinkedIn, Indeed, Greenhouse, and Lever, with a generic fallback) and posts it to the app, authenticated by a bearer token that is generated in Settings and stored in the OS keychain. Captured listings appear as new rows with a toast.

## Tailored versions

Creating a version from a listing opens the version editor:

- **Item selection**: check master items into the version (experiences with per-bullet control, education, skills, projects, certifications, other). Select-all and per-section toggles.
- **AI suggestion**: the AI receives the corpus with stable ids plus the listing text and proposes a selection with a rationale. You review the counts and rationale before applying; apply replaces the current selection and the confirmation toast offers Undo, which restores the previous selection.
- **Cover letter**: per-version plain text, written by hand or AI-generated against the listing, refinable, exportable to PDF.
- **Application lifecycle**: move the version through draft, applied, interviewing, offer, accepted, rejected, or archived. `applied_at` anchors "applied N days ago"; `state_changed_at` tracks movement. The versions list filters by state.
- **Styling**: pick a saved style template, edit custom CSS directly, or jump into the style designer with the current CSS.

## Style designer

A standalone workshop for resume appearance:

- **14 built-in CSS skeletons** (classic, modern, bold, editorial, compact, sidebar, minimal, banner, executive, ats, timeline, tech, right-sidebar, warm) rendered as live thumbnails.
- **Preview sources**: a built-in sample resume, your master corpus, or a real tailored version.
- **AI direction**: describe the look you want, or upload an example resume; the AI generates or adjusts CSS to match. Word documents are rendered to PDF via Word COM automation (Windows with Microsoft Word installed) so the model works from the actual appearance rather than raw markup. A refine loop with history lets you iterate.
- **Output**: apply CSS straight onto a version, or save it as a named template in the `style_templates` table (source-tagged as AI-mimicked, AI-described, or manual) for reuse across versions in the workspace.

All AI-produced CSS passes through the sanitizer before use.

## Export

From the version editor:

- **HTML, Markdown, plain text**: rendered client-side from the selected items.
- **DOCX**: built client-side with the docx library.
- **PDF**: rendered by the embedded webview. The backend loads the resume HTML into a hidden window over a custom `tailorpdf://` scheme and drives the print pipeline (WebView2 CDP `Page.printToPDF` on Windows). No external browser dependency. The Linux path (WebKitGTK print) exists but is untested; macOS is unimplemented.
- **Cover letter PDF**: through the same pipeline with a letter template.

An optional AI polish pass cleans up the final HTML before PDF rendering; its output is sanitized and the result is shown before download.

## Appearance

Light and dark themes plus a follow-the-OS option. Dark is the default. Because of CSS ordering, `@media (prefers-color-scheme: dark)` blocks must come last in component styles.

## Distribution and updates

- **Source**: private repo `Egibi-LLC/tailor`.
- **Artifacts**: public repo `Egibi-LLC/tailorra-releases` hosts the releases and the GitHub Pages docs site at https://egibi-llc.github.io/tailorra-releases/ (TypeDoc, rustdoc with source pages stripped, and the ADRs).
- **Installer**: Windows x64 NSIS installer, the only packaged target today. Releases list: https://github.com/Egibi-LLC/tailorra-releases/releases. Stable latest-installer link: https://github.com/Egibi-LLC/tailorra-releases/releases/latest/download/tailorra-x64-setup.exe (valid from the next release onward; older releases used a versioned asset name).
- **Auto-update**: the app checks the releases repo on launch via the Tauri updater plugin. Updates are minisign-signed; the public key is baked into `tauri.conf.json` and the manifest is `latest.json` on the latest release. Download, install, and relaunch happen in-app.
- **SmartScreen**: the installer is minisign-signed for the updater but not Authenticode-signed, so Windows SmartScreen shows "Windows protected your PC" on first run. Users click "More info", then "Run anyway". This is expected until the app gets a code-signing certificate (see Roadmap).

### CI

Three workflows in `.github/workflows/`:

- `ci.yml`: build checks on pushes and pull requests.
- `docs.yml`: builds TypeDoc, rustdoc, and the ADRs and publishes them to the Pages site on the releases repo.
- `release.yml`: dispatch-first releases from `main`; builds and signs the installer, creates the release as a draft, publishes it, and tags automatically. `docs/RELEASING.md` describes the flow, including the version bump (`npm run bump`, which also runs `cargo update -p tailorra` so `Cargo.lock` stays in step).

## Multi-user posture

- Distributed desktop app; each install is one user with local data.
- No auth, no cloud sync, no telemetry.
- App data lives in the OS-standard location for `dev.hubbard.tailor`; API keys and the capture token live in the OS keychain.

## Decision log

Locked 2026-04-27; full records in `docs/adr/`. The last column notes where the built app diverged.

| # | Decision | Choice (2026-04-27) | Implementation status |
|---|----------|---------------------|-----------------------|
| 1 | Storage | SQLite via `tauri-plugin-sql` | As decided. The renderer executes SQL directly through the plugin (deliberate); workspace import is the one Rust-side database path. |
| 2 | Export targets | Markdown, DOCX, PDF via print, plain text, HTML; Typst PDF in v1.5 | All five built. PDF prints through the hidden embedded webview (WebView2 CDP), no dialog and no external browser. The Typst plan is superseded by this pipeline. |
| 3 | Listing ingestion | Paste + file upload | Built, and exceeded: URL fetch and the browser-extension capture flow (both originally out of scope) are implemented. |
| 4 | Master scope | Single master per install; `project_id` in schema for later | The multi-workspace UI landed. Workspaces are first-class, with per-workspace backup files. |
| 5 | Cover letters | v1.5, schema-ready | Built early, as a `cover_letter` column on `tailored_versions` (migration 007) rather than the reserved separate table. |
| 6 | Submission tracking | Soft freeze via immutable `version_snapshots` (ADR-0006) | Diverged. Snapshots were never wired up; submission tracking became the seven-state `application_state` lifecycle with `applied_at` and `state_changed_at` (migration 006). The dead table was dropped in migration 015. |
| 7 | Provenance UI | Visible until reviewed; detail on demand; never in exports (ADR-0007) | Partially built. `source` and `reviewed` flags with badges and accept actions work. The `original_text` audit columns are reserved but never written (noted in migration 015); there is no change-history view. |
| 8 | AI safety boundaries | Schema split + automated output validation (ADR-0008) | Partially diverged. The structured-vs-prose schema split holds and prompts mark immutable fields. The automated verbatim output check was not built; enforcement is prompts, mandatory review, and the CSS sanitizer. See Roadmap. |
| 9 | App name + identifier | `tailor`, `dev.hubbard.tailor` (ADR-0009) | The product, crate, and npm package were renamed `tailorra`; the bundle identifier is frozen at `dev.hubbard.tailor` as ADR-0009 intended, since it pins the app data directory and update channel. |
| 10 | Onboarding API key | Contextual, deferrable | Built as the wizard's first step (connect a provider), skippable; the Claude CLI provider needs no key at all, only a local `claude login` session. |
| 11 | AI providers | Rust-side `AIProvider` trait; Anthropic, OpenAI, Gemini native plus OpenAI-compatible adapter (ADR-0011) | Diverged. Two providers exist: the Anthropic API and the Claude Code CLI (which the original design did not anticipate). There is no trait; dispatch is per-command match arms in `lib.rs`. OpenAI, Gemini, and OpenAI-compatible are enum stubs that return errors. See Roadmap. |
| 12 | Default template + extensibility | One built-in "Classic" template; templates as first-class data bundles in `app_data_dir/templates/` (ADR-0012) | Diverged in mechanism, kept in spirit. Styling is CSS over a fixed HTML class contract. Fourteen built-in skeletons live in the style designer, and user styles are rows in the `style_templates` table, not filesystem bundles. Template import/export is still open (Roadmap). |

## Roadmap

Designed or promised, not built. Nothing here is a commitment to a date.

- **More AI providers**: OpenAI, Google Gemini, and OpenAI-compatible endpoints (Ollama for local/offline use, Groq, Together, and similar). The enum variants and settings plumbing exist as stubs. Landing a second API provider is also the point to introduce the shared provider abstraction ADR-0011 called for, replacing the per-command dispatch in `lib.rs`.
- **Automated no-fabrication validation**: diff every AI-rephrased text against its source and reject output that drops or alters dates, numbers, or named entities before it reaches the user. The schema split that feeds it already exists.
- **Linux and macOS packaging**: the Linux PDF print path exists but is untested; the macOS path is unimplemented. Packaging, updater channels, and platform testing are all open.
- **Authenticode code signing**: removes the SmartScreen warning on the Windows installer.
- **Template import and export**: share saved style templates as files (the original design sketched a `.tailorra-template` format). Peer-to-peer file sharing only; no marketplace or hosting.
- **OS file association for `.tailorra`**: double-click to open a workspace backup; deferred to packaging work.
- **Per-bullet rephrase for a listing**: rewrite an individual bullet against the target listing, stored as an override on the version without touching the master. The `version_items.override_text` column is reserved for this.
- **Listing requirement parsing**: extract must-haves, nice-to-haves, and keywords from a listing into structured data (`job_listings.parsed_requirements` is reserved) to drive a requirements-versus-items review view.
- **Provenance change history**: populate the reserved `original_text` columns when AI modifies prose and show what changed on demand.

## Not planned

- Cloud sync, accounts, telemetry
- Resume scoring or ATS-optimization analytics
- Template marketplace or centralized template hosting
- Multi-language support
- Mobile
