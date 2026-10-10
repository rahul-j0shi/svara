# Svara — cumulative engineering review, rounds 1–5

**Latest review evidence:** 9 October 2026. **Earlier rounds and document split:** 8 October 2026. **Scope revision:** 9 October 2026. **Purpose:** evidence-backed review and bounded implementation guidance.

This standalone document preserves the Svara reviews from rounds 1–4 and adds the final fifth review with new controlled evidence, primary-source research, reasoned solutions, and an ordered implementation plan. The scope section and §6 govern current work boundaries; the cumulative ledger/corrections govern finding facts. Dated review sections retain their evidence and qualifications. The original combined fourth-round report is retained unchanged.

**Navigate:** [scope](#scope-and-how-to-use-this-review) · [execution plan](#6-prioritized-implementation-plan) · [final round](#final-round--9-october-2026) · [final fixtures](#12-final-round-reproduction-code) · [ledger](#2-cumulative-finding-ledger) · [corrections](#3-current-corrections) · [round 4](#4-round-4-findings) · [research](#5-research-and-reuse-decisions) · [verification](#8-verification-and-limits) · [historical reviews](#9-historical-project-reviews) · [reproduction code](#10-round-4-reproduction-code).

## Scope and how to use this review

**Scope edit: 9 October 2026.** This pass reviews the documents' boundaries and execution plans. It adds no new repository findings or test results. The evidence dates and reviewed commit remain those recorded below.

**Assessment:** the evidence was carefully qualified, but the work plan mixed a laboratory milestone with the entire assistant, put platform feasibility too late, grouped unrelated input/rendering/lifecycle repairs, and treated a long component shortlist too much like active work. The revised §6 is the single current execution plan. Historical action lists are context, not additional commitments.

| Boundary | Current scope |
|---|---|
| Purpose | Establish a trustworthy speech/avatar experiment and carry only proven contracts into the Ubuntu assistant. A bounded lab and the full desktop product have separate definitions of done. |
| Immediate target | Existing phone/PCM/performance-to-pose path, using the chosen native runner and one exact qualified free rig. Start with one final-pose regression/work unit; small independent fixes may proceed separately. |
| Lab defaults proposed by this review | One recorded voice/model pin, English input matching the current path, one rig/profile, one playback environment, and a small frozen corpus. Broader language/rig/provider claims are deferred until separately qualified. These defaults narrow the experiment, not the accepted release requirements. |
| Accepted runtime boundaries | Python 3.14, no PyTorch and no extra local service in v1; retain the small accepted kernel, existing generation fences, and synthesis-credit plan. A comparison candidate is not an approved architecture migration. |
| Early product feasibility | Keep Phase 0 foundation/platform/runtime/asset checks, including T0.10/T0.13, before dependent production work. A blocked hardware test stays blocked. Independent pure lab work may continue with its limitation named. |
| Excluded from the immediate lab | Full conversation orchestration, microphone/STT/cloud integration, every F01–F40 feature, custom fifteen-shape art, a framework swap, universal rigs/languages, and multiple reuse comparisons. Backchannel detection/resume remain outside accepted F13. |
| Later required work | The existing implementation guide's dependency order, required providers, desktop/audio gates, preview criteria, and full F01–F40 acceptance. “Later” does not mean optional or removed. |
| Optional extension | Extra voices/providers/rigs beyond the accepted requirements, alternative frameworks, quantization, and richer interruption policy, only after a demonstrated need. |

**Three separate acceptance levels:** a deterministic fixture proves a code path; a lab demonstrates a named corpus/rig/device result; a product/release passes the existing integrated acceptance gates. A mock does not promote itself to hardware evidence. A successful lab does not accept the assistant. Completing the whole assistant is also not required to publish an honestly labeled lab case study.

**Execution rules:** one active work unit, one baseline candidate at a time, and no open-ended search for the “best” avatar/TTS stack. Freeze the comparison inputs before measuring. Existing budget/accuracy targets come from the repository; this review introduces no easier replacement thresholds. An experiment can finish with an inconclusive result and a stated next decision without being labeled a feature pass.

**Authority:** this scope section and §6 govern work boundaries; the ledger/corrections govern issue facts; dated findings/reproductions are evidence; market tables and historical plans are reference. The review schedule never supersedes repository task dependencies or marks required v1 provider support optional.

Scope alignment checked against the existing [implementation guide](https://github.com/rahul-j0shi/svara/blob/f33c9943012d9dbbc331c4c8adea5c36a4cc09f2/docs/revamp/13-implementation-guide.md), particularly Phase 0, T2.4/T2.11, and T3.6–T3.15. This was a scope alignment check, not another source-code audit.


## Final round — 9 October 2026

**This is the fifth and final research/review round.** Rounds 1–4 remain below as dated evidence. This section records round-5 evidence and reasoning. The scope section and §6 govern current execution boundaries. The repository was fetched again; HEAD and origin/main remain `f33c9943012d9dbbc331c4c8adea5c36a4cc09f2`, with a clean working tree. No finding was fixed during this review.

### Final verdict: prove the boundaries before buying more sophistication

Svara has a worthwhile Ubuntu voice/avatar goal and a potentially distinctive native-timing lab. Its weakness is that the pieces have more local intelligence than shared authority: timing stages each reshape the mouth, asynchronous operations decide freshness at different boundaries, and metrics can report reassuring numbers while the requested cadence is missed.

**Roast:** the avatar is being asked to look alive while the plumbing still disagrees about which operation is current, which clock matters, and whether the text is actually text.

Keep the accepted Python 3.14, no-PyTorch, no-extra-local-service v1 scope. Reuse a qualified free avatar and existing rendering libraries. Custom phoneme/viseme work is justified only where the timing experiment demonstrates a benefit over a fair reused baseline. Another provider or a more elaborate face cannot repair the boundary contracts below.

### SV-22 — High-refresh callbacks turn the 60 FPS target into 48 FPS

**Evidence:** actual `AvatarRenderer` and `FrameProbe` code executed with virtual requestAnimationFrame timestamps and rendering dependencies mocked. Ten simulated seconds per cadence; no GPU or actual display measurement. **Priority:** P2 for spike diagnostics; P1 before using these metrics to claim the FPS/resource gate is met.

The renderer uses an elapsed threshold of `frameInterval × 0.88`, then resets its origin to the callback where a frame was rendered. At 144 callbacks/second, two callback intervals are shorter than that threshold; three are sufficient. It therefore renders every third callback: 48 frames/second. At 165 callbacks/second, the result is 55 frames/second.

| Virtual callbacks/second | Render calls in 10 s | Reported FPS | Reported dropped frames | Phase-preserving arithmetic prototype |
|---:|---:|---:|---:|---:|
| 60 | 600 | 59.9 | 0 | 600 |
| 120 | 600 | 59.9 | 0 | 600 |
| 144 | 480 | 47.9 | 0 | 600 |
| 165 | 550 | 54.9 | 0 | 600 |
| 240 | 600 | 59.9 | 0 | 600 |

The probe rounds each local interval divided by the target interval, then subtracts one. A 20.833 ms interval divided by 16.667 ms is about 1.25; rounding produces one and reports zero missed intervals. Consequently a consistent 20% cadence shortfall can show zero dropped frames. “Zero dropped” and “met target” are different claims.

**Roast:** the display can offer you more callbacks and your limiter rewards it with fewer frames. Then the probe congratulates itself for dropping nothing.

**Better solution:** preserve target phase with an absolute next deadline or accumulator. Render at most once per callback; advance the deadline past missed opportunities instead of doing an expensive catch-up loop. Keep the audio/performance clock independent of render cadence. For diagnostics, report callback availability, rendered FPS, target deficit, visibility state, and frame work separately. Define the measurement window/endpoints instead of hiding its small 59.9-versus-60 estimator difference.

The fixture includes a simple phase-preserving arithmetic prototype that counts 600 opportunities at all five cadences. This demonstrates the scheduling alternative; it is not an integrated renderer patch or evidence of real GPU/CPU performance. Check mode transitions, long stalls, and throttling before adopting it.

**Research:** the HTML standard describes rendering opportunities as dependent on hardware and user-agent conditions, including visibility. requestAnimationFrame is not a promise of a universal 60 Hz source. Test the cadence contract against actual opportunities and keep target-host performance as a separate measurement. [HTML rendering processing model](https://html.spec.whatwg.org/multipage/webappapis.html), [animation frame callbacks](https://html.spec.whatwg.org/multipage/imagebitmap-and-animations.html), checked 9 October 2026.

**Exit checks:** deterministic 50/60/90/120/144/165/240 Hz schedules; irregular stalls; idle/speaking transitions; hide/restore; no render bursts. Report unavoidable target deficit below 60 Hz honestly. Then rerun CPU/PSS and actual rendering on the required Ubuntu host, including a high-refresh display if it is in scope. Fixing cadence can increase rendering work; do not assume it improves the existing idle CPU budget.

**Source:** [avatar-renderer.ts](https://github.com/rahul-j0shi/svara/blob/f33c9943012d9dbbc331c4c8adea5c36a4cc09f2/packages/avatar/src/avatar-renderer.ts), [metrics.ts](https://github.com/rahul-j0shi/svara/blob/f33c9943012d9dbbc331c4c8adea5c36a4cc09f2/packages/avatar/src/metrics.ts).

### SV-23 — A serial focus queue keeps obsolete opens alive

**Evidence:** actual Electron main-process source executed in a Node VM with window/native-helper APIs mocked, specifically in opt-in `managed-hints` mode. No real process or desktop focus operation was performed. **Priority:** P1 before reusing this spike path for production interaction.

The fixture queues 20 open requests, requests close while the first open is pending, then completes the helper calls. Every obsolete open still runs and calls window focus after close was requested. Close eventually runs and blurs the window.

```text
queuedOpenRequests: 20
lateFocusCallsAfterCloseRequested: 20
nativeCalls: off, on × 20, off
finalAction: blur
```

The failure is transient stale focus plus accumulation of obsolete work, not permanent failure to close and not a reproduced Wayland incident. The frontend's interaction generation can ignore a stale returned result; it cannot undo focus effects already performed by the main process. The main process serializes calls, but the queued calls still represent old intent.

**Roast:** your queue faithfully honors every outdated wish. The user said close; the implementation hears “after these twenty opens.”

**Better solution:** model focus as replaceable desired state. Keep one active helper operation and one latest pending state, with a generation assigned when intent arrives. Replace pending obsolete work rather than storing every toggle. After an await, verify current identity before calling focus or publishing a result. If an old in-flight helper changed native hints, reconcile them to the current desired state; a JavaScript identity check alone cannot reverse that native effect.

Retain active ownership until the child exits. Cancellation is optional if supported and tested; it does not replace reconciliation or prove that a helper had no side effects. A stale completion should be a typed stale result, not a current UI failure. Preserve the existing owned-window sender check. A synchronous blur on close may be useful, but verify it against the actual window-manager behavior rather than treating the mock as a desktop specification.

**Why this is better:** FIFO is appropriate for mutations whose entire history must be applied. Focus is a current-state request. Coalescing bounds obsolete work and preserves the user's latest choice without adding a general scheduler framework.

**Exit checks:** open→close before/during/after helper completion; repeated toggles; helper failure; disposal; unauthorized sender rejected. Queue size remains bounded; obsolete completion cannot call focus after the latest close; final native hints match latest state. Separately verify real focus/click-through restoration on the mandatory Ubuntu/Wayland gates.

**Source:** [Electron main.cjs](https://github.com/rahul-j0shi/svara/blob/f33c9943012d9dbbc331c4c8adea5c36a4cc09f2/apps/desktop/electron/main.cjs). This extends the whole-operation ownership principle from SV-18–20 into a different process boundary; it does not establish that all current focus modes are broken.

### SV-24 — The phonemizer's argument position does not preserve literal text

**Evidence:** source-extracted actual `phonemize` function with a subprocess argument spy. The spy records the call; it does not run eSpeak or establish actual spoken output. Official eSpeak NG documentation/source independently establishes its option parsing. **Priority:** P1 small input-contract fix before expanding text coverage.

The function places text directly in the final argument position:

```text
[espeakExecutable, "-q", "--ipa=3", "-v", "en-us", text]
stdin input: absent
timeout: 10 seconds
```

For harmless literal text `--help` or `--version`, that argument is in an option-parsing position. There is no option terminator or stdin text boundary. eSpeak NG documents those switches and parses options with `getopt_long`. Passing a list of arguments already avoids a shell; it does not distinguish options from data inside the called program.

**Roast:** your language pipeline has a text boundary that depends on whether the sentence begins with a dash. That is a plumbing bug, not a hard phonetics problem.

**Better solution:** use the documented `--stdin` path and pass the exact text through subprocess input. Keep a fixed argv for executable/configuration, an explicit UTF-8 policy, return-code checking, the existing timeout, and the planned cancellation/active-job ownership. Validate returned phonemes against the declared inventory and preserve word provenance. Do not “solve” this by deleting hyphens or mutating legitimate language input.

A verified option terminator can be another approach for a particular pinned binary, but stdin handles paragraphs/leading dashes more directly. This does not require a new package or an extra service. Test actual pinned eSpeak behavior before marking the issue fixed.

**Exit checks:** ordinary sentences; literal leading `--help`/`--version`; hyphen-leading text; multiline Unicode, apostrophes, punctuation; explicit control/invalid-input policy; spawn failure, timeout, and cancellation. Assert fixed argv plus exact stdin, then compare actual phones on the pinned binary and a small declared corpus. Placeholder spy output is not speech evidence.

**Primary sources:** [eSpeak NG manual](https://github.com/espeak-ng/espeak-ng/blob/master/src/espeak-ng.1.ronn), [CLI option parser](https://github.com/espeak-ng/espeak-ng/blob/master/src/espeak-ng.c), checked 9 October 2026. **Project source:** [kokoro_spike.py](https://github.com/rahul-j0shi/svara/blob/f33c9943012d9dbbc331c4c8adea5c36a4cc09f2/spikes/kokoro/src/kokoro_spike.py).

### Research-to-decision register: where reuse actually helps

| Decision | Final recommendation | Reason and limit |
|---|---|---|
| Avatar creation | Qualify one existing free rig under its exact terms; use the accepted renderer. | Art work is not the current bottleneck. Free is not synonymous with CC0, and five basic vowel shapes cannot prove every phoneme distinction. |
| Phone/viseme path | Establish canonical PCM/frame extent, named phone inventory, and one owner for closure shaping before comparing alternatives. | Native timing may be valuable; repeated suppression and silent mappings obscure that value. A provider swap cannot repair a contradictory consumer contract. |
| Lip-sync baseline | Compare one native-timing path with one compatible reused/audio-driven baseline on the same rig/corpus where possible. | HeadTTS and earlier fallback candidates remain experiments, not newly verified drop-in replacements. Measure visible landmarks, first audible output, resource cost, and cancellation. |
| Async ownership | Share a small principle across asset selection and focus: intent identity, owned acquisition, stale-result suppression, rollback/reconciliation. | The operation starts before its final await. Keep specialized implementations; do not inflate the accepted small kernel into a new framework. |
| Text input | Fixed configuration argv plus literal stdin. | Resolves the program's option/data boundary with existing tooling; preserve actual text. |
| Rendering | Phase-preserving limiter plus honest deficit metrics. | Different callback rates should not accidentally degrade a reachable target. Real resource budget still needs measurement. |
| Runtime/provider breadth | Keep the accepted runtime constraints and pinned verified artifacts. | Python 3.14/ORT/native timing support must be demonstrated by a resolver and run, not inferred from a popular wrapper. Existing model hash/pin checks deserve credit. |

Only the framing/cadence/CLI primary references relevant to this round were newly checked. The broad market candidate table in §5 retains its prior research dates and compatibility qualifications. No new wrapper installation, dependency resolution, model inference, or all-provider survey occurred. “Latest” should mean verified compatibility for the option you actually choose, not a long list of fashionable tools.

### Current execution plan

The consolidated work units, dependencies, exclusions and stopping points are in [§6](#6-prioritized-implementation-plan). This dated evidence chapter does not maintain a second implementation schedule.

### What would make this stand out to a hiring manager

Your strongest artifact is a **small believable lab**, not a promise of the final assistant: one qualified free avatar, the same spoken corpus across native/reused paths, visible independently reviewed landmarks, first-audible timing, resource measurements, and an interrupt/replace demonstration with ownership traces. Keep platform-incomplete behavior labeled.

I would discount a “real-time AI companion” claim supported mainly by a specification and character render. I would take seriously a demo that shows a closure bug, fixes its single owner, explains output-clock uncertainty, and proves a late request cannot take over. That shows engineering judgment even before the complete desktop release exists.

**The custom work earns its place when you can answer:** what do native model timings improve over the reused baseline, for which phones/rig/language, at what startup/CPU cost, and under what interruption behavior? If the improvement is not demonstrated, keep the simpler compatible baseline. A phoneme/viseme distinction is worth understanding; it is not automatically a reason to build every adapter yourself.

### Final evidence limits

New execution: virtual-cadence actual renderer/probe, mocked main-process focus lifecycle, and subprocess-call capture from the actual phonemizer. Node 24.19.0 and Python were used in scratch harnesses. No real eSpeak binary was installed/run; no GPU/display benchmark, ONNX inference, native cancellation measurement, microphone/cloud request, PipeWire/AEC session, or target-Ubuntu focus experiment occurred. Mock render/focus calls are logical evidence, not hardware outcomes. The phase prototype is arithmetic only. Earlier reproductions retain their original dates and qualifications. Full fixture source and instructions appear in the final-round appendix.


## 1. Verdict and intent

**Svara is a worthwhile voice/avatar experiment whose useful components do not yet compose into a proven assistant.** Preserve the Ubuntu desktop goal and its accepted Python 3.14, no-PyTorch, and no-extra-local-service v1 constraints. Compare reusable components within those constraints.

The blunt criticism: the specification promises more than the current integration proves. A polished character will not compensate for indistinguishable mouth poses, unowned native work, or late startup operations undoing a user choice. Establish a qualified free rig, canonical performance boundaries, and executable lifecycle evidence.

**Reviewed repository:** [Svara](https://github.com/rahul-j0shi/svara/tree/f33c9943012d9dbbc331c4c8adea5c36a4cc09f2)

**HEAD and origin/main checked in round 4:** `f33c9943012d9dbbc331c4c8adea5c36a4cc09f2`. At that review the repository was unchanged from round 3 and clean. **No earlier finding is marked fixed.** Historical reproductions retain their original evidence date; they were not all rerun in round 4.

## 2. Cumulative finding ledger

These rows include defects, unproven guarantees, measurements, and presentation opportunities. **The row count is not a bug count.** Stable `SH` and `SV` IDs are assigned here so subsequent work can reference one issue without confusing the earlier reports' numbering.

Evidence labels:

- **Reproduced:** an executable fixture demonstrated the stated outcome. Read its qualification.
- **Source:** implementation or documentation establishes the narrower observation; no physical failure measurement implied.
- **Recorded:** measurements committed in the repository, not a fresh benchmark on Rahul's machine.
- **Design:** a contract inconsistency or engineering judgment.
- **Opportunity:** a useful next experiment or improvement, not a demonstrated defect.

Priorities: **P0** = silent loss/integrity problem before relying on existing durability claims; **P1** = correctness, lifetime, or contract work before expanding the affected feature; **P2** = next diagnostics or usability work; **P3** = later optional breadth. These are project priorities, not CVSS ratings or claims of production incidents.

| ID | First evidence | Finding | Evidence | Priority and current status |
|---|---|---|---|---|
| SV-01 | R1 §1 | VRoidPreset A–Z is described as CC0 despite its actual terms | Primary-source mismatch; **asset-selection plan**, no shipped-asset violation established | P1 exact asset/source/terms record |
| SV-02 | R1 §2 | Full FF closure collapses to a neutral basic mouth output | Reproduced player arithmetic/renderer formula, not a fresh perceptual test | P1 distinguish bilabial closure from labiodental shaping |
| SV-03 | R1 §3 | Full specification is far ahead of a working conversation | Design and documented foundation state | P2 bounded evidence slice, then ordered v1 work |
| SV-04 | R1 §4 | Sentence-first synthesis confidence lacks a fair first-audible-output comparison | Recorded timings and design critique | P1 latency experiment; earlier 634–1,476 ms samples are not the final acceptance workload |
| SV-05 | R1 §5 | Cancelling Python await does not automatically stop native inference or free its slot | Design/native ownership concern | P1 enforce active-job ownership; experiment with per-run ORT termination |
| SV-06 | R1 §6 | Visible idle resource baseline exceeds shell CPU target | Recorded: 16.123% of one logical core; summed RSS caveat | P2 profiling; minimized behavior already good |
| SV-07 | R1 §7 | Language and rig generality exceed tested coverage | Design | P1 qualification matrix; capability report proposed in R4 §6 |
| SV-08 | R1 §8 | LipSyncLab is the strongest specific demo; current timing needs independent validation | Opportunity/design | P2 lab and reviewed landmarks; preserve corrected evidence |
| SV-09 | R2 V1 | Native model-phone alphabet has unmapped symbols; unknown phones become silence | Source and synthetic mapping check | P1 coverage/provenance; **not proof current raw-espeak speech is wrong** |
| SV-10 | R2 V2 | PP suppression is applied at several stages, producing excessive vowel attenuation | Reproduced synthetic generator→player→renderer arithmetic | P1 one owner for suppression/coarticulation |
| SV-11 | R2 V3 | Correct total duration cannot establish internal phoneme alignment | Design; reinforces SV-08 | P1 independent reference/visual test, not more self-consistency claims |
| SV-12 | R2 V4 | Kernel/provider teardown contract needs executable lifecycle proof | Opportunity; accepted small kernel is justified | P1 prove existing fences/credits; do not invent a larger framework |
| SV-13 | R2 V5 | Avatar polish cannot replace unfinished desktop behavior | Design; refinement of SV-03 | P2 adhere to Wayland/AEC/integration gates |
| SV-14 | R3 V3.1 | Per-sentence normalization discards absolute level; sub-PCM-quantization signal can animate a silent file | Reproduced pure envelope/WAV functions | P1 declare activity-vs-loudness semantics and use the played PCM |
| SV-15 | R3 V3.2 | Integer-ms segment duration conflicts with an arbitrary-provider one-sample accuracy guarantee | Design plus arithmetic fixture | P1 authoritative frame count; current 25 ms Kokoro hop is a qualification |
| SV-16 | R3 V3.3 | Development playback accepts inconsistent audio and performance durations | Reproduced with fake Audio APIs and deliberately mismatched input | P1 boundary validation; no healthy Kokoro drift established |
| SV-17 | R3 V3.4 | Scheduling time can lead audible device output | Source/spec plus synthetic clock fixture; **no hardware offset measurement** | P2 output-clock experiment |
| SV-18 | R4 V4.1 | Delayed startup fetch can overwrite a later manual avatar selection | **New controlled lifecycle reproduction** using actual source | P2 current spike UX; P1 before adopting loader in production |
| SV-19 | R4 V4.2 | Stale startup rejection overwrites the status for a successful newer avatar | **New controlled lifecycle reproduction** using actual fetch chain | P2 same ownership fix as SV-18 |
| SV-20 | R4 V4.3 | Candidate scene is not disposed if a preprocessing utility throws before installation | **New fault injection** with actual loader control flow | P1 transactional candidate cleanup; not a measured GPU leak |
| SV-21 | R4 V4.4 | Generated release keys can exceed the duration required by the planned protocol; player accepts them | **New synthetic duration fixture** using actual generator/player | P1 settle segment-boundary contract before production validation |
| SV-22 | R5 final | Phase-reset limiter renders 48 FPS at virtual 144 Hz while probe reports zero dropped frames | Reproduced actual renderer/probe with virtual clocks and mocked graphics | P2 diagnostics; P1 before FPS acceptance claims |
| SV-23 | R5 final | Managed-hints FIFO executes obsolete opens after latest close request | Reproduced actual main-process logic with mock helper/window; eventual close succeeds | P1 before production reuse; bounded desired-state ownership |
| SV-24 | R5 final | Literal phonemizer text is placed in CLI option position without stdin/terminator | Captured actual function argv plus official option-parser source; no eSpeak execution | P1 fixed configuration argv plus literal stdin |


## 3. Current corrections

Read these before acting on the historical chapters. The ledger/corrections govern finding facts; the scope section and §6 govern execution. Earlier chapters retain their original framing.

| Topic | Current recommendation |
| --- | --- |
| Svara kernel | The accepted emit/waterfall/serial starting point supports real teardown/provider-switch requirements. Keep it narrow and test it; the first-round blanket skepticism is refined. |
| Svara cancellation/credit | Generation fences, renderer spoken-word progress, and credit over generated/transferred/queued sentences already exist in the plan. Their implementation is unproven, not their existence. |
| Barge-in | Backchannel detection and resuming interrupted speech are explicitly outside F13. They are optional later work. |
| Reuse | A convenient wrapper that excludes Python 3.14, a PyTorch dependency, or an extra local service is not a drop-in under the accepted v1 constraints. |
| Avatar license | Select an exact artifact under its actual terms. A permissive preset is not automatically CC0. No production asset was established as shipped improperly. |
| Native timing | A sums-match check is an internal consistency check. It does not certify where audible consonants occur. |
| RSS/CPU | Summed RSS can double-count shared memory; the idle CPU figure is a fraction of one logical core. Neither establishes a memory leak. |

Minor credibility corrections retained from round 1: choose Google STT for measured utility rather than because it appears on a resume; the public prototype-tag claim lacked corresponding remote tags at that earlier check. Neither observation is a new check performed during this document split.

## 4. Round 4 findings

The avatar tests use the **actual `AvatarRenderer` control flow**, with TypeScript syntax stripped and rendering/import dependencies replaced by deterministic test doubles. Startup tests also execute the actual fetch chain extracted from `spike-app.tsx`. They establish asynchronous ordering and disposal-call behavior. They do not render pixels, load a real malformed VRM, measure GPU memory, or simulate the entire Electron app.

### V4.1 / SV-18 — A startup default can undo a manual selection

**Type:** controlled reproduction. **Priority:** P2 for the development spike; P1 before this lifecycle is reused in a product.

**Research/evidence:**

1. The mount-time default-avatar fetch is pending.
2. The user manually selects B. B loads successfully.
3. The earlier startup fetch completes and calls `loadVrm(A)`.
4. Because the renderer's generation is assigned at the loader call, A is considered newer. A replaces and disposes B.

```text
statuses: VRM loaded: manual-B
          Loading ignored test VRM…
          VRM loaded: auto-A
disposed: manual-B
```

**Roast:** your loader knows which request arrived last, but your application forgot which choice the user made last.

**Criticism:** the renderer's fence is local to `loadVrm`. The operation actually starts earlier at fetch/selection intent. The delayed fetch and manual selection do not share an ownership token, so a technically newer loader call can represent older user intent.

**Solution:** assign a selection generation at the beginning of the whole operation. Capture it before fetch, check it after fetch and blob conversion, and permit model installation/status changes only while it is current. A manual selection invalidates the startup-default operation. Abort the owned startup fetch where possible, while retaining identity checks for work that completes anyway. Keep per-operation object URLs and revoke exactly the ones that operation owns.

**Exit tests:** default fetch resolves before/after manual selection; manual B succeeds/fails; A resolves/rejects late; disposal happens during either request. Late A must not replace B, rewrite B's status, or revoke B's URL. Every abandoned candidate and URL must have one owner.

### V4.2 / SV-19 — An old failure can lie about the current avatar

**Type:** controlled reproduction. **Priority:** P2; same change package as SV-18.

**Research/evidence:** startup A begins loading; manual B then succeeds; A rejects afterward. `AvatarRenderer` correctly suppresses its stale `onError` callback, but rethrows the rejection. The startup fetch chain's outer catch checks only whether its AbortSignal was aborted. It then overwrites the visible model status:

```text
Loading ignored test VRM…
VRM loaded: newest-B
Ignored test VRM unavailable: Error: late failure A
```

The current model remains B. The UI reports an obsolete failure.

**Roast:** one component carefully ignores the stale error, and the next catch proudly puts it back on screen.

**Criticism:** status messages are also effects owned by an operation. Protecting only scene installation is insufficient. The manual `loadFile` catch already relies on the renderer's useful callback; it is specifically the automatic startup catch that escapes this ownership rule.

**Solution:** gate the outer catch by the same selection identity as the rest of the operation. Treat stale completion as stale, not a user-facing failure. Check cancellation and identity at every boundary that mutates state. Choose one layer responsible for publishing current-operation status.

**Exit tests:** the reproduction must end with B's loaded status. A's late rejection may be retained in diagnostic logs with its identity but must not change the current UI. Do not swallow failures of the genuinely current selection.

### V4.3 / SV-20 — Pre-installation failure drops candidate ownership

**Type:** injected utility failure. **Priority:** P1 loader lifetime hardening.

**Research/evidence:** after GLTFLoader returned a candidate, I made `VRMUtils.removeUnnecessaryVertices` throw. The method rejected, but `deepDispose(candidate.scene)` was never called:

```text
Optimization fault candidate disposal count: 0
```

The code explicitly disposes stale loads and non-VRM glTF. Its generic catch does not retain ownership of a candidate that fails preprocessing.

**Qualification:** the utility exception was injected. This proves a missing cleanup branch under that failure, not that a particular real VRM triggers it or that measured GPU memory rises by a specific amount. Some resources may not have reached GPU upload at this point.

**Roast:** you clean up the errors you anticipated by name. The unexpected one walks out carrying the scene.

**Criticism:** asynchronous acquisition needs candidate ownership from load completion until successful installation. A catch that merely reports the exception is not a rollback mechanism. Replacement should also preserve the previous usable avatar until the new candidate has passed preparation and qualification.

**Solution:**

1. Keep candidate ownership in an outer scope and distinguish preparation from committed installation.
2. Optimize, qualify, and calculate framing before releasing the prior model.
3. On pre-commit failure or stale completion, remove the candidate if attached and dispose it exactly once.
4. After commit, ownership moves to the renderer; disposal later belongs to replacement or renderer shutdown.
5. Decide separately how observer callback failures are reported. A callback throwing after installation must not accidentally dispose the now-active model as if installation had never happened.

**Exit tests:** inject failure at each preparation stage and during qualification/framing; assert candidate cleanup exactly once, previous model still usable, and replacement callbacks consistent. Then run real repeated-load tests with render-memory diagnostics. Mock disposal counts do not replace that resource measurement.

### What the renderer already gets right

Two newly executed tests passed:

- A and B are both already inside `loadVrm`; B completes first and A completes late. B remains installed and A's scene is disposed.
- The renderer is disposed while C is pending. C's late scene is disposed and never installed.

Keep those fences. SV-18 concerns the earlier application operation; it does not mean the renderer's existing stale-success guard is broken. The prior round's successful stale audio-decode and stale `onended` tests remain credited as earlier evidence.

**Source for V4.1–V4.3:** [AvatarRenderer](https://github.com/rahul-j0shi/svara/blob/f33c9943012d9dbbc331c4c8adea5c36a4cc09f2/packages/avatar/src/avatar-renderer.ts), [startup fetch and manual selection](https://github.com/rahul-j0shi/svara/blob/f33c9943012d9dbbc331c4c8adea5c36a4cc09f2/apps/desktop/src/spike-app.tsx).

### V4.4 / SV-21 — Release keys can outlive their segment contract

**Type:** synthetic model-duration fixture; actual generator and actual pure player. **Priority:** P1 before implementing strict production segment validation.

**Research/evidence:** `build_viseme_keys` adds an 80 ms release after each hold, with no segment-duration parameter. The protocol plan requires time values within segment duration. The current player's validation does not reject a key after `durationMs`.

I supplied one audible vowel with four 25 ms duration frames, a zero-frame starting pad, and varied the final padding. These are accepted finite, non-negative inputs to `build_timeline`:

| Final pad frames | Audio/track duration | Latest generated viseme key | Keys beyond duration |
|---:|---:|---:|---:|
| 0 | 100 ms | 180 ms | 1 |
| 1 | 125 ms | 180 ms | 1 |
| 4 | 200 ms | 180 ms | 0 |

The actual player accepted the 100 ms track containing the 180 ms release key. At 99 ms its `aa` weight was 0.9; at 100 ms playback ended and returned all-zero weights. The planned release was never sampled.

**Qualification:** these are synthetic duration outputs, not inference results from the committed five-sentence corpus. The native model's real padding distribution was not measured in this round. Immediate neutral at utterance end may itself be an acceptable policy. The demonstrated problem is an inconsistent generator/player/planned-protocol contract, not proof of a visually bad transition in every real utterance.

**Roast:** the generator schedules the mouth to relax after the player has already gone home.

**Criticism:** articulation tails and PCM ownership need an explicit boundary policy. A future strict receiver cannot simply accept the current generator and enforce the planned rules without potentially rejecting such tracks. Consecutive segments and final utterance completion also need different treatment if smooth continuation is desired.

**Solution:**

1. Make PCM frame count and sample rate authoritative, as already recommended for SV-15.
2. Pass the derived segment extent into curve generation and choose a documented policy for attack/hold/release at that extent.
3. For strictly segment-local tracks, emit no key outside the interval. Preserve the curve's sampled boundary value where necessary; do not merely drop a distant final-zero key and silently alter interpolation.
4. For continuous articulation across consecutive segments, carry explicit boundary state into the next segment while keeping each segment's times valid. Reserve utterance-end neutralization for a declared end, rather than pretending every sentence is isolated. This is a policy option, not an unimplemented v1 defect by itself.
5. Enforce the chosen policy in the generator, schema/adapter validation, and player. Keep validation transactional so rejected input cannot partially replace accepted state.

Do not lengthen silent PCM by 80 ms just to make the animation parser happy without measuring the effect on sentence gaps and latency.

**Exit tests:** zero/short/long final pad, final bilabial closure, zero-duration tokens, empty/silent input, and consecutive segments. Every emitted key is inside the declared extent or uses an explicitly versioned cross-segment representation. Test the entire generator→validator→player path so individually passing layers cannot contradict one another.

**Source:** [native timeline and viseme generator](https://github.com/rahul-j0shi/svara/blob/f33c9943012d9dbbc331c4c8adea5c36a4cc09f2/spikes/kokoro/src/kokoro_spike.py), [PerformancePlayer](https://github.com/rahul-j0shi/svara/blob/f33c9943012d9dbbc331c4c8adea5c36a4cc09f2/packages/avatar/src/performance-player.ts), [protocol contract](https://github.com/rahul-j0shi/svara/blob/f33c9943012d9dbbc331c4c8adea5c36a4cc09f2/docs/revamp/13-implementation-guide.md).

## 5. Research and reuse decisions

Primary sources in this section were checked during round 4 on 8 October 2026. Earlier component research retains its own verification date in the historical chapters. This is dated research from the split, not a current shopping list. The scope edit adds no web research or component installation.

### Use the avatar library's existing introspection before building more custom rig plumbing

The current three-vrm API exposes expression maps and `getExpression(name)`, which returns null for an absent expression. Each expression exposes its bindings, binary behavior, and mouth override setting. VRM's expression design deliberately permits emotion expressions to alter procedural mouth output.

That matters because setting `aa` to a number does not prove the asset has a useful continuous `aa` shape, that the shape has bindings, or that another expression is not suppressing it. A binary expression changes continuous weights into a thresholded result. An emotion/animation can alter the mouth even when Svara's own arithmetic looks right.

**Recommendation:** use these existing APIs to qualify the single selected lab asset; broader rig coverage is deferred. This makes the already planned T2.4 metadata/retarget verification concrete; it is not a claim that T2.4 has been completed incorrectly.

| Check | What to record | Consequence |
|---|---|---|
| Format and identity | VRM version, asset hash, exact source, export version, profile ID | Reproducible choice; explicit compatibility |
| Humanoid | Required bones/usable head node | Supported pose/framing or a clear rejection |
| Basic mouth presets | Presence and nonempty binds for `aa`, `ee`, `ih`, `oh`, `ou` | Choose basic profile or report degraded capability |
| Continuous behavior | `isBinary` and a 0/0.25/0.5/0.75/1 pose sweep | Identify thresholded or visually ineffective shapes |
| Extended visemes | Actual custom expression names and bindings | Enable extended profile only when real shapes exist |
| Expression interaction | `overrideMouth` settings; mouth motion under intended emotions/animations | Avoid unexplained suppression/overshoot |
| Resources | Geometry/material/texture counts and replacement-cycle measurements | An actual resource budget rather than a filename-based assumption |

Use a free existing avatar first; qualify it. Do not buy or hand-sculpt fifteen consonant shapes just because the internal enum has fifteen labels. If a basic rig cannot distinguish FF/TH/PP, publish the approximation and compare alternatives on the same test corpus.

Sources: [three-vrm expression manager](https://pixiv.github.io/three-vrm/docs/classes/three-vrm-core.VRMExpressionManager.html), [expression properties](https://pixiv.github.io/three-vrm/docs/classes/three-vrm-core.VRMExpression.html), [VRM expression tutorial](https://vrm.dev/univrm1/vrm1_tutorial/expression/).

### Use an existing glTF validator for container checks; keep Svara-specific checks

[Khronos glTF Validator](https://github.com/KhronosGroup/glTF-Validator/blob/main/README.md) checks glTF 2.0/GLB structure, references, accessors, and supported extensions, and produces JSON issues and asset statistics. It has CLI/npm forms and a browser tool that processes locally. Reuse it as an asset-preparation or CI tool instead of implementing another container validator.

Its published supported-extension list does not list `VRMC_vrm`. Therefore a glTF validation pass is **not VRM qualification**, nor a license check, nor proof of useful visemes. Follow it with the small Svara capability checks above. No validator package was installed or run during this review. Avoid adding it to the runtime path merely because a developer validation step is useful.

### The latest renderer release is not a solution to these races

The official three-vrm releases page marks **3.5.5** as the latest stable release and also lists **3.6.0-beta.0**, adding spring-bone-limit support. Svara already pins **3.5.5**.

There is no evidence here that a version upgrade fixes application selection ownership, candidate rollback, or segment timing. Do not chase the beta as a substitute for these fixes. Evaluate a future upgrade against actual asset needs and regression tests.

Source: [official releases](https://github.com/pixiv/three-vrm/releases); repository `packages/avatar/package.json`.

### Keep earlier market comparisons bounded

**Parked research registry, not an active backlog.** V-D.2 selects at most one compatible baseline. Other candidates remain deferred unless a named unmet requirement promotes one; existing v1 provider obligations are handled by the repository plan.

| Candidate/group | Decision retained from prior research | Next evidence required |
|---|---|---|
| HeadTTS browser path | First comparison for the native-timing runner; not an automatic migration | Same corpus/voice/model where possible, timing quality, first PCM, resource cost, cancellation/ownership |
| three-vrm-lip-sync / HeadAudio | Audio-driven fallback baselines | Same rig; visible closures/transitions; documented latency/calibration and dependency fit |
| TalkingHead | Alternate rig/framework experiment | Account for GLB/ARKit/Oculus rig differences and conversion work |
| kokoro-onnx / Pipecat Kokoro | Not a drop-in under the previously checked Python/ORT pins | A current compatible resolver result and preserved native timing contract |
| Pocket TTS | Isolated latency comparison only under current v1 constraints | Deliberate PyTorch scope change before adoption; verify timing and artifact terms |
| Moonshine Voice | STT comparison candidate | Python 3.14 packaging, accent/entity accuracy, finalization delay, concurrent TTS cost, exact model terms |
| Montreal Forced Aligner | Offline evaluation reference | Appropriate model/dictionary plus reviewed anchors; not runtime plumbing |
| Supertonic archive | Optional comparison, not the obvious maintained successor | Maintenance and actual model/timing/runtime fit |
| ORT termination/quantization | Focused native-run experiments | Per-job termination behavior and measured latency/quality on the actual host |

This table retains previously checked decisions; those projects were not all refreshed in round 4. Full primary links and qualifications remain in the archived reports. Recheck compatibility when selecting a component for implementation.

## 6. Prioritized implementation plan

This is the **single current execution plan**. V-A–V-F retain their earlier package identities but are split below. These are recommended work units, not reports of completed implementation. Lab order and product dependency order are deliberately separate.

### Start here: one trustworthy performance path

| Work unit | Findings / dependency | Included work | Excluded work | Done / stop condition |
|---|---|---|---|---|
| V-A.1: final mouth-pose contract | SV-02, SV-10; first lab unit | Choose one owner for closure/coarticulation. Inspect the final basic-rig weights, including full FF and PP→vowel. State approximations for absent consonant shapes. | New rig art, extra providers, universal phoneme claims | Known arithmetic regressions and valid vowel controls pass through the final mapping. A selected real-rig sweep separately demonstrates appearance; arithmetic alone is labeled accordingly. |
| V-A.2: canonical audio/performance boundary | SV-14, SV-15, SV-16, SV-21; before lab comparisons | Make PCM frames/rate authoritative; state envelope semantics; validate audio/track pairing and in-range keys before starting playback. Choose one segment-tail policy. | Gapless utterance redesign or cross-segment articulation unless the existing acceptance contract requires it | Silent played PCM stays neutral; fractional-ms, mismatched audio, and short-pad cases behave as specified; matching input still plays. No self-consistency-to-accuracy leap. |
| V-A.3: literal text and declared phone inventory | SV-09, SV-24; before new synthesis inputs | Fixed configuration argv plus literal stdin; explicit unknown/pause handling and word provenance for the chosen provider inventory. | Multilingual G2P, changing model/voice, installing a phonetics framework | Argument-capture controls and actual pinned-binary corpus pass; each chosen inventory symbol is mapped or explicitly unsupported. No claim that synthetic native phones prove today's raw-English speech wrong. |

V-A.3's small literal-text repair can be separate from the inventory work. V-A.1 and V-A.2 also deserve separate regressions even if a shared patch changes both. This prevents an eight-finding package from becoming an all-or-nothing rewrite.

### Qualify the asset and protect intent

| Work unit | Findings / prerequisite | Included work and boundary | Completion evidence |
|---|---|---|---|
| V-B.1: one asset/profile qualification | SV-01, SV-07 | Record one exact free asset/hash/source/terms; use existing library introspection; sweep basic mouth presets and intended expression interactions. Fix the misleading CC0 plan wording. | Exact asset report, 0/0.25/0.5/0.75/1 sweeps and known limitations. If the first asset fails, evaluate one replacement; further searching needs a named missing capability. No custom fifteen-shape requirement. |
| V-B.2: avatar-operation ownership | SV-18, SV-19, SV-20 | Assign identity at selection intent; gate install/status; own URLs and candidates through prepare/commit/rollback. | Late default/success/error, preprocessing failure, replacement and disposal schedules preserve the current avatar/status and dispose owned abandoned candidates once. Real repeated-load resources remain a separate measurement. |
| V-E.1: replaceable focus intent | SV-23; standalone spike fix, before production reuse | One active helper plus latest pending desired state; suppress stale focus and reconcile native hints. Retain sender validation. | Controlled toggle/error/disposal cases prove bounded obsolete work and final desired state. Actual window-manager behavior is verified under platform gates; a VM pass is not a desktop pass. |
| V-E.2: pacing and metric definitions | SV-22, relevant SV-06 | Preserve target phase; count target deficit separately from callback availability; define visibility/window/mode behavior. | Virtual 50/60/90/120/144/165/240 Hz and stall cases pass without catch-up bursts. Recheck real CPU/PSS/cadence; logical success does not establish the resource budget. |

V-B.2/V-E.1/V-E.2 can be independent small fixes. They are not prerequisites for a headless arithmetic fixture. They are prerequisites when the lab/demo uses the affected loader, focus mode, or FPS metric. Do not add a common scheduler framework just because both ownership paths need generation checks.

### One bounded lab comparison, then stop and decide

| Work unit | Findings / prerequisite | Included experiment | Completion / decision |
|---|---|---|---|
| V-D.1: independent landmarks and output clock | SV-08, SV-11, SV-17; V-A contracts plus qualified rig for visual claims | Frozen short corpus, reviewed acoustic/visible landmarks, graph/output/display clock distinction, one named host/output device. | Trace/export plus uncertainty and device details; fallback behavior recorded. No requirement for an offline aligner or perfect ground truth. Unavailable hardware leaves that claim unverified. |
| V-D.2: one reuse comparison | SV-04 and lab evidence; after V-D.1 | Current native path versus **one** compatible reused baseline. HeadTTS browser mode is the first compatibility check; if unsuitable, choose one audio-driven baseline instead. | Compatibility decision and corpus comparison of first audible output, alignment, resources and ownership. Retain/adapt/replace/defer with reasons; no second candidate unless the first has a named failure or leaves a material decision unanswered. |
| V-C.1: native ownership before conversation reuse | SV-05, SV-12; selected runtime/provider | Enforce active-job ownership through native exit, existing credit and result fences. | Repeated cancel/replacement histories show no stale playback and no slot freed before exit. A Python-await cancellation result alone is insufficient. |
| V-C.2: optional termination experiment | SV-05; only if V-C.1's bounded completion misses an existing target | Compare per-run ORT termination with bounded completion on the actual model. | Stop-request/native-exit/replacement-start trace and reuse tests; keep fences. Inconclusive/no-gain is a valid experiment result, not a feature pass. |

**Proposed lab cap:** one pinned model/voice, one basic rig, one English corpus and one baseline. A starter corpus can be eight short utterances covering PP/FF, vowel transitions, pauses, short tails and leading-dash literal text, plus separate quiet/silent PCM controls; freeze text/fixtures before comparison. This corpus is proposed, not already run, and does not replace the repository's corrected five-sentence visual acceptance rerun. Record both where applicable. Match voices/models when possible; record confounders when not. Choose and record cold/warm repeat counts before running the comparison; report small-sample results as descriptive and do not infer reliable p95 or acceptance of the repository's latency target.

**Minimum useful portfolio stop:** V-A contract regressions, V-B.1's exact rig qualification, relevant ownership fixes, and one reviewed corpus replay with timing/export and stated limitations. A reuse decision makes the artifact stronger; failure to find a compatible baseline should be documented rather than trigger a stack rewrite. Do not call this artifact an accepted conversational assistant, and do not wait for every v1 feature to publish the lab honestly.

### Product route: early feasibility, existing dependency order

| Product checkpoint | Scope and evidence | Boundary |
|---|---|---|
| Foundation feasibility now | Keep T0.8's timing evidence, T0.9 runtime smoke, T0.10 presence/platform/AEC-transport/budget checks, T0.12 exact asset and T0.13 foundation exit in their existing dependency order. SV-03/SV-06/SV-13 explain why these matter. | Do not postpone feasibility until after a polished conversation. Pure lab work can continue independently; failed/unavailable gates block dependent production acceptance. The review does not execute reset/deletion tasks in Phase 0. |
| Text desktop, then voice | Follow Phase 1 → Phase 2 including T2.11 → Phase 3 including T3.15, with the implementation guide's stated task dependencies/pull-forwards. Carry relevant regressions from V-A/V-B/V-C/V-E into owning tasks. | The review's work units are not substitutes for kernel/protocol/desktop/STT/voice integration tasks. Headphones or push-to-talk do not accept the speaker/AEC requirement. |
| Preview/full v1 | Use the existing preview cut and final F01–F40/cross-feature/platform/budget/licensing criteria. | Required provider support stays required; additional providers beyond that requirement are V-F optional breadth. No platform threshold is relaxed here. |
| V-F: optional breadth | Extra rigs/languages/providers, quantization, richer interruption and alternative frameworks only for a measured need. | A separate bounded proposal; not part of the immediate lab and not a prerequisite for its case study. |

**Next session:** V-A.1 only: make the PP/FF final-weight regression explicit and settle one shaping owner. If continuing the product route, investigate still-open Phase 0 gates alongside independent lab work; do not advance dependent production acceptance on mock evidence.

**Status update format:** work unit → affected finding IDs/repository task → commit or experiment inputs → actual result → remaining limitation. Distinguish fixed code, decided contract, qualified asset, measured device and accepted release. Unknown remains unknown. A new candidate/tool enters active work only when it answers a named unmet requirement and replaces or extends one existing unit.


## 7. Hiring signal

**Svara:** a qualified free avatar, explicit five-vowel limitations, a fixed closure/suppression path, owned asynchronous work, and a timing lab that compares native and reused approaches is stronger than another polished character with unexplained mouth motion.

The strong story is: intent → small counterexample → principled change → regression → remaining limits. A longer roadmap or technology list cannot substitute for completing that loop.

## 8. Verification and limits

These are results from the fourth review, carried forward here. No tests were rerun merely to split the documents.

| Round-4 check | Actual result | What it establishes |
| --- | --- | --- |
| Fresh repository fetch/HEAD comparison and status | Svara unchanged; clean | Findings refer to the stated commits; no fixes occurred in this review |
| Avatar startup default after manual B | Default A replaced B | Whole-operation selection race under controlled ordering |
| Avatar stale startup rejection after B | Old failure overwrote status | Outer catch lacks current-selection ownership |
| Renderer-only stale success | Passed; late A disposed, B retained | Existing local generation fence works |
| Renderer disposed while load pending | Passed; late candidate disposed | Existing disposal fence works |
| Injected VRM optimization failure | Candidate disposal count zero | Missing pre-commit cleanup branch under injected failure |
| Generator short-pad fixtures | 180 ms key for 100/125 ms tracks | Accepted generator inputs can violate planned extent constraint |
| Actual PerformancePlayer on generated 100 ms track | Accepted; aa 0.9 at 99 ms, neutral at 100 ms | Player accepts distant release key and ends before sampling it |
| Primary-source research | Official VRM/three-vrm/Khronos sources inspected | Existing reuse mechanisms and their documented limitations |

Round-4 environment: Node 24.19.0 for the actual pure player and dependency-mocked avatar lifecycle, plus Python/NumPy for source-extracted timeline functions. TypeScript imports were redirected only in the scratch harness. No full dependency install, fresh ONNX inference/termination benchmark, real VRM render, microphone/cloud use, PipeWire/AEC session, target-Ubuntu desktop run, or hardware output-latency measurement occurred. Mock disposal calls do not establish GPU-memory behavior. Synthetic timing/PCM inputs do not establish corpus-wide perceptual accuracy.

Repository source was not edited. For future ledger updates use **ID → fixing commit → regression command → actual output → remaining limitation**. Mark fixed only when the affected behavior changes and the relevant regression passes.

## 9. Historical project reviews

The chapters below preserve the substantive project material from rounds 1–3, including original criticism, solutions, links, and available reproductions. Mixed tables are scoped to this project; unrelated project chapters are omitted. Use the scope section/§6 for execution and sections 2–3 for finding facts. Historical action lists are not extra active commitments. These chapters retain their original verification dates and historical numbering.

### Round 1

Evidence date: 8 October 2026. Review only. The project intent and all current qualifications above apply.

#### Svara: the roast, with engineering consequences

##### 1. The asset plan contains a concrete licence error
**Priority: immediate documentation correction. Evidence: primary-source mismatch.**

The avatar document says VRoidPreset_A–Z are CC0. The specific official [preset terms](https://vroid.pixiv.help/hc/en-us/articles/4402394424089-VRoidPreset-A-Z) explicitly say they are **not CC0**. Their permissive conditions allow many uses, but they prohibit redistributing them under a CC0 declaration, among other restrictions.

VRoid also has models actually designated CC0, such as [beta AvatarSample_1](https://vroid.pixiv.help/hc/en-us/articles/360012381793-%CE%B2-Ver-AvatarSample-1). The source and exact model matter.

**Roast:** licence metadata does not become correct because an architecture document says it confidently.

**Do next:** choose one exact model, retain its actual source/terms and record its export/metadata. Either use the preset under its real conditions or a verified CC0 asset. Keep the existing free-avatar strategy; it is already the right way to avoid unnecessary art work. I found a bad plan claim, not proof that a production asset has already been unlawfully distributed.

[Avatar plan](https://github.com/rahul-j0shi/svara/blob/f33c9943012d9dbbc331c4c8adea5c36a4cc09f2/docs/revamp/03-avatar-and-lipsync.md)

##### 2. Your FF viseme vanishes in the basic renderer
**Priority: before more lip-sync polish. Evidence: reproduced calculation using the current player and renderer mapping.**

PerformancePlayer uses max(PP, FF) as a closure weight and multiplies jaw opening by one minus that value. AvatarRenderer maps FF to ih × 0.2, then multiplies that result by the already suppressed jaw.

With a maximum envelope and only FF active:

| Input FF | Player jaw | Final basic ih weight |
|---:|---:|---:|
| 0.9 | 0.1 | 0.018 |
| 1.0 | 0 | **0** |

This reproduces the mathematical path; it is not a new rendered perceptual test. At full FF, the planned lower-lip/teeth distinction becomes a neutral closed-mouth output on the basic rig.

The anatomy matters: /p b m/ involve both lips; /f v/ involve the lower lip and upper teeth. See [University of Sheffield articulatory phonetics](https://sheffield.ac.uk/linguistics/home/all-about-linguistics/about-website/branches-linguistics/phonetics/what-do-phoneticians-study/articulatory).

**Do next:** separate bilabial closure from labiodental shaping and apply amplitude/coarticulation constraints in one well-defined place. A five-vowel VRM rig is an approximation; state its capabilities instead of suggesting the 15 internal labels create 15 visible shapes.

Compare the existing Papa and Five velvet flowers fixtures on the actual rig. The plateau unit tests currently prove curve construction, not the final mouth pose.

[Player](https://github.com/rahul-j0shi/svara/blob/f33c9943012d9dbbc331c4c8adea5c36a4cc09f2/packages/avatar/src/performance-player.ts) · [Renderer](https://github.com/rahul-j0shi/svara/blob/f33c9943012d9dbbc331c4c8adea5c36a4cc09f2/packages/avatar/src/avatar-renderer.ts)

##### 3. The specification is ahead of the basic conversation by an uncomfortable distance
**Evidence: design criticism, with explicit current-state documentation.**

The plan has 40 features, 114 acceptance criteria and 89 tasks. The handoff says the Python core is scaffolding and there is no product command, agent loop or real conversation yet.

The planned custom plugin kernel has legitimate requirements: feature teardown, provider replacement, live tools and scoped effects. Nevertheless, the assertion that a roughly 300-line kernel will be great in interviews comes before evidence that it makes the first conversation easier.

**Roast:** interview appeal is being used as an architecture input. The first useful interaction is still waiting behind your infrastructure curriculum.

**Do next:** separate an experimental speech/avatar slice from the full desktop release. Start with fixed composition, scoped cleanup and one provider per stage. Preserve the dynamic-kernel design as an experiment when an actual toggle/provider-switch scenario proves the need.

This is a recommendation to revise the plan deliberately, not silently bypass its documented Wayland release gates.

[Current-state handoff](https://github.com/rahul-j0shi/svara/blob/f33c9943012d9dbbc331c4c8adea5c36a4cc09f2/docs/revamp/15-developer-handoff.md) · [Kernel design](https://github.com/rahul-j0shi/svara/blob/f33c9943012d9dbbc331c4c8adea5c36a4cc09f2/docs/revamp/01-architecture.md)

##### 4. Current measurements challenge the chosen playback strategy
**Evidence: committed measurements and design criticism.**

The four-thread Kokoro follow-up reports **634–1,476 ms** inference across five cached-phoneme fixtures. The eventual target is 400 ms, with a separate prescribed workload. These samples are not that final benchmark, but they are enough to challenge confidence.

The design waits for a complete synthesized sentence before playback and asserts sub-sentence streaming saves only roughly 100–300 ms. I found no measured comparison establishing that assertion for this pipeline and hardware.

**Do next:** compare actual time to first audible output, cold/warm loads, segment gaps and interruption recovery. Test a shorter first sentence, alternate quantization/runtime and a streaming TTS baseline. Measure the complete STT/LLM/TTS/render workload under contention.

Do not make a <2-second conversational claim from individually measured stages. The pipeline document already recognizes this and calls for end-to-end measurement; implement that experiment.

[Timing evidence](https://github.com/rahul-j0shi/svara/blob/f33c9943012d9dbbc331c4c8adea5c36a4cc09f2/docs/spikes/T0.8-kokoro-lipsync.md) · [Voice design](https://github.com/rahul-j0shi/svara/blob/f33c9943012d9dbbc331c4c8adea5c36a4cc09f2/docs/revamp/04-voice-pipeline.md)

##### 5. Task cancellation does not give native inference back
**Evidence: design mismatch, not a reproduced concurrency defect.**

The architecture correctly notes that cancelling a TaskGroup cannot stop an already running inference thread. It then bounds the wasted work at a few hundred milliseconds. Current Kokoro measurements extend beyond a second.

A queue bounded by two sentences also does not automatically bound all submitted executor work across rapidly cancelled turns.

**Do next:** fence every result by turn/generation, cap submitted work as well as active work, and measure repeated interruptions. Ensure stale audio cannot play and a new turn does not wait behind a pile of obsolete inference. Keep one in-flight job per model as an enforced contract rather than a comment.

The existing spike playback already has useful ownership checks after asynchronous decoding, and stale VRM loads are disposed. Preserve those patterns.

##### 6. The always-present avatar currently misses its own idle CPU budget
**Evidence: repository measurement, not a new benchmark.**

The five-minute diagnostic baseline reports **16.123% of one logical core**, versus a <5% shell target, and about 775 MiB median summed RSS. That CPU is not 16% of the whole machine; summed RSS can double-count shared pages. The report explicitly explains both.

The renderer continually animates and submits 30 idle frames per second, requests high-performance GPU preference, enables antialiasing and allows pixel ratio up to two. These are candidates for profiling, not proof that any one setting causes the whole cost.

**Roast:** an assistant that decorates your desktop should not quietly spend its resource budget on decorative motion.

**Do next:** compare identical assets/settings while varying idle frame rate, resolution, spring-bone updates and render-on-change/sparse idle behavior. Measure CPU and GPU/frame work separately and observe sustained memory across several replacement cycles. Keep natural movement only where its visible benefit earns the cost.

The minimized baseline already falls to 0.033% of one core. Preserve that successful behavior. Do not proclaim a memory leak from one higher RSS reading.

[Resource evidence](https://github.com/rahul-j0shi/svara/blob/f33c9943012d9dbbc331c4c8adea5c36a4cc09f2/docs/spikes/T0.10-ubuntu-integration.md)

##### 7. Universal language and universal rig support are stated too easily
**Evidence: design criticism.**

An IPA mapping provides a useful common representation. It does not prove that every language's phoneme inventory, tokenizer, voice pronunciation and coarticulation behave correctly. Similarly, adding a retarget JSON does not automatically make an arbitrary ARKit GLB load and animate like a VRM avatar.

**Do next:** publish a tested language/voice/rig matrix. Label unsupported or approximate sounds. Keep the five-vowel profile as a useful baseline and make expanded consonant shapes optional. For a second language, test actual speech, punctuation, numbers, names and code-switching—not only whether a phoneme appears in a mapping table.

##### 8. The best demo is buried inside the future product
**Evidence: design criticism.**

LipSyncLab—a waveform, phoneme, viseme and rendered-mouth timeline—is a stronger demonstration of your engineering than another generic talking-avatar chat.

Duration sums matching waveform length are useful but are not independent phoneme ground truth. The foundation audit already corrected the earlier arithmetic onset claim; that honesty is good. Complete the visible test rather than turning that corrected claim into a new headline.

**Do next:** pull a small standalone lab forward: play a fixed corpus, switch native-timing and audio-driven baselines, scrub the timeline, inspect PP/FF, and export a comparison. Use manually reviewed anchors or an independent alignment reference for a subset, clearly stating uncertainty. Do not validate a model's timing solely against timing generated by the same model.

Also correct two small credibility distractions:
- Google STT is described as retained because it is on the resume. Choose providers for measured usefulness; let the resume follow the implementation.
- README says v0-prototype is preserved as a Git tag; the public remote returned no tags during this review. Publish that tag or correct the statement.

#### Current market options: what could actually save work
Primary documentation checked on 8 October 2026. Free code is not a guarantee of free hosting, compatible asset licences, accurate lip sync or easy integration.

| Candidate | Verified usefulness | Limitation that matters for Svara | Recommendation |
|---|---|---|---|
| [HeadTTS](https://github.com/met4citizen/HeadTTS) | MIT JavaScript TTS returning audio, phoneme timestamps and Oculus visemes; browser or Node backend; uses the timestamped Kokoro ONNX family already chosen here | English only in its current documentation; WebGPU availability, slower WASM fallback, model memory and different G2P/voice paths matter | **First comparison for the custom Kokoro runner.** Same voice/model pin where possible; adapt its output to Svara's clock/segment contract before deciding |
| [three-vrm-lip-sync](https://github.com/vlapky/three-vrm-lip-sync) | MIT ready-made audio/stream-to-VRM mouth animation; AudioWorklet MFCC vowel classification; can reuse AudioContext | Writes five vowel expressions, not exact timed consonants; calibration and maintenance/peer-version compatibility need checking | **Best first candidate for the audio-only fallback.** Compare it with the custom native-timing path on the same rig |
| [HeadAudio](https://github.com/met4citizen/HeadAudio) | MIT browser AudioWorklet producing Oculus-style viseme values without transcripts/timestamps | Author acknowledges accuracy/VAD weaknesses; its documented compensation adds 50–100 ms; needs VRM mapping | Useful second audio-driven baseline, not a substitute for proven phoneme alignment |
| [TalkingHead](https://github.com/met4citizen/TalkingHead) | Existing avatar speech, expressions and animation stack | Expects Mixamo-compatible GLB plus ARKit/Oculus shapes; **not a drop-in five-vowel VRM replacement** | Compare for a product-first alternate branch; account for rig conversion and lost learning before adopting |
| [Pocket TTS](https://github.com/kyutai-labs/pocket-tts) | CPU-oriented streaming TTS; current README supports Python 3.14 and seven languages | PyTorch dependency; its first-chunk figures are vendor measurements on different hardware; I have not verified a native per-phoneme timing interface; voice/model licences require their own check | Benchmark as a latency candidate with an audio-driven fallback. Do not assume streaming audio solves timed visemes |
| [Supertonic archive](https://github.com/supertone-oss-archive/supertonic) | Preserved ONNX local TTS; code MIT, model OpenRAIL-M | Official notice ends further development/support; all listed versions archived; no verified drop-in phoneme-timing contract here | Optional benchmark only. **Do not sell it as the obvious latest maintained replacement** |
| [VRoid preset terms](https://vroid.pixiv.help/hc/en-us/articles/4402394424089-VRoidPreset-A-Z) / [actual CC0 beta model](https://vroid.pixiv.help/hc/en-us/articles/360012381793-%CE%B2-Ver-AvatarSample-1) | Existing usable avatar assets; no custom character needed for the first slice | Specific terms vary; rig and metadata still need inspection | Keep the reuse strategy; fix the exact model/licence record |

The recommendation is not to replace Python or the whole avatar architecture because a JavaScript library exists. Python remains useful for the current STT/model ecosystem. A small adapter or controlled comparison is enough to discover whether the custom runner earns its maintenance.

##### Own versus reuse
**Keep your own:** the conversation/turn lifecycle, audio-clock scheduling, cancellation ownership, capability-aware retargeting, timing diagnostics and deliberate phoneme/coarticulation experiments.

**Reuse first:** default avatar creation, VRM loading, TTS inference/phonemization where an adequate maintained option exists, and the audio-only fallback.

**Defer until demonstrated:** an expanded 15-shape art workflow, inventory/customization breadth, multiple redundant providers and full dynamic plugin semantics.

Building a small mechanism to understand it remains valid. State the hypothesis and compare it with an existing baseline. That turns reinvention into an experiment rather than unnecessary plumbing.

#### A compact next-work sequence

| Order | Svara |
| --- | --- |
| 1 | Correct exact asset licensing and PP/FF handling; state the basic rig's limits |
| 2 | Build the small speech/avatar lab and compare current runner with HeadTTS plus an audio fallback |
| 3 | Measure first-audio latency, stale-work behavior and idle resources on the target host |
| 4 | Deliver one useful conversational loop, then complete the Wayland/AEC/release requirements deliberately |

Svara's first useful loop can use push-to-talk/headphones while speaker barge-in/AEC remains an explicitly separate experiment. This would be a revised milestone, not a claim that its existing speaker/AEC acceptance criteria have passed. PipeWire transport working is not acoustic cancellation working.

For Svara: a conversation that can be interrupted correctly, a mouth whose timing and rig limitations you can demonstrate, and a measured decision explaining why you reused one component and owned another.

### Round 2

Evidence date: 8 October 2026. The repository was unchanged from round 1. These are historical reproductions/source checks, not fixes.

##### Corrections and refinements to iteration 1

| Earlier emphasis | More accurate assessment after this pass |
| --- | --- |
| Replace the custom TTS runner with a convenient wrapper | The Python 3.14 compatibility objection is real. Reuse must satisfy the runtime and alignment contracts, not just produce speech. |
| The plugin kernel is unnecessary architecture | Provider swaps, toggles, and reversible effects justify a small lifecycle mechanism. The accepted ADR already says start with three dispatch modes and expand only for real uses. Prove that small mechanism. |
| Cancellation fencing and bounded synthesis are missing | The integration plan already specifies generation checks, renderer progress, and credit covering generated/transferred/queued sentences. Missing implementation evidence is different from a missing contract. |
| Backchannel handling should be part of barge-in | F13 explicitly excludes backchannel detection and resuming interrupted speech. More advanced behavior is an optional future tradeoff, not a current acceptance defect. |
| Use Pocket TTS or a local HeadTTS server | Svara explicitly excludes PyTorch and extra local services in v1. Such configurations belong in isolated comparisons unless you deliberately revise scope. |


#### 3. Svara: the roast, with receipts

**The avatar can receive valid timing, valid viseme keys, and a valid frame—and still produce the wrong mouth because each layer independently decides how strongly to close it.** That boundary is more valuable to solve than inventing another agent abstraction.

##### V1 — “IPA” is hiding a model-specific alphabet contract

**Priority: first alongside V2. Evidence: reproduced pure-function fixture plus upstream inventory.**

The current spike invokes eSpeak IPA, filters characters through the model vocabulary, and maps each surviving character through a generic viseme table. Unknown viseme symbols fall back to sil.

The pinned model vocabulary includes A, I, O, W, Y, and Q. [Misaki's native inventory](https://github.com/hexgrad/misaki/blob/main/EN_PHONES.md) uses these as model-specific vowel/diphthong symbols. Its [eSpeak adapter](https://github.com/hexgrad/misaki/blob/main/misaki/espeak.py) includes conversions that raw character filtering does not perform.

Feeding these symbols through Svara's actual timeline builder:

~~~
[('A', 'sil'), ('I', 'sil'), ('O', 'sil'),
 ('W', 'sil'), ('Y', 'sil'), ('Q', 'sil')]
~~~

**Important limit:** today's hardcoded raw-English eSpeak path does not normally emit those uppercase native symbols. I did not prove that its current spoken pronunciation is wrong. The maintained kokoro-onnx wrapper also uses raw eSpeak/filtering. The demonstrated defect is incomplete mapping for native-provider input and silent treatment of unsupported symbols.

An affricate fixture likewise shows character splitting/filtering rather than an explicit model-normalization contract. That is a warning to test the frontend, not independent evidence that a real word sounded wrong.

**Roast:** you call everything IPA, then quietly translate “I don't understand this symbol” into “the speaker is silent.” That hides the precise failure a debugging lab should reveal.

Fix the abstraction:

- Declare the alphabet and normalization version per adapter.
- Distinguish actual pause, unsupported phone, and best-effort mouth shape.
- Preserve original text → normalized spoken text → tokens → sample intervals.
- Record unknown-symbol counts and explicit fallback provenance.
- Test the provider's complete supported inventory, including stress, punctuation, diphthongs, affricates, and word boundaries.

Use the [exact pinned tokenizer](https://huggingface.co/onnx-community/Kokoro-82M-v1.0-ONNX-timestamped/blob/dd4401a9add81ac692d20e240d22ec9dda82cc29/tokenizer.json), not an assumed universal alphabet.

**Done means:** supported phones never silently become silence; unknowns are observable; timing stays attached to the actual provider tokens; names/numbers/abbreviations retain original-word provenance for captions and interruption progress.

##### V2 — Closure dominance becomes repeated suppression

**Priority: first. Evidence: reproduced synthetic curve, not a perceptual audio/video test.**

Current paths:

1. The generator extends short PP closures to a minimum 40 ms hold and adds an 80 ms release.
2. The player suppresses vowels above closure weight 0.5 and suppresses jaw continuously.
3. The renderer multiplies retargeted vowel expressions by that suppressed jaw and by (1 − PP) again.

I generated a 25 ms PP token followed by a 100 ms vowel, with maximum constant envelope. The actual player and the renderer's final multiplication give:

| Time | PP weight | Player aa | Final aa expression weight |
|---:|---:|---:|---:|
| 25 ms | 1.000 | 0.0000 | 0.000000 |
| 40 ms | 1.000 | 0.0000 | 0.000000 |
| 50 ms | 0.875 | 0.1125 | 0.001758 |
| 80 ms | 0.500 | 0.9000 | 0.225000 |
| 120 ms | 0.000 | 0.9000 | 0.900000 |

At 50 ms, the final weight is approximately **0.18% of full scale**. During the strong PP release, suppression compounds three times. The player's conditional also changes behavior at the 0.5 boundary.

This establishes the arithmetic effect for that fixture. It does not establish how every utterance looks on every rig.

**Roast:** three individually reasonable “make closure stronger” decisions collaborate to erase the next vowel. Your unit tests can all be green while the mouth loses the syllable.

Choose one owner for articulation blending. Separate phone weights, envelope/jaw intensity, and rig-specific retargeting. PP lip closure and FF lip-to-teeth contact should not be interchangeable. Treat the minimum hold/release as tunable, rig-dependent policy; compare with actual token length and recorded speech.

**Done means:** adjacent PP→vowel, FF→vowel, short closures, overlapping channels, silence, and end-of-speech tests check final expression weights. Then record actual rendered landmarks against audio. Testing the intermediate generated curve is insufficient.

Source: [generator](https://github.com/rahul-j0shi/svara/blob/f33c9943012d9dbbc331c4c8adea5c36a4cc09f2/spikes/kokoro/src/kokoro_spike.py), [player](https://github.com/rahul-j0shi/svara/blob/f33c9943012d9dbbc331c4c8adea5c36a4cc09f2/packages/avatar/src/performance-player.ts), [renderer](https://github.com/rahul-j0shi/svara/blob/f33c9943012d9dbbc331c4c8adea5c36a4cc09f2/packages/avatar/src/avatar-renderer.ts).

##### V3 — Correct total duration does not establish correct internal timing

**Priority: high. Evidence: existing recorded spike, not fresh inference.**

Your own [T0.8 report](https://github.com/rahul-j0shi/svara/blob/f33c9943012d9dbbc331c4c8adea5c36a4cc09f2/docs/spikes/T0.8-kokoro-lipsync.md) now correctly admits that matching waveform length confirms the decoder hop, not phoneme accuracy. A global offset or envelope correlation cannot prove internal closures, releases, or caption boundaries.

Do not replace that honest statement with a smoother but unsupported accuracy claim.

Start with a compact fixed corpus. Include bilabials, labiodentals, diphthongs, rapid transitions, punctuation, names, numbers, acronyms, and representative languages you actually intend to support. Keep voices, speeds, revisions, hashes, and hardware attached to every result.

Manually annotate a small set of obvious acoustic/visible landmarks first. [Montreal Forced Aligner](https://montreal-forced-aligner.readthedocs.io/en/latest/user_guide/workflows/alignment.html) can later supply independent offline phone/word intervals using suitable models and dictionaries. Its output is another estimate requiring review, not perfect ground truth or a recommendation to add a heavy aligner to v1 playback.

**Done means:** the lab displays model interval, reference interval, audio clock, and final rendered pose together. Report error distributions and bad examples, not only a mean that hides failed phrases.

##### V4 — Your lifecycle contract deserves proof more than another framework

**Priority: high after the platform prerequisites. Evidence: documented contract; integrated behavior not accepted.**

The integration document already handles generation fences, bounded synthesis credit, disconnects, hide/restore, and renderer-owned spoken progress. That is substantive design. The narrow kernel ADR is defensible.

The weakness is that those promises are not yet accepted behavior. A framework abstraction earns its place when a provider can be replaced or unmounted during active work without stale audio, listeners, approvals, or callbacks.

Implement the smallest accepted lifecycle surface. Test swapping a provider during synthesis, cancelling queued playback, losing a renderer connection, and disposing dependencies. Do not add dispatch modes because the architecture paper has names for them.

F13 intentionally stops at VAD-based interruption and excludes backchannels/resume. [LiveKit's turn guidance](https://docs.livekit.io/agents/logic/turns/) is useful for understanding richer future UX, not a reason to accuse your documented v1 scope of forgetting it.

##### V5 — A glamorous character will not rescue unfinished desktop behavior

**Priority: existing release gates. Evidence: repository status and recorded measurements.**

The real GNOME Wayland presence/focus/input experiments, AEC, audio timing, and resource budgets remain gates. A browser demo cannot certify desktop placement or microphone ownership. A minimized window's CPU cannot certify visible idle cost.

Use a licensed avatar with an explicit capability profile. Five standard VRM vowel expressions do not imply full 15-viseme articulation. Check what the actual asset supports before promising accurate consonants.

The first review's avatar-license correction still matters, but no production avatar has been accepted in assets. Treat that as an asset-selection blocker, not proof an already shipped application is infringing.

Run CPU and memory comparisons with the full process tree, a stated CPU denominator, and PSS where available. Keep cold-load, warm synthesis, time-to-first-playable-audio, total generation, and renderer cost separate. No new machine-specific benchmarks were run in this pass.

#### 4. Current research: what to reuse, compare, or leave alone

Checked against official repositories/docs during this iteration. “Available” does not mean “benchmarked successfully on Rahul's Ubuntu desktop.”

| Candidate | Useful contribution | Fit and decision |
|---|---|---|
| [HeadTTS](https://github.com/met4citizen/HeadTTS/blob/main/README.md) | Kokoro speech, word/phone timing, Oculus visemes; browser WebGPU/WASM or Node server; MIT code | **First comparison for the lip-sync frontend.** Browser mode avoids an extra service, but changes inference ownership and has memory/WebGPU/fallback tradeoffs. English is the documented supported language. Node-server adoption conflicts with the stated v1 service constraint. |
| [kokoro-onnx](https://github.com/thewh1teagle/kokoro-onnx/blob/main/pyproject.toml) | Existing maintained local wrapper and a useful preprocessing reference | Current metadata requires Python <3.14. **Keep the thin runner unless a compatible alternative proves the required duration/cancellation contract.** Wrapper convenience alone is not sufficient. |
| [Pipecat Kokoro](https://docs.pipecat.ai/api-reference/server/services/tts/kokoro) | Local TTS integration, streaming frames, pipeline behavior | Its Kokoro extra uses kokoro-onnx. Current [dependency metadata](https://github.com/pipecat-ai/pipecat/blob/main/pyproject.toml) also constrains ONNX Runtime to ~1.24.3, whereas Svara pins 1.30.0. **Not a drop-in under these pins.** Static incompatibility was checked; no resolver/install was run. |
| [Pipecat Smart Turn v3](https://docs.pipecat.ai/api-reference/server/utilities/turn-detection/smart-turn-overview) | Local CPU/ONNX semantic end-of-turn detection | **Optional isolated turn-detection comparison.** v3's ONNX path is distinct from heavier extras that include PyTorch. Measure incremental CPU and end-of-turn quality; do not import an entire framework to solve one policy. |
| [Moonshine Voice](https://github.com/moonshine-ai/moonshine/blob/main/README.md) | On-device streaming STT; Linux and Python interfaces | **New STT benchmark candidate**, not automatic faster-whisper replacement. Verify Python 3.14 packaging and native-library coexistence. Compare target-language accuracy and finalization lag under concurrent TTS. No local compatibility or accuracy run was performed. |
| [Moonshine licenses](https://github.com/moonshine-ai/moonshine/blob/main/LICENSE) | Artifact-specific reuse terms | Streaming STT and English STT models are MIT under current upstream terms; listed legacy non-English non-streaming models differ. TTS/G2P assets have separate terms. “Moonshine is MIT” is too coarse for choosing every artifact. |
| [Montreal Forced Aligner](https://montreal-forced-aligner.readthedocs.io/en/latest/user_guide/workflows/alignment.html) | Independent offline phone/word alignment for evaluation | **Developer evaluation tool**, outside the runtime path. Requires appropriate acoustic model, dictionary, and reviewed annotations. |
| Pocket TTS / full avatar frameworks | Alternative voice/rendering approaches raised in iteration 1 | **No default adoption now.** Preserve no-PyTorch/no-extra-service v1 constraints. A framework swap must demonstrate lower integration cost and required rig capabilities. |

The research conclusion is not “custom code is bad.” It is:

**Keep custom code that expresses the project's learning or product contract. Compare commodity components. Replace only when the comparison demonstrates compatibility and a concrete advantage.**

##### Minimum fair comparison protocol for Svara

Use one fixed corpus, voices/models recorded, the same target host, and offline prerecorded input where possible. Keep these columns:

- Cold model load and warm first playable PCM; total generation separately.
- Phone inventory coverage, unknown/fallback count, word provenance.
- Audio/visible landmark error, especially closures and rapid transitions.
- Cancellation of in-flight work, late-result rejection, and queue bounds.
- CPU, PSS/process-tree memory, concurrent STT/TTS behavior.
- Dependency/runtime compatibility and exact code/model/asset terms.

For STT, add word error rate, command/entity mistakes, interim stability, and finalization delay. A vendor's general accuracy claim is not evidence on your accent, vocabulary, or mixed-language usage.

#### Historical round-2 execution plan

| Order | Package | Concrete work | Done/evidence | Size |
|---:|---|---|---|---|
| 1 | Establish mouth input/output contracts | Per-provider alphabet; explicit unsupported/pause; one blend owner; actual avatar capabilities/license | Complete supported inventory coverage; PP/FF→vowel tests inspect final pose weights | M |
| 2 | Make LipSyncLab an evaluator | Small fixed corpus; replay audio with pose; manual anchors; optional offline alignment | Side-by-side bad cases; reproducible timing/weight traces; no unmeasured accuracy claim | M |
| 3 | Run one bounded reuse comparison | Start with HeadTTS frontend/timing against thin runner; investigate streaming STT separately | Decision note: retain/adapt/replace, backed by the comparison columns above | M |
| 4 | Prove one integrated conversation path | One provider, typed/audio input under permissions, output, cancellation, generation fence, bounded queue; narrow kernel | Stale audio cannot reach a new turn; progress reflects playback; teardown leaves no owned work | L |
| 5 | Close existing desktop/release gates | Real Wayland sessions, focus/input, AEC, hide/restore, budgets, packaging and accepted features | Evidence for existing gates; explicitly retain failed/unknown gates | L |

**Scope detail:** all documented F01–F40 requirements remain. This is a proposed focus order, not permission to delete requirements, relax budgets, or bypass Phase 0. Independent pure-function labs and reference comparisons can proceed while platform gates remain blocked; dependent integration/release acceptance cannot.

Do not make a comparison silently introduce PyTorch, another service, paid usage, microphone capture, account login, or system configuration. None of those were performed during this review.

**Next concrete Svara session:** reproduce V2 in a final-expression regression and settle the blend ownership contract. Then compare a few audio/pose examples; do not expand the plugin API first.

#### 6. What would impress a serious engineering interviewer

| Project | Strong evidence to show | Weak substitute to avoid |
| --- | --- | --- |
| Svara | Provider-alphabet adapter; final-pose regressions; audio/visual reference comparison; stale-result/cancellation trace; real Wayland evidence | A polished avatar video or 40-feature architecture list standing in for integration quality |
| Shared lesson | A decision to retain or replace a component based on compatibility, costs, and evidence | More frameworks, metrics names, or speculative scale targets |

I would not reject either project for being a learning project. I would lose confidence if you presented planned guarantees as implemented, repeated arithmetic as independent accuracy evidence, or could not explain how failure changes ownership.

The standout profile is a developer who can show the embarrassing counterexample, explain the broken assumption, fix the right boundary, and demonstrate why it stays fixed.

#### Historical round-2 reproductions

Run only against disposable owned fixture data. The prior review used the actual core/pure functions with the limitations described above.

##### Svara: exact synthetic input and arithmetic

Use the source's TokenTiming, build_timeline, and build_viseme_keys functions. The timeline fixture supplies native characters AIOWYQ and one duration frame per padded token. Inspect the resulting viseme for each character.

For the suppression fixture:

~~~python
keys = build_viseme_keys([
    TokenTiming(25, "p", 0, "PP"),
    TokenTiming(125, "a", 25, "aa"),
])
track = {
    "durationMs": 220,
    "envelope": [255] * 22,
    "envelopeHopMs": 10,
    "visemes": keys,
}
~~~

Load this track into PerformancePlayer at clock origin 0. At each time in [25, 40, 50, 80, 120], calculate:

~~~typescript
const frame = player.update(timeMs);
const finalAa = frame.weights.aa * frame.jaw * (1 - frame.weights.PP);
~~~

That last line is the current renderer's final basic-vowel multiplication for this aa-only fixture. It does not instantiate a WebGL renderer or prove visible geometry. The results are the table in V2.

### Round 3

Evidence date: 8 October 2026. Same unchanged reviewed commit. Reproduced malformed inputs, design inconsistencies, and proposed experiments are distinguished in each finding.

#### 3. Svara

##### V3.1 — Your loudness envelope forgets loudness

**Evidence: reproduced using the actual rms_envelope and write_wav functions. Priority: high alongside the earlier blending fix.**

The envelope divides every sentence's RMS by that sentence's peak RMS.

A fixed tone and the same tone at 1/32 amplitude yield:

~~~
32x amplitude change yields identical envelopes= True
~~~

A much smaller tone rounds entirely to zero in the saved 16-bit PCM, but its pre-quantization envelope still yields:

~~~
Sub-quantization signal: PCM nonzero samples= 0 ; envelope peak= 255
~~~

**Roast:** a whisper gets promoted to a shout, and numerical dust gets a full-scale jaw signal.

**Criticism:** peak normalization supplies relative activity shape, not absolute level. Your design says the envelope distinguishes whispers and shouts and carries real loudness. The implementation discards that distinction. It also computes the envelope from a different representation than the final clipped/quantized audio.

This is a contract mismatch, not proof that normalization is universally wrong. Normalization can be useful for stable articulation across voices if that is explicitly the intended behavior.

**Solution:**

- Separate relative speech activity from calibrated articulation intensity.
- Compute the audible-level path from the actual output PCM representation, or an explicitly equivalent conversion.
- Use a stable per-voice calibration/dB range and an explicit silence floor, rather than independently maximizing every sentence.
- Keep smoothing and headroom; measure the chosen gain rather than inventing universal constants.
- Keep user playback volume separate unless the UX deliberately links it to mouth amplitude. The existing volume-zero behavior preserves playback timing.

**Exit tests:** all-zero output PCM has zero audible envelope; amplitude-scaled fixtures retain the intended intensity distinction; finite values remain in range; sentence boundaries do not create unexplained gain jumps. Then review actual speech across voices before claiming perceptual quality.

Source: [Kokoro spike functions](https://github.com/rahul-j0shi/svara/blob/f33c9943012d9dbbc331c4c8adea5c36a4cc09f2/spikes/kokoro/src/kokoro_spike.py), [avatar design](https://github.com/rahul-j0shi/svara/blob/f33c9943012d9dbbc331c4c8adea5c36a4cc09f2/docs/revamp/03-avatar-and-lipsync.md).

##### V3.2 — “One-sample accuracy” and integer milliseconds disagree

**Evidence: design inconsistency plus arithmetic fixture; not a current Kokoro inference failure. Priority: before implementing provider/protocol contracts.**

The design's PerformanceSegment uses integer duration_ms. The protocol requires duration to match the PCM sample count within one sample.

For 339 samples at 24 kHz:

| Quantity | Value |
|---|---:|
| Exact duration | 14.125 ms |
| Nearest integer milliseconds | 14 ms |
| Difference | 3 samples |

The same issue occurs with longer audio whenever its duration falls between integer milliseconds. Native Kokoro's current 25 ms frame-derived durations happen to fit; arbitrary provider output need not.

**Roast:** the contract promises a ruler precise to one sample, then supplies a ruler marked only in milliseconds.

**Solution:** make integer frame count and sample rate authoritative. Derive duration for scheduling from those values; use milliseconds as a display/annotation representation. A floating-point duration can also address this particular inconsistency, but exact frame count makes the intended invariant clearer.

Validate mono PCM byte alignment and supported rates before deriving frames. When decodeAudioData resamples, validate duration/sample-count relationships at the appropriate original/decoded rates.

**Exit tests:** fractional-millisecond fixtures are accepted when exact frame counts agree; truncated PCM and genuine count mismatches are rejected; provider tests include outputs beyond Kokoro's 25 ms grid.

Source: [PerformanceSegment design](https://github.com/rahul-j0shi/svara/blob/f33c9943012d9dbbc331c4c8adea5c36a4cc09f2/docs/revamp/03-avatar-and-lipsync.md), [audio protocol rules](https://github.com/rahul-j0shi/svara/blob/f33c9943012d9dbbc331c4c8adea5c36a4cc09f2/docs/revamp/13-implementation-guide.md).

##### V3.3 — The development player accepts incompatible audio and animation

**Evidence: reproduced using the actual SpikePlayback and PerformancePlayer with fake audio APIs. Priority: useful probe regression; production validation is already planned.**

A 200 ms track with a decoded 2,000 ms audio buffer is accepted. The performance ends while the longer fake buffer would still be playing:

~~~
200 ms track + 2000 ms decoded audio accepted= true
animation ended 300 ms after scheduled start= true
~~~

**Qualification:** today's generated Kokoro fixtures already check duration/sample equality upstream. This test deliberately introduces incompatible inputs. It does not prove that the existing healthy sample files drift.

**Roast:** the probe validates each half independently and assumes they describe the same sentence.

**Solution:** reject inconsistent audio/track inputs at the playback boundary. Use authoritative frame-based duration and validate envelope coverage/clock units. Carry the same fixture into the planned protocol acceptance suite; do not mistake a future rule for a current check.

**Exit tests:** incompatible inputs never start a source; matching inputs do; cancellation still prevents late work from starting. This is a small extension to the probe, not a demand to implement the whole product now.

Source: [SpikePlayback](https://github.com/rahul-j0shi/svara/blob/f33c9943012d9dbbc331c4c8adea5c36a4cc09f2/apps/desktop/src/spike-playback.ts).

##### V3.4 — Shared scheduling time is not necessarily speaker output time

**Evidence: source finding and synthetic clock fixture; no physical measurement. Priority: existing timing gate.**

The probe samples AudioContext.currentTime. W3C distinguishes graph time from device output position. getOutputTimestamp supplies a context/performance mapping; outputLatency estimates latency. Check support in the selected Electron build. Source: [Web Audio 1.1 working draft](https://www.w3.org/TR/webaudio-1.1/).

Synthetic result:

~~~
graph elapsed ms=150 ; output elapsed ms=30
~~~

**Roast:** the components agree, but the speakers have not arrived.

**Solution experiment:** retain currentTime scheduling; evaluate output-position sampling for the face and heard progress:

~~~typescript
const stamp = context.getOutputTimestamp();
const outputContextTime =
  stamp.contextTime + (performance.now() - stamp.performanceTime) / 1000;
~~~

Validate startup, suspension, device changes, timestamp availability, and display delay. Keep compensation separate from artistic anticipation.

**Exit tests:** injected delay changes face sampling without changing scheduling; fallback is explicit; target-device landmark recordings establish the remaining offset.

##### What passed: frontend stale-result ownership

The actual probe passed two additional mock cases:

~~~
Stale old onended closed new context= 0
Replacement B accepted= true ; late decode A accepted= false ; B start count= 1
~~~

An old completion callback did not close replacement playback. A late decode from A did not replace active B.

Keep those guards and add the cases as regressions. They prove those frontend paths under controlled mocks, not the unimplemented core/provider/kernel lifecycle.

**A serious review should report this too. Roasting everything indiscriminately is as unhelpful as praising everything.**

#### Historical round-3 research

##### ONNX Runtime already offers per-run termination

The current concurrency ADR correctly says cancelling Python does not stop a native thread automatically. It uses short bounded inference plus stale-result disposal.

[ONNX Runtime's Python API](https://onnxruntime.ai/docs/api/python/api_summary.html) additionally exposes RunOptions.terminate: it requests termination for calls using that options object, which return an error status.

**Proposed experiment:** give each job its own RunOptions, pass it to session.run, and request termination on cancellation. Retain generation fencing. Keep a model's active-job slot owned until native work actually finishes; a cancelled Python await is not proof its thread stopped.

Measure stop-request-to-native-exit, delay before a replacement job starts, and repeated cancellation/reuse. Do not share one options object across unrelated turns. No model-level termination benchmark was run here, and no immediate-stop guarantee is claimed.

This reuses the existing runtime without adding PyTorch, a local service, or a new framework. It is a potential improvement to the accepted design, not evidence that every native backend becomes cancellable.

##### Reuse testing tools before inventing testing infrastructure

| Tool | Relevant capability | Recommendation |
| --- | --- | --- |
| Existing [Hypothesis](https://hypothesis.readthedocs.io/en/latest/stateful.html) in Svara | Stateful rules and invariants | Reuse for the planned Python lifecycle contracts; it is already in dev dependencies. |
| [fast-check scheduler](https://fast-check.dev/docs/advanced/race-conditions/) | Controlled promise resolution ordering | Optional if JS async schedules outgrow explicit deferred-promise tests. It controls wrapped promises, not arbitrary native/browser work. |

The immediate fixes need no new testing dependency. Keep the small deterministic reproductions first.

##### Quantization is an experiment, not a promised rescue

[ONNX Runtime's quantization guidance](https://onnxruntime.ai/docs/performance/model-optimizations/quantization.html) explicitly makes gains model/hardware-dependent; overhead can make inference slower.

If pursuing it later, check speech quality, token-duration outputs, frame-hop equality, cancellation, memory, and target-host latency together. A smaller model file or faster isolated inference is insufficient if lip-sync alignment degrades. No quantization or model replacement was performed.

The previous HeadTTS/Moonshine comparison candidates remain candidates. This pass found more immediate value in correcting boundaries inside the chosen stack.

#### 5. Solutions in an order you can execute

Do not create another giant feature backlog. These extend the existing plans.

| Order | Project | Work | Evidence to produce |
| ---: | --- | --- | --- |
| 1 | Svara | One articulation boundary package: earlier repeated suppression plus this pass's envelope/PCM consistency | Final-pose traces and silent/quiet/loud fixtures, then reviewed speech examples |
| 2 | Svara | Settle frame-based duration; validate audio/performance pairing | Fractional-ms and deliberately mismatched fixtures; provider contract rules |
| 3 | Svara | Extend LipSyncLab with graph/output/display timing distinctions | Controlled clock traces and real device landmark recordings under the existing timing gate |
| 4 | Svara | Compare ORT per-run termination with bounded completion | Native-exit and replacement-start measurements; retain stale-result fences |

Independent pure fixtures and developer comparisons can proceed while platform gates remain unresolved. Nothing here waives Wayland, AEC, budget, dependency, or release requirements. All documented v1 features remain; this is a proposed engineering focus order.

##### Portfolio output from this work

For each repaired boundary, publish one compact case study:

1. The promised invariant.
2. The tiny fixture that breaks it.
3. The incorrect assumption.
4. The change and its scope.
5. The regression and remaining limits.

For Svara, “the clock was consistent but represented the wrong stage of playback” demonstrates integration understanding beyond assembling an avatar demo.

**The next iteration should follow code changes or new measurements.** Three reviews of unchanged heads have now produced enough actionable work. Another increasingly dramatic report would add less value than closing one of these counterexamples.

#### Historical round-3 minimal fixture parameters

| Case | Construction |
| --- | --- |
| Envelope scale invariance | Fixed 220 Hz, 100 ms float32 tone versus the same samples divided by 32 |
| PCM/envelope disagreement | Same tone multiplied by 10^-6; compare write_wav PCM with rms_envelope output |
| Duration precision | 339 frames / 24000 Hz = 14.125 ms; integer 14 ms differs by 3 frames |
| Playback mismatch | Fake decoded audio: 48000 frames at 24000 Hz; performance duration 200 ms |
| Stale playback | Replace A with B, then invoke saved old onended; separately finish A's deferred decode after B starts |
| Device clock | start = 1 s, currentTime = 1.15 s, output timestamp contextTime = 1.03 s at matching performance time |

The fixture construction is intentionally narrow. Use the evidence labels in each finding; no broad crash, perceptual, or hardware accuracy guarantee follows from these parameters.

## 10. Round 4 reproduction code

These diagnostic sources were executed during round 4. They are embedded here for reproducibility and are not patches to the repository.

Extract the four files below into one diagnostic directory. Set `SVARA_REPO` to the absolute path of the pinned checkout. Run `fourth-pass-svara.py` before `fourth-tail-check.mjs`. Python needs NumPy; Node 24 provides native TypeScript stripping. The avatar harness creates a scratch transformed module beside itself and replaces rendering/loader imports with the included test doubles. It proves ordering/disposal branches, not rendered or GPU behavior.

```bash
python3 fourth-pass-svara.py
node --disable-warning=ExperimentalWarning fourth-tail-check.mjs
node --disable-warning=ExperimentalWarning fourth-avatar-check.mjs
```

#### fourth-pass-svara.py

```python
import ast, json, math, os
from pathlib import Path
from dataclasses import dataclass
from typing import Final
import numpy as np

source=Path(os.environ.get('SVARA_REPO','/workspace/scratch/f19504e07c45/tmp/project-audit/svara'))/'spikes/kokoro/src/kokoro_spike.py'
tree=ast.parse(source.read_text())
nodes=[n for n in tree.body if isinstance(n,ast.AnnAssign) or isinstance(n,ast.ImportFrom) and n.module=='__future__' or isinstance(n,ast.ClassDef) and n.name=='TokenTiming' or isinstance(n,ast.FunctionDef) and n.name in {'build_timeline','build_viseme_keys'}]
ns={'np':np,'math':math,'dataclass':dataclass,'Final':Final,'__name__':'__main__'}
exec(compile(ast.Module(body=nodes,type_ignores=[]),str(source),'exec'),ns)
out=[]
for final_pad in (0,1,4):
    timeline, frames=ns['build_timeline'](['a'],np.array([0,4,final_pad],dtype=np.float32))
    keys=ns['build_viseme_keys'](timeline)
    duration=frames*25
    out.append({'finalPadFrames':final_pad,'durationMs':duration,'maxKeyMs':max(k['tMs'] for k in keys),'outOfRangeKeys':sum(k['tMs']>duration for k in keys)})
print(json.dumps(out,indent=2))
track={'durationMs':100,'envelope':[255]*10,'envelopeHopMs':10,'visemes':ns['build_viseme_keys']([ns['TokenTiming'](100,'a',0,'aa')])}
Path(__file__).with_name('fourth-tail-track.json').write_text(json.dumps(track))
```

#### fourth-tail-check.mjs

```javascript
import {readFileSync} from 'node:fs';
import assert from 'node:assert/strict';
import {pathToFileURL} from 'node:url';
const repo=process.env.SVARA_REPO??'/workspace/scratch/f19504e07c45/tmp/project-audit/svara';
const {PerformancePlayer}=await import(pathToFileURL(repo+'/packages/avatar/src/performance-player.ts'));
const track=JSON.parse(readFileSync(new URL('./fourth-tail-track.json',import.meta.url)));
const player=new PerformancePlayer();player.play(track,0);
const before=player.update(99),atEnd=player.update(100);
console.log(JSON.stringify({acceptedOutOfRangeKey:track.visemes.at(-1),beforeEnd:before,atEnd},null,2));
assert.equal(before.weights.aa,.9);assert.equal(atEnd.ended,true);assert.equal(atEnd.weights.aa,0);
```

#### fourth-avatar-stubs.mjs

```javascript
export const pending=new Map();
export const disposed=[];
export const faults={optimize:false};
class Vec {x=0;y=1;z=0;set(x,y,z){this.x=x;this.y=y;this.z=z;return this;}}
class Obj {position=new Vec();rotation=new Vec();scale=new Vec();children=[];visible=true;parent=null;
  add(o){this.children.push(o);o.parent=this;}clear(){this.children=[];}removeFromParent(){if(this.parent)this.parent.children=this.parent.children.filter(x=>x!==this);this.parent=null;}}
export class Group extends Obj {}
export class Scene extends Obj {}
export class Mesh extends Obj {constructor(g,m){super();this.geometry=g;this.material=m;}}
export class PerspectiveCamera extends Obj {constructor(f){super();this.fov=f;}lookAt(){}updateProjectionMatrix(){}}
export class WebGLRenderer {constructor({canvas}){this.domElement=canvas;}setClearColor(){}setPixelRatio(){}setSize(){}dispose(){}render(){}}
export class HemisphereLight extends Obj {}
export class DirectionalLight extends Obj {}
export class MeshStandardMaterial {}
export class MeshBasicMaterial {}
export class SphereGeometry {}
export class CapsuleGeometry {}
export class Vector3 extends Vec {}
export class Box3 {setFromObject(){return this;}getSize(v){v.y=2;return v;}getCenter(v){return v;}}
export const MathUtils={degToRad:x=>x*Math.PI/180};
export const SRGBColorSpace='srgb';
export class GLTFLoader {register(){}loadAsync(source){return new Promise((resolve,reject)=>pending.set(source,{resolve,reject}));}}
export class VRMLoaderPlugin {}
export const VRMExpressionPresetName={Aa:'aa',Ee:'ee',Ih:'ih',Oh:'oh',Ou:'ou'};
export const VRMUtils={deepDispose:s=>disposed.push(s.name??'fallback'),removeUnnecessaryVertices(){if(faults.optimize)throw new Error('injected optimization failure');},combineSkeletons(){},rotateVRM0(){}};
export class FrameProbe {snapshot(){return {};}record(){}}
export function model(name){const scene=new Group();scene.name=name;return {scene,userData:{vrm:{scene,meta:{name},update(){}}}};}
```

#### fourth-avatar-check.mjs

```javascript
import {readFileSync,writeFileSync} from 'node:fs';
import {stripTypeScriptTypes} from 'node:module';
import assert from 'node:assert/strict';
import {pending,disposed,faults,model} from './fourth-avatar-stubs.mjs';
const base=(process.env.SVARA_REPO??'/workspace/scratch/f19504e07c45/tmp/project-audit/svara').replace(/\/$/,'')+'/';
let source=stripTypeScriptTypes(readFileSync(base+'packages/avatar/src/avatar-renderer.ts','utf8'));
source=source.replaceAll('"@pixiv/three-vrm"','"./fourth-avatar-stubs.mjs"').replaceAll('"three"','"./fourth-avatar-stubs.mjs"').replaceAll('"three/examples/jsm/loaders/GLTFLoader.js"','"./fourth-avatar-stubs.mjs"').replaceAll('"./metrics"','"./fourth-avatar-stubs.mjs"').replaceAll('"./performance-player"',JSON.stringify('file://'+base+'packages/avatar/src/performance-player.ts'));
writeFileSync(new URL('./fourth-avatar-instrumented.mjs',import.meta.url),source);
globalThis.window={devicePixelRatio:1,setTimeout,clearTimeout};
globalThis.ResizeObserver=class {observe(){}disconnect(){}};
globalThis.requestAnimationFrame=()=>1;globalThis.cancelAnimationFrame=()=>{};
const {AvatarRenderer}=await import('./fourth-avatar-instrumented.mjs');
const canvas={clientWidth:100,clientHeight:100};
const app=readFileSync(base+'apps/desktop/src/spike-app.tsx','utf8');
const start=app.indexOf('    void fetch("/spike-assets/avatar.vrm"');
const end=app.indexOf('\n\n    const resizeObserver',start);
assert(start>=0&&end>start);
const chain=app.slice(start,end).replace('void fetch','const autoLoad = fetch');
const wrapper=stripTypeScriptTypes('function run(renderer,abort,setModelStatus,objectUrlRef) {\n'+chain+'\n return autoLoad;\n}');
const run=new Function(wrapper+'\nreturn run;')();
const settle=async()=>{for(let i=0;i<8;i++)await Promise.resolve();};
URL.createObjectURL=b=>b.id;URL.revokeObjectURL=()=>{};

let statuses=[];let fetchResolve;
globalThis.fetch=()=>new Promise(resolve=>fetchResolve=resolve);
let renderer=new AvatarRenderer(canvas,{onModelLoaded:n=>statuses.push('VRM loaded: '+n),onError:e=>statuses.push('VRM load failed: '+e.message)});
const auto=run(renderer,new AbortController(),s=>statuses.push(s),{current:null});
let manual=renderer.loadVrm('manual-B');pending.get('manual-B').resolve(model('manual-B'));await manual;
fetchResolve({ok:true,headers:{get:()=> 'model/gltf-binary'},blob:async()=>({id:'auto-A'})});await settle();
pending.get('auto-A').resolve(model('auto-A'));await auto;
console.log('Delayed startup fetch after manual selection:',JSON.stringify({statuses,disposed:[...disposed]}));
assert.equal(statuses.at(-1),'VRM loaded: auto-A');assert(disposed.includes('manual-B'));renderer.dispose();

statuses=[];disposed.length=0;
globalThis.fetch=async()=>({ok:true,headers:{get:()=> 'model/gltf-binary'},blob:async()=>({id:'auto-fails'})});
renderer=new AvatarRenderer(canvas,{onModelLoaded:n=>statuses.push('VRM loaded: '+n),onError:e=>statuses.push('VRM load failed: '+e.message)});
const failingAuto=run(renderer,new AbortController(),s=>statuses.push(s),{current:null});await settle();
manual=renderer.loadVrm('newest-B');pending.get('newest-B').resolve(model('newest-B'));await manual;
pending.get('auto-fails').reject(new Error('late failure A'));await failingAuto;
console.log('Stale startup error after B success:',JSON.stringify(statuses));
assert(statuses.at(-1).includes('late failure A'));renderer.dispose();

statuses=[];disposed.length=0;
renderer=new AvatarRenderer(canvas,{onModelLoaded:n=>statuses.push(n)});
const a=renderer.loadVrm('stale-success-A'),b=renderer.loadVrm('current-B');
pending.get('current-B').resolve(model('current-B'));await b;pending.get('stale-success-A').resolve(model('stale-success-A'));await a;
assert.deepEqual(statuses,['current-B']);assert(disposed.includes('stale-success-A'));
console.log('Renderer-only stale successful load: PASS (B retained; A disposed)');
const c=renderer.loadVrm('after-dispose-C');renderer.dispose();pending.get('after-dispose-C').resolve(model('after-dispose-C'));await c;
assert(disposed.includes('after-dispose-C'));console.log('Renderer disposed while load pending: PASS (candidate disposed)');

disposed.length=0;renderer=new AvatarRenderer(canvas);
faults.optimize=true;const broken=renderer.loadVrm('candidate-error');pending.get('candidate-error').resolve(model('candidate-error'));await assert.rejects(broken,/injected optimization failure/);faults.optimize=false;
console.log('Optimization fault candidate disposal count:',disposed.filter(n=>n==='candidate-error').length);
assert(!disposed.includes('candidate-error'));renderer.dispose();
```

## 11. Document provenance

Derived from [Shale-Svara-Engineering-Review-Iteration-4.md](sandbox:/workspace/scratch/f19504e07c45/Shale-Svara-Engineering-Review-Iteration-4.md), read in its current saved form before splitting. The combined original remains unchanged.

**Source SHA-256:** `f36dfaecea774cfca41d6f146b1cc3aaea98d4d03c6b7193543f5b177c21d280`

Historical project chapters retain their original content where applicable, with table scoping and headings adjusted for this standalone structure. The split itself claimed no new research results or fixes. The fifth review added the separately dated evidence above; the scope revision reorganizes recommendations and acceptance boundaries without new test results.

## 12. Final-round reproduction code

These harnesses execute reviewed source with dependencies mocked or subprocess calls captured. They do not operate the real desktop, run a native helper, render a VRM, or synthesize speech.

For the cadence fixture, extract the shared stub below into `tmp/audit-repro/fourth-avatar-stubs.mjs` and the final scripts into sibling `tmp/final-review/`. It creates three scratch `.mjs` adapters next to itself. Node 24.19.0 supplies built-in TypeScript stripping. Set `SVARA_REPO` to the absolute checkout path if different; the recorded default path is the review checkout. The focus fixture also accepts that variable. The Python fixture's `source` line records the review path; point it at the same reviewed `kokoro_spike.py` when reproducing elsewhere.

```bash
node --disable-warning=ExperimentalWarning tmp/final-review/final-cadence.mjs
node tmp/final-review/final-focus.cjs
python3 tmp/final-review/final-phonemize.py
```

The cadence prototype counts arithmetic opportunities, not real rendered performance. The focus fixture's process API is a test double and never spawns a helper. The phonemizer spy returns placeholder output only; its assertions concern argv/stdin construction. All three completed successfully during this review. The final section records the observed counts and their limits.

### fourth-avatar-stubs.mjs

```javascript
export const pending=new Map();
export const disposed=[];
export const faults={optimize:false};
class Vec {x=0;y=1;z=0;set(x,y,z){this.x=x;this.y=y;this.z=z;return this;}}
class Obj {position=new Vec();rotation=new Vec();scale=new Vec();children=[];visible=true;parent=null;
  add(o){this.children.push(o);o.parent=this;}clear(){this.children=[];}removeFromParent(){if(this.parent)this.parent.children=this.parent.children.filter(x=>x!==this);this.parent=null;}}
export class Group extends Obj {}
export class Scene extends Obj {}
export class Mesh extends Obj {constructor(g,m){super();this.geometry=g;this.material=m;}}
export class PerspectiveCamera extends Obj {constructor(f){super();this.fov=f;}lookAt(){}updateProjectionMatrix(){}}
export class WebGLRenderer {constructor({canvas}){this.domElement=canvas;}setClearColor(){}setPixelRatio(){}setSize(){}dispose(){}render(){}}
export class HemisphereLight extends Obj {}
export class DirectionalLight extends Obj {}
export class MeshStandardMaterial {}
export class MeshBasicMaterial {}
export class SphereGeometry {}
export class CapsuleGeometry {}
export class Vector3 extends Vec {}
export class Box3 {setFromObject(){return this;}getSize(v){v.y=2;return v;}getCenter(v){return v;}}
export const MathUtils={degToRad:x=>x*Math.PI/180};
export const SRGBColorSpace='srgb';
export class GLTFLoader {register(){}loadAsync(source){return new Promise((resolve,reject)=>pending.set(source,{resolve,reject}));}}
export class VRMLoaderPlugin {}
export const VRMExpressionPresetName={Aa:'aa',Ee:'ee',Ih:'ih',Oh:'oh',Ou:'ou'};
export const VRMUtils={deepDispose:s=>disposed.push(s.name??'fallback'),removeUnnecessaryVertices(){if(faults.optimize)throw new Error('injected optimization failure');},combineSkeletons(){},rotateVRM0(){}};
export class FrameProbe {snapshot(){return {};}record(){}}
export function model(name){const scene=new Group();scene.name=name;return {scene,userData:{vrm:{scene,meta:{name},update(){}}}};}
```

### final-cadence.mjs

```javascript
import {readFileSync,writeFileSync} from 'node:fs';
import {stripTypeScriptTypes} from 'node:module';
import assert from 'node:assert/strict';
const repo=process.env.SVARA_REPO??'/workspace/scratch/f19504e07c45/tmp/project-audit/svara';
const folder=new URL('./',import.meta.url);
let stubs=readFileSync(new URL('../audit-repro/fourth-avatar-stubs.mjs',folder),'utf8');
stubs='export let renders=0; export function resetRenders(){renders=0;}\n'+stubs.replace('render(){}','render(){renders++;}');
writeFileSync(new URL('final-render-stubs.mjs',folder),stubs);
writeFileSync(new URL('final-metrics.mjs',folder),stripTypeScriptTypes(readFileSync(repo+'/packages/avatar/src/metrics.ts','utf8'),{mode:'transform'}));
let code=stripTypeScriptTypes(readFileSync(repo+'/packages/avatar/src/avatar-renderer.ts','utf8'));
code=code.replaceAll('"@pixiv/three-vrm"','"./final-render-stubs.mjs"').replaceAll('"three"','"./final-render-stubs.mjs"').replaceAll('"three/examples/jsm/loaders/GLTFLoader.js"','"./final-render-stubs.mjs"').replaceAll('"./metrics"','"./final-metrics.mjs"').replaceAll('"./performance-player"',JSON.stringify('file://'+repo+'/packages/avatar/src/performance-player.ts'));
writeFileSync(new URL('final-renderer.mjs',folder),code);
let callback,clock=0;
globalThis.performance={now:()=>clock};
globalThis.window={devicePixelRatio:1,setTimeout:()=>1,clearTimeout:()=>{}};
globalThis.ResizeObserver=class {observe(){}disconnect(){}};
globalThis.requestAnimationFrame=fn=>{callback=fn;return 1;};globalThis.cancelAnimationFrame=()=>{};
const {AvatarRenderer}=await import('./final-renderer.mjs');
const mock=await import('./final-render-stubs.mjs');
const results=[];
for(const hz of [60,120,144,165,240]) {
  clock=0;mock.resetRenders();
  const renderer=new AvatarRenderer({clientWidth:100,clientHeight:100});renderer.setMode('speaking');
  const measurement=renderer.measure(10000);
  for(let i=1;i<=hz*10;i++){clock=i*1000/hz;callback(clock);}
  renderer.setMode('idle');const m=await measurement;
  let due=1000/60,reference=0;
  for(let i=1;i<=hz*10;i++){const t=i*1000/hz;if(t+1e-7>=due){reference++;due+=(Math.floor((t-due+1e-7)/(1000/60))+1)*(1000/60);}}
  results.push({callbackHz:hz,renders:mock.renders,reportedFps:m.fps,reportedDropped:m.droppedFrames,phasePreservingPrototype:reference});renderer.dispose();
}
console.log(JSON.stringify(results,null,2));
assert.equal(results.find(x=>x.callbackHz===144).renders,480);
assert.equal(results.find(x=>x.callbackHz===144).reportedDropped,0);
assert(results.every(x=>x.phasePreservingPrototype===600));
```

### final-focus.cjs

```javascript
const fs=require('node:fs');const vm=require('node:vm');const util=require('node:util');const path=require('node:path');const assert=require('node:assert/strict');
const repo=process.env.SVARA_REPO??'/workspace/scratch/f19504e07c45/tmp/project-audit/svara';
const handlers=new Map(),pending=[],calls=[],actions=[];let window;
class Window {
  constructor(){window=this;this.webContents={setWindowOpenHandler(){},on(){}};this.focused=false;}
  isDestroyed(){return false;}setAlwaysOnTop(){}loadURL(){return Promise.resolve();}showInactive(){}getBounds(){return {};}
  getNativeWindowHandle(){return Buffer.from([1,0,0,0]);}isFocusable(){return true;}isFocused(){return this.focused;}isAlwaysOnTop(){return true;}
  focus(){this.focused=true;actions.push('focus');}blur(){this.focused=false;actions.push('blur');}
}
const electron={BrowserWindow:Window,ipcMain:{handle:(name,fn)=>handlers.set(name,fn)},app:{commandLine:{appendSwitch(){}},setPath(){},getPath(){return '/fixture';},whenReady(){return Promise.resolve();},on(){},exit(code){throw Error('exit '+code);}}};
const fakeExec=(exe,args,options,cb)=>{calls.push(args[1]);pending.push(()=>cb(null,{stdout:JSON.stringify({inputHint:args[1]==='on',takeFocus:args[1]==='on',overrideRedirect:false})}));};
const context={require:name=>name==='electron'?electron:name==='node:child_process'?{execFile:fakeExec}:name==='node:util'?util:name==='node:path'?path:name==='./validation.cjs'?require(repo+'/apps/desktop/electron/validation.cjs'):(()=>{throw Error(name);})(),__dirname:repo+'/apps/desktop/electron',process:{env:{SVARA_SPIKE_FOCUS_MODE:'managed-hints',SVARA_SPIKE_INPUT_HINT_HELPER:'/fixture/no-real-process'}},Buffer,console:{log(){},error(e){throw e;}}};
vm.runInNewContext(fs.readFileSync(repo+'/apps/desktop/electron/main.cjs','utf8'),context);
const settle=async()=>{for(let i=0;i<12;i++)await Promise.resolve();};
(async()=>{
  await settle();assert.equal(pending.length,1);pending.shift()();await settle();
  const event={sender:window.webContents};const handler=handlers.get('svara-spike:set-interaction');
  const opens=Array.from({length:20},()=>handler(event,true));await settle();
  const close=handler(event,false);const actionsAtClose=actions.length;
  while(pending.length){pending.shift()();await settle();}
  await Promise.all([...opens,close]);
  const late=actions.slice(actionsAtClose);
  console.log(JSON.stringify({queuedOpenRequests:20,lateFocusCallsAfterCloseRequested:late.filter(x=>x==='focus').length,nativeCalls:calls,finalAction:actions.at(-1)},null,2));
  assert.equal(late.filter(x=>x==='focus').length,20);assert.equal(actions.at(-1),'blur');
})().catch(e=>{console.error(e);process.exitCode=1;});
```

### final-phonemize.py

```python
import ast,json
from pathlib import Path
from types import SimpleNamespace
source=Path('/workspace/scratch/f19504e07c45/tmp/project-audit/svara/spikes/kokoro/src/kokoro_spike.py')
tree=ast.parse(source.read_text());nodes=[n for n in tree.body if isinstance(n,ast.ImportFrom) and n.module=='__future__' or isinstance(n,ast.FunctionDef) and n.name=='phonemize']
calls=[]
def run(argv,**kwargs):
    calls.append({'argv':argv,'stdinInput':kwargs.get('input'),'timeout':kwargs.get('timeout')})
    return SimpleNamespace(stdout='ə')
ns={'subprocess':SimpleNamespace(run=run),'Path':Path}
exec(compile(ast.Module(body=nodes,type_ignores=[]),str(source),'exec'),ns)
for text in ['hello','--help','--version','- hello']:
    ns['phonemize'](Path('/fixture/espeak-ng'),text)
print(json.dumps(calls,indent=2))
assert all(x['stdinInput'] is None for x in calls)
assert calls[1]['argv'][-1]=='--help' and '--' not in calls[1]['argv']
```
