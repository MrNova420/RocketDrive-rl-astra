# 0009 — DOM/CSS HUD instead of an in-canvas HUD

- **Status:** Accepted
- **Date:** 2026-09-07

## Context

The match UI (scoreboard, boost gauge, clock, toasts, messages, menus, control status, live input monitor) is dense, text-heavy, and reactive, and it must support responsive/compact layouts, reduced-motion preferences, keyboard focus, and inert menus. Drawing all of this in WebGL would mean reimplementing text layout, accessibility, focus management, and media queries.

## Decision

Build the HUD with DOM elements and CSS. `HUD` (`hud.ts`) owns the elements and updates them from simulation state each frame; `boost-gauge.ts` renders the dial as inline SVG; styling lives in `match-hud.css`, `style.css`, and per-panel CSS files. Menus use real buttons with focus/inert management (`syncMenu`), and `ControlsMenu`/`VisualSettings` are DOM overlays. The WebGL canvas carries an `aria-label` and sits beneath the HUD.

## Consequences

- Text, responsiveness, accessibility, and focus come from the platform.
- HUD updates must be cheap; scoreboard textures and DOM values update only when displayed state changes.
- HUD code is browser-only and kept out of the headless simulation.
