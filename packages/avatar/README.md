# `@svara/avatar`

Framework-agnostic TypeScript for loading VRM avatars and playing visemes, expressions, gestures,
and idle behaviours. This package must not import React or desktop-shell APIs.

The T0.8 `PerformancePlayer` is pure clock-driven code: it interpolates the 15 internal viseme
channels, applies closure dominance and the amplitude envelope, and lets `AvatarRenderer` retarget
the result to standard VRM mouth expressions.

Known review findings in this spike code (recorded, not yet fixed; owners in
[16](../../docs/revamp/16-lab-track-and-review-findings.md)): closure suppression is applied in both
player and renderer and FF collapses on the basic rig (SV-02/SV-10, ADR-0018); out-of-range keys and
mismatched durations are accepted (SV-16/SV-21, ADR-0017); a candidate that fails preprocessing is not
disposed (SV-20, ADR-0019); the frame limiter under-renders at some refresh rates and the probe's
dropped-frame count can hide a cadence deficit (SV-22, ADR-0020).
