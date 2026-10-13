# 0001 — Record architecture decisions with ADRs

- **Status:** Accepted
- **Date:** 2026-09-07

## Context

The project has accumulated several load-bearing decisions (shared headless simulation, pinned deterministic physics, client prediction with full-checkpoint rollback, a two-transport netcode design, RocketSim ball-material parity, and a separate renderer). The research documents explain *what was measured*, but not *why the code is shaped this way*. New contributors and future changes need that rationale in a durable form.

## Decision

Record architecturally significant decisions as numbered, immutable ADRs under `docs/adr/`, one decision per file, using Context / Decision / Consequences. Supersede rather than rewrite: a changed decision gets a new ADR that links to the one it replaces.

## Consequences

- The reasoning behind hard-to-reverse choices is discoverable next to the code.
- Each new significant change should add or supersede an ADR; trivial changes should not.
- These records are retroactive where the decision predates this file.
