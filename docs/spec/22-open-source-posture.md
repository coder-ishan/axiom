# 22 — Open-source posture

> Part of the [Axiom architecture spec](README.md).

## License

**Apache-2.0** for Axiom's own code (chosen over MIT for the explicit patent grant — meaningful for a project that orchestrates third-party model weights). See `LICENSE` at the repo root.

## Component license hygiene (ATTRIBUTION.md)

Every model, runtime, and library the stack depends on is inventoried in `ATTRIBUTION.md` with its license and pin hash. Two rules:

1. **OSI-preferred by default.** Apache-2.0 / MIT / CC-BY-4.0 components only.
2. **Any non-OSI component is marked swappable** with a named Apache-2.0 alternative (e.g., if a vision or executor base ships under a custom license, the roster lists its drop-in replacement). Axiom's *own* code never depends on a non-OSI license to function — that's what "open source, every option" means operationally.

## Repo furniture for open-source readiness

| File | Purpose |
|---|---|
| `LICENSE` | Apache-2.0, full text |
| `CONTRIBUTING.md` | Spec-first workflow: invariants win; ADR for any change; which gates a PR must pass |
| `SECURITY.md` | Threat model pointer, scope, private reporting channel, honest residual-risk statement |
| `ATTRIBUTION.md` | Every component + model: license, source, pin-at-time-of-use |
| `README.md` | Public face: what/why, guarantees, targets, reading path |

## The "fully local" guarantees (public claims, all checkable)

- **No network in the agent path.** Model downloads only, pinned by hash; the orchestrator has no egress dependency.
- **No telemetry.** Metrics are local (§12); nothing phones home, ever.
- **No credential storage.** Passwords are user-typed (§09); Axiom does not manage secrets.
- **Eval-gated everything.** Model updates, LoRA promotions — replay-gated against traps (§07, §19).
- **Kill switch + audit log.** Forensics always possible (§06); the user always wins (§09).
- **Open-source stack end to end** — the roster (§00, §21) is downloadable, inspectable, runnable offline.
