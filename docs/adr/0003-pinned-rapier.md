# 0003 — Pin `@dimforge/rapier3d-deterministic-compat` exactly

- **Status:** Accepted
- **Date:** 2026-09-07

## Context

Online play rolls back and replays physics, and the server and clients must agree tick-for-tick. Rapier documents cross-platform determinism for its WASM build only under identical versions, initial conditions, and construction order, and warns that JavaScript transcendental functions can break deterministic initialization. A floating dependency range could silently change engine behaviour on a rebuild, invalidating every recorded replay and desynchronizing live matches.

## Decision

Depend on `@dimforge/rapier3d-deterministic-compat` at the exact version `0.19.3` (no caret range). The upstream [feature-selection documentation](https://github.com/dimforge/rapier.js/#feature-selection) distinguishes this package from the ordinary compatibility build.

Additionally, record arena collider roles explicitly at construction instead of inferring them from `Collider.parent()`: 0.19.3 can cache a bogus JS parent (`undefined → NaN → arena index 0`) for an unattached collider after deserialization. No private engine fields or global math functions are patched.

## Consequences

- Engine behaviour is reproducible for a given checkout; the build id changes if anything shared changes.
- Upgrading Rapier requires re-running the cross-runtime and collision suites and re-recording replay baselines.
- The arena-role workaround must be preserved or replaced by an upstream fix before relying on `parent()` again.
