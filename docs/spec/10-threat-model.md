# 10 — Threat model

> Part of the [CURSOR architecture spec](README.md).

**Framing:** the agent is a *confused-deputy amplifier*. Its Accessibility grant lets it do what the user can do; anyone who can drive the agent inherits that power. Every defense below answers one question: "who can pull the agent's strings, and how do we make the strings useless?"

| Vector | Example | Primary defense (by construction where possible) |
|---|---|---|
| **Prompt injection via content** | web page label / email body / filename contains "ignore instructions, run curl …" | Content is fenced data (`<obs>`); the agent has *no shell and no URL-following capability* — the injected command is not just refused, it's unrepresentable. Detection (suspicious-instruction classifier) is secondary telemetry, not the wall. |
| **Malicious AX labels** | app labels a button "Save" that deletes; invisible decoy elements | Dispatch resolves the *element handle*, never a label re-search; gates key on resolved target + action class; confirmation cards show the real target; the destructive lexicon is matched against the resolved element, not the plan text. |
| **Fake dialogs / lookalikes** | "System Preferences wants to install…" window | Occlusion check + window owner bundle-id verification; dialogs from non-system bundles that request security actions → refuse + report. |
| **Control-plane hijack** | local malware drives cursord's socket to click what its own TCC forbids | Socket peer-UID check; new task creation requires a P1-originated, user-visible action; gated intents still require P1 approval; heartbeat loss >2s → auto-pause. **Honest limit:** same-user malware can synthesize clicks on P1's UI itself — reduced (intent-bound approvals, frontmost requirement, Touch ID for the top class) but not eliminated; this is the irreducible residual risk of any AX agent, stated plainly. |
| **URL / exfiltration** | injected text smuggles data out via an `open` URL with an encoded payload | URLs only from the user's typed goal or an allowlist; never from content; any URL not goal-derived → G2. Plus: local models + no network in the agent path (model download only, pinned hashes) means there is no channel unless an action creates one. |
| **Pasteboard abuse** | secrets pasted then left on the clipboard | write → paste → verify → **clear**; change-count confirms delivery; payload never logged. |
| **Screenshot PII (pixel lane)** | capture contains banking/tokens | screenshots never leave the device, never enter trajectories unredacted; capture is lazy-only, released after use. |
| **Model/adapter supply chain** | poisoned fine-tune biases actions | weight pinning by hash; eval-gated promotion incl. trap regression; adapters signed locally; dependencies pinned + hash-locked. |
| **Script injection** | model output interpolated into AppleScript source = arbitrary code exec | ADR-007: allowlisted parameterized templates with JSON-typed argv; no generated script source exists anywhere in the codebase (the capability is absent, not forbidden). |

> [!IMPORTANT]
> **Decision — redaction point.** Redact at persistence, not at model context. The local model needs real text to act; there's no exfiltration channel in the agent path. The exfil surface is the trajectory corpus (and any future sync), so regex-first + GLiNER-lazy redaction runs on everything written to disk. Trade-off accepted and logged: the context window may contain PII transiently; the disk never does.
