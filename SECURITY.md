# Security Policy

Axiom holds an Accessibility grant on macOS — it is, by design, one of the most powerful classes of user-space software. Security is treated accordingly.

## Scope of what we defend

The full threat model is [docs/spec/10-threat-model.md](docs/spec/10-threat-model.md). Headlines:

- **Prompt injection via content** (web pages, emails, filenames, AX labels) — defended by construction: content is fenced data; shell and URL-following capabilities don't exist.
- **Control-plane hijack** — peer-UID socket checks, intent-bound approvals, heartbeat auto-pause.
- **Speaker injection** (voice channel) — G2 approvals are never voice-approved; TTS never speaks secrets.

## Honest residual risk (stated in the spec, not hidden)

A same-user local malware can synthesize clicks on the approval UI itself. We reduce this (intent-bound approvals, frontmost requirement, Touch ID for the top gate class) but **cannot fully eliminate it** — it is the irreducible residual of any Accessibility-powered agent. v1 is supervised-only for exactly this reason.

## Reporting a vulnerability

- **Preferred:** GitHub *Private vulnerability reporting* on this repository (Security tab → Report a vulnerability).
- **Do not** open a public issue for anything exploitable.
- Include: macOS version, reproduction steps, and — if safe — a journal file (redacted; it's JSONL and grep-able).
- We aim to acknowledge within **72 hours** and ship a fix or a documented mitigation within **14 days** for critical issues.

## Safe harbor

We consider good-faith research following the spec's own trap methodology (§13) to be authorized activity, provided you never exfiltrate user data, never persist audio/PII, and report rather than exploit.
