# ADR-0007: Provenance UI (visible until reviewed)

- **Status**: Accepted
- **Date**: 2026-04-27

## Context

Every item in the master carries provenance metadata: `source` (`user_entered` / `ai_extracted` / `ai_suggested`) and a `reviewed` flag. How visibly should this be surfaced in the UI?

Tradeoffs:

- **Always-visible badges**: maximum transparency, but visual clutter on every item forever (feels like the app distrusting the user even after they've personally reviewed and accepted an AI suggestion)
- **Visible until reviewed, hidden after**: workflow-style signal. The badge does work while review is pending and disappears once the user has vouched for the item.
- **Hidden by default**: cleanest, but loses the at-a-glance signal of what still needs attention

## Decision

**Visible until reviewed; hidden after; detail-on-demand for the audit trail.**

- Items where `reviewed = false` show a "needs review" badge in the master editor
- Once accepted or edited (sets `reviewed = true`), the badge disappears; the item looks like any other
- A detail panel (one click away) shows full provenance + change history for any item
- **Provenance is never rendered in export output** (recruiters don't see it)

## Consequences

- **Pro:** Workflow signal (badges) appears exactly where action is required
- **Pro:** User-vouched items aren't perpetually flagged as "AI"
- **Pro:** Audit trail still available for accountability and forensics
- **Con:** Slightly more state to manage in the UI (visibility logic)
- **Con:** Reverting a "vouched" item to flag-on requires an explicit "unverify" action (not in v1 scope)
