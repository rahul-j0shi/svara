# 16 — Lab track and engineering-review finding ledger

Adopted **2026-10-10** from the
[engineering review, rounds 1–5](../reviews/2026-10-09-engineering-review-rounds-1-5.md) (reviewed
commit `f33c994`). Decisions are in ADRs
[0017](../adr/0017-make-pcm-frames-the-performance-segment-authority.md)–[0021](../adr/0021-run-a-bounded-lab-track-with-separate-acceptance-levels.md);
the per-document change list is the
[reconciliation note](../reviews/2026-10-10-engineering-review-reconciliation.md).

**Authority.** [12](12-feature-specification.md) still controls product scope, [13](13-implementation-guide.md)
the product task graph and [14](14-acceptance-and-integration.md) acceptance. This document is the
single execution plan for the **lab track** and the owner map for review findings SV-01–SV-24. It
adds no feature, removes no requirement and relaxes no target. Lab work unit IDs (`V-…`) are not
product tasks and never mark a task, gate or AC passed.

## 1. Acceptance levels and the two routes

| Level | Proves | Never proves |
|---|---|---|
| **Fixture** | A code path under stated, usually mocked or synthetic, inputs | Hardware, perception, a lab result or a feature |
| **Lab result** | A named corpus × rig × device × build outcome, with its limits | A product AC, a platform gate or the assistant |
| **Product / release** | The integrated ACs, cross-feature checks and gates in 14 | Anything outside its recorded environment |

- **Lab track (this document):** a trustworthy phone → PCM → performance → pose path, using the
  existing native Kokoro runner, the code under `spikes/kokoro`, `packages/avatar` and the diagnostic
  desktop spike. It may proceed while Phase 0 platform gates are blocked, because it changes no
  product surface. Its limits are always named.
- **Product route ([13 §8](13-implementation-guide.md#8-ordered-backlog)):** unchanged order — Phase 0
  gates (T0.8, T0.10, T0.11, T0.12, then T0.13) before Phase 1, then Phases 1–7. "Later" is not
  "optional": every required provider, gate and F01–F40 AC remains.

Execution rules: one active work unit; one baseline candidate at a time; freeze comparison inputs
before measuring; use repository targets only; an inconclusive experiment ends with a stated next
decision, not a pass. A new candidate or tool enters active work only to answer a named unmet
requirement, replacing or extending one existing unit.

## 2. Lab scope

| Boundary | Lab value |
|---|---|
| Model/voice | Pinned Kokoro-82M timestamped ONNX revision `dd4401a9…` with `af_heart` (T0.8 pins) |
| Language | English (`en-us` espeak-ng voice), matching the current path |
| Rig/profile | One qualified free VRM (V-B.1, the same asset T0.12 selects when it qualifies), `vrm-basic` unless the report proves extended shapes |
| Playback environment | The primary dev host and one named output device, both recorded; X11 is acceptable for lab results because no platform gate is claimed |
| Runtime | Python 3.14, ONNX Runtime 1.30, no PyTorch, no extra local service |
| Corpus | §4, frozen with hashes before any comparison |

**Excluded from the lab:** full conversation orchestration; microphone, STT and cloud calls; every
F01–F40 feature; custom fifteen-shape art; a framework swap; universal rig or language claims; more
than one reuse comparison; backchannel detection or resuming speech (outside F13).

## 3. Work units

Order: V-A.1 → V-A.2 → V-A.3 → V-B.1 → V-D.1 → V-D.2 → V-C.1 (V-C.2 only if triggered).
V-B.2, V-E.1 and V-E.2 are independent small fixes, required only before the lab uses the affected
loader, focus mode or FPS metric. V-A.1 and V-A.2 get separate regressions even if one patch touches
both; V-A.3's literal-text fix may land separately from its inventory work.

| Unit | Findings | Included | Excluded | Done / stop condition | Carries into |
|---|---|---|---|---|---|
| **V-A.1** final mouth-pose contract | SV-02, SV-10 | Implement ADR-0018's ownership: generator timing only, player as the single blender, linear retarget. Make PP/FF final-weight regressions explicit (full FF, PP→vowel 25/40/50/80/120 ms, FF→vowel, short closure, overlap, silence, end). State `vrm-basic` consonant approximations | New rig art, providers, universal phoneme claims | Arithmetic regressions and vowel controls pass through the **final** mapping; a real-rig sweep (after V-B.1) separately shows appearance; arithmetic results are labeled as such | T3.7, T3.8 |
| **V-A.2** canonical audio/performance boundary | SV-14, SV-15, SV-16, SV-21 | ADR-0017: frame-count extent, envelope from played PCM with calibrated range and silence floor, pairing and key-range validation before playback, segment-local tail policy | Gapless/cross-segment wire format | Silent played PCM stays neutral; 1/32-amplitude copy keeps a lower intensity; 339-frame/24 kHz fixture accepted; mismatched audio and out-of-range keys rejected without starting a source; 0/1/4-frame final pads emit no key beyond extent; matching input still plays | T1.6, T3.7, T3.8 |
| **V-A.3** literal text and declared inventory | SV-09, SV-24 | Fixed argv + `--stdin`, UTF-8 policy, return code, timeout; pinned-tokenizer inventory with every symbol mapped or explicitly unsupported; pause vs unsupported; unknown counts; word provenance | Multilingual G2P, model/voice change, a phonetics framework | Argv/stdin capture for ordinary, `--help`, `--version`, `- hello`, multiline Unicode, apostrophes, punctuation; spawn failure/timeout/cancel; then actual pinned `espeak-ng` on the corpus. No claim that today's raw-English output is wrong | T3.6, T3.7 |
| **V-B.1** one asset qualification | SV-01, SV-07 | ADR-0007 selection and record; three-vrm introspection report and 0/0.25/0.5/0.75/1 sweeps; intended expression interaction; glTF Validator container check | Custom shapes, broad rig survey | Exact asset report with known limits. If it fails, evaluate one replacement; more searching needs a named missing capability | T0.12, T2.4 |
| **V-B.2** avatar-operation ownership | SV-18, SV-19, SV-20 | ADR-0019 intent identity at selection; gated install/status; owned URLs; prepare/commit/rollback | A generic scheduler | Late default success/failure, manual success/failure, failure at each preparation stage, disposal during either load: current avatar/status preserved, abandoned candidates/URLs disposed exactly once. Real repeated-load memory remains a separate measurement | T2.4 |
| **V-E.1** replaceable focus intent | SV-23 | One active helper + latest desired state; stale focus suppressed; native hints reconciled; sender check kept | New focus modes | open→close before/during/after helper completion, repeated toggles, helper failure, disposal, unauthorized sender: bounded queue, no focus after the latest close, final hints match latest state. A VM/mock pass is not a desktop pass | T2.3 (G-PRESENCE in T0.10) |
| **V-E.2** pacing and metric definitions | SV-22, SV-06 | ADR-0020 limiter and metrics; record frame-time distribution, long tasks and input-to-paint (ADR-0022, X27) on the spike | Renderer optimization work | Virtual 50/60/90/120/144/165/240 Hz and stall cases without bursts; deficits reported; then re-measure CPU/PSS/cadence with refresh rate recorded | T2.5, T2.11, G-BUDGET |
| **V-D.1** independent landmarks and output clock | SV-08, SV-11, SV-17 | Frozen §4 corpus; human-reviewed acoustic and visible landmarks; graph/output/display clock distinction; one named host/device | Offline aligner, perfect ground truth | Trace/export with uncertainty, device details and fallback behavior. Unavailable hardware leaves that claim unverified | T0.8 rerun, T3.12, G-TIMING |
| **V-D.2** one reuse comparison | SV-04 | Native path vs **one** baseline: HeadTTS browser mode first; if incompatible, one audio-driven baseline (three-vrm-lip-sync) | A second candidate without a named failure; any migration | Decision note retain/adapt/replace/defer, with first audible output (cold/warm), alignment, resources, cancellation/ownership on the same corpus and rig | T3.13; any adoption needs a new ADR + user decision |
| **V-C.1** native ownership | SV-05, SV-12 | ADR-0019 slot held until native exit, submitted-work cap, generation/credit fences | Termination APIs | Repeated cancel/replace histories show no stale playback and no slot freed before exit | T3.6, T3.15 |
| **V-C.2** termination experiment (optional) | SV-05 | Only if V-C.1 misses an existing target: per-run `RunOptions.terminate` vs bounded completion | Making it a requirement | Stop-request → native-exit → replacement-start trace; fences kept. No gain is a valid result | T3.6 |
| **V-F** optional breadth | — | Extra rigs/languages/providers beyond v1's required set, quantization, richer interruption, alternative frameworks | Everything until a measured need | A separate bounded proposal; not part of the lab or its case study | New ADR if adopted |

## 4. Frozen corpus and comparison protocol

Proposed starter corpus (it does **not** replace T0.8's corrected five-sentence visual rerun; record
both). V-D.1 commits it as a text file plus fixture hashes before the first comparison run; after
that, changes need a new baseline.

1. `Papa made a blue paper boat.` — bilabials, existing T0.8 fixture
2. `Five velvet flowers shimmer beside the fountain.` — labiodentals, existing fixture
3. `Bob met Mia by the bay.` — PP→vowel transitions
4. `Very fine fish fly over waves.` — FF/V→vowel
5. `Oh, I see. You ate early.` — vowel-to-vowel transitions and a sentence pause
6. `Wait. Stop... then go.` — pauses and short tails
7. `--help me read this line.` — leading-dash literal text
8. `Come home, Tom.` — final bilabial closure at the segment end

Controls (PCM fixtures, not synthesized text): all-zero PCM; utterance 1 scaled by 1/32; utterance 1
scaled by 10⁻⁶ (rounds to zero in s16); and a 339-frame 24 kHz segment for the fractional-ms rule.

Runs: 1 cold + 5 warm per utterance per path, chosen now and recorded. Results are descriptive; no
p95 or repository-target acceptance is inferred from this sample. Record voice/model/revision,
hashes, host, output device, refresh rate and build for every result. Match voices/models where
possible; record confounders where not.

**Minimum useful portfolio stop:** V-A contract regressions, V-B.1's qualified rig, the ownership
fixes the demo uses, and one reviewed corpus replay with timing export and stated limits. A V-D.2
decision strengthens it; failing to find a compatible baseline is documented, not a trigger for a
stack rewrite. It is a lab case study, never "an accepted conversational assistant", and publishing
it needs the user's approval. Case-study format: promised invariant → smallest breaking fixture →
wrong assumption → scoped change → regression → remaining limits.

## 5. Finding ledger (SV-01–SV-24)

Status vocabulary: **Decided** = contract chosen in an ADR/doc, code unfixed; **Corrected** = a
documentation claim fixed; **Open** = experiment still to run; **Fixed** only with
ID → fixing commit → regression command → actual output → remaining limitation. As of 2026-10-10 no
code finding is fixed. Priorities follow the review (P0 integrity, P1 contract before expansion,
P2 diagnostics/usability, P3 optional breadth).

| ID | Finding | Evidence | Pri. | Decision / doc | Lab unit | Product owner | Status |
|---|---|---|---|---|---|---|---|
| SV-01 | VRoidPreset A–Z described as CC0 | Primary-source mismatch | P1 | ADR-0007, 03 §1, 12 F14 | V-B.1 | T0.12 | Corrected; asset unselected |
| SV-02 | Full FF collapses to a neutral basic mouth | Reproduced arithmetic | P1 | ADR-0018 | V-A.1 | T3.7, T3.8 | Decided |
| SV-03 | Specification far ahead of a working conversation | Design | P2 | ADR-0021, this doc | — | T0.13 then 13 order | Decided |
| SV-04 | Sentence-first latency confidence lacks a first-audible comparison | Recorded + design | P1 | 03 §3, 04 §5 | V-D.2 | T3.13 | Open |
| SV-05 | Python cancellation does not stop native work or free its slot | Design | P1 | ADR-0019, ADR-0005, 01 §5 | V-C.1/V-C.2 | T3.6, T3.15 | Decided |
| SV-06 | Visible idle CPU above budget (16.123% of one core) | Recorded | P2 | ADR-0020, 14 §5 | V-E.2 | T2.11 (G-BUDGET) | Open |
| SV-07 | Language and rig generality exceed tested coverage | Design | P1 | ADR-0018, 03 §4.4/§4.6, 12 F15 | V-B.1 | T3.7, T7.4 | Corrected |
| SV-08 | LipSyncLab needs independent timing validation | Opportunity | P2 | 03 §6, 12 F18 | V-D.1 | T3.12 | Open |
| SV-09 | Unmapped native phones silently become silence | Source + synthetic | P1 | ADR-0018 | V-A.3 | T3.6, T3.7 | Decided |
| SV-10 | PP suppression applied at three stages | Reproduced arithmetic | P1 | ADR-0018 | V-A.1 | T3.8 | Decided |
| SV-11 | Correct total duration ≠ internal alignment | Design | P1 | 03 §4.8, 14 §5 | V-D.1 | T3.12 | Open |
| SV-12 | Kernel/provider teardown needs executable proof | Opportunity | P1 | ADR-0004, ADR-0005 | V-C.1 | T1.1, T3.15 | Open |
| SV-13 | Avatar polish cannot replace desktop gates | Design | P2 | ADR-0021, 14 §2 | — | T0.10, T2.11, T3.15 | Decided |
| SV-14 | Per-sentence normalization; sub-quantization signal animates silence | Reproduced | P1 | ADR-0017 | V-A.2 | T3.7 | Decided |
| SV-15 | Integer ms vs one-sample guarantee | Design + arithmetic | P1 | ADR-0017, 13 §7 | V-A.2 | T1.6, T3.6 | Decided |
| SV-16 | Player accepts mismatched audio/track durations | Reproduced (fake APIs) | P1 | ADR-0017 | V-A.2 | T3.8 | Decided |
| SV-17 | Graph time can lead audible output | Source + synthetic clock | P2 | ADR-0017 | V-D.1 | T3.8, T3.12 | Open (measurement) |
| SV-18 | Startup fetch overwrites a later manual avatar | Reproduced | P2 spike / P1 product | ADR-0019 | V-B.2 | T2.4 | Decided |
| SV-19 | Stale startup error overwrites newer status | Reproduced | P2 | ADR-0019 | V-B.2 | T2.4 | Decided |
| SV-20 | Candidate not disposed on preprocessing failure | Fault injection | P1 | ADR-0019 | V-B.2 | T2.4 | Decided |
| SV-21 | Release keys outlive the segment; player accepts them | Synthetic fixture | P1 | ADR-0017 | V-A.2 | T3.7, T3.8 | Decided |
| SV-22 | 48 fps at 144 Hz while the probe reports zero drops | Reproduced (virtual clocks) | P2 / P1 for FPS claims | ADR-0020 | V-E.2 | T2.5, T2.11 | Decided |
| SV-23 | Focus FIFO executes obsolete opens after close | Reproduced (mocked main process) | P1 before reuse | ADR-0019, ADR-0002 | V-E.1 | T2.3 | Decided |
| SV-24 | Phonemizer text in CLI option position | Argv capture + eSpeak NG source | P1 | ADR-0018, ADR-0009 | V-A.3 | T3.6 | Decided |

Retained corrections: the renderer's in-loader stale-success fence, its disposal-while-pending
fence, and the stale `onended`/late-decode playback guards **passed** review fixtures; keep them
as regressions. Barge-in backchannels and resume stay outside F13. Summed RSS can double-count shared
pages; neither the CPU nor RSS figure establishes a leak.

Two credibility corrections from round 1 are also closed in documentation: the Google Cloud STT
adapter is kept because F10 requires it and its fit is measured like any adapter (04 §4), and the
`v0-prototype` tag was never pushed (§7).

## 6. Parked research registry — not a backlog

V-D.2 selects at most one baseline. These remain parked unless a named unmet requirement promotes
one; re-check compatibility at selection time. Research dates are those in the review.

| Candidate | Retained decision | Evidence needed before any use |
|---|---|---|
| HeadTTS (browser mode) | First V-D.2 comparison; not a migration | Same voice/model where possible, timing quality, first PCM, memory, WebGPU/WASM fallback, cancellation |
| three-vrm-lip-sync / HeadAudio | Audio-driven baselines | Same rig; visible closures; latency/calibration; peer-version fit |
| TalkingHead | Alternate framework experiment | GLB/ARKit/Oculus rig conversion cost; not a VRM drop-in |
| kokoro-onnx / Pipecat Kokoro | Not drop-ins under Python 3.14 / ORT 1.30 pins | A current resolver result preserving the duration contract |
| Pocket TTS | Isolated latency comparison only | Deliberate PyTorch scope change (user decision) before adoption |
| Moonshine Voice | STT comparison candidate | Python 3.14 packaging, accuracy, finalization delay, artifact terms |
| Montreal Forced Aligner | Offline evaluation reference | Model/dictionary plus reviewed anchors; never runtime |
| Supertonic archive | Optional comparison; archived upstream | Maintenance and timing fit |
| ORT termination / quantization | V-C.2 / V-F experiments | Actual-host latency, quality, duration equality, cancellation |
| fast-check scheduler | Optional if deferred-promise tests outgrow themselves | Hypothesis (already a dev dependency) covers Python lifecycle tests |

## 7. Open items that need the user (not developer work)

| Item | Why it cannot be closed in the repository | Who / when |
|---|---|---|
| Wayland 24.04 session and 26.04 GNOME VM | G-PRESENCE/G-PLATFORM need real sessions; containers and X11 do not count | User supplies; blocks T0.10 → T0.13 |
| Speaker/mic consent | G-AEC transport and F13 trials capture audio | User consent at T0.10/T3.11 |
| `v0-prototype` tag | The prototype is preserved in history at merge commit `2389194`, but no tag exists on the remote (`git ls-remote --tags` returned none on 2026-10-10). Pushing a tag is outward-facing | User approves; then `git tag v0-prototype 2389194 && git push origin v0-prototype` closes T0.1's remaining step |
| Account/provider tests | T0.11 subscription bridge and any paid call | User consent at T0.11 |
| Publishing a lab case study or preview | Publication is never implied by a passing check | User approval |

## 8. Status updates and next session

Status update format: **work unit → finding IDs / task → commit or experiment inputs → actual
result → remaining limitation.** Report fixed code, decided contract, qualified asset, measured
device and accepted release separately. Unknown remains unknown.

**Next session:** V-A.1 only — make the PP/FF final-weight regression explicit and implement the
single shaping owner from ADR-0018 in the spike player/renderer. If the user supplies Wayland
sessions, resume T0.10 on the product route in parallel; do not advance dependent production
acceptance on mock evidence.
