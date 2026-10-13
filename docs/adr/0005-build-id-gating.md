# 0005 — Gate joins on a content-hashed simulation build id

- **Status:** Accepted
- **Date:** 2026-09-07

## Context

Client and server must run byte-identical simulation code. A stale browser tab, a partially deployed server, or a dependency change could otherwise connect two incompatible worlds and produce persistent desync that looks like a netcode bug.

## Decision

Derive `__SIM_BUILD__` as a SHA-256 over every `src/**/*.ts` file plus `package-lock.json`, truncated to 20 hex characters (`scripts/build-id.mjs`). Vite injects it into the browser bundle; esbuild injects it into the server bundle. The client includes it in `join`; the server rejects any mismatch. Baselines also carry `build` and `epoch` and are validated on arrival.

## Consequences

- Any shared-code change (including geometry and physics tuning) changes the build id and forces a redeploy of both sides together.
- A missed match fails fast with an "outdated game" message rather than desyncing silently.
- The build id is a drift/version guard, not an authentication token.
