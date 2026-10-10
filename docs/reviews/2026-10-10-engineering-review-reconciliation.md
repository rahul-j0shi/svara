# Engineering review reconciliation — 2026-10-10

Request: understand Svara and the
[engineering review, rounds 1–5](2026-10-09-engineering-review-rounds-1-5.md), then update ADRs,
plans and the README end to end so that nothing is left open and everything stays well scoped.
**Documentation and plan change only.** No source code, test, dependency, asset or tag was changed.

The review was written against commit `f33c994`; `main` and this branch's base were still that
commit when this reconciliation was made, so every finding applies unchanged.

## 1. What the review asked for and where it now lives

| Review element | Adopted as |
|---|---|
| Scope section and §6 "single current execution plan" | [ADR-0021](../adr/0021-run-a-bounded-lab-track-with-separate-acceptance-levels.md) and [16](../revamp/16-lab-track-and-review-findings.md) §1–§4, §8 |
| Three acceptance levels (fixture / lab / product) | ADR-0021; 12 rule 7; 14 §1; 15 §6 template |
| Finding ledger SV-01–SV-24 | 16 §5, each with decision, lab unit, product owner and status |
| SV-14/15/16/17/21 segment boundary | [ADR-0017](../adr/0017-make-pcm-frames-the-performance-segment-authority.md); 03 §3/§5; 13 §7.2/§7.4; T1.6/T3.7/T3.8 |
| SV-02/09/10/24 (and SV-07) articulation and input | [ADR-0018](../adr/0018-give-articulation-one-owner-and-declare-phone-inventories.md); 03 §4; ADR-0008/0009 amendments; T3.6–T3.8 |
| SV-05/18/19/20/23 ownership | [ADR-0019](../adr/0019-own-asynchronous-operations-by-intent.md); 01 §5; 02 §3; 03 §2; ADR-0002/0005 amendments; T2.3/T2.4/T3.6; X23–X25 |
| SV-22 (and SV-06) pacing | [ADR-0020](../adr/0020-pace-frames-by-target-phase-and-report-deficits.md); 12 F02; 02 §8; 14 §5; T2.5/T2.11 |
| SV-01 avatar licence | ADR-0007 rewritten; 03 §1; 12 F14 + AC3; 14 G-ASSETS; T0.12; 09; assets README |
| SV-04 latency, SV-11 timing evidence | 03 §3/§4.8; 04 §5; 12 F12; T3.13; 14 §5 |
| SV-03/08/13 lab vs product | ADR-0021; 09 lab-track section; 11 G89 |
| Round-1 credibility corrections | README and 04 §4 (Google STT); README, revamp README, 09, 13 T0.1, foundation audit (tag) |
| Parked research registry | 16 §6 (not a backlog) |
| Next session = V-A.1 only | 15 §4.1; 16 §8; README |

## 2. Where this reconciliation decided what the review left to a work unit

The review asked several units to "choose" a contract. To leave nothing open, the choices are made
now in ADRs; the units implement and test them. Each choice stays inside the review's stated bounds.

| Open point in the review | Decision taken | Why it is within scope |
|---|---|---|
| V-A.1 "choose one owner for closure/coarticulation" | `PerformancePlayer` owns blending; generator owns timing only; retarget is linear; PP drives closure dominance, FF is a partial jaw reduction (ADR-0018) | Removes all three duplicate suppressions with the fewest moved responsibilities; the player already performs smoothing and envelope gain |
| V-A.2 "choose one segment-tail policy" | Strictly segment-local keys clamped to the extent; continuity carried by the player's smoothing state; neutral on `performance.end`/stop/discard/gap; no cross-segment wire format (ADR-0017) | The review lists this as an acceptable policy and calls cross-segment representation optional |
| SV-14 "declare activity-vs-loudness semantics" | Articulation intensity from played PCM, per-voice calibrated range, explicit silence floor, no per-sentence normalization | Matches the existing design intent ("whispers and shouts look different") while fixing the silent-file case |
| SV-15 frame-count vs float | Integer `frameCount` authoritative; `durationMs` derived and informational | The review prefers exact frame counts for the invariant |
| SV-17 clock | Schedule on graph time; sample the face/progress from `getOutputTimestamp` with recorded fallback | The review's proposed experiment, made the default with an explicit fallback |
| Lab corpus "can be eight short utterances" | Eight utterances and four PCM controls listed in 16 §4; frozen with hashes by V-D.1 | Same coverage the review lists (PP/FF, transitions, pauses, short tails, leading dash, quiet/silent) |
| "Choose and record cold/warm repeat counts" | 1 cold + 5 warm per utterance per path, descriptive only | No p95 or target acceptance inferred, as the review requires |
| SV-01 asset choice | Selection order: publisher-designated CC0 first, else preset under its actual terms if redistribution is permitted (ADR-0007) | The review requires exact terms, not a specific asset; the actual pick remains T0.12's job with a re-check of the terms |

Protocol additions made to carry these contracts: `frameCount` (required), `timingTier` and
`unknownPhones` on `performance.segment` (13 §7.2). They serve F15/F18 diagnostics; no new feature.

## 3. Deliberately not adopted or left for the user

- **Round-1 "first useful loop with push-to-talk/headphones":** not adopted as a milestone. The
  review's own later rounds say headphones or push-to-talk do not accept F13's speaker requirement;
  the product route keeps its order and gates.
- **Pushing the `v0-prototype` tag:** outward-facing; recorded as a user action in 16 §7 with the
  exact command. The docs now say the prototype lives at `2389194` and the tag is unpublished.
- **Running the review's fixtures, installing the glTF Validator or eSpeak:** implementation and
  lab work, assigned to V-A/V-B units. Nothing was executed against the code here.
- **Framework or provider swaps (HeadTTS, TalkingHead, Pocket TTS, Moonshine, …):** parked in
  16 §6; only V-D.2's single comparison is active, and any adoption needs an ADR and user decision.
- **Upgrading three-vrm to 3.6.0-beta:** not a fix for these findings (ADR-0007).

Items only the user can unblock are listed once, in 16 §7: Wayland 24.04 session and 26.04 VM,
speaker/mic consent, the tag push, account/provider tests and any publication.

## 4. Files changed

- New: ADR-0017–0021; `docs/revamp/16-lab-track-and-review-findings.md`; this note; the review
  itself, copied unchanged as `2026-10-09-engineering-review-rounds-1-5.md` (SHA-256
  `ca35c9c6114866d674e45a523de0dfe61dbfeabd05a6aae8bd041e9a86407f65`).
- Amended ADRs: 0002, 0004, 0005, 0006, 0007 (rewritten decision), 0008, 0009; ADR index.
- Plan: 01, 02, 03, 04, 09, 11 (gap log G83–G92, risks, over-engineering check), 12 (F02/F12/F14/
  F15/F18/F40, §0.1, rule 7, change log), 13 (§1, ADR index, §7, task "Done when" cells for
  T0.1/T0.8/T0.10/T0.12/T0.13/T1.6/T2.3/T2.4/T2.5/T2.11/T3.6/T3.7/T3.8/T3.12/T3.13), 14 (gates,
  X23–X25, §5, §6), 15 (§1 known findings, read order, §4.1 next unit, template), revamp README.
- Evidence notes: T0.8 and T0.10 spike reports, foundation audit (tag row and pointer).
- READMEs: root, `assets/`, `packages/avatar/`, `spikes/kokoro/`, `spikes/ubuntu-integration/`.

No task ID, dependency cell, feature, AC count or numeric limit was added, removed or relaxed. The
task graph still has 89 tasks; 40 features / 114 ACs remain mapped.

## 5. Verification

- `node docs/revamp/check-plan.mjs`: task DAG, AC mapping, R1–R38 and local links pass.
- `git diff --check`: no whitespace errors.
- Every ADR/doc link added here was checked to resolve to a file in the repository.
- No GUI, Wayland, microphone, provider, model or code test was run; none was needed for a
  documentation change, and none is claimed.

## 6. Addendum — liquid-smooth experience contract (same day)

Follow-up request: the plan had functional UX (avatar-only surface, state visuals ≤100 ms, bubble
timing, 30/60 fps) but no definition of how interactions must *feel*. Added, without changing any
feature, AC, task dependency or existing limit:

- [ADR-0022](../adr/0022-hold-a-liquid-smooth-experience-contract.md) and
  [17 — Experience & motion](../revamp/17-experience-and-motion.md): principles, one motion
  vocabulary, choreography for every moment, perceived-latency timeline, render/audio rules,
  measurement and owners.
- 12 §0.5 global budgets B1–B9 (input acknowledgement, frame consistency, long tasks, drag, transitions,
  audio ramps, window first paint, layout stability, reduced motion); F02 renders 60 fps during any
  motion.
- 14 §3.4 checks X26–X30 with owners and a smoothness measurement protocol in §5; 13 conventions and
  "Done when" for T2.3/T2.5/T2.6/T2.8/T2.9/T2.10/T2.11/T3.8/T3.10/T3.15/T6.6/T7.5; R39/G93 and a
  risk in 11; lab V-E.2 records frame-time and long tasks; 02, 03, 09, 15, READMEs cross-referenced.

Deliberately excluded: earcons, custom menus, fake typing/thinking delays and filler speech, skeleton
screens. The quality ladder is a logged mitigation, never a budget pass.
