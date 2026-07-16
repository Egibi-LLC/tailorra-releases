# ADR-0005: Cover letters (v1.5, schema-ready)

- **Status**: Accepted
- **Date**: 2026-04-27

## Context

Cover letters take the same input shape as tailored resumes (master + listing) but produce a different output (free-form letter rather than structured doc). They're a major time sink in real job applications.

The full v1 implementation would need:

- New editor screen
- Different prompt path (free-form vs. structured)
- Salutation / closing fields
- Tone configuration (formal/casual)

Most underlying infrastructure (master, listing, AI providers, exports, truthfulness validation) already exists. Marginal effort: roughly 20-30% of the resume tailoring flow.

## Decision

Defer cover letter implementation to **v1.5**, but reserve the schema in v1: `cover_letters` table with FK to `tailored_versions`. No migration needed when v1.5 lands.

## Consequences

- **Pro:** v1 ships sooner; focus stays on getting resume tailoring right first
- **Pro:** No schema churn when cover letters arrive
- **Pro:** Truthfulness rule (ADR-0008) extends naturally
- **Con:** Users still write cover letters by hand in v1 (an explicit limitation)
