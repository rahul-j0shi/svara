# 0021 — Run a bounded speech/avatar lab track with separate acceptance levels
Status: Accepted

Context: The plan specifies 40 features, 114 ACs and 89 tasks, while the runtime is still scaffolding
and the Phase 0 platform gates are blocked on Wayland sessions that are not available. The
engineering review found that the plan mixed a laboratory milestone with the whole assistant, that
mock evidence risked being read as hardware evidence, and that a long shortlist of reusable
components was drifting toward active work. The review's §6 is a single bounded execution plan; this
ADR adopts it without weakening any accepted requirement. See
[16](../revamp/16-lab-track-and-review-findings.md) and
[the review scope](../reviews/2026-10-09-engineering-review-rounds-1-5.md#scope-and-how-to-use-this-review).

Decision:
- **Three acceptance levels, never promoted implicitly:** a *fixture* proves a code path; a *lab
  result* demonstrates a named corpus/rig/device outcome; a *product/release* passes the integrated
  ACs and gates in 14. A mock is not hardware evidence; a successful lab does not accept the assistant.
- **Two routes with separate definitions of done:**
  - *Lab track* (16 §2–§4): trustworthy phone→PCM→performance→pose path with the existing native
    runner, one pinned voice/model, English, one qualified free rig, one playback environment and a
    small frozen corpus. It may run while Phase 0 gates are blocked because it changes only spike and
    pure package code, and it never marks a T-task, gate or AC passed.
  - *Product route* (13 §8): unchanged dependency order, Phase 0 gates first, then Phases 1–7.
    Regressions from lab work units are carried into their owning product tasks.
- **One active work unit, one baseline candidate at a time.** Freeze comparison inputs before
  measuring; use repository targets, never easier replacement thresholds; an inconclusive experiment
  ends with a stated next decision, not a feature pass.
- **Reuse is a bounded comparison, not a migration:** exactly one compatible baseline (HeadTTS
  browser mode first, else one audio-driven baseline) is compared with the native path. Any adoption
  that changes runtime, dependency or service boundaries (Python 3.14, no PyTorch, no extra local
  service in v1) needs its own ADR and user decision. Other candidates stay in a parked registry.
- **A lab case study may be published honestly labeled** (lab, not assistant) without waiting for
  v1; publication still needs the user's approval.

Consequences:
- Work proceeds on independent pure contracts while platform environments are unavailable, without
  bypassing G-PRESENCE/G-PLATFORM/G-AEC/G-BUDGET or relabeling blocked checks.
- The status format is fixed: work unit → finding IDs/task → commit or experiment inputs → actual
  result → remaining limitation; fixed code, decided contract, qualified asset, measured device and
  accepted release are reported distinctly.
- Optional breadth (extra rigs, languages beyond the qualified path, providers beyond v1's required
  set, quantization, richer interruption, alternative frameworks) needs a named unmet requirement.

Evidence:
- [Engineering review rounds 1–5](../reviews/2026-10-09-engineering-review-rounds-1-5.md), §6 and
  scope section; reconciled in
  [the 2026-10-10 reconciliation](../reviews/2026-10-10-engineering-review-reconciliation.md).
