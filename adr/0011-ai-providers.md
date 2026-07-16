# ADR-0011: AI providers (multi-provider via Rust-side abstraction)

- **Status**: Accepted
- **Date**: 2026-04-27

## Context

The app could hard-wire to a single AI provider (Anthropic) for simplicity, but that:

- Locks users into one cost structure
- Excludes users without an Anthropic account
- Excludes privacy-conscious users who'd prefer local models (Ollama)
- Couples the application to one company's pricing/availability decisions

A multi-provider design is meaningful additional work (abstraction layer, per-provider integrations, output normalization) but is much harder to retrofit later than to build in.

## Decision

**Provider-agnostic AI via a Rust-side `AIProvider` trait.** All AI logic lives in Rust; the frontend calls Tauri commands (`invoke('ai_extract', ...)`, `invoke('ai_tailor', ...)`, etc.). Frontend never sees raw API keys or raw provider responses, only validated, structured operation outputs.

### v1 providers

| Provider | Integration | Notes |
|----------|-------------|-------|
| **Anthropic** | Native (default) | Prompt caching for master corpus context |
| **OpenAI** | Native | Structured outputs (JSON schema) where supported |
| **Google Gemini** | Native | Structured outputs via response schema |
| **OpenAI-compatible** | Generic adapter | Covers Ollama (local, fully offline), Groq, Together, Anyscale, and similar. User configures base URL, key, and model name. |

### Model tier abstraction

Each provider exposes a "fast" tier (extract, tailor, suggest, rephrase) and a "deep" tier (gap-find, critique). Defaults: Anthropic Sonnet 4.6 / Opus 4.7; OpenAI GPT-4o / o-series; Gemini 2.5 Flash / 2.5 Pro; OpenAI-compatible: user-configured per endpoint.

### Capability degradation

- Output validation (truthfulness check, ADR-0008) runs uniformly across providers
- Operations recommending the "deep" tier warn (don't block) if user has only configured a weak model
- Local models via Ollama enable a fully offline, private mode (no cloud calls, no external API keys)

## Consequences

- **Pro:** Users pick their preferred provider, including local
- **Pro:** Privacy-conscious workflow possible (Ollama)
- **Pro:** API keys never enter the renderer process; security improved over a frontend-only Anthropic SDK approach
- **Pro:** Truthfulness invariant enforced uniformly across providers
- **Con:** More v1 effort than single-provider; multiple integrations to write and maintain
- **Con:** Capability differences across providers must be handled gracefully (e.g., not all support JSON schema)
