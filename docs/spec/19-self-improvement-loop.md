# 19 — Self-improvement loop (the moat)

> Part of the [Axiom architecture spec](README.md).

The loop the cloud can't run: every day of use makes *your* executor better — locally, on your trajectories, gated by replay.

## The four stages

1. **Record.** Every verified trajectory (AX deltas + actions + outcomes) is automatically a training example. The journal already contains the data — this stage is a view, not a pipeline.
2. **Synthesize.** Failed tasks get corrected by the escalation model (9B); (state, corrected-action) pairs become new examples. This is the FunctionGemma recipe — 270M going 46% → 90% purely via synthetic fine-tuning.
3. **Fine-tune nightly.** MLX LoRA on the 270M executor (minutes on a 16GB Mac); weekly QLoRA on the 2B pilot (32GB+ machines only).
4. **Verify before promote.** The candidate LoRA replays the last-100 trajectories *plus* the trap suite; promote only on non-regression (§07 update gating). Rollback is a file swap.

> [!IMPORTANT]
> **Decision — the executor (0.27B) is the fine-tuning surface, not the pilot.** It's cheap to train, narrow in output space, and its errors are caught by the validator. The pilot stays general; the executor absorbs your machine's specifics — your apps, your workflows, your naming.

## Guardrails the loop must respect

- All training data passes the PII redaction pipeline **before** persistence (§10).
- Promotion is eval-gated (trap regression blocks outright) — a nightly job can never silently make the agent worse (§07).
- Training runs use the memory-pressure-aware scheduler (§07): a nightly LoRA job yields to user activity; it does not compete with foreground GPU work.
- A promoted adapter that misbehaves in production is one file-swap from rollback; the audit viewer shows which adapter acted at every step.
