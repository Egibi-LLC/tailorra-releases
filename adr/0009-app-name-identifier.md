# ADR-0009: App name and Tauri identifier

- **Status**: Accepted; the app-name half is superseded by [ADR-0013](./0013-tailorra-rename-freezes.md) (renamed to `tailorra`), the identifier half stands
- **Date**: 2026-04-27

## Context

The app needs:

- A user-facing name (window title, display in OS app lists)
- A package name (`package.json`, `Cargo.toml`)
- A reverse-DNS bundle identifier (Tauri config, OS integration, code signing, update channels)

The identifier is *especially* important to lock early: it determines the OS app data directory and update channel routing. Changing it after shipping to real users is painful (app data migration, update channel break).

## Decision

- **App name (productName + package name)**: `tailor`
  - Names the central metaphor ("I tailored this resume in Tailor")
  - Short, two syllables, easy to type in CLI commands
  - Distribution-ready: clear what it does
- **Bundle identifier**: `dev.hubbard.tailor`
  - `dev.*` is the indie/personal-developer convention for apps without a registered domain
  - Easy to flip pre-launch with a single config edit if a domain is registered later

## Consequences

- **Pro:** Memorable, descriptive name; not coupled to other projects
- **Pro:** Bundle ID safe under solo-developer convention; no domain claims
- **Con:** If the project ends up under an organization or domain later, the bundle ID may need to change before the first public release
