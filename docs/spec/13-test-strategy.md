# 13 — Test strategy per module

> Part of the [Axiom architecture spec](README.md).

| Module | Tests |
|---|---|
| AX reader + differ | golden fixture library: AXD snapshots from ~30 real apps (PII-redacted). **Property tests:** applying a delta stream == full resnapshot at every step; serialization determinism (same tree → identical bytes) — hard CI gate; id-stability under reorder/append |
| State machine | exhaustive transition tests incl. timeout-at-every-state; kill/lock/YIELD races (deterministic scheduler) |
| Policy engine | gate-class matrix exhaustively enumerated; destructive-lexicon fuzzing (homoglyphs, unicode tricks in labels) |
| Dispatch layer | stub-app harness: scripted AX trees with controllable responsiveness (hung/empty/exploding); assert occlusion check rejects occluded points; Secure Input simulation |
| Journal/recovery | crash injection at every record boundary; corrupt/truncated journals; ambiguous-intent reconciliation cases incl. at-most-once actions |
| Models | record/replay determinism; grammar-rejection tests (attempt invalid JSON — must be unrepresentable); confidence-calibration set (τ tuned on the eval suite, not vibes) |
| Chaos (system-level) | **gremlin harness:** random modal dialogs, notification banners, focus stealing, slow-app delays injected mid-task → assert: never click through foreign dialogs, always yield to user input, always halt bounded |
| Evals (task-level) | 100 curated macOS tasks (30 held-out) · 20 trap tasks: injection traps (fake instructions in page/file/email), double-click traps, renamed-target traps, money/delete gates · ScreenSpot-V2 subset for the pixel lane · latency CI: p50/p95 regression > 10% fails the build |
| Soak | 500-step session: RAM flat (KV bounded), journal rotation, no fd/leak growth |

## Reference points for the evals

- Cloud pixel agents, 2025 published numbers: OpenAI CUA 36.4% / Claude 28–38% on OSWorld; UI-TARS-1.5 42.5%.
- Realistic ambition for a local 2B-pilot AX-first agent: **50–65% on macOS-style tasks by Phase 3** — the AX advantage is what makes that plausible where pixel agents need 70B.
- Latency methodology: warm cache, median of 3, realistic agent-shaped prompts — never UI-reported tok/s alone (the wall-clock trap).
