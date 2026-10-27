# 17 — Risk register (architecture-aware)

> Part of the [CURSOR architecture spec](README.md).

| Risk | Sev | Mitigation | Residual |
|---|---|---|---|
| AX gaps in Electron/canvas apps | High | manual-accessibility + pixel lane + per-app adapters; coverage % tracked as a first-class metric | some apps remain pixel-only and slower — bounded, not eliminated |
| Same-user malware hijacks the agent | High | §10 layered defenses; Touch ID top gate; socket peer checks | **irreducible** — stated in §10, not hidden |
| TCC churn disrupts dev + users | Med | stable signing identity; doctor command; document the Terminal-grant trap | low |
| 2B pilot too weak for hard multi-step tasks | Med | confidence-gated escalation; specialist offload; budget forcing | escalation rate is the tracked symptom; if chronically > 20% → pilot-upgrade decision point |
| Secure Input / occlusion class of dispatch hazards | Med | contracts + dispatch-time checks + chunked typing with read-back | low — the contract table (§05) owns it |
| Alert fatigue makes gates rubber stamps | Med | effect-statement cards, intent binding, ask budget, Touch ID for top class, ask-rate metric | medium — human factors never fully close |
| MLX multi-model instability | Low | process isolation + backoff restarts + journal; llama.cpp fallback lane for v1 | low |
| Apple API drift (AX, TCC, capture) | Low | adapters isolate `perceive/` + `execute/`; macOS-version pins in manifest | low |
