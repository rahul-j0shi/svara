# Documentation assessment and scope closure — 2026-10-10

Request: assess the whole repository's documentation (architecture docs, ADRs, plans, READMEs,
explanation docs) for (1) readability — does each component's what/why/where/how/when come through
by reading; (2) scope — can someone read and start developing without gaps; (3) end goal — is the
vision clear and backed by sensible targets; (4) avatar customisation — is there a settings path for
look, style and persona; (5) avatar smoothness and motion — is it planned end to end. Then improve
the writing and close the scope. **Documentation only;** no code, test, dependency or asset changed.

Base: `main` at `94b1d8a` (after PR #2). Everything under `docs/`, the root and package READMEs were read.

## 1. Findings and what changed

| Factor | Finding before this pass | Change made | Remaining limit |
|---|---|---|---|
| 1. Readability | Individual docs (01–08, 12–14) are precise and specific, but there was no front door. Readers met `F`/`AC`/`T`/`R`/`G`/`G-NAME`/`X`/`B`/`SV`/`V` IDs with no glossary; "where does X live and why" needed four documents; several intros had grown into stacked dated notes (13 §1, 15 intro, revamp README); a stale "Spike A" column in 02 §1.2 | New [00 — Vision and overview](../revamp/00-vision-and-overview.md): end goal, one conversation end to end, component atlas (what/why/where/how/when/today), reading paths by role, glossary. Revamp README restructured (index, decisions, current status); 13 §1, 15 intro and 03 §1 rewritten as present-tense current truth with history left to reviews; 02 column fixed | Dense cross-references (`03 §4.3`) remain by design; the glossary and atlas make them navigable. Historical spike reports keep their dated style as evidence |
| 2. Scope closure | Product scope was complete (40 features, 114 ACs, 89 tasks, gates), but three holes existed: per-frame animation composition, a user control for avatar size, and visual style of transient UI | 03 §5.1 animation stack; F08/T6.2 size slider; 17 §2.1 visual style; owners wired into T3.9/T6.2 and X28 | Work that depends on unavailable environments (Wayland sessions, consent, the tag push) stays listed once in 16 §7 |
| 3. End goal | Vision was scattered (README paragraph, "because this is for a job search" in 09, non-goals in 12) and blurred product vs portfolio motives; targets had no external reference | 00 §1 states the goal (daily companion first; portfolio as outcome), audience, positioning against existing VRM companions, a success table pairing every v1 bar with an external reference and a post-v1 north star, and the milestone path; root README and 09 reframed | No v1 number was changed. The north-star column (e.g. ≤ 0.8 s voice latency) is direction only and needs a future scope decision |
| 4. Customisation | Configure already had Avatar looks (`avatar.json` editor + live preview + import/reset), Persona (editor + test) and Voice (form controls) — file-first by the user's decision R13. Missing: a summary of what can be changed where; any UI for avatar size; a plan for friendlier tools | 02 §6.1 "What you can make your own in v1, and where" (14 rows); size slider; post-v1 **Avatar Studio** in five bounded steps (09) and a 12 §0.2 non-goal row | Form-based looks, presets, persona switching and a voice gallery are deliberately post-v1, each needing its own scoped proposal |
| 5. Smoothness and motion | 17/ADR-0022 (previous pass) covered budgets and choreography, but how the renderer composes idle, state, gesture, gaze, emotion, blink, mouth and physics each frame was unspecified, and there was no per-frame script budget | 03 §5.1: layer table with one owner per channel, frame order, `dt` clamping, hide/restore rule, pure compose function with goldens; B3 adds ≤ 10 ms per-frame script work (RAIL); T3.9 and X28 own it | All smoothness evidence is still to be produced (lab V-E.2, T2.5, T2.11, T3.15) |

## 2. Research used

Checked 2026-10-10. Used only to position targets and the product; no v1 limit was relaxed or added
beyond B3's per-frame budget.

- Voice latency: [Twilio — core latency in cascaded voice agents](https://static0.twilio.com/en-us/blog/developers/best-practices/guide-core-latency-ai-voice-agents)
  (~1.1 s target, 1.4 s upper mouth-to-ear); [Future AGI — measuring voice AI latency](https://futureagi.com/blog/how-to-measure-voice-ai-latency-2026/)
  (~800 ms p95 for conversational flow). Vendor guidance, not standards; boundary definitions differ.
- Turn-taking: [Stivers et al. 2009, PNAS](https://forms.mpi.nl/node/50939) — languages share
  overlap avoidance and short gaps.
- Lip sync: ITU-R BT.1359 detectability ≈ +45 ms audio-leading / −125 ms audio-lagging
  ([summary](https://www.forasoft.com/learn/audio-for-video/articles-audio/lip-sync-itu-r-bt-1359-tolerance-windows),
  [TV Technology](https://www.tvtechnology.com/opinions/av-synchronization-how-bad-is-bad)); F15's < 45 ms
  sits at the stricter edge.
- Responsiveness: [web.dev RAIL](https://web.dev/articles/rail) (respond within 100 ms, ~10 ms of
  script per frame); [web.dev INP](https://web.dev/articles/inp) ("good" ≤ 200 ms). B1's 100 ms is
  stricter than INP and equal to RAIL.
- Comparable products: [AikoMate](https://gg.deals/application/aikomate/), AIvy,
  [Liqu Companion](https://awesome.ecosyste.ms/projects/github.com%2Fcameroncodesstuff%2Fliqu-companion),
  Utsuwa — VRM desktop companions with personas and avatar swapping; none targets Ubuntu Wayland with
  measured lip sync and a permissioned tool engine.

## 3. Files changed

- New: `docs/revamp/00-vision-and-overview.md`, this record.
- Plan: 02 (§1.2 column, §6 size slider, §6.1 customisation map), 03 (§1 wording, §5.1 animation
  stack), 09 (milestone framing, Avatar Studio), 11 (R40–R41, G94–G97, R1–R41), 12 (F08 size slider,
  B3, §0.2 row, change log), 13 (§1, T3.9, T6.2), 14 (X28, R1–R41), 15 (intro, read order), 17 (§2.1,
  §5 link, owners), revamp README (restructured), `check-plan.mjs` (retains R1–R41).
- Root README: "What it is for" and the start-here path.

## 4. Verification

`node docs/revamp/check-plan.mjs` (89 tasks, 40 features / 114 ACs, R1–R41, local links) and
`git diff --check` pass; a repository-wide Markdown link check finds no broken local links outside
code blocks in the verbatim review. No product behaviour was tested; none is claimed.

## 5. Decisions (taken 2026-10-10 at the user's request)

- **End-goal framing: daily companion first, portfolio second** (00 §1). Real daily use forces the
  properties worth demonstrating; demo-first design produces something that only works in the demo,
  and the engineering review already warned against portfolio appeal as an architecture input.
- **North-star targets are after-v1 direction only** (00 §1.1). A v1 ≤ 0.8 s gate would force
  streaming and runtime changes before the basic local path is proven; v1 measures on the same
  boundaries so the gap stays visible.
- **Avatar Studio order:** form controls → voice gallery → look presets → characters (persona + voice
  + look) → inventory (09). Highest daily value first, cheapest reuse second, then dependency order
  (presets need form controls; characters need presets and voices); inventory last because it adds
  packaging, licence and trust work. Each step still needs its own scoped proposal before it starts.
