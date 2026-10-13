# 0008 — Keep the renderer out of the simulation

- **Status:** Accepted
- **Date:** 2026-09-07

## Context

The simulation must run under Node for the dedicated server, for headless tests, and for cross-runtime determinism checks. Three.js is convenient for vector/quaternion math but its renderer and scene graph are browser-only and non-deterministic across environments. If simulation and presentation were entangled (as they originally were, when the match loop lived inside `Game`), the server could not run the match and offline/online logic would diverge.

## Decision

Enforce a one-way dependency:

- `physics.ts` / `simulation.ts` use Three **math types only** and never import the renderer, DOM, or any presentation module.
- `Game` (`game.ts`) and `GameRenderer` (`render.ts`) consume simulation state and events; they never mutate it.
- `tsconfig.server.json` includes only `server/`, `simulation.ts`, and `network/{authority,prediction,protocol}.ts`.

Simulation → presentation communication is the `MatchEvent` stream plus per-frame state reads.

## Consequences

- The identical simulation runs in Chrome and Node (validated by `test:cross-runtime`).
- Events (goal, demo, goal blast origin, etc.) must carry enough data for presentation instead of presentation reading internal objects — e.g. the blast origin is stored in `Simulation.celebration`, not read from the explosion object.
- Adding a presentation need requires a simulation state/event change, not direct access.
