# 0002 — One shared headless simulation at a fixed 120 Hz

- **Status:** Accepted
- **Date:** 2026-09-07

## Context

The game needs offline play, a bot opponent, and a dedicated-server 1v1 mode. Rocket League-style gameplay depends on a fixed timestep, and multiplayer correctness requires the server and every client to step identical physics with identical inputs. If offline and online each had their own copy of the match logic, they would drift apart, and every offline change would need to be mirrored online (or would silently break parity).

## Decision

Extract the match into a single headless `Simulation` class (`src/simulation.ts`) that owns `Physics`, the clock, score, phases, and events, and advances by exactly `STEP = 1/120` per tick.

- Offline: `Game extends Simulation` — the browser **is** the simulation.
- Online: the server's `Authority` holds a `Simulation`; the client's `Prediction` holds two.
- `simulation.ts` and `physics.ts` may use Three **math types only**, never the renderer or DOM, so they run unchanged under Node.

Presentation is a consumer of simulation state and `MatchEvent`s; it never mutates the simulation. Render frames run at the display rate and interpolate between physics poses.

## Consequences

- One implementation of match rules for both modes.
- The same code is exercised by headless Node tests and the browser.
- The simulation must stay free of DOM/renderer dependencies — enforced by `tsconfig.server.json` including only `simulation.ts` and the `network/*` netcode files.
- Bot behaviour is just another `Input` producer, so it reuses the same step function.
