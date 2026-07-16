# ADR-0008: AI safety boundaries, hard-immutable fields, enforced architecturally

- **Status**: Accepted
- **Date**: 2026-04-27

## Context

The truthfulness invariant ("AI may select, reorder, and rephrase, but never fabricate") is a core principle of the product. A prompt instruction alone is insufficient: under stress (long context, complex prompts, weaker models, ambiguous inputs), an LLM may silently introduce or alter facts. Real-world job applications cannot tolerate fabrication.

## Decision

Enforce the invariant **architecturally**, not advisorially:

### Hard-immutable fields (AI cannot modify, ever)

- Dates (employment, education, project, certification)
- Employer names, job titles, locations
- Degree names, institution names, GPAs
- Certification names, issuers, IDs
- Specific numbers and metrics (team sizes, %, $, durations)
- Named entities (products, technologies, projects, people)

### Enforcement mechanisms

1. **Schema split**: immutable fields live in dedicated columns separate from prose (descriptions, bullet text). Queries treat them differently.
2. **Prompt construction**: AI receives immutable fields as labeled context with explicit "do not modify" instructions; only prose is presented as a rewrite target.
3. **Output validation**: every AI-rephrased bullet is automatically diffed against its source. Any number, date, or named entity in the source must appear verbatim in the output, or the output is rejected before reaching the user.
4. **Review UI**: diffs highlight structured-field drift in red and prose changes in normal review styling.

The validator is provider-agnostic (ADR-0011); it runs uniformly across Anthropic, OpenAI, Gemini, and OpenAI-compatible endpoints.

## Consequences

- **Pro:** Truthfulness is a property of the system, not a hope about the model
- **Pro:** Weaker models cannot bypass; they fail validation more often, which is *correct* behavior
- **Pro:** Safety scales across all providers without re-engineering per provider
- **Con:** Validation logic must be maintained alongside prompt and schema evolution
- **Con:** Some legitimate AI rephrasing (e.g. expanding "5" to "five") will be blocked unless we add carefully-scoped exceptions; an accepted fail-safe trade
