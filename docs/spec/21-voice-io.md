# 21 — Voice I/O: STT / TTS / wake word

> Part of the [Axiom architecture spec](README.md).

Axiom is voice-optional but voice-complete: wake word, speech-in, speech-out — **all local, all open source, all from the specialist-floor shelf** (the same research that put a 631MB STT model above Whisper large-v3).

## The voice roster

| Role | Model | Size | Resident? | Job |
|---|---|---|---|---|
| **VAD** | Silero VAD | ~2MB | **Yes (always, in P1)** | speech-activity detection; wakes the voice stack |
| **Wake word** | openWakeWord | ~2MB | **Yes (always, in P1)** | "Hey Axiom" |
| **STT** | Parakeet 0.6B (TDT) | ~631MB | Lazy (resident in voice-first mode) | task dictation, corrections; beats Whisper large-v3 |
| STT (tiny lane) | Moonshine 27M | ~50MB | optional | fast short-clip capture |
| **TTS** | Kokoro-82M | ~160MB | Lazy (resident in voice-first mode) | gate read-backs, status announcements |

Voice-first mode (say the word, hands on keyboard elsewhere) makes Parakeet + Kokoro resident: **+~800MB**, still under the 2.5GB-class budget with the standard roster. VAD + wake word add ~5MB — negligible, always on.

## Integration rules (voice serves the loop; it never bypasses it)

- **Voice is a channel, not an authority.** In: task creation, correction, "stop". Out: status lines, gate effect-statements, completion announcements. Nothing else.
- **G2 is never voice-approved.** This is a hard rule (Invariant I14) with a concrete threat: *speaker injection* — any audio played near the mic can forge a spoken "approve". G2 requires Touch ID or a physical click, every time, even mid-voice-flow. Voice may confirm G0/G1.
- **TTS never speaks secrets.** The TTS path runs the §10 redaction filter: passwords, card numbers, tokens are announced as placeholders ("the password field"), never as values.
- **Audio is transient.** Mic audio is never persisted; transcripts pass redaction before the journal; recordings cannot leak because they don't exist.
- **"Stop."** — a dedicated, high-precision-only voice kill phrase, equivalent to the kill switch (§03): halts at the next state boundary, releases assertions.
- **Arbitration is unchanged.** YIELD watches mouse/keyboard (§09); speaking *commands* the agent rather than pausing it — that's the point of a voice channel.
- **Latency targets:** wake → listening < 300ms · a spoken sentence → text < 1s · TTS first audio < 400ms. All comfortably real-time-factors on M-series silicon.

## Where it lives

- **P1** (menu-bar app): VAD + wake word — tiny, no model server needed; P1 additionally holds the **Microphone** grant (§02 permission topology). Overlays (§20) and audio I/O co-owning P1 keeps the "display+sound only" process distinct from the acting process.
- **P4 lazy servers:** Parakeet and Kokoro load on first use (or stay resident in voice-first mode), subject to the memory-pressure ladder (§07 — voice models drop *before* the pixel stack: perception of the task outranks narration of it).

## Edge cases enumerated

- **The mic indicator:** macOS shows the orange mic dot system-wide — Axiom must never suppress or obscure it; the menu-bar dot turns a small mic glyph while listening. Honest mic usage is part of the trust surface.
- **It talks over you:** TTS output is ducked/canceled when the VAD detects user speech mid-announcement — Axiom never barges in.
- **Noisy environment / mis-wakes:** false wake → the mic glyph + a subtle chime (no card, no task). "Hey Axiom, cancel" works *during* the wake window.
- **Multilingual users:** Parakeet handles it; Kokoro voice quality varies by language — degrade to text-only cards gracefully when a language's TTS is weak.
- **Deaf/HoH users are first-class:** every spoken announcement has a visual twin (§12 live status + §20 surfaces). Voice adds a channel; it never *is* the channel.
