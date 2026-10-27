# 08 — Caching rules & invalidation (the speed story, made safe)

> Part of the [Axiom architecture spec](README.md).

| Cache | Contents | Key | Invalidated when | Failure mode if stale | Defense |
|---|---|---|---|---|---|
| L1 prefix (system+schema) | system prompt, action JSON schema, AXD format spec | hash(prompt + schema + model + adapter) | model/adapter/schema/prompt version change | model "forgets" its output format → schema violations | key includes all versions; mismatch = cold prefill, never silent |
| L2 app card | per-app summary + workflow hints | (bundle-id, app-version, window-kind) | app switch, app relaunch, 5-min TTL | hints describe a UI that changed | card carries an `as-of` tree-hash; header mismatch → rebuild |
| L3 tree state | last AXD doc (delta base) | seq + app + window | app switch, window open/close burst, delta-overflow | delta computed against the wrong base → garbage diff | header check (app/pid/window-hash) before diffing; mismatch → full resnapshot |
| KV cache | pilot session context | task id | task switch (always — privacy + correctness), compaction | cross-task context bleed (a privacy bug, not just a quality bug) | task-scoped; 8-bit quantized; LRU-evicted |
| Process cache | warm lazy servers | — | ladder / idle timers (§07) | — | — |

> [!IMPORTANT]
> **The cacheability invariant.** Same UI state ⇒ byte-identical AXD ⇒ prefix cache hits. Canonical ordering (§04) is therefore a *correctness* requirement, not a nicety — and the determinism property test (§13) is a hard CI gate.
