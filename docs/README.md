# Developer documentation

Architecture, module reference, and decision records for Champions Field. Start here if you are changing or extending the codebase.

> User-facing feature docs and run instructions live in the [root README](../README.md).

## Read in this order

1. **[architecture.md](architecture.md)** — how the system is put together: the two runtime modes over one shared simulation, the fixed 120 Hz loop, data flow offline and online, the determinism strategy, the physics model, the rendering pipeline, and a task→file index.
2. **[modules.md](modules.md)** — a file-by-file map of `src/`, `server/`, and `scripts/`, grouped by concern with notable exports and a `sim`/`view` layer tag.
3. **[adr/](adr/)** — the reasoning behind the load-bearing decisions.

## Decision records

| ADR | Decision |
| --- | --- |
| [0001](adr/0001-record-architecture-decisions.md) | Record architecture decisions with ADRs |
| [0002](adr/0002-shared-headless-simulation.md) | One shared headless simulation at a fixed 120 Hz |
| [0003](adr/0003-pinned-rapier.md) | Pin `@dimforge/rapier3d-deterministic-compat` exactly |
| [0004](adr/0004-client-prediction-full-checkpoint-rollback.md) | Client prediction with full-checkpoint rollback |
| [0005](adr/0005-build-id-gating.md) | Gate joins on a content-hashed simulation build id |
| [0006](adr/0006-webrtc-transport.md) | WebRTC data channel for gameplay, WebSocket for control |
| [0007](adr/0007-ball-material-split.md) | Split ball car/world materials with dual-mask colliders |
| [0008](adr/0008-renderer-separate-from-simulation.md) | Keep the renderer out of the simulation |
| [0009](adr/0009-dom-css-hud.md) | DOM/CSS HUD instead of an in-canvas HUD |
| [0010](adr/0010-quality-tiers.md) | Quality tiers as a single renderer switch |

## Topic deep-dives

These predate this index and cover specific investigations and the deployed alpha:

- **[multiplayer-alpha.md](multiplayer-alpha.md)** — the operating guide for the deployed dedicated 1v1 alpha: final architecture, transport/recovery, verification commands, and deployment procedure.
- **[multiplayer-validation.md](multiplayer-validation.md)** — measured collision/prediction results, server capacity, and the problems found and fixed.
- **[multiplayer-plan.md](multiplayer-plan.md)** — historical research and the original feasibility investigation (retained for rationale, not current state).
- **[ball-physics-parity.md](ball-physics-parity.md)** — the RocketSim ball-model comparison and its measurements.
- **[research/](research/)** — the reproducible feasibility probe (`multiplayer-feasibility.mjs`) and its recorded output.

## Conventions

- **Layer tags.** `sim` modules are headless and must run under Node; `view` modules are browser-only. See [ADR-0008](adr/0008-renderer-separate-from-simulation.md).
- **Build id.** `__SIM_BUILD__` is a content hash of `src/**/*.ts` + `package-lock.json`; browser and server must match to connect. See [ADR-0005](adr/0005-build-id-gating.md).
- **Commands.** `npm run dev`, `npm run build`, `npm test`; online suites via `npm run test:netcode`, `test:collisions`, `test:cross-runtime`, etc. Browser suites need `CHROME_PATH` on non-macOS hosts.
