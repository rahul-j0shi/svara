# 0008 — Build a phoneme-to-viseme performance engine
Status: Accepted

Context: Lip sync is a defining product capability, not a decorative UI effect. Provider timing
quality varies, and the engine must work across languages, TTS providers, and VRM models without
binding the core to a renderer. The data model and timing tiers are specified in
[Avatar §§3–5](../revamp/03-avatar-and-lipsync.md#3-the-performance-data-model).

Decision:
- Normalize timed phonemes into a 15-viseme Oculus-style vocabulary in the Python core, from a
  declared per-adapter phone inventory (ADR-0018).
- Prefer provider phoneme timings, then word timings plus G2P, then estimated timings, while exposing
  the timing tier in diagnostics.
- Produce immutable performance segments containing viseme keyframes, an amplitude envelope,
  expressions, and gestures.
- Retarget canonical visemes through a per-avatar map.
- Schedule audio and facial performance against the renderer's single `AudioContext` clock, with
  renderer-side anticipation, coarticulation, and smoothing.

Consequences:
- TTS adapters and avatar rendering stay independent behind a stable performance contract.
- The engine requires golden timing tests and a visual lip-sync lab, not only unit tests.
- Lower timing tiers degrade gracefully but cannot match native phoneme timestamps.
- Svara owns language normalization, retargeting, and synchronization quality.

Evidence:
- T0.8 exercises a viseme track against an ignored test VRM, not the unselected production asset.
  Duration extraction is confirmed; true phoneme/visible accuracy remains open.
- [Feature F15](../revamp/12-feature-specification.md#f15--lip-sync-performance-engine) defines the
  quality and fallback acceptance criteria.
- 2026-10-10 amendment after the engineering review: segment extent, key ranges, envelope and clock
  semantics are fixed by [ADR-0017](0017-make-pcm-frames-the-performance-segment-authority.md);
  articulation ownership, declared phone inventories and literal G2P input by
  [ADR-0018](0018-give-articulation-one-owner-and-declare-phone-inventories.md). The 15 canonical
  labels are an internal vocabulary; a basic rig shows five vowels plus named approximations.
  Language support is a tested qualification matrix, not a property of IPA. The custom engine keeps
  its place only where the bounded lab comparison (ADR-0021, V-D.2) shows a benefit over one reused
  baseline; otherwise the simpler compatible baseline is kept by a recorded decision.
