# ADR-0003: Job listing ingestion

- **Status**: Accepted
- **Date**: 2026-04-27

## Context

The tailoring flow needs the job listing as input. Possible sources:

- Pasted text (textarea)
- URL fetch (LinkedIn, Indeed, etc.)
- Browser extension that grabs DOM
- Uploaded file (PDF / TXT / DOCX of a saved listing)

URL fetching is fragile: many job sites actively block scraping, redesign frequently, and have login walls. A browser extension is essentially a separate product. Paste covers 100% of cases reliably.

## Decision

v1 supports **paste + file upload (PDF / TXT / DOCX)**. The same parser used for resume upload (ADR-related to extraction flow) handles listing files.

**Out of scope for v1**: URL fetching, browser extensions.

## Consequences

- **Pro:** Reliable across every job site
- **Pro:** Reuses the file parser already needed for resume upload
- **Pro:** No maintenance treadmill chasing site redesigns
- **Con:** User has a manual paste/upload step per listing (that's actually fine; the workflow already involves reading the listing carefully before applying)
