# Svara

Svara is being rebuilt as an Ubuntu-native, voice-first desktop assistant with an animated 3D avatar.
Its name comes from **svara** (स्वर): voice, tone, musical note, and vowel—the same unit at the heart
of its speech and lip-sync pipeline.

## Where the project stands

Implementation is in **Phase 0**. There is no assistant runtime, CLI, tray or settings window yet;
the only runnable desktop build is a diagnostic spike, not the avatar-only product UI. ADR-0006
selected Electron as the Ubuntu desktop shell after the measured Tauri/Electron comparison.

- **Product route:** the remaining T0.10/T0.8 gates from the
  [foundation audit](docs/reviews/2026-09-27-foundation-audit.md) come first. They are blocked on a
  real Ubuntu 24.04 Wayland session and a 26.04 GNOME VM, which are not yet available. Phase 1 does
  not start until T0.13 passes.
- **Lab track:** a bounded speech/avatar lab (one pinned voice, English, one qualified free rig, a
  frozen corpus, one reuse baseline) proceeds in parallel on spike and pure package code. It fixes
  the contracts found by the engineering review and produces timing evidence. A lab result never
  accepts a product feature. Next unit: **V-A.1**, the final mouth-pose contract.
- **Engineering review:** the [five-round review](docs/reviews/2026-10-09-engineering-review-rounds-1-5.md)
  (24 findings, SV-01–SV-24) is adopted through ADRs 0017–0021 and
  [16 — Lab track & review findings](docs/revamp/16-lab-track-and-review-findings.md). Its findings are
  recorded and owned; none is fixed in code yet. See the
  [reconciliation note](docs/reviews/2026-10-10-engineering-review-reconciliation.md) for what changed.

Evidence is reported at three levels: a *fixture* proves a code path, a *lab result* proves a named
corpus/rig/device outcome, and only the integrated acceptance checks accept a product feature.

## Documentation

- [`docs/revamp/`](docs/revamp/README.md) — the plan. The authoritative v1 scope is
  [12 — Feature specification](docs/revamp/12-feature-specification.md); the ordered backlog is
  [13 — Implementation guide](docs/revamp/13-implementation-guide.md); acceptance owners and gates
  are in [14](docs/revamp/14-acceptance-and-integration.md).
- **Start here:** [15 — Developer handoff](docs/revamp/15-developer-handoff.md), then
  [16 — Lab track & review findings](docs/revamp/16-lab-track-and-review-findings.md).
- [`docs/adr/`](docs/adr/README.md) — architecture decisions (accepted decisions, not passed tests).
- [`docs/spikes/`](docs/spikes/) and [`docs/reviews/`](docs/reviews/) — evidence and reviews.

## Known corrections

- The default avatar is **not** a CC0 VRoid preset: the official VRoidPreset A–Z terms say they are
  not CC0. T0.12 selects one exact, licence-verified free VRM and records its actual terms
  (ADR-0007). No avatar asset has been committed.
- The previous browser-based prototype is preserved in Git history at merge commit `2389194`. The
  planned `v0-prototype` tag has **not** been pushed to the remote; publishing it awaits approval.
- Lip sync is built on IPA, but only the qualified path (English, the pinned Kokoro voice, the
  default rig) is claimed. Other languages are best-effort and untested.

## Development

The repository uses Node 24 with pnpm 12, uv-managed Python 3.14, and GNU Make. Install `uv`, then
bootstrap the locked Python and TypeScript workspaces:

```bash
make setup
```

Use `make lint`, `make test`, and `make gen` for the same checks run by CI. For planning edits, also
run `node docs/revamp/check-plan.mjs`. The `dev` and `demo` targets become runnable after their
corresponding core and desktop tasks are implemented.

VRM and VRMA assets are tracked with Git LFS. Install Git LFS before adding or checking out model
assets; the first production avatar is intentionally deferred to T0.12.
