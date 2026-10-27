# 20 — Visual presence: the dot

> Part of the [Axiom architecture spec](README.md).

An agent that drives your machine is invisible by nature — that's a trust problem, not a feature. Axiom is designed to be *ambiently visible*: you should always be able to answer "where is it acting and what state is it in" without opening any window. This is the §12 live status line, extended into space.

## The three surfaces

**1. Menu-bar state dot (P1 owns it).** A small status item that is always the first thing you check:

| Machine state (§03) | Menu dot |
|---|---|
| IDLE | gray, steady |
| Step cycle running | blue, slow pulse |
| GATE (awaiting your approval) | orange, steady — gently emphasized |
| YIELD (you have the machine) | dim gray, hollow |
| HALTED / FAILED | red, steady |
| DONE | green for 2s, then gray |

Click → the task card: current intent in plain language, last action, kill switch.

**2. The action dot (the on-screen dot).** A floating dot rendered at the **exact dispatch point** of every action — the element center for AX actions, the coordinate for clicks:

- Appears ~80ms *before* the action lands (a micro "here it comes"), fades out ~400ms after.
- Blue for normal dispatches; **orange ring** while the pixel lane is active; **red flash** for G2-class actions; green blip on successful VERIFY.
- This is the "dot like agent apps on macOS" — but with a contract, not decoration: it maps 1:1 to journal INTENT records. Every dot you saw is an action you can audit.

**3. The field halo.** When the agent mutates text (AXSetValue / paste / chunked typing), a soft ring traces the field for the mutation's duration. "What did it change?" becomes visible at a glance — text mutations are the actions users worry about most.

## Rendering contract (the part that keeps it safe)

- **Click-through, always.** The dot sits *exactly at the click point*; if it ever intercepted input, the agent would click itself. The overlay is `NSWindow.ignoresMouseEvents = true`, non-activating, borderless. (Invariant I13.)
- **Display-only, owned by the powerless process.** The overlay lives in P1 — which holds no action powers and no AX grant. The dot can never become a control surface; driving it accomplishes nothing.
- **Excluded from capture:** `NSWindow.sharingType = .none` — the dot never pollutes screenshots or screen recordings. This matters twice over: recordings stay clean, and the pixel lane (§04) never grounds on our own overlay.
- **Occlusion-check exemption:** the §05 window hit-test filters our own windows by PID, so the dot can never fail our own occlusion check.
- **One pooled overlay window, moved not recreated** — no window churn, no space-switch flashes, negligible GPU cost.

## Edge cases enumerated

- **Multi-display:** the dot renders on the `NSScreen` containing the global dispatch point; coordinates come from the same source as the dispatch itself, so dot and click can't disagree.
- **Spaces / fullscreen apps:** overlay level above fullscreen, joins all spaces; if the target is on another space, the dot waits at the dispatch moment — it never chases.
- **Lock screen:** nothing renders (the machine is HALTED by policy §09 anyway).
- **Secure fields:** dispatch dot shows; the field halo is suppressed — no visual invitation to look at password state.
- **Reduced-motion:** pulse/trail animations off, steady dots only. A "quiet mode" shrinks the dot to a 6px point for daily-driver users.
- **The dot is honest about ignorance:** in pixel-lane grounding with low confidence, the dot renders hollow/dashed — never solid over a guess.
