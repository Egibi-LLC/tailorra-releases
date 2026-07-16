# ADR-0012: Default template and extensibility (Classic for v1; templates as first-class data)

- **Status**: Accepted
- **Date**: 2026-04-27

## Context

Resume styling is a contentious space. Common formats:

- Classic single-column (boring, ATS-safe, never wrong)
- Modern minimalist (similar but with subtle accents)
- Two-column with sidebar (visually striking, real ATS parsing risks)

The user explicitly wants:

1. A reasonable default for v1
2. Multiple templates / themes / styles in the future
3. **Users can submit / export / save their own templates**

Goal #3 means templates must be **data, not code**: the v1 default template lives in the same shape that user-imported templates will eventually use.

## Decision

### v1: one built-in template, architected for many

- Ship a single built-in template: **Classic** (single-column, sans-serif, bold section headers with subtle horizontal rule, dates right-aligned, plain bullets, black on white, generous line height, ATS-safe)
- Templates are **first-class data**: stored on disk under `app_data_dir/templates/`, loaded by the rendering pipeline
- Built-in templates ship as system-installed, read-only entries
- Settings has a real template picker (only "Classic" listed in v1, but it's a real picker, no no-op)

### Template format

A template is a structured bundle:

- **Structure** (target-agnostic): section order, item layout, conditional blocks
- **HTML/CSS** (preview, HTML export, PDF-via-print): Handlebars-style templating with a fixed safe context, **no script tags, no remote URLs, no event handlers**; rendered in an isolated WebView
- **DOCX styling** (`docx-rs`): font, sizes, weights, spacing, color mappings (DOCX has no CSS analog)
- **Plain text / Markdown**: structure only; visual styling ignored

### v1.5+ (planned, additive)

- Additional built-in templates (Modern minimalist, Two-column with sidebar)
- Template editor UI (visual + code-edit)
- Import/export user templates as `.tailor-template` bundle files
- Theme variations within a template (color palette, font selection)

**Out of scope**: a template marketplace or any centralized hosting. Templates are files users share peer-to-peer.

### Security

User templates render in a sandboxed WebView (no network, no scripts, no event handlers). Data binding is structured (named fields only); template authors cannot inject arbitrary code into the renderer or access app state. Imported templates require explicit user approval before becoming active.

## Consequences

- **Pro:** v1 ships a clean, ATS-safe default
- **Pro:** Adding more templates in v1.5 is purely additive, no migration, no rendering rewrite
- **Pro:** User-shareable templates are an explicit product capability, not bolted on
- **Pro:** Sandboxed rendering blocks the obvious template-as-malware vector
- **Con:** Slightly higher v1 effort than hardcoding the classic template (rendering pipeline must read templates as data from day 1)
- **Con:** DOCX requires a separate styling representation from HTML; the duplication is accepted for fidelity
