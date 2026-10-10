# 0017 — Make PCM frames the authority for every performance segment
Status: Accepted

Context: The engineering review (SV-14, SV-15, SV-16, SV-17, SV-21) showed that the performance
contract had several competing notions of extent and time. `duration_ms` was an integer while the
protocol promised one-sample agreement (339 frames at 24 kHz is 14.125 ms); the generator emitted an
80 ms release key beyond a 100 ms track and the player accepted it; the development player accepted a
200 ms track paired with 2,000 ms of audio; the envelope was peak-normalized per sentence and computed
before quantization, so a 1/32-amplitude tone and a file whose PCM rounds to zero both produced a
full-scale envelope; and the face sampled `AudioContext.currentTime`, which is graph time, not the
time audio leaves the device. Each layer was locally reasonable; together they disagreed about what a
segment is. See [03 §3, §5](../revamp/03-avatar-and-lipsync.md#3-the-performance-data-model) and
[the review ledger](../reviews/2026-10-09-engineering-review-rounds-1-5.md#2-cumulative-finding-ledger).

Decision:
- **Extent.** A segment's integer `frameCount` and `sampleRate` are authoritative. Milliseconds are
  derived (`frameCount / sampleRate × 1000`) for scheduling and display; `durationMs` on the wire is
  an informational, derived value and is never used to validate PCM. PCM must be mono s16le,
  byte-aligned, at a supported adapter rate, and contain exactly `frameCount` frames.
- **Keys are segment-local.** Every viseme, cue and word-span time is finite, monotonic where the
  protocol says so, and within `[0, extent]`. The generator receives the extent and clamps attack,
  hold and release to it, preserving the curve's sampled value at the boundary instead of dropping a
  distant zero key. No PCM is padded only to make animation keys fit.
- **Continuity across sentences** is the player's job, not the wire's: its smoothing state carries
  over back-to-back segments, and it neutralizes the mouth only on `performance.end`, `stop`, discard
  or a queue gap. A versioned cross-segment key representation is not part of v1.
- **Validation happens before playback, transactionally.** The adapter boundary (core) and the
  playback boundary (renderer) both reject a segment whose decoded frame count, rate, envelope
  coverage or key ranges disagree; a rejected segment never starts a source and never partially
  replaces accepted state. It is reported as `discarded` with a typed error.
- **Envelope semantics.** The envelope is *articulation intensity* computed from the PCM that is
  actually played (after clipping/quantization), using a per-voice calibrated dB range with an
  explicit silence floor. It is not peak-normalized per sentence. All-zero PCM yields an all-zero
  envelope. User output volume does not change it (volume 0 keeps timing, 12 F12).
- **Clocks.** Audio is scheduled on `AudioContext.currentTime` (graph clock). The face and
  `playback.progress` sample an *output-position* estimate derived from
  `getOutputTimestamp()` when the Electron build exposes a valid value, and fall back to graph time
  with the fallback recorded in diagnostics. Compensation for measured device/display offset is kept
  separate from artistic anticipation (`lookaheadMs`).

Consequences:
- Protocol 1 gains `frameCount` on `performance.segment`; T1.6 generates and drift-checks it. One
  sample precision holds for any provider rate, not only Kokoro's 25 ms grid.
- Generator, schema/adapter validation and player enforce the same rules, so a strict receiver can
  never reject what the default generator emits.
- Whispers and shouts can look different only to the extent the calibrated envelope shows; the
  calibration is measured per voice in T3.7, not invented as a universal constant.
- Output-clock sampling may be unavailable or noisy; the residual offset is a measured lab result
  (14 §5), never assumed to be zero.

Evidence:
- Review rounds 3–4 fixtures (synthetic PCM, generator→player, fake Audio APIs); reproduction code is
  preserved in the review. These prove contract inconsistencies, not perceptual failures of the
  healthy Kokoro fixtures, whose durations already match their PCM.
- Implementation and regressions: lab work unit V-A.2 now, then T1.6/T3.6/T3.7/T3.8 for the product
  ([16](../revamp/16-lab-track-and-review-findings.md)).
