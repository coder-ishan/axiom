# 18 — Explicit non-goals (v1)

> Part of the [Axiom architecture spec](README.md).

- No autonomous/unattended operation
- No multi-task concurrency (tasks queue serially)
- No gaming/canvas automation beyond pixel-assist
- No Windows/iOS (different AX worlds entirely)
- No credential management (no password storage, ever)
- No cloud component anywhere; no telemetry anywhere

Each non-goal is a *deferred decision with a named trigger* — e.g., autonomy returns only after 3 months of clean gate/audit telemetry, not before. Non-goals are load-bearing: they are what makes the v1 threat model honest.
