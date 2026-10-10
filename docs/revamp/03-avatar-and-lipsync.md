# 03 — Avatar, lip sync & expressions

Lip sync and expressions are part of the **core pipeline**, not a rendering trick. The core produces a
*performance* (audio + viseme track + amplitude envelope + expression and gesture cues). The renderer
plays that performance on one clock.

## 1. Avatar format: VRM 1.0, not Ready Player Me

Ready Player Me is gone as a platform: Netflix acquired it in Dec 2025 and it ended its public
services, including the avatar creator and APIs, on 31 Jan 2026. Only previously exported GLBs still
work. Even before that, RPM avatars were hard to change on the fly (baked meshes, a remote creator).

| Option | Customisable on the fly | Face rig | Tooling | Licence clarity | Verdict |
|--------|------------------------|----------|---------|-----------------|---------|
| **VRM 1.0** (glTF extension) + `@pixiv/three-vrm` | High: per-mesh visibility, material colours/textures, spring-bone hair/cloth, custom expressions | Standard presets: `aa ih ou ee oh`, `happy angry sad relaxed surprised`, `blink*`, `look*`, plus **custom expressions** | VRoid Studio (free, character creator), Blender VRM add-on, UniVRM | Licence metadata is embedded in the file | **Chosen** |
| ARKit-52 GLB (Avaturn, Character Creator, MetaHuman exports) | Medium | 52 blendshapes, the richest mouth | Paid or complex pipelines | Varies | Supported later through the same retarget map |
| Live2D | High for 2D | Good | Proprietary SDK licence | Restrictive | No |
| Fully procedural (code-generated) character | Total | Whatever we build | Our own | Ours | Too much art work for v1; possible "inventory" direction later |

**Decision:** VRM 1.0 as the native format, with the renderer written against an *avatar profile*
abstraction (§4.4). ARKit-52 GLBs are a post-v1 option: a retarget map is necessary but not
sufficient, because loading, humanoid mapping and framing of an arbitrary GLB also need their own
qualification. No "drop in a JSON file" claim is made.

**Renderer:** three.js + `@pixiv/three-vrm`, because it is the reference VRM runtime and actively
maintained. Babylon.js has only community VRM loaders. (Resume note: this replaces "Babylon.js".)

**Default avatar (decided 2026-10-10, ADR-0007): one exact, licence-verified free VRM.** An
earlier version of this plan said the VRoid Studio presets (`VRoidPreset_A` … `Z`) are CC0. Their
[official terms](https://vroid.pixiv.help/hc/en-us/articles/4402394424089-VRoidPreset-A-Z) say they
are **not** CC0 (engineering review SV-01). Some VRoid models are designated CC0, for example
[β AvatarSample_1](https://vroid.pixiv.help/hc/en-us/articles/360012381793-%CE%B2-Ver-AvatarSample-1).
T0.12 selects in this order, re-checking the publisher's page on the day:

1. a model its publisher designates CC0;
2. otherwise a VRoid preset used under its actual, recorded, non-CC0 conditions, only if those
   conditions permit redistribution inside the `.deb` and this repository.

Licences are never relabelled. "Free" is not the same as CC0.

- **Getting the file on Ubuntu:** VRoid Studio has no Linux build. A VRoid model is exported **once**
  as VRM 1.0 by running VRoid Studio under Proton/Wine (community-reported to work) or on any
  Windows/macOS machine, or downloaded directly where the publisher distributes a `.vrm`. three-vrm
  loads both VRM 0.x and 1.0, so no conversion is required.
- **Qualification (required before use, ADR-0007/0018):** a committed report records VRM version,
  hash, source, export version and profile; humanoid bones and head; presence and non-empty binds of
  `aa ee ih oh ou`; `isBinary`; `overrideMouth`; a 0/0.25/0.5/0.75/1 sweep of each mouth preset;
  mouth behaviour under the intended expressions; and geometry/material/texture counts. It uses
  three-vrm's own `expressionManager`/`getExpression()` introspection, plus the Khronos glTF
  Validator for container checks only (it does not understand `VRMC_vrm`, licences or visemes). If
  the first asset fails, evaluate one replacement; further searching needs a named missing capability.
- **Mouth shapes:** VRoid exports normally include the five vowel shapes, so v1 uses the
  **`vrm-basic`** retarget (§4.4); the qualification report confirms this for the chosen file. The
  extended 15-viseme mouth (adding `PP`, `FF`, `TH`, `DD`, `kk`, `CH`, `SS`, `nn`, `RR` shape keys in
  Blender with the VRM add-on) is an **optional later item**; no custom fifteen-shape art is
  required. The engine is identical either way; only the retarget file changes.
- **Customisation in v1** happens only through `avatar.json` (§2): colours, mesh visibility,
  framing, spring bones, face and idle parameters. No mesh editing inside the app.
- **Licence record:** `assets/LICENSES.md` states the exact asset name, source URL, terms URL and the
  terms' wording on the checked date, SHA-256, export tool/version, embedded VRM metadata and every
  modification. The model is committed with Git LFS, only once.
- **Respect VRM licence metadata:** imported `.vrm` files carry usage permissions (avatar
  permission, commercial use, modification). The import dialog shows them; if modification is
  prohibited, looks edits are blocked with an explanation (F14), not merely warned about.

**Animation assets (idle and gestures)** are needed and must be licence-clean:

| Source | Use | Licence note |
|--------|-----|--------------|
| Procedural (code) | Breathing, sway, blink, saccades, head nod/tilt | Ours. Covers most of the "alive" feeling with zero assets |
| pixiv's free VRMA motion samples | Wave, greeting and similar clips | Check the distribution terms before committing them to the repo; otherwise download at first run |
| Mixamo → retargeted to VRMA | More gestures | Allowed inside apps, but **redistributing the raw animation files is not**. Keep them out of the public repo; bake them into our own edited clips or skip them |
| Own clips recorded in Blender or with webcam mocap | Signature gestures | Ours |

v1 needs only **3 gestures** (wave, nod, shrug) + procedural idle. That is enough for the demo.

## 2. Looks as code: `avatar.json`

For v1 the "looks" section is a schema-validated code editor with a live preview. A later inventory UI
just writes this same file, so nothing is thrown away.

```jsonc
{
  "$schema": "./avatar.schema.json",
  "model": "models/default.vrm",              // or an imported .vrm
  "framing": { "height": 1.0, "offsetY": -0.1, "cameraFov": 28, "shot": "bust" },
  "materials": {                               // match by material name, glob allowed
    "Hair_*":  { "color": "#2b1d3a", "emissive": "#000000" },
    "Eye_Iris": { "color": "#3a7bd5" }
  },
  "meshes": { "Glasses": { "visible": true }, "Hat_*": { "visible": false } },
  "springBones": { "stiffness": 1.0, "gravity": 1.0, "wind": 0.1 },
  "face": {
    "expressionIntensity": 0.8,
    "blink": { "meanIntervalMs": 3800, "doubleBlinkChance": 0.15 },
    "gaze": { "followCursor": true, "saccades": true }   // cursor is only known over the avatar window (§4.5)
  },
  "idle": { "breathing": 0.6, "sway": 0.3, "animations": ["idle_1.vrma"] },
  "lipsync": {
    "profile": "vrm-basic",                    // default; extended only for custom rigs (§4.4)
    "jawGain": 1.0, "smoothingMs": { "attack": 35, "release": 80 }, "lookaheadMs": 60
  },
  "gestures": { "wave": "gestures/wave.vrma", "nod": "gestures/nod.vrma", "shrug": "gestures/shrug.vrma" }
}
```

- Body animations use **VRMA** (VRM Animation) clips, which three-vrm can load and retarget.
- The live preview renders in the Configure window using the same `packages/avatar` renderer.
- **Apply path:** saving `avatar.json` → the core validates it → `config.changed` → the avatar window
  applies it. Changes to materials, meshes, face and idle apply in place. A change of `model`
  reloads the VRM (cross-fade, ~1 s). Assets load through the core's `/assets` route (01 §4).
- An invalid file keeps the old look and shows the error in the editor (same rule as all config).
- **Model replacement is an owned operation (ADR-0019):** a generation is assigned when the
  selection intent arrives (startup default, `avatar.json` change or import), before the fetch. The
  candidate is prepared (optimise, qualify, frame) while the previous model stays visible; only a
  current, fully prepared candidate is committed. A stale or failed candidate is disposed exactly
  once and its object URLs revoked; a stale completion never changes the visible status.
- **Later ("inventory"):** items become small packages (`item.json` + assets) that patch
  `materials`/`meshes` or attach a mesh to a bone. "Publish" means exporting an item package. This
  stays out of scope for the desktop v1.

## 3. The performance data model

```python
# core/src/svara_core/performance/model.py (sketch)

Viseme = Literal["sil","PP","FF","TH","DD","kk","CH","SS","nn","RR","aa","E","I","O","U"]  # 15, Oculus-style set

@dataclass(frozen=True, slots=True)
class VisemeKey:
    t_ms: float        # offset from the start of this segment's audio, 0 ≤ t_ms ≤ extent
    viseme: Viseme
    weight: float      # 0..1 peak weight

@dataclass(frozen=True, slots=True)
class Cue:
    t_ms: int
    kind: Literal["expression", "gesture", "gaze"]
    name: str          # "happy", "wave", "look_user", …
    intensity: float
    duration_ms: int | None

@dataclass(frozen=True, slots=True)
class PerformanceSegment:
    id: str
    text: str                      # the sentence as spoken (tags removed)
    audio_stream_id: int           # binary audio frames carry this id
    sample_rate: int
    frame_count: int               # authoritative extent (ADR-0017); duration is derived from it
    visemes: tuple[VisemeKey, ...] # sorted by t_ms, every t_ms within the extent
    envelope: bytes                # articulation intensity per 10 ms, u8 (ADR-0017)
    cues: tuple[Cue, ...]          # sorted by t_ms
    word_spans: tuple[tuple[int, int, int, int], ...]  # (start_ms, end_ms, char_start, char_end)
```

**Delivery rule (v1): one segment = one fully synthesised sentence.** The core sends the sentence's
binary audio frames first, then `performance.segment` with the complete viseme track. The renderer
schedules a segment only when both have arrived. Sentence-level synthesis keeps alignment simple and
exact. The cost is time to first audible output on the first sentence. An earlier estimate that
streaming within a sentence would save only 100–300 ms was never measured (engineering review
SV-04): the recorded Kokoro sentences took 0.6–2.2 s to synthesise. The short-first-sentence rule in
04 §2 is the v1 mitigation; T3.13 (and lab unit V-D.2) measure first audible output cold and warm. If
the F12 or 04 §5 targets are missed, the measured options go to the user as a scope decision.
Sub-sentence streaming stays post-v1 until then. The data model allows it later (a segment can be
split into chunks with the same id), but v1 does not build it.

**Why keyframes plus an envelope and not dense per-frame weights:** keyframes are small on the wire
(a sentence is about 40 keys), easy to test (golden files) and independent of frame rate. The renderer
turns them into smooth curves. The envelope carries articulation intensity, so the jaw moves with
the voice. Plosives get their closure from the keyframes, and intensity from the audio.

**Segment contract (ADR-0017):**
- `frame_count` and `sample_rate` are authoritative; milliseconds are derived. Every key, cue and word
  span lies inside the extent. The generator clamps attack/hold/release to it and keeps the sampled
  boundary value; PCM is never padded only to fit animation keys.
- Smoothing continues across back-to-back segments inside the player; the mouth neutralises on
  `performance.end`, stop, discard or a queue gap. There is no cross-segment key format in v1.
- The envelope is computed from the PCM that is actually played (after clipping/quantisation), over a
  per-voice calibrated dB range with an explicit silence floor. It is **not** normalised per sentence:
  all-zero PCM gives an all-zero envelope, and a quieter rendition gives a lower envelope. User volume
  does not change it.
- Core and renderer both reject a segment whose PCM, rate, envelope coverage or keys disagree,
  before any audio starts.

## 4. The viseme engine (core)

### 4.1 Sources of timing, best first

Each TTS adapter declares what alignment it can provide. The engine uses the best one available:

| Tier | Source | Examples | Quality |
|------|--------|----------|---------|
| A | **Phoneme timings** from the TTS itself | Kokoro (`pred_dur` from the timestamped ONNX export, §4.8), Azure TTS viseme events | Best |
| B | **Word or character timings** + our G2P | ElevenLabs `with-timestamps` (character alignment), Cartesia (word timestamps) | Good |
| C | **Text only**, then G2P + forced alignment against the audio | Possible future OpenAI TTS path | Good; costs CPU; not a v1 adapter promise |
| D | **Audio only**: spectral vowel estimation (formant/MFCC-based), computed **in the core** over the finished sentence audio, so it uses the same data model | Anything, including raw audio | Fallback: vowels only, no closures |

Tier C uses a small aligner (e.g. a CTC phoneme model through onnxruntime) and only runs if Tier A/B
are not available. Tiers A, B and D are enough for v1. Tier C is post-v1, **unless** Spike B shows the
chosen default TTS gives no usable timing, in which case Tier C moves into Phase 3b.

### 4.2 Pipeline

```
sentence text ──► normalise (numbers, abbreviations, emoji → words)
              ──► G2P  (espeak-ng subprocess, fixed argv + text on stdin, see §4.6)
              ──► phoneme sequence in the adapter's declared alphabet (ADR-0018)
timing source ──► phoneme timeline [(phoneme, start_ms, end_ms)]
                   · Tier A: direct
                   · Tier B: split each word/char span across its phonemes by per-class duration priors
                             (vowels long, plosives short) and normalise to fit the span
              ──► map phoneme → viseme (data/phoneme_viseme.json, per declared alphabet → 15 visemes;
                   every symbol mapped or explicitly unsupported, never silently `sil`)
              ──► timing rules (the generator owns timing only, no vowel suppression)
                   · bilabials (p b m) → PP reaches weight ≥ 0.9 for ≥ 40 ms (lips visibly close)
                   · labiodentals (f v) → FF, same minimum hold
                   · merge runs of the same viseme, drop keys < 20 ms unless they are closures
                   · add sil at real pauses > 120 ms
                   · every key clamped inside the segment extent (ADR-0017)
              ──► VisemeKey list + provenance (text → spoken text → tokens → sample intervals)
                   + unknown-symbol count
audio         ──► RMS envelope (10 ms hop, u8) ──► attached to the segment
```

### 4.3 Coarticulation and smoothing (renderer, one owner)

Real mouths anticipate the next sound. The `PerformancePlayer` in `packages/avatar` is the **single
owner** of articulation blending (ADR-0018); the generator before it owns timing only and the
retarget after it is a pure linear map. Each step below is applied once:

1. Reads keys with a **lookahead** (~60 ms): a key starts ramping before its timestamp.
2. Uses **attack/release** smoothing per viseme channel (critically damped spring or exponential,
   ~35 ms attack and ~80 ms release), which stops the mouth jittering. The smoothing state carries
   over between back-to-back segments.
3. Keeps **bilabial closures dominant**: while PP is active, open-vowel channels and the jaw are
   suppressed. FF (lower lip to upper teeth) is not a full closure: it reduces jaw opening partially
   and keeps its own output, so it never collapses to a neutral mouth.
4. Applies **jaw gain from the envelope** (articulation intensity, §3), so quiet and loud speech can
   look different within the voice's calibrated range.
5. **Blends with expressions:** mouth-area parts of an expression (e.g. a `happy` smile) are reduced
   while a strong viseme is active, so a smile never fights a closed "m". Expressions whose
   `overrideMouth` setting would suppress visemes are recorded in the rig qualification report.

Its output is final canonical weights plus jaw. Tests assert the **final rig weights** for PP→vowel,
FF→vowel, short closures, overlapping channels, silence and end of speech; testing the generated
curve alone is not evidence of the pose.

### 4.4 Retarget maps (per avatar profile)

The 15 internal visemes map to whatever the model has. This is data, not code:

```jsonc
// performance/data/retarget/vrm-extended.json: default avatar with custom mouth expressions
{ "PP": {"PP": 1.0}, "FF": {"FF": 1.0}, "aa": {"aa": 1.0}, "E": {"ee": 0.8, "aa": 0.2}, "I": {"ih": 1.0},
  "O": {"oh": 1.0}, "U": {"ou": 1.0}, "SS": {"ih": 0.4}, "CH": {"CH": 1.0}, "kk": {"aa": 0.3}, … }

// performance/data/retarget/vrm-basic.json: any VRM with only the 5 vowel presets
{ "PP": {}, "FF": {"ih": 0.2}, "aa": {"aa": 1.0}, "E": {"ee": 1.0}, … }     // {} = mouth closed
// FF/TH/SS/CH on a basic rig are documented approximations; exact weights are tuned in lab V-A.1
// and recorded with the rig report. The map is linear: no extra jaw or PP multiplication here.

// performance/data/retarget/arkit52.json: later, ARKit-rigged GLBs
{ "PP": {"mouthClose": 0.8, "mouthPressLeft": 0.4, "mouthPressRight": 0.4}, … }
```

### 4.5 Expressions and gestures

- **Where they come from:** the LLM is told (through a prompt section registered by the
  `performance` plugin) that it may use inline tags: `[happy]`, `[thinking]`, `[surprised:0.6]`,
  `[gesture:wave]`, `[gesture:nod]`. The **segmenter** strips the tags from the spoken text and turns
  them into `Cue`s timed to the next word. Unknown tags are dropped.
- **Fallback:** if the model uses no tags, a cheap rule-based sentiment on each sentence sets a mild
  default expression. You can switch this off.
- **States drive ambient behaviour** (every change blends with the `pose`/`expression` springs of
  17 §2 and retargets from the current pose if interrupted, ADR-0022): `listening` gives a slight head tilt and gaze at the user;
  `thinking` gives gaze up-left and a slower blink; `acting` (tool running) gives a small "working"
  loop; `error` gives a brief `sad`.
- **Idle layer (always on):** stochastic blinking (Poisson, with occasional double blinks), breathing,
  micro-sway, saccades, and gaze following the cursor (clamped). These are cheap and they make the
  avatar feel alive.
- **Gaze under Wayland:** the global cursor position is not available to our XWayland window when the
  pointer is over Wayland-native apps (02 §1.2). So `gaze.followCursor` only follows the cursor while
  it is over the avatar window. Otherwise the avatar looks at the viewer (camera) with natural
  saccades.
- **Feature flags:** `lipsync`, `expressions`, `gestures`, `idle.gazeFollowsCursor` can each be
  turned off.

### 4.6 Language

**No language-specific work is planned.** The pipeline is language-agnostic by construction:

- **Reply language follows the input.** A built-in prompt rule says "reply in the language the user
  wrote or spoke in". The persona can override it, as free text in `persona.md`. STT auto-detects the
  spoken language.
- **Lip sync can work for other languages** because the viseme table maps phones, not letters, but
  an IPA table does not prove a language's inventory, tokenizer, voice pronunciation or
  coarticulation (SV-07). Only rows of the published qualification matrix (language × voice × rig)
  that passed are claimed; v1 qualifies English with the pinned Kokoro voice and the default rig.
  G2P for every language, English included, is the **`espeak-ng` executable** as a subprocess with a
  fixed configuration argv and the exact text on stdin (`espeak-ng -q --ipa=3 -v <lang> --stdin`;
  100+ languages). Passing text as the final argument let literal text such as `--help` be parsed as
  an option (SV-24); text is never edited to work around that.
  **Which language:** for a spoken turn, the language STT detected; for a typed turn,
  `tts.defaultLanguage` (default `en`). No language-ID library in v1. If espeak-ng doesn't cover the
  language, Tier D (audio-only) still moves the mouth.
- **Voice:** the TTS voice is whatever the user picked. An optional `tts.voicesByLanguage` map
  (`{"de": "…", "es": "…"}`) picks a matching voice when one exists. If the active TTS can't speak the
  language, that turn falls back to bubble-only with a small note, so we never read text aloud with the
  wrong phonology.
- v1 is developed and tested in **English**. Other languages are best-effort and listed as untested
  in the README until a qualification row passes (actual speech, punctuation, numbers, names and
  code-switching, not only table coverage).

### 4.7 Licences to watch

The repo is **Apache-2.0**, and no GPL code is loaded into our process:

- `espeak-ng` is **GPL-3.0**. We run the `espeak-ng` **executable** as a separate process (installed as
  a `.deb` dependency), which is aggregation, not linking. The installer ships its licence text and
  a source pointer.
- **Not used, on purpose:** `phonemizer` / `phonemizer-fork` / `espeakng-loader` (they load the GPL
  espeak library **in-process**), `kokoro-onnx` (which depends on them, and also requires Python
  < 3.14), and `misaki` (which requires Python < 3.13 and pulls in `phonemizer-fork` + spaCy).
- Kokoro-82M model weights and the timestamped ONNX export are Apache-2.0. The Silero VAD model is
  MIT. faster-whisper is MIT, and Whisper weights are MIT.
- The full generated review (`THIRD_PARTY_NOTICES.md`) is a Phase 7 task.

### 4.8 Kokoro: our own thin runner (decided, verified Sep 2026)

- **Model:** the **timestamped** Kokoro v1.0 ONNX export
  (`onnx-community/Kokoro-82M-v1.0-ONNX-timestamped`, fp16, ~163 MiB, Apache-2.0). Unlike the plain
  export, it exposes the graph's predicted per-token duration as the **`durations`** output (the
  internal `pred_dur`) in decoder frames. That is **Tier A timing** for free.
- **Runner (`providers/tts/kokoro.py`, ~150 lines):**
  1. espeak-ng IPA for the sentence (text on stdin).
  2. Map IPA symbols to Kokoro token ids with `model.vocab` from the pinned `tokenizer.json`; the
     tokenizer's inventory is declared, dropped/unknown symbols are counted (ADR-0018).
  3. Run onnxruntime with the tokens, the voice style vector (from the voices file, indexed by token
     count) and speed, holding the model's executor slot until the native call returns (ADR-0019).
  4. Return 24 kHz PCM with its frame count plus `(phoneme, start_frame, end_frame)` from
     `pred_dur` × frame hop.
- **Why not `kokoro-onnx` or `kokoro`:** Python < 3.14 pin and in-process GPL (§4.7), or PyTorch
  (gigabytes).
- **Spike B (T0.8) confirmed** a 600-sample / 25 ms frame hop exactly on five real waveforms. Raw
  envelope-onset error was 40 ms mean / 70 ms max. Subtracting a nominal 35 ms attack gives an
  arithmetic estimate of 25 ms mean / 35 ms max, **not a measured visible onset**. This does not
  pass the audio-to-mouth or true-phoneme accuracy gates. Matching total duration confirms the
  decoder hop, not internal phoneme alignment (SV-11). See `docs/spikes/T0.8-kokoro-lipsync.md`.
- **Known spike findings (unfixed, owned by lab V-A.1–V-A.3):** text in eSpeak's option position
  (SV-24), unknown native symbols mapped to `sil` (SV-09), per-sentence peak-normalised envelope
  (SV-14), release keys beyond the segment (SV-21), and triple closure suppression across
  generator/player/renderer (SV-10, SV-02). See [16 §5](16-lab-track-and-review-findings.md#5-finding-ledger-sv-01sv-24).

## 5. Renderer playback (`packages/avatar`)

```ts
class PerformancePlayer {
  // one AudioContext for the whole app; everything is scheduled on ctx.currentTime
  enqueue(segment: PerformanceSegment, pcm: AudioBuffer): void   // schedules audio + keys + cues
  stop(): void                                                    // barge-in: stop sources, clear queues
  update(nowSec: number, dt: number): FaceFrame                   // called every animation frame
}
```

- Segments are queued back-to-back: `segment.startAt = max(ctx.currentTime + 0.05, prevEnd)`, after
  the segment passed validation (ADR-0017). A rejected segment never starts a source.
- **Clocks:** audio is scheduled on `ctx.currentTime` (graph time). The face and `playback.progress`
  sample an output-position estimate from `ctx.getOutputTimestamp()`
  (`contextTime + (performance.now() − performanceTime)/1000`) when the Electron build provides a
  valid value, else graph time with the fallback recorded. Measured device/display offset
  compensation is kept separate from `lookaheadMs`.
- **Frame pacing:** the render loop uses a phase-preserving deadline and reports deficits
  (ADR-0020); a skipped frame never shifts the mouth in time.
- **Bubble-only mode (no audio):** cues still play. They are scheduled against the bubble's text reveal
  (an estimated 15 characters per second), so expressions and gestures still happen without voice.
- The viseme cursor per frame is a pointer walk over the sorted key array: O(1) amortised, no search.
- `FaceFrame` = weights per expression name. `AvatarRenderer` applies them through
  `vrm.expressionManager.setValue()` and then `vrm.update(dt)`.
- Everything is testable without WebGL: `PerformancePlayer.update` is pure given a time, so we can
  write vitest golden tests ("at t=120 ms the PP channel is ≥ 0.9").

## 6. Lip-sync lab (dev tool and portfolio piece)

A hidden route in the Configure window (`Advanced → Lip-sync lab`):

- Type a sentence and choose a TTS provider/voice. It synthesises and shows a **timeline**: waveform,
  phonemes, viseme keys, envelope and cues, with a scrubber.
- You can play it at 0.25×, compare timing tiers A/B/D side by side, and export the track as JSON (this
  is how golden test files are made).
- It shows **final rig weights**, unknown-phone markers and the graph/output clock difference, and
  can load human-reviewed landmark annotations for a fixed corpus so model interval, reference
  interval and rendered pose appear together. A model's durations compared with themselves are never
  presented as accuracy. An offline aligner (e.g. Montreal Forced Aligner) may supply a reviewed
  reference for evaluation, never runtime timing.
- The bounded lab track (16) builds the evidence this tool will display before the product lab
  exists; its first comparison is one reused baseline on the same corpus and rig.
- A short screen recording of this makes a strong README section.

## 7. Quality bar (exit criteria for lip sync)

- On a fixed set of 30 test sentences, every bilabial /p b m/ produces a PP closure of ≥ 40 ms that
  lands within ±40 ms of the true phoneme time (Tier A). Tested automatically.
- Audio-to-mouth offset is below 45 ms (the perceptual threshold is roughly 45 ms audio-leading).
  This is measured on the rendered avatar against audible output with a calibrated capture
  (14 §5), not by comparing the envelope with the model's own timing.
- 60 fps while speaking on an integrated GPU with zero target deficit (ADR-0020), and < 5% CPU while
  idle.
