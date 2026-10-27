# 07 — Model lifecycle & resource management

> Part of the [CURSOR architecture spec](README.md).

## Load matrix

| Model | Loaded | Unloaded |
|---|---|---|
| Resident set (pilot+draft, executor, router, embedder, ~2GB) | at daemon start (cold-start budget: first action ≤ 3s from page cache) | idle > 30min → release (configurable; default: stay resident) |
| Escalation 9B (~5.5GB) | first low-confidence step; kept warm 10 min | memory-pressure ladder or 10-min idle |
| Pixel stack (~2.1GB incl. capture) | first pixel-lane step; kept warm 5 min | ladder or 5-min idle |
| GLiNER PII (0.3B) | only at trajectory-redaction time (log path, never the agent path) | immediately after |

## Memory-pressure ladder (responds to `DISPATCH_SOURCE_TYPE_MEMORYPRESSURE`)

1. **Warning:** shorten keep-warm timers; no new lazy spawns.
2. **Critical:** drop GLiNER → drop escalation → drop pixel stack, in order. *Never* drop the resident set — that's the agent's competence.
3. If escalation/pixel is needed while critical → unavailable → degrade: more budget-forcing on the pilot, lower ambition, or GATE ("need memory to continue well"). The agent never competes with the user's apps for RAM; its own lazy models are the only things it evicts.

## Compute contention & watchdog

- MLX and the user's apps share the GPU cooperatively. Under heavy external GPU load, our step latency balloons. **Watchdog:** EWMA of step latency; 3 consecutive steps > 1.5× p95 budget → auto-pause (YIELD) with status "system under load — resume when ready." No thrashing.
- While a task is active: scoped sleep assertion (prevent idle *system* sleep), hard cap 10 min, released at every terminal state, disclosed in the status line. Display sleep is *not* prevented.

## Cancellation, hot-swap, updates

- **Model-call cancellation:** generation streams; cancel between chunks. If a full action JSON was already emitted and validated when cancel arrives, it is discarded at the state boundary — dispatch never begins after a cancel.
- **LoRA hot-swap:** executor adapters swap without reloading base weights. The pilot/draft pair is version-locked (speculation requires vocab/family match — changing the pilot invalidates the draft): enforced by a roster manifest, not convention.
- **Model updates are eval-gated:** no weight/adapter change is promoted without replaying the 100-task suite + 20 traps; a regression on trap tasks blocks promotion outright (pin by hash in the manifest).
