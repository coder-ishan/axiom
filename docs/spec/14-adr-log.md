# 14 — ADR log: decisions with rejected alternatives

> Part of the [Axiom architecture spec](README.md).

| ADR | Decision | Alternatives rejected | Why / consequence |
|---|---|---|---|
| 001 | AX-first perception, pixel lane lazy | screenshot-first (universal); AX-only (no fallback) | macOS exposes the UI as structured text → 10–30× less prefill, exact grounding, 0.3B-class executor. Consequence: per-app AX coverage becomes a first-class metric; the pixel lane exists for the rest. |
| 002 | 3 process classes (§02) | all-in-one (crash coupling, permission mixing); process-per-model (5× IPC + supervision sprawl) | crash isolation where crashes happen; permission locality; clean lazy unload = process exit. |
| 003 | Python + pyobjc for P2/P3 in v1 | Swift-native everything | ~2× iteration speed; the AX reader runs 30–80ms in Python — inside budget; a Swift port is a later optimization, only if p50 misses. |
| 004 | localhost HTTP between P2 and model servers | XPC (Swift-only, more code); in-process models (crash coupling) | OpenAI-compatible surface matches mlx-lm/llama.cpp ecosystems, and the future dLLM motor plugs in unchanged. ~1–3ms IPC is noise at these latencies. |
| 005 | Journal = append-only JSONL WAL | SQLite (also viable) | trivially inspectable/greppable/fsync-able; SQLite reserved for the trajectory *index*. Consequence: recovery logic reads a log, not a database — simpler to prove. |
| 006 | Text entry: AXSetValue → paste → chunked keystrokes | synthetic typing as default | synthetic keys break under Secure Input and interleave badly; setValue is atomic + read-back-verifiable. Keystrokes are the last resort, chunked with verification. |
| 007 | AppleScript only via allowlisted parameterized templates | generated script source (model writes AppleScript) | generated script = arbitrary code execution via prompt injection. Templates + JSON argv = the SQL-parameterization move. Cost: exotic actions wait on new templates — accepted. |
| 008 | Redact at persistence; model context stays faithful | redact before the model (breaks grounding on PII-bearing UIs) | local-only context has no exfil channel; the disk (trajectories) does. See §10. |
| 009 | Router is advisory; hard heuristics own the extremes | router as the authority | a 66M classifier will be confidently wrong sometimes; hard evidence (empty tree, rich tree) must win regardless. Hysteresis bounds flapping. |
