# Module map

A file-by-file reference for the `src/`, `server/`, and `scripts/` trees. Each entry lists the module's role and its notable exports. For how the pieces fit together, see [architecture.md](architecture.md).

Legend: **sim** = headless (no DOM/renderer, safe under Node), **view** = browser-only presentation.

## Core simulation

| Module | Layer | Role |
| --- | --- | --- |
| `src/config.ts` | sim | All shared constants and types: `FIELD` (width 40.96, length 51.2, height 20.48, goal + ramp + ball dimensions), `CAR` (RocketSim-derived handling values), `STEP = 1/120`, `MATCH_LENGTH = 300`, team colors, and the `Input` interface + `emptyInput()`. Single source of truth for tuning that must match across browser and server. |
| `src/simulation.ts` | sim | `Simulation` — the shared headless match. Owns `Physics`, match clock/score/phase/overtime, the `MatchEvent` stream (`emit`/`onEvent`), `step(inputs, start)`, `kickoff`, `resetMatch`, `score`, `end`, `matchState`, `checkpoint`/`restore`, and `digest`. `Phase`, `MatchState`, `MatchEvent`, `Checkpoint`, `Slot` types live here. |
| `src/game.ts` | view | `Game extends Simulation` — the offline host. Wires `Controls`, `HUD`, `GameRenderer`, menus, online menu, visibility pause, and the render/accumulator loop. Owns `action()` (input→command dispatch), `botInput()`, `presentEvent()` (events→audio/HUD/effects/shake), `frame()`, `restart()`, and dev-only `scenario()`/`advance()`/`snapshot()` test hooks. |
| `src/main.ts` | view | Entry point. Imports global CSS, constructs `Game`, calls `init()`, and renders a WebGL-2 fallback message on failure. |

## Physics and arena

| Module | Layer | Role |
| --- | --- | --- |
| `src/physics.ts` | sim | `Physics` — the Rapier world and the tuned car/ball model. Two car bodies with four-suspension wheel traces, jump/dodge/flip mechanics, boost, recovery, demolitions, goal detection, pad pickups, camera-clearance queries, and the event callbacks (`onHit`, `onPad`, `onDemo`, …). `checkpoint()`/`restore()` serialize Rapier bytes plus car memory; `canonicalState()` feeds the digest. `COLLISION_GROUPS` implements the dual-mask ball. |
| `src/ball-physics.ts` | sim | RocketSim parity constants (`BALL`) and `ballHitVelocity()` — the relative-speed hit curve and direction adjustment used for ball strikes. |
| `src/arena.ts` | sim | Procedural arena geometry shared by physics and rendering: `SurfaceKind`, `ArenaSurface`, `BoundaryPoint`, `arenaSurfaces()`, `goalArchHeight()`. Curved collision/render meshes, tapered goal entrances, rounded interiors, honeycomb. |
| `src/goal-frame.ts` | view | Layered beveled goal surrounds, silver U-borders, inset team lamps, hex-tiled sills, and the three translucent own-goal shields. |
| `src/ramp-decoration.ts` | view | Ramp panel materials and decoration: `createRampMaterials()`, `decorateRamp()` — pale metal panels, lane stripes, drainage borders; transparent backs for the camera. |
| `src/arena-lettering.ts` | view | Arena signage text: `arenaLettering()` and the `goalAward` markup used on banners. |
| `src/score-numerals.ts` | view | `scoreNumerals()` — thin arena numeral textures for the scoreboard panels. |

## Netcode

| Module | Layer | Role |
| --- | --- | --- |
| `src/network/protocol.ts` | sim | Wire format. `PROTOCOL`, `MAX_LEAD = 64`, `HISTORY = 256`, `SERVER_HISTORY = 3600`, `MAX_PACKET = 1200`. `canonicalInput()` quantization, `heldInput()` (clears the jump edge), `Command`/`Frame` types, and `encodeCommands`/`decodeCommands` (19-byte stride, ≤16) and `encodeFrames`/`decodeFrames` (43-byte stride, ≤24), each with a strict `valid()` length/version check. |
| `src/network/authority.ts` | sim | `Authority` — the server-side match. Per-slot applied-input queues, bounded-lead acceptance, duplicate/expired rejection, a 12-tick held-input fallback, rematch gating, an 8-tick digest cadence, and a `SERVER_HISTORY` ring of frames. `step()` produces a `Frame`. |
| `src/network/prediction.ts` | sim | `Prediction` — the client-side rollback engine. Holds `confirmed` and `predicted` `Simulation`s, applies local input immediately while predicting the opponent's held input, and `reconcile()`s against server frames (applied-input diff + digest). Corrections restore a full confirmed checkpoint and replay ≤64 ticks. Emits `CorrectionSample` with same-tick error. |
| `src/network/client.ts` | view | `OnlineClient` — orchestrates a browser session: WebSocket control plane + WebRTC `physics` data channel, baseline restore, ping/anchor clock, target-tick pacing, stall/resync detection, and exposing `window.multiplayerStatus()`. |
| `src/network/online.css` | view | Styling for the online lobby/session panel. |

## Server

| Module | Layer | Role |
| --- | --- | --- |
| `server/index.ts` | node | HTTP + WebSocket host. Serves the static build and `/api/health`, `/api/rooms` (create), `/api/join` (code→seat), `/api/admin/status`; upgrades `/api/connect` for signaling. Manages rooms, per-room worker lifecycle, WebRTC peer setup with rewritten ICE candidates, TURN HMAC credentials, rate budgets, replay logging, and shutdown. |
| `server/room.ts` | node | Per-room worker. Runs the fixed 120 Hz `Authority` loop with bounded catch-up, refuses to drift time on overload (fatal after 2 s behind), and posts `frames`, `recover`, `baseline`, `stats`, and `replay` messages to the host. |
| `server/benchmark.ts` | node | Isolated physics-throughput benchmark used for capacity planning (documented in [multiplayer-validation.md](multiplayer-validation.md)). |

## Input and controls

| Module | Layer | Role |
| --- | --- | --- |
| `src/bindings.ts` | view | `ACTIONS` (id, label, category, default keys/pad bindings), `ControlSettings`, `STORAGE_KEY`, `defaults()`, key/pad label helpers, and validators (`validKey`, `validPad`). |
| `src/controls.ts` | view | `Controls` — merges keyboard + polled Gamepad API into `Input`. Edge vs held actions, deadzones, sensitivities, hotplug/`padBlocked` handling, rebind capture, menu navigation, and persistence. |
| `src/controls-menu.ts` | view | `ControlsMenu` + `navigateMenu` — the rebinding UI, controller status, live input monitor, and tuning controls. |

## Rendering orchestration

| Module | Layer | Role |
| --- | --- | --- |
| `src/render.ts` | view | `GameRenderer` — scene, camera, `WebGLRenderer`, `EffectComposer` pass chain, quality tiers, sun/shadow management, asset loading, offscreen shader warm-up (`prepareGraphics`), car/ball sync with network offsets, and the per-frame `update`/`draw`. `QualityLevel`, `BoostStyle` types. |
| `src/assets.ts` | view | Procedural fallback `createCarModel`/`createBall` (and `CarModel`, material/geometry helpers) used before detailed models load. |
| `src/models.ts` | view | `loadDetailedModels`, `detailedCar`, `detailedBall` — GLB Octane and ball assets, material setup, independent wheel pivots. |
| `src/car-paint.ts` | view | `CarPaint`, `PAINT_JOBS`, `PaintJob`, `validPaintJob` — paint presets applied to car materials, plus the paint preview animation. |
| `src/octane-lighting.ts` | view | `OctaneLighting` — segmented engine-core glow, brake lights, and exhaust-outlet heat driven by throttle/boost/braking. |
| `src/octane-outlets.ts` | view | `BOOST_OUTLET`, `EXHAUST_OUTLET` — shared car-scale socket coordinates used by effects. |

## Stadium and scenery

| Module | Layer | Role |
| --- | --- | --- |
| `src/stadium.ts` | view | `Stadium` — procedural grass-ish turf mesh, seating, crowds, goal nets, wall/truss geometry, banners, boost-pad visuals, monument, and `glowTexture()`. Updates per frame and per camera. |
| `src/stadium-presentation.ts` | view | `StadiumPresentation` — live scoreboards, LED ribbons, animated light channels, cloth/crowd motion, goal-celebration lighting, and Ultra roof beams. `setMatch()` pushes score/clock/phase. |
| `src/ultra-grass.ts` | view | `UltraGrass` — up to 1,000,000 wind-animated grass blades with tire-contact compression, used only in Ultra. |

## Effects

| Module | Layer | Role |
| --- | --- | --- |
| `src/effects.ts` | view | `Effects` — the shared GPU particle pool plus the goal explosion, demolitions, and ball speed trail; emits boost sparks and short-lived ball sparks. |
| `src/goal-explosion.ts` | view | `GoalExplosion` — the "Stellar Collapse" goal celebration (corona, singularity, accretion disk, GPU particles, lightning, shockwave) and its `impact` scalar. |
| `src/blast-pass.ts` | view | `createBlastPass()` — full-screen refraction/blast post pass centered on the explosion. |
| `src/demolition.ts` | view | `Demolitions` — reusable fireballs, shock rings, spark streaks, tumbling debris, BOOM lettering, and smoke; `updateCamera()` for near-camera fade. |
| `src/rocket-boost.ts` | view | `RocketBoost` — the Classic orange afterburner (cores, rippling edges, drift, release). |
| `src/flame-smoke.ts` | view | `FlameSmoke` — the Inferno boost plume (multicolored smoke, birth-orientation, world-space retention). |
| `src/exhaust-backfire.ts` | view | `ExhaustBackfire` — throttle-release flames from the side exhaust pipes. |
| `src/speed-trails.ts` | view | `SpeedTrails` — grounded supersonic rear-tire ribbons; `configureWheels()` binds tire geometry. |
| `src/supersonic-streaks.ts` | view | `SupersonicStreaks` — camera-relative world-space speed wisps. |
| `src/tire-smoke.ts` | view | `TireSmoke` — brief e-brake/drift smoke above the tallest grass. |
| `src/turf-debris.ts` | view | `TurfDebris` — short-lived grass/soil clumps thrown from grounded tire contacts; exposes `contacts` for grass compression. |
| `src/ball-speed-trail.ts` | view | `BallSpeedTrail` — sparse short-lived ball speed sparks. |
| `src/ball-marker.ts` | view | `createBallMarker()` — the height-scaled ground reticle projected onto floor and ramps. |
| `src/ball-direction.ts` | view | `BallDirection` — the off-ball pointer arrow when ball cam is off. |

## Camera

| Module | Layer | Role |
| --- | --- | --- |
| `src/follow-camera.ts` | view | `FollowCamera` — analytic critically damped spring orbit, shared lens/distance across camera modes, focus transitions, floor clearance, ball-priority framing, and manual look. |
| `src/camera-settings.ts` | view | `CAMERA_DEFAULTS`, `CAMERA_RANGES`, `loadCameraSettings()` — persisted FOV/height/distance/angle/shake preferences. |
| `src/cinematic-pass.ts` | view | `CinematicMotionPass` (exposes `depthPass`) and `createCinematicGrade()` — depth prepass, motion blur, and color grade passes. |
| `src/bloom-pass.ts` | view | `createBloomPass()` — configured Unreal bloom pass. |
| `src/ultra-occlusion.ts` | view | `UltraOcclusion extends GTAOPass` — Ultra-only ambient occlusion. |
| `src/ultra-sky.ts` | view | `createUltraSky()` — Ultra layered sunset sky. |

## HUD and UI

| Module | Layer | Role |
| --- | --- | --- |
| `src/hud.ts` | view | `HUD` — DOM/CSS match UI: score, clock, boost, speed, toasts, messages, opponent boost, net stats, pause/help panels, and action dispatch (`hud.onAction`). |
| `src/boost-gauge.ts` | view | `BoostGauge` — boost dial SVG (amber→coral segments, numerals, drain glow, pickup surges, low-fuel/supersonic states). |
| `src/visual-settings.ts` | view | `VisualSettings` — the Camera & Visuals options panel (quality, boost style, paint, sliders) and its change/close callbacks. |
| `src/style.css` | view | Global layout and shared UI styling. |
| `src/match-hud.css` | view | Match HUD styling (scoreboard, boost, messages, responsive/compact, reduced-motion). |
| `src/original-hud.css` | view | Legacy/reference HUD styling retained for tests. |
| `src/visual-settings.css` | view | Camera & Visuals panel styling. |

## Audio

| Module | Layer | Role |
| --- | --- | --- |
| `src/game-audio.ts` | view | `GameAudio` — WebAudio mixer: asset load/unlock, throttle-aware motor, boost/tire/ball/jump/pickup/goal/demolition/UI sounds, distance+pan from `soundPosition()`, mute and pause control. |
| `src/audio-clips.ts` | view | `AUDIO_CLIPS` — the local sample manifest (23 clips). |

## Build tooling and tests

| Module | Layer | Role |
| --- | --- | --- |
| `scripts/build-id.mjs` | node | `simulationBuild()` — SHA-256 over all `src/**/*.ts` + `package-lock.json`, truncated to 20 chars. Injected as `__SIM_BUILD__`. |
| `scripts/build-server.mjs` | node | esbuild bundle of `server/{index,room,benchmark}.ts` into `dist-server/*.mjs` (Node 22, external packages). |
| `scripts/verify-replay.mjs` | node | `npm run verify:replay` — replays a logged room against its original build and checks recorded hashes. |
| `tests/*.mjs` | node/browser | 58 test files (suites plus shared fixtures/helpers): headless gameplay/physics/controls/HUD checks (run via `npm test`) and Playwright browser suites (`test:netcode`, `test:collisions`, `test:multiplayer-browser`, `test:cross-runtime`, `test:server-safety`, `test:deployed`, `test:turn-auth`). Reusable fixtures live in `collision-scenarios.mjs`, `collision-assertions.mjs`, `collision-budgets.mjs`, `collision-server.mjs`. |

See [architecture.md §11](architecture.md#11-where-to-make-a-change) for a task→file index.
