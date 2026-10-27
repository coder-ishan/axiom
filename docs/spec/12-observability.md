# 12 — Observability & the trust surface

> Part of the [CURSOR architecture spec](README.md).

- **Trace model:** task → step → span (observe/route/plan/validate/dispatch/verify), each with latency + token counts + cache-hit flags. The per-step span is the unit of every latency claim in this spec.
- **Live status (P1):** one line at all times — current state, current intent in plain language, last action's outcome. A user should never wonder what the machine is doing; this is the trust product.
- **Metrics (local, no telemetry):** step p50/p95, cache hit rates, correction rate, escalation rate, ask rate, gate rate, sick-app events. Correlations of these are how we find design flaws.
- **Postmortem flow:** every HALTED/FAILED carries a structured diagnosis (state, last intents, diffs) into the audit viewer; failed trajectories feed the correction-synthesis loop (§19).
