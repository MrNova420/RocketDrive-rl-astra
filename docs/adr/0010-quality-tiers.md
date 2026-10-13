# 0010 — Quality tiers as a single renderer switch

- **Status:** Accepted
- **Date:** 2026-09-07

## Context

The detailed scene (million-blade grass, ambient occlusion, SMAA, bloom, 4096px shadows, layered sky, dense effects) is expensive, but the game must stay playable on modest hardware and the online collision/render tests need a predictable-cost preset.

## Decision

Expose three quality levels — `performance` (default), `high`, `ultra` — as a single switch in `GameRenderer.setQuality()` that controls pixel ratio (1 / ≤1.5 / ≤2), bloom on/off, ambient occlusion (Ultra only), SMAA (Ultra only), grass/sky/atmosphere visibility, shadow-map size (2048 / 4096), and the sun/lit profile. The choice persists to `localStorage` under `champions-field.quality` and is cycled by the user.

Effects that affect gameplay feel (engine cores, exhaust heat, brake lights, boost) run in every tier; only the heavy scene dressing is tier-gated.

## Consequences

- One code path with tier flags, rather than separate renderers.
- Changing a tier re-sizes buffers and rebuilds the shadow map when needed.
- Browser performance tests pin `performance` to bound cost; measurements are workstation-specific, not a cross-device guarantee.
