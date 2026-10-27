# 06 — The journal: crash recovery semantics

> Part of the [Axiom architecture spec](README.md).

An append-only JSONL write-ahead log, one file per task. Flush rule: **INTENT records for gated/irreversible actions are fsynced before dispatch**; everything else is buffered (≤200ms loss window).

## Record shapes

```json
{"t":"INTENT","task":42,"step":7,"intent_id":"i-3f2c","action":"press",
 "target":"a1b2c3d4","app":"com.apple.mail","gate":"G2","approval":"ap-991"}
{"t":"RESULT","intent_id":"i-3f2c","outcome":"ok|failed|timeout|partial",
 "effect":"sheet-opened","latency_ms":212}
{"t":"STATE","step":7,"axd_hash":"9f13...","summary":"replied to Kai, attached PDF"}
```

## Crash → restart protocol (resume-to-HALTED, never auto-resume)

1. launchd relaunches P2. It reads the newest task journal. If the last record is INTENT without RESULT → **ambiguous: the action may have fired.**
2. **Reconciliation protocol:** re-observe; search for the contract's effect signature (§05).
   - Found → write a synthetic RESULT ("effect observed post-crash").
   - Not found → outcome *unknown*:
     - if the action is **pure** → safe to mark failed; re-execution allowed after explicit user resume.
     - if **at-most-once** (send/pay/delete) → the task is marked **AMBIGUOUS** and surfaced to the user with the state diff. The agent *never* re-fires an at-most-once action whose effect it cannot confirm or deny.
   - This is why gated actions exist: an ambiguous crash can only ever happen at a place the user already saw and approved.
3. The agent resumes only to **HALTED** and shows a card: "These steps already happened: … I stopped because … Resume?" Resumption is a user click, never automatic.

> [!WARNING]
> **Degradation: disk full / log failure.** Journal write failure → actions continue, but gated/irreversible actions are *refused* (I7: no dispatch without a durable intent record). Everything else buffers to RAM with a 1MB cap, dropping oldest. A missing audit trail must not be possible for dangerous actions — that's the one case where the log outranks the work.
