# 09 — Human arbitration: the user always wins

> Part of the [Axiom architecture spec](README.md).

## Yield semantics

- **User-input detection:** a listen-only `CGEventTap` (mouse + keyboard; Input Monitoring grant) — never swallows events, observes only. Any user mouse/keyboard activity during an active task → YIELD within ~200ms at the next state boundary (an in-flight dispatch <150ms completes first; keystroke bursts stop between chunks).
- Corollary invariant: **the agent never types while the user is typing** — arbitration is time-division, not priority.
- Screen locked / fast user switch / session change → immediate HALTED. AX reads at the login window are useless, and key-sending near it is a security hazard; policy treats lock as "the machine left my care."

## Confirmation design (against alert fatigue — the real vulnerability)

- Cards state the **effect in plain language** ("Send email titled 'Invoice' to kai@…") plus the resolved target (app, element role, label) — the user confirms *reality*, not the model's story.
- Approvals are bound to intent IDs; an approved card cannot silently cover a different action. One approval = one intent.
- **No batch auto-approve.** "Always allow" memory is per action-pattern, capped at 5 uses, and the G2-top class (pay/install/security/delete-all) never memorizes — Touch ID every time (LocalAuthentication in P1).
- **Asking budget: ≤ 3 asks per task** → beyond that, HALT ("this task needs decisions I keep having to make — let's do it together"). More asks = the agent is out of its depth; saying so is the honest output.
- **Secure fields (passwords):** the agent focuses the field, then yields: "Please type your password." Completion is verified by observing the downstream state (submit enabled / next screen), never by reading the field. Passwords never touch the journal, the trajectory corpus, or the context window. The agent does not manage credentials at all in v1; if a workflow needs one, it asks the user live.

## Voice arbitration (§21 integration)

- Voice commands (wake word → dictation) are an explicit input channel; they do not trigger YIELD — they *direct* the agent.
- **"Stop."** — a dedicated, high-precision-only phrase, equivalent to the kill switch.
- **G2 is never voice-approved** (speaker-injection defense, §10): gate read-backs are announced via TTS, approval still requires Touch ID or a physical click.
- TTS announcements duck/cancel when the VAD hears the user speaking — Axiom never talks over you.
