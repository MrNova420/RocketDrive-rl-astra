# 0004 — Client prediction with full-checkpoint rollback

- **Status:** Accepted
- **Date:** 2026-09-07

## Context

A dedicated server owns the truth, but a client that waits for server confirmation before responding to input feels laggy. Predicting only body poses is insufficient: car and ball contacts depend on Rapier's solver caches, sleeping state, and contact manifolds, so a position-only correction will not reproduce the server's contact outcome.

## Decision

Each client runs a **confirmed** simulation (steps only server-applied inputs in order) and a **predicted** simulation (applies local input immediately and predicts the opponent's held input for the unknown horizon).

- Drift is detected two ways: an every-8-tick digest comparison, and an immediate diff when the applied inputs for a tick differ from what was predicted (so a surprise jump/steer/boost corrects at once).
- On any mismatch, the predicted sim is restored from a **complete confirmed checkpoint** — Rapier snapshot plus car controller state, pads, timers, and flags — then replays up to 64 ticks of local commands.
- Corrections are measured at the **same tick** (predicted-before vs predicted-after at tick T), so ordinary between-frame motion is not mistaken for error.
- Displayed error is smoothed separately: `GameRenderer.correctNetwork()` decays offsets with a 65 ms time constant and snaps on large errors or kickoff teleports.

## Consequences

- Responsiveness is local; authority remains server-side.
- Worst-case correction cost is bounded by the replay window (≤64 ticks).
- The engine must support complete save/restore of *all* interacting state; a position-only serializer is explicitly out of scope.
- Baseline payloads are large (~12 MB raw, 3–4 MB compressed). They are transferred rarely (join/resync), never per tick.
