# 03 — The agent loop as an explicit state machine

> Part of the [Axiom architecture spec](README.md).

No ad-hoc loop. The agent is a state machine with named states, guarded transitions, per-state timeouts, and a persisted step pointer. Every transition appends to the journal (§06), so any crash lands in a *known* state.

```text
                    goal
        ┌──────┐        │
        │ IDLE ├───────► BOOT ───infeasible──────► FAILED
        └──────┘           │ (permission check, task card, budget init)
                           ▼
   ┌────────────────────── step cycle ──────────────────────┐
   │                                                        │
   │  OBSERVE ─► ROUTE ─► PLAN ─► VALIDATE ─► GATE? ─► DISPATCH ─► SETTLE ─► VERIFY ─┐
   │    ▲         │        │         │        │(await user)  │         │       │   │
   │    │         │pixellane│escalate │block   │deny/timeout  │         │diff   │ok │
   │    │         ▼        ▼         ▼        └─cancel action ▼         ▼       │   │
   │    └── lane ok   PLAN′    CORRECT ◄────────────────────────────────────┘    │
   │                 ▲          │ >3 corrections or detected cycle               │
   │                 └──────────┼────────────────────────────────────────────────┘
   │                            ▼
   │                          HALTED   (bounded stop: budget/kill/lock/panic)
   └── goal met ──► DONE        │ user resumes → back to OBSERVE (never auto)
                                │
   ANY STATE ──user input──► YIELD ──user resumes──► OBSERVE (same step re-done)
   ANY STATE ──kill/lock/FUS─► HALTED  (kill also releases event taps + sleep asserts)
```

## State semantics & timeouts

| State | Does | Timeout | On timeout |
|---|---|---|---|
| BOOT | Verify grants (doctor), build task card, init budgets (steps ≤ 60, wall ≤ 10min, asks ≤ 3, corrections ≤ 3) | 2s | FAILED with diagnosis |
| OBSERVE | AX read of frontmost app + delta vs last (AXD/v1, §04) | 400ms total (AX messaging timeout 300ms) | Mark app *sick*; route to pixel lane or skip-step (max 2 skips) |
| ROUTE | Perception lane decision: heuristics first, router model for ambiguous middles (§04) | 50ms | Heuristic-only decision (router is advisory by design) |
| PLAN | Pilot 2B (+ draft speculation) → scratchpad + action JSON, confidence = logit margin | 2.5s decode budget | Escalate to 9B once; still over → GATE with "I'm not sure" framing |
| VALIDATE | Schema ✓ (grammar already), target resolves in *live* tree, args typed, policy check | 20ms | Executor reformat loop ≤ 2 → CORRECT |
| GATE | Async: approval card in P1, bound to intent ID (§09). Only for G2-class actions | 120s | Cancel action (not task), re-ask once, then HALT task |
| DISPATCH | Pre-dispatch checks (§05), then execute primitive. Composite actions chunked | 150ms/primitive, 1.5s/script | Record partial, treat as SETTLE-ambiguous (§06) |
| SETTLE | Wait for the contract's effect signature or settle timeout (default 400ms) | per action contract | Verify anyway with "weak" expectation → CORRECT if diff contradicts |
| VERIFY | Re-observe; compare against contract's expected-effect predicate | 500ms | CORRECT (counts toward budget) |
| CORRECT | Re-plan with the diff attached ("expected X, got Y") | — | >3 → HALT with diagnosis summary |
| YIELD | Parked; user owns the machine. No timers | ∞ | — |
| HALTED / DONE / FAILED | Terminal; card shown with what happened, what was already done, resume offer | — | — |

> [!IMPORTANT]
> **Loop-level rules**
>
> - **At most one action in flight** — ever. No pipelining of dispatches.
> - **Cancellation is honored at state boundaries only.** The kill switch sets an abort flag read at every transition; an in-flight dispatch (<150ms) completes, then the machine halts. A keystroke *burst* is chunked — abort happens between chunks, never mid-field, and the field value is reconciled on halt.
> - **Cycle detector:** keep the last 6 state-hashes (hash of AXD doc). A repeated hash with the same planned action → declared stuck → HALT with hypothesis ("maybe a dialog needs attention").
> - **Task concurrency: none.** Tasks queue serially. Two agents clicking at once is state interference nobody can debug; parallelism is a v2+ question.
