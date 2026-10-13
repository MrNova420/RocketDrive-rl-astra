# 0007 — Split ball car/world materials with dual-mask colliders

- **Status:** Accepted
- **Date:** 2026-09-07

## Context

[RocketSim ball-physics parity](../ball-physics-parity.md) needs different effective materials for the ball against a **car** versus against the **arena**: ball/car restitution `0` and friction `2`, ball/arena restitution `0.6` and friction `0.35`. Rapier's installed JavaScript API does not expose solver-material modification hooks, so a single collider cannot carry two material behaviours.

## Decision

Give the ball body **two coincident sphere colliders** with mutually exclusive collision masks (`COLLISION_GROUPS` in `physics.ts`): one resolves ball↔car, the other ball↔world/arena. Only one collider can ever resolve a given pair, so no contact is applied twice. The second collider has **zero mass**, preserving the sphere's mass and inertia. Both collider handles are serialized in checkpoints and restored on rollback, and wheel queries recognize either sphere.

## Consequences

- RocketSim's separate ball materials are reproduced without engine hooks or forking Rapier.
- Checkpoint/restore must rebind both handles; dropping one would change ball behaviour after a correction.
- The dual-collider setup is a targeted workaround; if Rapier adds per-pair material hooks it can be simplified.
