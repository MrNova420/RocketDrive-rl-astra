# 0006 — WebRTC data channel for gameplay, WebSocket for control

- **Status:** Accepted
- **Date:** 2026-09-07

## Context

Gameplay frames are superseding: a newer input batch or authoritative frame makes the previous one obsolete. A reliable, ordered transport (plain WebSocket/TCP) suffers head-of-line blocking — a lost older packet delays newer ones, adding latency exactly when the network is worst. But shipping, joining, WebRTC signaling, large baselines, and reliable catch-up benefit from an ordered, reliable channel.

## Decision

Use two transports:

- **WebSocket** at `/api/connect` for the control plane — JSON join/auth, WebRTC signaling (`description`, `candidate`), gzip baselines (binary, leading byte `0x1f`), reliable `recover` catch-up, `ping`/`pong`, and seat/room events.
- **WebRTC data channel** named `physics`, negotiated `id: 0`, `ordered: false`, `maxRetransmits: 0`, for redundant input batches and authoritative frames.

Chrome creates the SDP offer so it owns ICE nomination, including TCP-TURN fallback. The server builds `node-datachannel` against **libnice** because the default libjuice backend rejects Chrome's zero-valued ICE tiebreaker on TCP-TURN checks (without weakening DTLS verification).

WebTransport was considered (Chrome-only) but rejected for this alpha because the inspected Node WebTransport package documented incomplete HTTP/3 features; the transport layer is kept independent of the simulation so it can change later.

## Consequences

- A lost old gameplay packet never blocks a newer one.
- Two transports must be wired and kept consistent (signaling on one, data on the other).
- Server-side limits apply per channel: input budget (~180/s) and control budget (~60/s), with oversized/binary-frame guards.
- Deployment must expose UDP for ICE and run a TURN service; ICE candidates are rewritten for the Docker/public address mapping.
