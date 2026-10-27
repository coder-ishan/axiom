# Axiom

![License](https://img.shields.io/badge/license-Apache--2.0-blue) ![Platform](https://img.shields.io/badge/platform-macOS%20·%20Apple%20Silicon-black) ![Cloud](https://img.shields.io/badge/cloud-none-green) ![Status](https://img.shields.io/badge/status-specification--phase-orange)

A fast, small, **fully local** computer-use agent for macOS — AX-first, ~2GB resident, sub-500ms steps, with a presence dot and full voice I/O. Every model is open-source and runs on your machine. Zero cloud, zero telemetry, zero credential storage.

> **Status: specification phase.** No agent code yet. This repo contains the full architecture spec in [`docs/spec/`](docs/spec/README.md) — every decision pre-made and justified, with rejected alternatives, failure modes, and invariants. Implementation starts at Phase 0 ([`docs/spec/16-build-phases.md`](docs/spec/16-build-phases.md)).

## The thesis in one sentence

On macOS, the fastest and smallest computer-use agent doesn't look at pixels most of the time — it reads the **Accessibility tree**, plans with a ~2B model, executes with a ~0.3B action model, and keeps a lazily-loaded pixel stack only for apps with broken accessibility.

**Targets:** step p50 < 500ms · resident RAM < 2.5GB · 100% gate on irreversible actions · cold start to first action < 3s.

## What Axiom is

- **Computer use, fully local** — the agent observes via the Accessibility tree, acts via AX actions / AppleScript templates / CGEvents, and verifies every action against action contracts ([§05](docs/spec/05-action-contracts.md)).
- **The dot** ([§20](docs/spec/20-visual-presence.md)) — a menu-bar state dot plus a floating on-screen dot at the exact dispatch point of every action, click-through and capture-excluded. Where the agent acts is always visible; every dot maps to an auditable journal record.
- **Voice, end to end** ([§21](docs/spec/21-voice-io.md)) — wake word (openWakeWord), speech-in (Parakeet 0.6B, beats Whisper large-v3), speech-out (Kokoro-82M), VAD always-on (Silero). Voice directs the agent; it never bypasses a gate.
- **Learning that compounds** ([§19](docs/spec/19-self-improvement-loop.md)) — trajectories → nightly executor LoRA → replay-gated promotion.

## The "fully local" guarantees (all checkable, not vibes)

- **No network in the agent path** — model downloads only, pinned by hash; no egress dependency.
- **No telemetry** — metrics are local (§12); nothing phones home, ever.
- **No credential management** — passwords are user-typed (§09); Axiom stores no secrets.
- **Eval-gated everything** — model updates and LoRA promotions replay the trap suite; a regression blocks promotion.
- **Kill switch + append-only audit journal** — forensics always possible, including after a crash (§06).
- **Open-source stack end to end** — see [`ATTRIBUTION.md`](ATTRIBUTION.md) for every component's license.

## Start reading

1. [`docs/spec/00-overview.md`](docs/spec/00-overview.md) — thesis, model roster, latency budget
2. [`docs/spec/01-design-principles.md`](docs/spec/01-design-principles.md) — the 8 principles
3. Full index: [`docs/spec/README.md`](docs/spec/README.md)

The compressed version of the entire spec is [the invariants](docs/spec/15-invariants.md) — the constitution the implementation must uphold.

## Repo furniture (open-source ready)

| | |
|---|---|
| [`LICENSE`](LICENSE) | Apache-2.0 (patent grant — appropriate for a project orchestrating third-party model weights) |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | Spec-first workflow; invariant-wins rule; the 5 gates every PR must pass |
| [`SECURITY.md`](SECURITY.md) | Threat-model pointer, private reporting channel, honest residual-risk statement |
| [`ATTRIBUTION.md`](ATTRIBUTION.md) | Every model/runtime dependency with its license and swap plan |

## Planned repo layout

```text
axiom/
  agent/        loop.py (state machine) · policy.py (gates, kill switch) · yield.py (arbitration)
  perceive/     ax.py (reader + differ — the core IP) · axd.py (AXD/v1 format) · screen.py (lazy pixel lane)
  models/       roster.py (manifest, version-locked) · server.py (resident set) · escalate.py · confidence.py
  execute/      axactions.py · applescript.py (allowlisted templates only) · cgevents.py · contracts.py
  voice/        vad.py (Silero) · wake.py (openWakeWord) · stt.py (Parakeet) · tts.py (Kokoro)
  presence/     menubar.py (state dot) · overlay.py (action dot + field halo)
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
- `docs/reference/computer-use-agent-plan.html` — the styled HTML snapshot of this spec.
