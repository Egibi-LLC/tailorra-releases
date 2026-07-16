# tailorra

A Windows desktop app that helps you customize your resume for each job you apply to. Built with Tauri 2 and Angular 20. You import your existing resume (or enter your history by hand), capture job listings, and generate tailored resume versions and cover letters with AI assistance. Everything lives in a local SQLite database; nothing is sent anywhere except the AI calls you trigger.

The idea: you keep one big "master" corpus with every job, project, and skill you've ever had. When you find a listing, tailorra picks the relevant parts and helps you put together a focused version: choosing what to include, suggesting how to phrase it, and laying it out. The AI helps with the writing, but it won't invent experiences, change dates, or make up numbers. You review every AI-generated change before it lands on a resume you actually send.

This is the private source repo (`Egibi-LLC/tailor`). Public installers and the docs site live in `Egibi-LLC/tailorra-releases`. The project, crate, and npm package are all named `tailorra`; the bundle identifier stays `dev.hubbard.tailor` (frozen, see ADR 0009).

## Download (end users)

- Docs site: [egibi-llc.github.io/tailorra-releases](https://egibi-llc.github.io/tailorra-releases/)
- All releases: [github.com/Egibi-LLC/tailorra-releases/releases](https://github.com/Egibi-LLC/tailorra-releases/releases)
- Latest Windows installer (stable link): [tailorra-x64-setup.exe](https://github.com/Egibi-LLC/tailorra-releases/releases/latest/download/tailorra-x64-setup.exe)

The installer is minisign-signed for the auto-updater but not Authenticode-signed, so Windows SmartScreen shows "Windows protected your PC" on first run. Click "More info", then "Run anyway". Once installed, the app checks for updates on launch and updates itself.

## Stack

- **Frontend**: Angular 20, SCSS, lucide-angular. Light and dark themes (dark is the default).
- **Desktop shell**: Tauri 2 (Rust). Windows x64 NSIS installer is the only packaged target today.
- **Storage**: SQLite via `tauri-plugin-sql`. The Angular renderer executes SQL directly; this is deliberate, not an oversight. 15 migrations. Multi-workspace via a `projects` table with `project_id` scoping. Workspace backup and restore uses a `.tailorra` JSON file (legacy `.json` still loads); import runs through a transactional Rust command (`src-tauri/src/import.rs`) with column whitelists and id regeneration.
- **AI providers**: two work today.
  - Anthropic API, with the key stored in the OS keychain (`keyring` crate).
  - Claude Code CLI passthrough, which uses your local `claude` login session and needs no key.
  - OpenAI, Gemini, and OpenAI-compatible endpoints exist as enum stubs that return "not yet supported". There is no provider trait; dispatch is per-command match arms in `src-tauri/src/lib.rs`.
- **Job listing capture**: manual entry, URL fetch, or a browser extension (`extension/`) that talks to a local axum server on `127.0.0.1:7341` with bearer-token pairing.
- **Exports**: HTML, Markdown, plain text, and DOCX client-side; PDF through the embedded webview print pipeline (WebView2 CDP on Windows). Legacy `.doc` is not an export target (convert the DOCX in Word if needed).
- **Onboarding**: a guided 11-step wizard; pages with draft state register per-page Save & Next handlers.
- **Auto-updater**: minisign-signed NSIS installer published to the GitHub releases channel.

Other pieces: resume file import with AI extraction (pdf, docx, doc, txt, md), a style designer with 14 CSS skeleton variants plus AI direction and refine, AI cover letters, undo for AI suggestions, and an AI activity log.

## Documentation

| Document | Purpose |
|----------|---------|
| [`SPEC.md`](./SPEC.md) | Specification of the app as built, plus roadmap and decision log |
| [`ARCHITECTURE.md`](./ARCHITECTURE.md) | Code map, runtime topology, key invariants |
| [`CONTRIBUTING.md`](./CONTRIBUTING.md) | Dev setup, conventions, PR flow |
| [`docs/RELEASING.md`](./docs/RELEASING.md) | Release flow (version bump, tagging, publish) |
| [`docs/USER_GUIDE.md`](./docs/USER_GUIDE.md) | End-user guide |
| [`docs/adr/`](./docs/adr/) | Architecture Decision Records, one per locked design choice |

`SPEC.md` and `ARCHITECTURE.md` are kept in step with the implementation; where they disagree with the code, the code wins. Generated API docs (TypeDoc and rustdoc) are published to the Pages site by `docs.yml` and can be built locally (see below).

## Prerequisites

- **Node.js** 20+
- **Rust** 1.77+ via [`rustup`](https://rustup.rs)
- **Tauri 2 system requirements**: see [tauri.app/start/prerequisites](https://tauri.app/start/prerequisites/). On Windows that mainly means WebView2, which ships with Windows 10/11.

## Run locally

```bash
npm install
npm run tauri dev
```

The Angular dev server runs on port 1420 (Tauri default); the app window opens automatically.

## Build

```bash
npm run tauri build
```

Produces the NSIS installer under `src-tauri/target/release/bundle/`. Real releases go through CI instead: `release.yml` runs dispatch-first on `main`, builds a draft, then publishes and auto-tags. See [`docs/RELEASING.md`](./docs/RELEASING.md).

## Generate documentation

```bash
npm run docs            # both
npm run docs:ts         # TypeDoc, output to docs/api/ts/
npm run docs:rust       # rustdoc, output to src-tauri/target/doc/
```

CI workflows: `ci.yml` runs build checks on every push; `docs.yml` builds TypeDoc, rustdoc, and the ADRs and publishes them to the Pages site on the public releases repo (rustdoc source pages are stripped first).

## Status

Working app, pre-1.0. Windows is the only supported platform: the installer, updater, PDF export, and Word export are all Windows-verified. A Linux PDF path exists but is untested; macOS PDF export is unimplemented. Expect breaking changes to the database schema and file formats before 1.0.

## License

Proprietary, all rights reserved. Not currently licensed for redistribution.
