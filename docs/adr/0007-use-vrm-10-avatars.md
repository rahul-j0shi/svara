# 0007 — Use VRM 1.0 avatars with a licence-verified, qualified default
Status: Accepted (amended 2026-10-10: the default is no longer assumed to be CC0)

Context: Svara needs a portable humanoid format with standardized expressions, look-at behavior, and
mouth blend shapes. Ready Player Me is no longer a viable platform dependency, and a proprietary
avatar service would make the product and portfolio demo fragile. The asset and renderer design is
in [Avatar §1](../revamp/03-avatar-and-lipsync.md#1-avatar-format-vrm-10-not-ready-player-me).
The original decision named "a CC0 VRoid preset". The engineering review (SV-01) found that the
official VRoidPreset A–Z terms state that those presets are **not** CC0, while some other VRoid models
(for example β AvatarSample_1) are designated CC0. No asset had been committed, so this is a plan
correction, not a distribution incident.

Decision:
- Use VRM 1.0 rendered by three.js and `@pixiv/three-vrm`.
- Ship one **exact, named** free avatar whose actual terms permit redistribution inside the `.deb` and
  this Apache-2.0 repository. Selection order for T0.12: (1) a model that its publisher designates
  CC0 (verified on the publisher's page at selection time); (2) otherwise a VRoid preset used under
  its actual, non-CC0 conditions, recorded as such, only if those conditions permit this
  redistribution. Never relabel an asset's licence.
- Record in `assets/LICENSES.md`: asset name, source URL, terms URL and the terms' wording as checked
  on a stated date, SHA-256, export tool/version, VRM version, embedded VRM metadata, and every
  modification made.
- **Qualify before use** (ADR-0018): a committed report using three-vrm introspection — humanoid
  bones/head, presence and non-empty binds of `aa ee ih oh ou`, `isBinary`, `overrideMouth`, a
  0/0.25/0.5/0.75/1 mouth sweep, intended expression interaction, and geometry/material/texture
  counts. The report selects `vrm-basic` or `vrm-extended` or records degraded capability. If the
  first asset fails, evaluate one replacement; further searching needs a named missing capability.
- Use the Khronos glTF Validator as a developer/asset-preparation check for container validity only;
  it does not validate `VRMC_vrm`, licences or visemes, and it is not added to the runtime.
- Load rendering through the framework-agnostic `@svara/avatar` package; keep per-avatar appearance,
  framing, retargeting and animation choices in `avatar.json` rather than forking model files.
- Track VRM and VRMA binaries with Git LFS and record all asset licenses.
- Stay on the stable three-vrm line already pinned (3.5.x); a beta upgrade is not a fix for
  ownership or timing defects and needs its own regression evidence.

Consequences:
- Avatars remain local, inspectable, replaceable, and reusable by the future extension or web demo.
- The renderer must handle VRM coordinate conventions and validate required humanoid and expression
  metadata; loads follow the prepare/commit/rollback ownership of ADR-0019.
- Models with different mouth shapes need explicit retarget profiles. No custom fifteen-shape art is
  required for v1; a basic rig's consonant approximations are documented, not hidden.
- Svara does not provide an avatar marketplace or depend on a hosted creator in v1.

Evidence:
- T0.12 selects the asset, commits it through LFS, records its licence and commits the qualification
  report (lab unit V-B.1 produces the same report). T2.4 verifies loading, framing, and live
  application of `avatar.json`.
- [Engineering review SV-01](../reviews/2026-10-09-engineering-review-rounds-1-5.md#2-cumulative-finding-ledger)
  cites the [VRoidPreset A–Z terms](https://vroid.pixiv.help/hc/en-us/articles/4402394424089-VRoidPreset-A-Z)
  and the [CC0 β AvatarSample_1 page](https://vroid.pixiv.help/hc/en-us/articles/360012381793-%CE%B2-Ver-AvatarSample-1);
  T0.12 re-checks both at selection time.
