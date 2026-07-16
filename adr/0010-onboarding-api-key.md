# ADR-0010: Onboarding API key (contextual prompt)

- **Status**: Accepted
- **Date**: 2026-04-27

## Context

When a new user installs the app, when do they have to provide an AI provider API key?

- **Gate** (require during onboarding): simpler app states, but hard friction. Users without an Anthropic/OpenAI account hit a wall before exploring the app.
- **Defer** (skip during onboarding): fully usable for manual entry; AI buttons prompt later. More UI states to handle.
- **Contextual**: onboarding has the field but it's optional; AI-dependent paths inline-prompt for it.

The app must support a fully manual workflow (typing the resume by hand, no AI). That serves both as a real feature and as a way for users to evaluate the app before paying for AI.

## Decision

**Contextual prompt**:

- Onboarding has the API key field but it's optional
- Choosing **"upload existing resume"** (which requires AI extraction) inline-prompts for provider and key (the key is needed *now*)
- Choosing **"start from scratch"** lets the user proceed without; AI buttons elsewhere disable gracefully with a tooltip pointing to Settings
- Settings always has the full provider config

## Consequences

- **Pro:** Friction matches user intent: users only see the key prompt when they need it
- **Pro:** Manual-only workflow fully supported
- **Pro:** Important for distributable app where some users have no Anthropic account
- **Con:** More UI states to handle than a hard gate (the "no key configured" mode must work everywhere)
