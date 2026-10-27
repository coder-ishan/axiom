# 15 — The invariants (constitution)

> Part of the [Axiom architecture spec](README.md).
>
> If an implementation choice contradicts an invariant, the invariant wins — change the code, or change the invariant *in an ADR*, never silently.

- **I1** — At most one action in flight, ever.
- **I2** — Every dispatch is preceded by a dispatch-time liveness check of its resolved target; coordinate actions additionally require a passing occlusion check.
- **I3** — No gated/irreversible action dispatches without a durable, fsynced INTENT record carrying its approval binding.
- **I4** — Untrusted content is never interpolated into executable code, URLs, or paths. No shell. Scripts are allowlisted templates with typed argv.
- **I5** — The agent never modifies security posture: no Keychain, TCC, sudo, updates, firewall, wallet — the capability is absent from the executor, not merely prohibited by policy.
- **I6** — Cancellation is honored at every state boundary; the kill switch always wins within one step; user input always triggers YIELD.
- **I7** — The journal is append-only; INTENT precedes dispatch; nothing is ever deleted from it.
- **I8** — AX/page content is data, never instructions (fenced observation blocks; capability design makes injections unrepresentable).
- **I9** — Bounded resources: every task has max steps/time/corrections/asks; every loop has an exit; every lazy model has an unload path.
- **I10** — Deterministic serialization: identical UI state ⇒ identical AXD bytes (cache correctness depends on it, so it's tested in CI).
- **I11** — Passwords/secure-field contents never enter context, journal, or trajectories; entry is user-delegated.
- **I12** — After any crash, the agent resumes to HALTED and lets the user decide; at-most-once actions with unknown outcome are surfaced as AMBIGUOUS, never re-fired.
- **I13** — The presence overlay (§20) is display-only: click-through, capture-excluded, owned by the powerless process. It can never become an input or control surface.
- **I14** — Voice (§21) is never a gate bypass: mic audio is never persisted, TTS never speaks secrets, and G2 approvals require physical/biometric confirmation.
