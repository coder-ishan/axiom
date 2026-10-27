# 11 — Failure-mode taxonomy (per state, with responses)

> Part of the [CURSOR architecture spec](README.md).

| Where | Failure | Detection | Response |
|---|---|---|---|
| OBSERVE | app hung (AX IPC stall) | messaging timeout 300ms | mark sick 30s → pixel lane or skip (≤2) → GATE |
| OBSERVE | no/empty tree (Electron, game, canvas) | element-count heuristic | manual-accessibility attempt ×1 → pixel lane |
| OBSERVE | tree explosion (Chrome DOM 10k+) | cap counters | prioritized truncation + T markers; targeted expansion on demand |
| ROUTE | router misclassification | downstream signals (pixel grounding low / AX dangling refs) | hysteresis limits damage to 1 switch per 5 steps; router is advisory |
| PLAN | goal infeasible | pilot emits "cannot" class | FAILED with reason — never silent guessing |
| PLAN | low confidence | logit margin < τ | escalate once → still low → GATE ("not sure" framing) |
| PLAN | decode budget blown | token/time budget | escalate → GATE; also signals the prompt is too big → compact |
| VALIDATE | dangling target ref (element vanished) | live-tree resolution fails | re-resolve by fuzzy match (embedder) → re-observe → CORRECT |
| VALIDATE | schema violation | should be impossible (grammar); validator double-checks | executor reformat ≤2 → CORRECT (grammar bug hunt if ever hit) |
| DISPATCH | occlusion changed since plan | the dispatch-time check itself | abort → re-observe → CORRECT (never click blind) |
| DISPATCH | Secure Input active → synthetic keys suppressed | chunk read-back mismatch | switch to AXSetValue/paste; if impossible (true password field) → user delegation (§09) |
| DISPATCH | AX action returns false / script fails | API result | 1 template-true retry → CORRECT with error attached |
| SETTLE/VERIFY | no effect observed | contract signature absent after settle | classify: pure → 1 careful retry after re-resolve; at-most-once → reconcile, no blind retry |
| VERIFY | unexpected diff (foreign dialog, notification) | delta contains elements outside plan scope | new state; re-plan (dialog owner checked per §10); never click through |
| VERIFY | loop (same state, same action) | state-hash ring, K=6 | HALT with diagnosis |
| Any | model server crash | HTTP failure / health check | restart ≤3 with backoff (P3), lazy respawn (P4) → else HALT |
| Any | memory pressure | dispatch source | ladder (§07) → degrade → GATE |
| Any | kill switch / lock / fast user switch / P1 heartbeat lost | abort flag / distributed notifications / heartbeat | HALTED at next boundary; bursts stop between chunks; assertions released |
| Any | P2 crash mid-task | launchd relaunch + journal read | resume-to-HALTED + reconciliation (§06) |
