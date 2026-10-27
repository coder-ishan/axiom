# Attribution

Every third-party component Axiom builds on. Rule (docs/spec/22): **pin at build time and verify each license against the model card / repo at pin time** — this table records intent, the manifest records exact hashes.

## Models — the agent roster

| Component | Used for | License (verify at pin) | Open-source? |
|---|---|---|---|
| Qwen3.5-2B / 0.6B / 9B | pilot, draft, escalation | Apache-2.0 | ✅ |
| 270M executor (FunctionGemma recipe) | action emission | Gemma family — **custom license**; marked **swappable** with Apache-2.0 bases (e.g., Qwen3-0.6B) | ⚠️ base license |
| DistilBERT (66M, fine-tuned) | router | Apache-2.0 | ✅ |
| all-MiniLM-L6-v2 | embedder | Apache-2.0 | ✅ |
| Holo1.5-3B | pixel-lane grounding (lazy) | permissive — verify model card; swappable | ✅* |
| GLiNER 0.3B | PII detection (lazy, log path) | MIT (verify) | ✅ |

## Models — the voice stack

| Component | Used for | License (verify at pin) | Open-source? |
|---|---|---|---|
| Silero VAD | voice activity detection (~2MB) | MIT | ✅ |
| openWakeWord | wake word ("Hey Axiom") | Apache-2.0 | ✅ |
| Parakeet 0.6B (NVIDIA TDT) | STT — beats Whisper large-v3 | CC-BY-4.0 | ✅ |
| Moonshine 27M | tiny STT lane | MIT (verify) | ✅ |
| Kokoro-82M | TTS | Apache-2.0 | ✅ |

## Runtimes & libraries

| Component | Used for | License |
|---|---|---|
| MLX / mlx-lm | model runtime | MIT |
| llama.cpp | fallback runtime lane (GBNF grammars) | MIT |
| pyobjc | AX tree reader, AX actions, CGEvents | MIT |
| appscript | AppleScript bridge (allowlisted templates only) | MIT |
| FastAPI | agent loop + policy engine HTTP | MIT |
| SQLite | trajectory index (journal is JSONL — our own) | Public domain |

## Notes

- **Fully-local claim:** every component above is downloadable, inspectable, and runnable offline. No API keys, no phone-home, no accounts.
- The ⚠️/✅* marks are the honest record: where a base model ships under a custom (non-OSI) license, Axiom names its Apache-2.0 replacement in the roster manifest so the project remains open-source-clean under any configuration.
