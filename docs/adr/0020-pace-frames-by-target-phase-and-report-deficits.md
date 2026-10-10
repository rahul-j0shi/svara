# 0020 — Pace frames by target phase and report deficits honestly
Status: Accepted

Context: The renderer limits frames with `elapsed ≥ frameInterval × 0.88` and then resets its origin
to the callback it rendered. With virtual callbacks at 144 Hz it renders every third callback
(48 fps instead of 60); at 165 Hz, 55 fps. The probe computes dropped frames as
`round(interval / target) − 1`, so a steady 20.8 ms interval reports zero drops (SV-22). The HTML
standard ties rendering opportunities to hardware and visibility; `requestAnimationFrame` is not a
universal 60 Hz source. F02 promises 30 fps idle and 60 fps while speaking.

Decision:
- Limit frames with an **absolute next deadline** (phase-preserving accumulator): render at most once
  per callback when the deadline has passed, then advance the deadline past any missed opportunities;
  never run catch-up render bursts.
- The audio/performance clock (ADR-0017) is independent of render cadence; a skipped frame never
  shifts the mouth in time.
- Metrics report separately: callback availability (rate of opportunities), rendered fps, target
  deficit (frames the target needed that were not rendered), frame work time, visibility state, and
  mode transitions, over a defined measurement window with stated endpoints. "Zero dropped" and
  "met target" are different claims, and a display slower than the target reports its unavoidable
  deficit.
- Hidden or minimized means zero submitted frames (unchanged).

Consequences:
- Deterministic tests cover 50/60/90/120/144/165/240 Hz schedules, irregular stalls, idle↔speaking,
  hide/restore and no bursts. Logical success does not establish the resource budget: correct pacing
  can *increase* work on high-refresh displays, so CPU/PSS is re-measured on the target host.
- Earlier X11 measurements remain valid as rendered-frame counts; their "zero counted drops" figure
  means only "no drops counted by the old estimator". The host refresh rate was not recorded, so
  every new frame measurement records it.

Evidence:
- Review round 5: actual `AvatarRenderer`/`FrameProbe` with virtual clocks and mocked graphics; a
  phase-preserving arithmetic prototype produced 600 opportunities in 10 s at all five rates. No GPU
  or display measurement was part of that evidence.
- Implementation: lab unit V-E.2; product owners T2.5/T2.11 and the 14 §5 animation protocol
  ([16](../revamp/16-lab-track-and-review-findings.md)).
