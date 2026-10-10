# 00 — Vision and overview: start here

This is the front door to the plan. It says what Svara is for, how to tell whether it succeeded, how
one conversation flows through the system, where every component lives and why it is built that way,
and what the plan's vocabulary means. Every other document goes deeper into one part.

## 1. The end goal

**Svara is a small, always-present voice companion for the Ubuntu desktop.** You talk to it like a
person; it answers out loud with a face that visibly speaks; it can act on your computer only as far
as you allow; and it uses the AI provider you choose, including fully local operation. Idle, it is a
calm character in the corner of the screen that costs almost nothing and never steals focus. In use,
it feels immediate and fluid — **liquid smooth** — from the first click to the last spoken word.

**Who it is for.** One person on one Ubuntu 24.04/26.04 machine, using it daily: quick spoken
questions while working, reading and summarising local files, small desktop actions behind
approvals, and a hands-free conversation when typing is inconvenient (12 §0.1). The same build is
also the project's public engineering evidence. That is an *outcome* of building it well, not a
design input: no component exists only to impress a reviewer (ADR-0004, ADR-0021). **Decided
2026-10-10:** daily companion first, portfolio second. Building for real daily use forces exactly the
properties a reviewer should see: low latency, clean interruption, smoothness, safety and honest
measurements. Building for the demo first tends to produce something that only works in the demo.

**Why it is worth building.** Existing VRM desktop companions — for example
[AikoMate](https://gg.deals/application/aikomate/), AIvy,
[Liqu Companion](https://awesome.ecosyste.ms/projects/github.com%2Fcameroncodesstuff%2Fliqu-companion)
and Utsuwa — show that people want a character on the desktop with a persona and swappable avatar.
Svara's distinct combination is: native Ubuntu GNOME Wayland support, measured phoneme-level lip
sync, real desktop tools behind a permission engine and audit log, any LLM provider including a
Claude subscription, and a local-first, no-telemetry default. (Products checked 2026-10-10; Svara does
not depend on or copy any of them.)

### 1.1 What success looks like

The v1 bar is the acceptance contract in 12/14; nothing here relaxes it. The reference column shows
where each target sits against published guidance, so the targets are understood rather than
arbitrary. **Decided 2026-10-10:** the north-star column is direction for after v1 only; it is
**not** a v1 requirement and needs a scoped decision before work starts. Making ≤ 0.8 s a v1 gate would
force sub-sentence streaming and runtime changes before the basic local path is even proven (Kokoro
currently takes 0.6–2.2 s per sentence on CPU). v1 measurements use the same boundaries as the north
star (speech end → first audible output, 14 §5), so the remaining gap is always visible.

| Outcome | v1 bar (normative) | External reference | North star (post-v1) |
|---|---|---|---|
| Responsive conversation | Speech end → avatar audibly talking p50 ≤ 2.0 s, stretch 1.2 s (04 §5) | Cascaded voice-agent guidance targets ~1.1–1.4 s mouth-to-ear ([Twilio](https://static0.twilio.com/en-us/blog/developers/best-practices/guide-core-latency-ai-voice-agents)) and ~0.8 s p95 for conversational flow ([Future AGI](https://futureagi.com/blog/how-to-measure-voice-ai-latency-2026/)) | ≤ 0.8 s p50, e.g. via sub-sentence streaming — a later scope decision (03 §3) |
| A face that speaks | Mouth/audio offset < 45 ms; every /p b m/ visibly closes ≥ 40 ms (F15) | ITU-R BT.1359 detectability ≈ 45 ms audio-leading / 125 ms audio-lagging ([summary](https://www.forasoft.com/learn/audio-for-video/articles-audio/lip-sync-itu-r-bt-1359-tolerance-windows)) | Extended-viseme rigs and more qualified languages (16 §6 parked) |
| Feels immediate | Any input acknowledged ≤ 100 ms (12 §0.5 B1) | RAIL: respond within 100 ms ([web.dev/rail](https://web.dev/articles/rail)); INP "good" ≤ 200 ms ([web.dev/inp](https://web.dev/articles/inp)) | Same |
| Liquid smooth | 60 fps with ≥ 99% of frames ≤ 25 ms during motion, per-frame work ≤ 10 ms, no pops or clicks (12 §0.5 B2–B9) | RAIL: ~10 ms of script per 60 fps frame | Same on high-refresh displays |
| Interruptible | Barge-in silence ≤ 350 ms; Stop ≤ 300 ms (F13, F27) | Human turn-taking avoids overlap and long gaps across languages ([Stivers et al. 2009](https://forms.mpi.nl/node/50939)) | Smarter turn-taking (backchannels) — outside F13 |
| Unobtrusive presence | Idle shell < 5% and core < 3% of one core; core < 250 MB; hidden = 0 fps (F02, 01 §3a) | — | Lower on battery |
| Safe and private | Read-only by default; approvals and audit; no telemetry; fully offline mode works (F24, F34, T5.5) | — | Same |
| Yours | Persona, voice, looks and every feature switchable live (F08, F30) | — | Avatar Studio (09 "After v1") |

A target that fails stays failed until it passes on recorded evidence; changing a target is a spec
change with the user's approval (12, "Changing this specification").

### 1.2 Milestones toward it

1. **Foundation (Phase 0)** — prove the platform, runtime and assets: Wayland presence, dependencies,
   timing, the default avatar (13 §8, T0.13 exit).
2. **Lab case study (parallel)** — a trustworthy speech→face path on one rig and corpus, honestly
   labeled as a lab result (16, ADR-0021).
3. **Text desktop (Phases 1–2)** — the character lives on the desktop and answers typed questions.
4. **Voice (Phase 3)** — a full spoken conversation with lip sync and interruption.
5. **v0.5 preview (Phase 4a)** — read-only desktop tools with approvals, a `.deb`, measured README.
6. **v1 (Phases 4b–7)** — act tools, MCP, skills, every setting, onboarding, final packaging.
7. **After v1** — browser extension (10), Avatar Studio and the items in 09 "After v1".

## 2. One conversation, end to end

You double-click the character (or press the Talk shortcut). Within 100 ms it turns attentive and,
once the **core** confirms the microphone is open, a glow appears (F04). The core captures audio from
PipeWire, the voice-activity detector notices when you stop, and speech-to-text produces your words,
shown as a faint ghost line. The **agent** sends the conversation to your chosen model, streaming text
back; if the model wants a tool, the **permission engine** allows it, denies it or asks you in a bubble.
As text streams, the **segmenter** cuts it into sentences; each sentence is synthesised to audio and
turned into a **performance** — timed mouth shapes, a loudness envelope and expression cues. The
**shell's** avatar window plays the audio and drives the face from the same clock, revealing bubble
words as they are spoken. Speak again mid-answer and the turn is cancelled, audio fades out and Svara
listens. Everything that happened is appended to a local event log you can browse in Conversations.

Read next: 01 (architecture), 04 (voice), 03 (avatar), 06 (tools), 17 (how it must feel).

## 3. Component atlas

Paths marked *target* do not exist yet. "Today" is the state on `main`; 15 §1 has detail.

| Component | What it does | Why built this way | Where | How it works | When | Today |
|---|---|---|---|---|---|---|
| Desktop shell | Windows, tray, menus, click-through, CLI actions, starts/stops the core | Chromium's mature Linux GPU path; XWayland for positioning (ADR-0002/0003/0006) | `apps/desktop/electron/` | 02 §1–3, 01 §3a | T2.1, T2.3, T2.7 | Diagnostic probe |
| Avatar window UI | Bubble, type box, approvals, gestures, states | Character-only idle surface (F02) | `apps/desktop/src/windows/avatar/` *target* | 02 §3–5, 17 | T2.3, T2.6, T2.8, T4.3 | Diagnostic page only |
| Avatar renderer | Loads the VRM, runs the animation stack, plays performances | Framework-free so the extension/web demo reuse it (ADR-0007/0008) | `packages/avatar/src/` | 03 §5, 17 | T2.4, T2.5, T3.8, T3.9 | Spike code, known findings (16 §5) |
| Kernel | Mounts/unmounts plugins and reverts their effects | Live toggles and provider swaps without restarts (ADR-0004) | `core/src/svara_core/kernel/` | 01 §2 | T1.1 | Scaffold |
| Config, storage, secrets | JSONC files, live reload, SQLite event log, keychain | Human-editable files; one auditable log (ADR-0014/0015) | `config/`, `storage/`, `secrets/` *target* | 08 | T1.2–T1.5 | Scaffold |
| Protocol | Typed WebSocket messages and audio frames between core and windows | One contract for every surface (ADR-0003/0017) | `protocol/`, `packages/protocol/` | 01 §4, 13 §7 | T1.6, T2.2 | Placeholder |
| Agent and model providers | The tool-calling loop and LLM adapters | Provider freedom; Claude subscription via the Agent SDK (ADR-0011/0012) | `agent/`, `providers/llm/` | 05, 06 §1 | T1.7–T1.12, T4.0, T5.3 | Scaffold |
| Voice pipeline | Mic, VAD, STT, segmenter, TTS, barge-in | Native capture with echo cancellation; local-first models (ADR-0005/0009/0010) | `audio/` *target*, `pipeline/`, `providers/stt/`, `providers/tts/` | 04 | T3.1–T3.6, T3.11 | Kokoro spike |
| Performance engine | Phones → visemes, envelope, cues | Lip sync as a pipeline stage, not a UI effect (ADR-0008/0017/0018) | `performance/` | 03 §3–4 | T3.7 | Spike only |
| Tools and permissions | Desktop tools, rules, approvals, audit | Safe by default; enforcement inside handlers (ADR-0013) | `tools/`, `tools/desktop/` | 06 | T4.1–T4.14 | Not started |
| MCP, skills, persona | External tools, reusable instructions, who the avatar is | Open formats users already have (07) | `mcp/`, `skills/`, `persona/` | 07 | T1.7, T5.1, T5.2 | Not started |
| Conversations and Configure | History viewer and every setting | On-demand windows, never beside the avatar (F07/F08) | `apps/desktop/src/windows/` *target* | 02 §6–7 | T2.9, T2.10, T6.1 | Not started |
| Packaging | `.deb`, removal, notices | One supported install path (ADR-0016) | packaging scripts *target* | 09 Phase 7 | T4.16, T7.1–T7.3 | Not started |
| Lab track | Bounded speech→face evidence | Prove contracts before breadth (ADR-0021) | `spikes/`, `packages/avatar/` | 16 | V-A.1 next | In progress (planning) |

## 4. Reading paths

| You want to… | Read |
|---|---|
| Understand the product in 15 minutes | This page → [README](README.md) principles → 17 §1–3 |
| Start developing today | [15](15-developer-handoff.md) → [16](16-lab-track-and-review-findings.md) → the task's row in [13 §8](13-implementation-guide.md#8-ordered-backlog) → its features in [12](12-feature-specification.md) and checks in [14](14-acceptance-and-integration.md) |
| Work on the avatar, animation or lip sync | 03 → 17 → ADR-0017/0018/0020/0022 → 16 §3 |
| Work on the core or voice | 01 → 04 → 05/06 → 08 → ADR-0004/0005/0019 |
| Review decisions | [ADR index](../adr/README.md) → [reviews](../reviews/) |
| Check scope boundaries | 12 §0 (global rules, non-goals, budgets) |

**Which document wins:** 12 for scope; 13 for task order; 14 for acceptance; ADRs for decisions;
everything else explains. A contradiction is fixed in 12 first (15 §2).

## 5. Glossary

| Term | Meaning |
|---|---|
| Core / shell / surface | The Python process that owns all logic; the Electron app; any window or client connected to the core |
| Plugin, fiber, effect | A unit the kernel mounts; its lifetime; a registration undone automatically on unmount (01 §2) |
| Turn, generation | One user input and the reply; the identity used to ignore late results from a cancelled turn |
| Segment / performance | One synthesised sentence: PCM audio plus timed visemes, envelope and cues (03 §3) |
| Phone, viseme | A speech sound; the mouth shape for it (15 canonical labels, five vowels on a basic rig) |
| Timing tier A/B/C/D | Where mouth timing comes from: TTS phonemes / word or character timings / forced alignment / audio only (03 §4.1) |
| Envelope | Per-10 ms articulation intensity from the played audio, driving jaw opening |
| Retarget profile | Map from canonical visemes to a rig's expressions (`vrm-basic`, `vrm-extended`) |
| VRM / VRMA | The avatar file format / its animation clip format |
| XWayland, portal | X11 compatibility inside GNOME Wayland; the permission-mediated desktop API |
| AEC, barge-in, duck-and-gate | Echo cancellation; interrupting by speaking; the no-AEC fallback |
| Spike | Disposable measurement code under `spikes/` or the diagnostic page, never product |
| Fixture / lab result / product acceptance | The three evidence levels (ADR-0021) |
| `Fnn`, `AC-Fnn.k` | Feature and its acceptance criterion (12, 14 §4) |
| `Tn.m` | Backlog task (13 §8) |
| `Rnn` | Original wishlist requirement (11 §1) |
| `Gnn` vs `G-NAME` | A gap fixed in an earlier review (11 §2) vs a Phase 0 decision gate such as G-PRESENCE (14 §2) |
| `Xnn` | Cross-feature regression check (14 §3) |
| `Bn` | Experience budget (12 §0.5) |
| `SV-nn` | Engineering-review finding (16 §5) |
| `V-…` | Lab-track work unit (16 §3) |
