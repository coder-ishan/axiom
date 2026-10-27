# 16 — Build phases & gates

> Part of the [CURSOR architecture spec](README.md).

| Phase | Scope | Gate to pass |
|---|---|---|
| **0 — Honest baseline (1 wk)** | Qwen3.5-8B + raw screenshots + CGEvent clicks, no caching. Everything is measured against this | runs 20/50 curated tasks; step p50 ≤ 5s; all numbers recorded |
| **1 — AX core (2 wks)** | AXD/v1 reader + differ + fixtures; state-machine skeleton; action contracts for the top 6 classes; journal; policy engine + kill switch | success ≥ baseline; step p50 ≤ 800ms; property tests green (differ idempotence + determinism); crash-injection suite passes (resume-to-HALTED everywhere) |
| **2 — Speed (2 wks)** | prefix caching + delta prefill + speculative decoding + grammar-constrained executor; watchdog + memory ladder | p50 ≤ 500ms / p95 ≤ 1.2s; TTFT(warm) ≤ 120ms; memory-pressure drill passes; determinism gate still green |
| **3 — Fallback + cascade (2 wks)** | router (advisory), lazy pixel stack, escalation path, occlusion checks, secure-field protocol, Electron manual-AX | broken-AX app subset success ≥ 60%; gremlin chaos suite passes (no foreign-dialog clicks, 100% yield to user input); no regression on AX subset |
| **4 — Learning (3 wks)** — *the moat* | trajectory recording (redacted) → nightly executor LoRA → replay-promote gate → correction synthesis | week-over-week success +5pts on held-out tasks; a trap-task regression blocks any promotion (tested by attempting one) |
| **5 — Product shell (2 wks)** | P1 menu-bar app, onboarding doctor (grants + whose-grant-am-I-using), approval cards + Touch ID, audit viewer, model updater | 2 weeks daily-driver dogfood; every failure logged honestly; alert-fatigue review (ask rate < 1.5/task on the 100-task suite) |

Each gate is measurable on purpose: a phase ends when the gate passes, not when the calendar says so.
