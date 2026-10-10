# 17 — Experience & motion: the liquid-smooth contract

Adopted **2026-10-10** at the user's request ("it should be liquid smooth"). Decision:
[ADR-0022](../adr/0022-hold-a-liquid-smooth-experience-contract.md). Numeric budgets are normative in
[12 §0.5](12-feature-specification.md#05-experience-quality-global); checks X26–X30 and their owners
are in [14 §3.4](14-acceptance-and-integration.md#34-experience-quality); this document is the design.
It adds no feature: it defines how the existing features (F02–F07, F14–F17, F24, F27) must feel.

## 1. Principles

1. **Immediate.** Every deliberate input gets a visible answer within 100 ms, even if the real work
   takes seconds. Waiting is always visibly *someone working*, never a frozen character.
2. **Continuous.** Nothing pops, flashes, jumps or clicks. Every property moves from where it *is*,
   including when a transition is interrupted halfway.
3. **Calm.** An always-on desktop character is peripheral. Idle motion is slow and small; attention
   motion (listening, approval) is clear but brief. No bouncing for its own sake.
4. **Honest.** Motion never pretends: the glow means the mic is open (F04), the mouth follows the
   heard audio (ADR-0017), progress is real. No fake typing or fake thinking delays.
5. **Unblocked.** The avatar's animation path never waits on the network, the core, disk, parsing or
   GC-heavy work. Heavy work happens elsewhere or in small slices.

## 2. Motion vocabulary (one module, shared by renderer and windows)

| Token | Value | Used for |
|---|---|---|
| `instant` | 0–80 ms, ease-out | Press/hover feedback, button states, glow on/off onset |
| `quick` | 150 ms, ease-out in / ease-in out | Chips, tooltips, ghost line, bubble fade, window fade |
| `standard` | 220 ms, standard curve `cubic-bezier(0.2, 0, 0, 1)` | Bubble appear/resize/flip, type box open, approval reveal |
| `pose` | critically damped spring, ~250 ms settle, no overshoot | Body/head state poses (listening tilt, thinking gaze, acting loop entry) |
| `expression` | spring, ~300 ms in, 400 ms ease back to neutral (F16) | Face expressions; reduced while a strong viseme is active |
| `model` | ~1 s cross-fade (F14) | Avatar model replacement, after the candidate is prepared (ADR-0019) |
| `audio-ramp` | 5–20 ms linear gain ramp | Every audio start/stop/pause/duck/volume change |

Rules: animate only `opacity`/`transform` in DOM UI (no layout-thrashing properties); springs and
easings take clock time `dt`; an interrupted animation retargets from current value and velocity;
exits are slightly faster than entries. **Reduced motion** (`prefers-reduced-motion`): sway, bounce
and spring overshoot off; movement becomes ≤150 ms opacity changes; blink and lip sync stay (F02).

## 3. Moment-by-moment choreography

| Moment | What the user perceives | Mechanism |
|---|---|---|
| Launch | Nothing, then the character fades in already alive; never an empty, black or half-loaded window | Window maps only after the first rendered frame; `quick` fade; `booting` greyed pose until core ready (F04) |
| Idle | Slow breathing, blinks, saccades; feels present, costs little | 30 fps, `dt`-driven idle layer (F17) |
| Hover | Subtle attentiveness (gaze toward cursor while over the window) | Elevate to 60 fps while hovered |
| Drag | The character is glued to the pointer; no jump on grab, no lag trail | Grab offset kept; position updates every frame; render 60 fps; input region = window during drag (02 §3) |
| Double-click → listening | Within 100 ms: attentive `pose`, then the glow; ghost line fills with partial transcript | The pose starts locally on the gesture; the glow appears only when the core confirms the mic is open (honesty rule, F04); the local round-trip still fits 100 ms |
| Speech end / Enter → thinking | No dead air on screen: `thinking` gaze and "…" chip within 100 ms | F04 state message; ghost line stays visible |
| First audio → speaking | Mouth, voice and bubble words arrive together | Bubble reveals by `word_spans` (F05), face samples output time (ADR-0017), `audio-ramp` in |
| Between sentences | Continuous speech; mouth relaxes naturally on short gaps | Player smoothing across segments (ADR-0017/0018); gaps ≤150 ms (F12) |
| Tool running | A one-line chip slides in; the character shows a small working loop | `quick` chip, `pose` transition to acting |
| Approval | The bubble expands with clear buttons and keyboard focus on *Allow once* | `standard` reveal; window focusable only now (F02, ADR-0019) |
| Interruption (barge-in, Stop, new input) | Voice fades out within the stop bound; mouth closes softly; listening/idle pose blends in | `audio-ramp` out inside F13 ≤350 ms / F27 ≤300 ms; pose retargets mid-transition |
| Reply ends | Bubble lingers, then fades; character eases back to idle | `expression` back to neutral 400 ms; auto-hide fade `quick` (F05) |
| Error | Gentle: brief `sad`, an error bubble, return to idle | No modal dialog, no shake |
| Hide / show | Fades away / returns already animating, never showing a stale frame | Render one fresh frame before mapping; `quick` fade; 0 fps while hidden |
| Scale change | Character resizes smoothly around its anchor, no clipping | Window and framing animate together over `standard` |
| Model change | Old look stays until the new one is ready, then cross-fades | ADR-0019 prepare/commit; `model` token |
| Core restart / reconnect | Character stays alive (greyed, "Reconnecting…"), then resumes | Renderer independent of core; snapshot on reconnect (F40) |
| Configure / Conversations | Opens to a themed first frame, never a white flash; edits apply live with a subtle confirmation | `ready-to-show` + OS-theme background; Monaco lazy-loaded; lists paged (F07) |

## 4. Perceived-latency timeline for a spoken turn

`0 ms` user stops speaking → `≤100 ms` thinking visual (B1) → partial transcript already visible →
LLM first token → first sentence synthesised → **first audible output** (04 §5: p50 ≤2.0 s end of
speech → avatar talking) with mouth and bubble in step. Nothing on screen may freeze during this
interval; the character keeps idle-layer motion under the thinking pose. The design never invents
fake audio or filler speech to mask latency; real latency is reduced and measured (T3.13).

## 5. Rendering and process rules

- **Off the animation path:** VRM fetch/parse/optimise, `decodeAudioData`, protocol JSON for large
  payloads, Markdown sanitising and settings editors. Use workers, chunked work or the separate
  windows. A model swap is the only moment an unavoidable parse may run on the avatar thread; it
  happens behind the still-visible previous model and each slice stays ≤100 ms.
- **Frame rate:** 60 fps target while speaking, dragged, hovered or transitioning; 30 fps idle;
  0 fps hidden; phase-preserving limiter (ADR-0020). Everything advances by `dt`.
- **Allocation discipline:** no per-frame allocation in the player/renderer hot path (reuse arrays,
  vectors); player update ≤1 ms (F15).
- **Never block:** shell IPC and native helpers are asynchronous (ADR-0019); the core runs in its own
  process; no synchronous IPC from renderers.
- **Quality ladder (sustained miss only):** after 2 s over the frame budget, step 1 pixel ratio
  2→1.5→1, step 2 antialiasing off, step 3 spring-bone update 30 Hz; step back up after 10 s within
  budget. Logged and visible in Advanced diagnostics. It never changes the idle/speaking budgets and
  never hides a G-BUDGET failure.

## 6. Audio smoothness

Every gain change — start, stop, pause voice, barge-in duck to 25% and restore (13 §6.1), volume
slider, mute — ramps over 5–20 ms on the final gain node, so there are no clicks or pops. The ramp is
inside, not added to, the F13/F27 stop bounds. Volume 0 keeps timing (F12). Segment boundaries are
sample-accurate (ADR-0017), so back-to-back sentences need no crossfade.

## 7. Measuring smoothness

Recorded in the renderer with standard browser APIs, on the primary host, with refresh rate noted
(14 §5): `requestAnimationFrame` timestamps for frame intervals and deficit (ADR-0020),
`PerformanceObserver` long tasks for main-thread blocking, Event Timing (`event` entries) for
input-to-next-paint, gain-node automation traces for ramps, and window `ready-to-show`/first-frame
timestamps. A short screen recording of the scripted demo (open, drag, talk, interrupt, approve,
hide/show) is reviewed for pops and flashes; a recording is supporting evidence, not a substitute for
the numbers. Results publish with the latency table (T7.5).

## 8. Owners

| Area | Task(s) | Check |
|---|---|---|
| Drag, hide/show, first-frame mapping, focus reveal | T2.3 | X26, X29 |
| Idle layer, `dt` animation, fps elevation, quality ladder | T2.5 | X27 |
| State poses and interruptible transitions | T2.6 | X28 |
| Bubble/type box/chip motion and layout stability | T2.8 | X28 |
| Window first paint (Conversations, Configure) | T2.9, T2.10 | X29 |
| Text-desktop integration of all of the above | T2.11 | X26–X29 |
| Audio ramps, speaking smoothness, face/voice/bubble together | T3.8, T3.10 | X30 |
| Voice-load smoothness (STT/TTS/LLM running) | T3.15 | X26–X30 |
| Reduced motion and multi-monitor/scale transitions | T6.6 | X28 |
| Published smoothness numbers | T7.5 | — |

Lab track: V-E.2 records frame-time distribution and long tasks alongside its pacing work (16 §3).

## 9. Explicitly not done

Earcons or state sounds (F04 out of scope); custom-styled native menus (F03); idle chatter or random
speech (F17); fake typing/thinking delays; skeleton screens that pretend content exists; animated
backgrounds or panels around the character (F02 production-surface invariant).
