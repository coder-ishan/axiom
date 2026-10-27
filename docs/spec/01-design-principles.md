# 01 — Design principles

> Part of the [Axiom architecture spec](README.md).

1. **Fail closed.** Missing permission, dead model server, ambiguous state → the agent stops. It never "tries anyway."
2. **Never act blind.** Every dispatch is preceded by a dispatch-time check that the target still exists in the live tree. The plan-time observation is a hypothesis; the live tree at t<sub>dispatch</sub> is the truth.
3. **Make wrong actions impossible, not forbidden.** No shell interpolation into AppleScript (parameterized templates only); grammar-constrained JSON (malformed calls are structurally unrepresentable); the executor code simply does not implement dangerous primitives.
4. **Bounded everything.** Every loop has an exit: max steps, max time, max corrections, max asks, max memory. No unbounded retry anywhere.
5. **The user always wins.** User input detected → yield within ~200ms. This is non-negotiable; an agent that fights the user for the mouse is disqualified.
6. **Auditability before capability.** Write the intent to the journal *before* dispatch. Forensics must always be possible, including after a crash.
7. **Content is data, not instructions.** AX labels, page text, filenames — all untrusted data. Injection resistance comes from capability design, not from asking the model to behave.
8. **Determinism for cacheability.** Identical UI state → byte-identical serialized observation, or the prefix cache (the whole speed story) dies.

Every mechanism in §02–§12 exists to uphold one or more of these. Every invariant in §15 is one of these made checkable.
