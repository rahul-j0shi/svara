# 0018 — Give articulation one owner and declare each provider's phone inventory
Status: Accepted

Context: The review reproduced two shaping bugs and two input-contract gaps in the performance path.
Closure suppression was applied three times (generator hold/release, player vowel and jaw suppression,
renderer `× jaw × (1 − PP)`), shrinking the vowel after a 25 ms /p/ to about 0.18% of full scale at
50 ms (SV-10). FF was counted as a bilabial-style closure and then retargeted as `ih × 0.2 × jaw`, so a
full labiodental became a neutral closed mouth on the basic rig (SV-02). The runner filtered eSpeak
IPA through the model vocabulary and mapped unknown symbols to `sil`, so a native symbol such as `A`
or `Q` silently became silence (SV-09). Text was passed in eSpeak's option position, so literal text
such as `--help` is parsed as an option (SV-24). See
[03 §4](../revamp/03-avatar-and-lipsync.md#4-the-viseme-engine-core).

Decision:
- **Generator (core) owns timing only:** phone intervals, the minimum-closure hold (/p b m/ ≥ 40 ms at
  weight ≥ 0.9, per F15) and key placement inside the segment extent (ADR-0017). It does not apply
  vowel suppression.
- **`PerformancePlayer` is the single owner of articulation blending:** lookahead, attack/release,
  coarticulation, envelope jaw gain and closure dominance, each applied exactly once, producing final
  canonical weights plus jaw. Closure dominance is driven by bilabial PP. FF is labiodental: it reduces
  jaw opening partially but is not a full closure and never zeroes its own output.
- **Retargeting is a pure linear map** from final canonical weights to the rig's expressions, with no
  further multiplication by jaw or PP. Profiles state their approximations: `vrm-basic` has five
  vowels, so PP is "mouth closed", and FF/TH/SS/CH are named approximations, not distinct shapes.
  The 15 internal labels do not imply 15 visible shapes on a basic rig.
- **Each TTS/G2P adapter declares its alphabet** (inventory, normalization version, pause symbols).
  Every inventory symbol is mapped to a viseme or explicitly marked unsupported. Pause, unsupported
  phone and best-effort shape are distinct outcomes; unknown-symbol counts and fallback provenance are
  recorded. Unsupported symbols never become silent `sil`.
- **Provenance is preserved:** original text → normalized spoken text → provider tokens → sample
  intervals, so captions, word spans and interruption notes refer to the user's words.
- **Literal G2P input:** `espeak-ng` is invoked with a fixed configuration argv (`-q`, IPA mode,
  voice) and `--stdin`; the exact text goes through stdin with an explicit UTF-8 policy, return-code
  checking, the existing timeout and the active-job ownership of ADR-0019. Text is never edited to
  avoid option parsing.
- **Rig capability is qualified, not assumed:** the asset report (ADR-0007) uses three-vrm's
  expression introspection (presence, binds, `isBinary`, `overrideMouth`, 0/0.25/0.5/0.75/1 sweeps)
  to choose a profile or report degraded capability.

Consequences:
- Tests assert *final* rig weights for PP→vowel, FF→vowel, short closures, overlapping channels,
  silence and end of speech; intermediate-curve tests alone are not evidence.
- A provider swap cannot hide an inventory gap: the contract suite fails on an unmapped symbol.
- Language claims are bounded by a published qualification matrix (language × voice × rig). English
  with the pinned Kokoro voice is the qualified v1 path; other espeak-ng languages are best-effort.
- Custom consonant shapes remain optional (`vrm-extended`), added only when a rig really has them.

Evidence:
- Review rounds 1–2 and 5 fixtures (player/renderer arithmetic, timeline builder on native symbols,
  phonemizer argv capture) and eSpeak NG's documented `--stdin` and `getopt_long` parsing. No real
  eSpeak run or perceptual test was part of that evidence.
- Implementation: lab units V-A.1 and V-A.3 now; product regressions in T3.6/T3.7/T3.8
  ([16](../revamp/16-lab-track-and-review-findings.md)).
