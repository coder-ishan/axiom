# Project CURSOR

A fast, small, **local** computer-use agent for macOS — AX-first, ~2GB resident, sub-500ms steps. Every model is open-source and runs on your machine. Zero cloud, zero telemetry.

> **Status: specification phase.** No agent code yet. This repo contains the full architecture spec in [`docs/spec/`](docs/spec/README.md) — every decision pre-made and justified, with rejected alternatives, failure modes, and invariants. Implementation starts at Phase 0 ([`docs/spec/16-build-phases.md`](docs/spec/16-build-phases.md)).

## The thesis in one sentence

On macOS, the fastest and smallest computer-use agent doesn't look at pixels most of the time — it reads the **Accessibility tree**, plans with a ~2B model, executes with a ~0.3B action model, and keeps a lazily-loaded pixel stack only for apps with broken accessibility.

**Targets:** step p50 < 500ms · resident RAM < 2.5GB · 100% gate on irreversible actions · cold start to first action < 3s.

## Start reading

1. [`docs/spec/00-overview.md`](docs/spec/00-overview.md) — thesis, model roster, latency budget
2. [`docs/spec/01-design-principles.md`](docs/spec/01-design-principles.md) — the 8 principles
3. Full index: [`docs/spec/README.md`](docs/spec/README.md)

The compressed version of the entire spec is [the invariants](docs/spec/15-invariants.md) — the constitution the implementation must uphold.

## Planned repo layout

```text
cursor/
  agent/        loop.py (state machine) · policy.py (gates, kill switch) · yield.py (arbitration)
  perceive/     ax.py (reader + differ — the core IP) · axd.py (AXD/v1 format) · screen.py (lazy pixel lane)
  models/       roster.py (manifest, version-locked) · server.py (resident set) · escalate.py · confidence.py
  execute/      axactions.py · applescript.py (allowlisted templates only) · cgevents.py · contracts.py
  journal/      wal.py · reconcile.py
  memory/       trajectories.py (redaction pipeline lives here)
  train/        nightly.py · promote.py (replay-gated)
  evals/        tasks/ · traps/ · bench/
  fixtures/     axd/ (golden AXD snapshots from ~30 real apps)
  tests/        property/ · chaos/ (gremlin harness) · fault/
  docs/         spec/ (this architecture) · reference/ (research guides)
```

## Companion research

- `docs/reference/local-inference-guide.html` — 30-chapter deep dive on local LLM inference on Apple Silicon (foundations, roofline, quantization, speculative decoding, micro-models, diffusion LMs, the "beat MLX" program).
- `docs/reference/computer-use-agent-plan.html` — the styled HTML version of this spec.

## License

TBD — MIT or Apache-2.0 recommended (everything this project depends on is open source; the repo should be too).
