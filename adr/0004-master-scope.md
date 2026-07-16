# ADR-0004: Master scope (single master per install)

- **Status**: Accepted
- **Date**: 2026-04-27

## Context

How many master resumes does one install support? The motivating case (a software engineer with prior military experience tailoring for `.NET` jobs vs. transportation jobs) is solved by tailored *versions* off one master, not separate masters. But what about truly disjoint identities, e.g., professional musician + software engineer, where even the summary and framing differ?

## Decision

**One master per install in v1.** UI exposes a single master; no project switcher. Schema includes a `project_id` column in every table from day 1, defaulting to `1`. Multi-master becomes a purely additive UI feature later (no migration needed).

Alternatives considered:

- **Multiple masters from v1**: adds friction for the 99% case
- **Tag-based filtering**: doesn't solve cases where the *whole identity* differs (different summary, different name presentation)

## Consequences

- **Pro:** Clean v1 onboarding, no "create a project" step
- **Pro:** Most users never need more than one master
- **Pro:** Forward-compatible: multi-master is purely additive, gated by UI not schema
- **Con:** Power users with truly disjoint careers must wait or install twice in v1
- **Con:** Slight schema overhead from carrying `project_id` everywhere from day 1
