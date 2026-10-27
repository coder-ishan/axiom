# Contributing to Axiom

Thanks for your interest. Axiom is a safety-critical piece of software by design — it drives a real machine — so contribution discipline is the product.

## Spec-first workflow

- The architecture lives in [`docs/spec/`](docs/spec/README.md). If a code change contradicts an invariant ([`15-invariants.md`](docs/spec/15-invariants.md)), **the invariant wins**: change the code, or change the invariant *through an ADR* ([`14-adr-log.md`](docs/spec/14-adr-log.md)) — never silently.
- Behavioral changes without a spec section are rejected at review; write the section first, even if it's one paragraph.

## What a PR must pass

1. **Property tests** — differ idempotence, serialization determinism (same UI state → byte-identical AXD).
2. **Fault-injection suite** — crash at any record/state boundary must land in resume-to-HALTED.
3. **Chaos (gremlin) suite** — no foreign-dialog clicks; 100% yield to user input.
4. **Trap evals** — any regression on the 20 trap tasks blocks the PR outright.
5. **Latency CI** — step p50/p95 regression > 10% fails.

## House rules

- **No network in the agent path.** No cloud calls, no telemetry, no analytics — anywhere, ever. CI must prove it, not just review.
- **No credential management.** PRs that add secret storage are rejected by policy ([`18-non-goals.md`](docs/spec/18-non-goals.md)).
- **Capabilities are made impossible, not forbidden** (§01 principle 3) — prefer removing the primitive over gating it, when both work.
- **Every external call has a timeout and a fallback** (§03). If you added an unbounded wait, you added a bug.
- PII discipline: fixtures are redacted; trajectories pass the redaction pipeline (§10) before touching disk.

## Commit style

Conventional Commits (`feat:`, `fix:`, `spec:`, `adr:`, `eval:` …). `spec:` changes touch `docs/spec/` only.

## Licensing

By contributing you agree your contributions are licensed under Apache-2.0 (see `LICENSE`). New third-party dependencies must be OSI-approved and recorded in `ATTRIBUTION.md` with a pin hash.
