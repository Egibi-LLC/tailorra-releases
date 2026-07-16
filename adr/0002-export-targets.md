# ADR-0002: Export targets v1

- **Status**: Accepted
- **Date**: 2026-04-27

## Context

A finalized tailored version needs to be exportable in formats that match real-world job application paths:

- Web forms (Workday, Greenhouse, LinkedIn) often want pasted text (Markdown or plain text)
- Recruiters frequently request `.docx` for ATS systems
- Some recruiters want PDF
- Native PDF rendering in a Tauri app is non-trivial; multiple paths exist with very different costs

## Decision

Ship five formats in v1:

| Format | Method | Cost |
|--------|--------|------|
| **Markdown** | Direct render from data | Trivial |
| **DOCX** | `docx-rs` Rust crate | Moderate |
| **PDF (via WebView print dialog)** | `window.print()` from rendered HTML | Free |
| **Plain text** | Strip Markdown formatting | Trivial |
| **HTML** | Rendered preview, saved to disk | Trivial |

Defer **native PDF (Typst-rendered)** to v1.5: it adds a binary dependency (Typst) and complicates packaging. The print-dialog PDF covers most needs adequately; we'll only build the polished version if real users complain about quality.

## Consequences

- **Pro:** Day-one coverage of every common submission path
- **Pro:** No binary dependencies in v1 (DOCX is pure-Rust)
- **Pro:** HTML preview already needed for the rendering pipeline; PDF-via-print is a free byproduct
- **Con:** PDF-via-print uses browser-default styling; not as polished as a Typst-rendered PDF
- **Con:** Rendering pipeline must support five output targets from the start (mitigated by ADR-0012's template architecture)
