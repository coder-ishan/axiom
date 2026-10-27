# 05 — Action contracts: TOCTOU, idempotency, partial actions

> Part of the [Axiom architecture spec](README.md).

This section is the heart of the architecture. Every action class declares a **contract**: what it expects to change, how we verify it, whether retrying it is safe, and what gate it needs. Contracts turn "the UI didn't change — should I click again?" from a guess into a policy.

## Contract table

| Action class | Idempotency | Effect signature (VERIFY looks for) | Settle | Retry policy | Default gate |
|---|---|---|---|---|---|
| AXPress — button | varies by button | tree/window/value change, sheet opens, focus moves | 400ms | Retry only if classified *pure* (nav/select); else GATE on 2nd attempt | G2 if label matches destructive set |
| AXPress — menu item | pure (mostly) | menu closes, state/value change | 300ms | safe to retry after re-resolve | G0 |
| AXSetValue — text field | idempotent | field value == intended (read-back) | 100ms | set again is safe | G0 (G2 if field is a recipient/address) |
| Paste — text | idempotent per target | field value diff matches payload prefix | 150ms | re-paste safe; pasteboard change-count confirms delivery; **clear pasteboard after, never log payload** | G0/G2 by target |
| Keystroke burst | *append* semantics | read-back after each chunk | per chunk | chunked (≤32 chars); abort between chunks; never resend from scratch blindly | G0 |
| Key combo (⌘S…) | varies wildly | sheet/dialog/appearance change | 400ms | 1 retry after re-observe | G1 |
| Click @ coordinates | varies | same as press | 400ms | last resort only (see occlusion check) | G1 min |
| Scroll / drag | pure | scroll-position attr delta | 150ms | safe | G0 |
| Launch app / open URL | mostly pure | process exists + window appears (or URL handler ran) | 1s | safe | URLs: G2 unless derived from user's typed goal (§10) |
| Window close / app quit | destructive-ish | window gone / pid gone | 600ms | **never auto-retry** | G1; G2 if unsaved-document state detected |
| File move/delete, send, pay, install | **at-most-once** | domain-specific (Finder item check, Sent-folder check at +2s) | varies | **never blind-retry:** reconcile state first (§06) | G2 (+ Touch ID for top class) |

## The dispatch-time protocol (5 checks, every dispatch, no exceptions)

1. **Liveness:** target id still present in a *freshly re-read* minimal subtree around it (cheap partial re-observe, ~20–50ms). The plan-time observation is stale by definition.
2. **Occlusion (coordinate actions only):** the topmost window under the point must belong to the target app/window — `CGWindowListCopyWindowInfo` bounds+PID hit-test (works without window-name permission; names of other apps are redacted but bounds/PID are not). A coordinate click into an occluded point hits *whatever is on top* — the classic wrong-window bug; without a passing check, the dispatch is disallowed.
3. **Policy:** app allowlist, capability caps, gate-class resolution (matrix below).
4. **Gate:** G2 → await approval bound to this intent ID.
5. **Journal INTENT record flushed** (§06) — then dispatch.

> [!IMPORTANT]
> **Decision — when to ask the user: the irreversibility × verifiability matrix**

| | Verifiable (strong) | Verifiable (weak) | Not verifiable |
|---|---|---|---|
| **Reversible** | G0 auto | G0 auto | G1 notify |
| **Semantic-risk** (sends, closes unsaved) | G0 auto | G1 notify | G2 confirm |
| **Irreversible** (delete, pay, install, security) | G2 confirm | G2 confirm | **Refuse** (out of capability) |

Hard floor (always G2 + Touch ID, regardless of what anything "believes"): Keychain, TCC/security settings, sudo, software updates, wallet/payment, `rm`-class file deletion, anything matching the destructive lexicon — matched on the **resolved target + action**, never on the model's claim of what it's doing.

Gate classes: **G0** auto (read-only / reversible / in-app navigation) · **G1** notify (visible to user, low risk) · **G2** confirm (irreversible or data-leaving).

## Partial-action semantics

- **Actions are atomic primitives, not stories.** "Fill the form and send" is a task of many primitives; the journal records each. There is no multi-primitive transaction — after any interruption, the state is whatever the primitives left, and reconciliation (§06) reads the UI to find out.
- **The interrupted keystroke burst:** abort between chunks; on resume, read the field, diff against the intended payload, and set the remainder via AXSetValue (idempotent completion of an append — never re-type the prefix).
- **The double-click trap:** a click whose dialog is slow to appear, followed by a "retry" click, lands inside the dialog — possibly on a same-labeled button ("Delete" → confirm dialog also has "Delete"). Countermeasure: after any dispatch, *re-resolve the target before any repeat*; a second press of the same id requires the contract's explicit retry permission. Never "click the same frame again because it felt like a no-op."
