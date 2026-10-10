# 0022 — Hold a liquid-smooth experience contract across every surface
Status: Accepted

Context: The plan specified *what* the avatar and its transient UI do (12 F02–F06, 02 §3–§5) and a few
isolated timings (state visual ≤100 ms, bubble ≤300 ms ahead of speech, 30/60 fps), but not how the
experience must *feel*: how fast any input is acknowledged, how states blend, what happens when a
transition is interrupted, how much jank is tolerated, or how audio starts and stops. The user asked
for an experience that is "liquid smooth". Without a measurable contract, smoothness is decided ad
hoc per component and silently traded away. The design is in
[17 — Experience & motion](../revamp/17-experience-and-motion.md).

Decision:
- Adopt global experience budgets in [12 §0.5](../revamp/12-feature-specification.md#05-experience-quality-global)
  for input acknowledgement, frame-time consistency, main-thread long tasks, drag tracking, window
  first paint and audio gain changes. They apply to every feature, like the privacy and permission
  rules, and are verified by cross-feature checks X26–X30 in 14.
- **Continuity rule:** every visible or audible property changes continuously. Body, face, glow,
  bubbles, windows and gain are animated by time-based easing or critically damped springs that
  retarget from their *current value and velocity* when interrupted. Nothing restarts from zero,
  pops, flashes or clicks. Mouth weights remain owned by `PerformancePlayer` (ADR-0018).
- **Time, not frames:** all animation advances by clock time (`dt`), so 30↔60 fps changes and
  dropped opportunities never change animation speed (ADR-0020).
- **The render loop never waits:** VRM parsing/preparation, audio decoding, large JSON, Markdown and
  Monaco work run off the avatar's animation path (workers, chunking, idle callbacks or the separate
  windows). The core and shell never block the renderer; IPC is asynchronous.
- **Elevate frame rate for motion:** the avatar renders at the 60 fps target while speaking, dragged,
  hovered, or while any transition is in flight, then returns to 30 fps idle and 0 fps hidden.
- **One motion vocabulary:** shared duration/easing tokens (17 §2) in one module used by the
  renderer and all windows; reduced motion swaps movement for short opacity changes.
- **Bounded quality ladder:** if frame-time budgets are missed for a sustained window, step down a
  fixed ladder (pixel ratio, then antialiasing, then spring-bone rate) and step back up after
  recovery. Every step is logged and shown in diagnostics; the ladder is a mitigation and never a
  budget waiver (G-BUDGET still records the miss).

Consequences:
- UX smoothness becomes testable: frame-time distribution, long-task count, input-to-frame latency
  and gain-ramp traces are recorded like latency (14 §5).
- Components cannot add a blocking load or a hard cut without breaking a named check.
- Higher frame rate during motion costs CPU/GPU; it must still fit the idle and speaking budgets,
  which remain unchanged.
- Out of scope remain: earcons/sounds for state changes (F04), custom-styled native menus (F03),
  and theming beyond following the OS.

Evidence:
- None yet; this is a new contract. The diagnostic spike's 10-second samples (29.9 fps idle) say
  nothing about frame-time variance or input latency. Lab unit V-E.2 adds frame-time/long-task
  recording; product owners are T2.3, T2.5, T2.6, T2.8, T2.11, T3.8, T3.10, T3.15 and T6.6.
