# 00 — Overview: thesis, model roster, latency budget

> Part of the [Axiom architecture spec](README.md).

## The thesis in one sentence

On macOS, the fastest and smallest computer-use agent doesn't look at pixels most of the time — it reads the **Accessibility tree**, plans with a ~2B model, executes with a ~0.3–0.6B action model, and falls back to a lazily-loaded pixel stack only for apps with broken accessibility.

## Why AX-first beats screenshot-first on macOS

Every computer-use agent (UI-TARS, Holo1.5, OpenAI CUA, Claude CUA) is screenshot-first because it must work on every OS with no cooperation from apps. That constraint costs them everything: screenshots are 1–3k vision tokens of **prefill per step** (the expensive phase), OCR-grade text recognition is the hardest sub-problem (documented failure mode: "clicks into the void"), and grounding accuracy caps task success.

macOS is the one platform where the agent can skip all of it: the Accessibility API (`AXUIElement`) exposes the live UI as **structured text** — every button's role, label, position, value, and enabled state.

| | Screenshot-first | AX-first (macOS) |
|---|---|---|
| Perception per step | 1–3k vision tokens → 1–4s prefill | 1–4k *text* tokens, delta-compressed → 100–400ms prefill (~0 with prefix caching) |
| Grounding | VLM regression to coordinates (2B+ model, error-prone) | Text ranking over labeled elements with known frames — click targets are *exact* |
| Smallest viable model | 2B (GUI-actor floor) | 0.3–0.6B (it's text processing) |
| Reliability | Depends on rendering, resolution, theme | Deterministic UI metadata; works in dark mode, any resolution |
| Coverage | Everything, poorly sometimes | ~90–95% of apps well; pixel fallback for games/canvas/broken AX |

## The model roster — ~2.3GB resident total

| Role | Model | Params | On disk (Q4) | Resident? | Job |
|---|---|---|---|---|---|
| **Pilot (planner)** | Qwen3.5-2B (thinking toggle ON) | 2B | ~1.3GB | Yes | Decompose goal → next action + reasoning; the only model that "thinks" |
| **Draft (for pilot)** | Qwen3.5-0.6B (vocab-matched) | 0.6B | ~0.4GB | Yes | Speculative decoding ×2 on the pilot |
| **Executor (action emitter)** | fine-tuned 270M-class (FunctionGemma recipe) | 0.27B | ~0.16GB | Yes | Pilot's decision → strict tool-call JSON; re-formats failed calls |
| **Router** | fine-tuned DistilBERT (66M) | 0.07B | ~0.03GB | Yes | AX-sufficiency check: {use AX, need pixels, escalate} — ~10ms |
| **Embedder** | all-MiniLM-L6-v2 | 0.02B | ~0.03GB | Yes | Element search ("the send button") over the AX tree; trajectory memory |
| **VAD** | Silero VAD | ~0.001B | ~2MB | Yes | Speech-activity detection; wakes the voice stack (§21) |
| **Wake word** | openWakeWord | ~0.002B | ~2MB | Yes | "Hey Axiom" |
| STT (voice lane) | Parakeet 0.6B | 0.6B | ~0.63GB | voice-first: Yes | Dictation, corrections; beats Whisper large-v3 |
| TTS (voice lane) | Kokoro-82M | 0.08B | ~0.16GB | voice-first: Yes | Gate read-backs, status announcements |
| Grounding (pixel fallback) | Holo1.5-3B | 3B | ~1.9GB | **LAZY** | Coordinate grounding on screenshots when AX is unusable |
| Guardrail (optional) | GLiNER 0.3B (PII) — regex first | 0.3B | ~0.16GB | **LAZY** | Redact before logging trajectories; flag sensitive actions |

> [!IMPORTANT]
> **Decision — perception.** AX-first by default, pixel stack as a lazy fallback. The pixel stack loads *only* when the AX tree for the frontmost app is empty/incomplete, and unloads after.
>
> **Decision — roster.** Resident ~1.9–2.3GB (pilot + draft + executor + router + embedder; +5MB for the always-on voice trigger stack, §21). Lazy: +2GB only when pixels are needed, +~0.8GB when voice-first mode keeps STT/TTS resident. Fits a 16GB Mac. Escalation: if the pilot's confidence is low (logit margin on the action token — cheap to compute), hand the step to a bigger model (Qwen3.5-9B, loaded on demand).

## The loop at a glance

```text
┌─ agent step (target: <500ms, typical ~250ms) ───────────────────────┐
 1. OBSERVE      read AX tree of frontmost app (native, ~10–30ms)
                 delta-diff vs previous step (only changes serialized)
 2. ROUTE        router model: AX enough? → yes: continue (~10ms)
                 no → wake pixel stack (300ms first time, then cached)
 3. RETRIEVE     embedder: fuzzy-match user goal phrases → element refs (~5ms)
 4. PREFILL      system prompt + tool schema  [PREFIX CACHED — ~0ms]
                 + app context card           [CACHED per app — ~0ms]
                 + delta AX snapshot          (100–300ms, the only real prefill)
 5. DECIDE       pilot 2B + draft speculation → action + confidence (100–250ms)
 6. VALIDATE     executor: strict JSON tool call; validator checks args vs
                 live AX element existence (5ms; retry with executor on mismatch)
 7. ACT          execute: AX action / AppleScript template / CGEvent (10–50ms)
 8. VERIFY       re-read AX tree: did the UI change as predicted? (~30ms)
                 no → self-correct: re-plan with the diff (max 3 loops)
└────────────────────────────────────────────────────────────────────┘
```

### The four speed mechanisms

| Mechanism | Saves | Spec section |
|---|---|---|
| AX tree instead of screenshots | 1–4s of vision prefill → 0.1–0.3s text prefill | §04 |
| Radix prefix caching (system prompt + app card + schema are stable across steps) | ~90% of prefill on steps 2–N of a task | §08 |
| Delta AX snapshots (only changed elements serialized) | keeps per-step prefill ~100–300 tokens | §04, §08 |
| Speculative decoding on the pilot | ×2 on the 100–250ms decision leg | §07 |

## Execution layer priority

| Priority | Mechanism | Use for | Why |
|---|---|---|---|
| 1 | **Accessibility actions** (`AXPress`, `AXConfirm` on elements) | Buttons, menus, checkboxes | Most reliable: no coordinates at all; works when windows are occluded |
| 2 | **AppleScript / System Events + JXA** | App control, menu navigation, window management, file ops | Deterministic, scriptable, no coordinates; the macOS-native "API" |
| 3 | **CGEvent synthetic mouse/keyboard** | Click at exact AX-reported frames; typing; scrolls; drag | Universal fallback; coordinates come from AX metadata (not OCR), so precision is free |
| 4 | URL schemes / `open` / Shortcuts | App launch, deep links, automations | Fastest path when it exists |

> [!IMPORTANT]
> **Decision — execution.** Prefer semantic actions (AX/AppleScript) over coordinate clicks; use CGEvents only with AX-supplied frames. Pixel-first agents must *predict* coordinates (error-prone); Axiom's coordinates come from OS metadata (exact). See §05 for the dispatch-time checks that make even coordinate actions safe.

## Speed gates

> [!TIP]
> Step latency p50 < 500ms / p95 < 1.2s (AX path) · TTFT < 120ms with warm cache · pixel-lane grounding < 600ms · resident RAM < 2.5GB · cold start to first action < 3s.

## Self-improvement (the moat)

Trajectory recording → nightly MLX LoRA on the executor → replay-gated promotion. Every day of use makes *your* executor better — a compounding loop no cloud agent can offer on your machine's behalf. Full spec: [§19](19-self-improvement-loop.md).
