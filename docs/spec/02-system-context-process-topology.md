# 02 — System context & process topology

> Part of the [CURSOR architecture spec](README.md).

```text
┌────────────────────────────────────────────────────────────────────┐
│  P1 cursor (menu-bar app / CLI v0)  — holds NO special powers       │
│     approval cards · kill switch · live status · audit viewer      │
│     Touch ID prompts for the top gate class (LocalAuthentication)  │
└──────△─────────────────────────────────────────────────────────────┘
       │ unix domain socket, SO_PEERCRED (same-UID only), heartbeat 1s
┌──────▽─────────────────────────────────────────────────────────────┐
│  P2 cursord (orchestrator, Python)                                 │
│     agent loop state machine · policy engine · journal (WAL)       │
│     AX tree reader + delta differ   ← holds Accessibility grant    │
│     action dispatch: AX actions / AppleScript templates / CGEvents │
└──────△─────────────────────────────────────────────────────────────┘
       │ localhost HTTP (OpenAI-compatible + /route /embed endpoints)
┌──────▽──────────────────────────┐  ┌─────────────────────────────────────────┐
│  P3 cursor-models (resident ~2GB)│  │  P4.x lazy servers (spawned on demand,  │
│  pilot 2B + draft 0.6B (spec.)   │  │  killed under memory pressure):         │
│  executor 270M (grammar-constr.) │  │   · escalation Qwen3.5-9B ~5.5GB        │
│  router 66M · embedder MiniLM    │  │   · pixel stack: Holo1.5-3B ~1.9GB      │
└─────────────────────────────────┘  │     + ScreenCaptureKit capture          │
                                     │   · GLiNER PII 0.3B (log-time only)     │
                                     └─────────────────────────────────────────┘
```

## Why this topology

- **Crash containment:** the crash-prone part is the models (Metal OOM, engine asserts). They die in P3/P4; P2's journal is on disk and launchd relaunches. The AX-granted process (P2) is the *stable* one.
- **Permission locality:** TCC grants bind to a code-signing designated requirement. P2 holds Accessibility + Automation; P1 holds LocalAuthentication + (fallback) Screen Recording. Never one process holding everything.
- **Control-plane hijack defense:** the P1↔P2 socket checks peer UID. A same-user process can still reach it — see §10 (Threat model) for the honest limits and layered mitigations.

> [!WARNING]
> **Edge case: the TCC re-grant churn (hits every developer of such tools).** Accessibility grants are keyed to the binary's designated requirement. Ad-hoc-signed binaries get a *new* requirement on every rebuild → macOS revokes and re-prompts. Mitigations: one stable signing identity for dev builds (self-signed cert with fixed CN, not ad-hoc), fixed bundle ID, and a first-run doctor command that *verifies* all grants and explains any failure precisely ("re-grant after rebuild #4 — expected, here's why").
>
> A second, subtler trap: launching from Terminal means the agent inherits **Terminal's** AX grant — it works, silently, under the wrong identity. The doctor must report whose grant is actually in effect.

> [!IMPORTANT]
> **Decision — supervision model.** v1 is supervised-only. No unattended/autonomous operation: the agent acts only while the user is at the machine and has an active task. This is what makes the threat model (§10) tractable. Autonomy is a later product decision, gated on the gate/audit machinery being proven — not a v1 feature.
