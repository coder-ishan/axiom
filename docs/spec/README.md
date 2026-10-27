# Axiom Architecture Specification — Index

> Version 2 (October 2026). v1 said *what* to build; this spec says *how it behaves when everything goes wrong* — which is 80% of what a responsible agent architecture actually is.

## Reading order

| # | Doc | What it answers |
|---|-----|-----------------|
| 00 | [Overview](00-overview.md) | Thesis, why AX-first, model roster, where every millisecond goes |
| 01 | [Design principles](01-design-principles.md) | The 8 non-negotiables every other section derives from |
| 02 | [System context & process topology](02-system-context-process-topology.md) | Which process holds which power, crash containment, permission topology |
| 03 | [Agent loop as a state machine](03-agent-loop-state-machine.md) | States, transitions, per-state timeouts, cancellation semantics |
| 04 | [AX perception subsystem](04-ax-perception-subsystem.md) | Reader mechanics, stable IDs, the AXD/v1 format, lane decision |
| 05 | [Action contracts](05-action-contracts.md) | TOCTOU, idempotency, partial actions, the dispatch-time protocol |
| 06 | [Journal & crash recovery](06-journal-crash-recovery.md) | WAL records, ambiguous-intent reconciliation, never-auto-resume |
| 07 | [Model lifecycle & resources](07-model-lifecycle.md) | Load matrix, memory-pressure ladder, watchdog, hot-swap, update gating |
| 08 | [Caching rules](08-caching-rules.md) | Every cache, its key, its invalidation, its stale failure mode |
| 09 | [Human arbitration](09-human-arbitration.md) | Yield semantics, confirmation design, secure-field protocol |
| 10 | [Threat model](10-threat-model.md) | Injection, malicious labels, control-plane hijack — defenses and honest limits |
| 11 | [Failure-mode taxonomy](11-failure-mode-taxonomy.md) | Failure × detection × response, per state |
| 12 | [Observability](12-observability.md) | Traces, live status, metrics, postmortems |
| 13 | [Test strategy](13-test-strategy.md) | Property tests, fixtures, fault injection, chaos, evals, latency CI |
| 14 | [ADR log](14-adr-log.md) | 9 decisions with rejected alternatives |
| 15 | [Invariants](15-invariants.md) | The constitution — I1…I12 |
| 16 | [Build phases & gates](16-build-phases.md) | 6 phases, each with a measurable gate |
| 17 | [Risk register](17-risk-register.md) | Risks, mitigations, honest residuals |
| 18 | [Explicit non-goals](18-non-goals.md) | What v1 refuses to do, and what would change that |
| 19 | [Self-improvement loop](19-self-improvement-loop.md) | Trajectories → nightly LoRA → replay-gated promotion |
| 20 | [Visual presence — the dot](20-visual-presence.md) | Menu-bar dot, on-screen action dot, field halo — with a rendering contract |
| 21 | [Voice I/O](21-voice-io.md) | Wake word, STT, TTS — fully local; voice never bypasses gates |
| 22 | [Open-source posture](22-open-source-posture.md) | License, attribution, the "fully local" guarantees |

## How the docs fit together

- **01 (principles) → 15 (invariants)** are the contract; **02–12** are the mechanisms; **13** proves the mechanisms; **14** records why each mechanism won; **16–18** bound the work; **19** compounds it.
- If an implementation choice contradicts an invariant, the invariant wins — change the code or change the invariant *in an ADR*, never silently.
