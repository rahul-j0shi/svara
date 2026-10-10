# Architecture Decision Records

Svara records durable architecture choices here using the process in
[ADR-0001](0001-record-architecture-decisions.md). Product behavior remains authoritative in the
[feature specification](../revamp/12-feature-specification.md). An accepted ADR is a decision, not
a passed feature test; implementation status lives in the
[developer handoff](../revamp/15-developer-handoff.md) and
[lab/finding ledger](../revamp/16-lab-track-and-review-findings.md).

| ADR | Decision | Status |
|---|---|---|
| [0001](0001-record-architecture-decisions.md) | Record architecture decisions | Accepted |
| [0002](0002-support-ubuntu-and-use-xwayland.md) | Support Ubuntu and run the avatar window through XWayland | Accepted |
| [0003](0003-separate-python-core-and-web-shell.md) | Separate the Python core from the web desktop shell | Accepted |
| [0004](0004-use-reversible-plugin-kernel.md) | Use a small reversible plugin kernel | Accepted |
| [0005](0005-use-python-314-structured-concurrency.md) | Use standard Python 3.14 with structured concurrency | Accepted (amended 2026-10-10) |
| [0006](0006-select-desktop-shell-by-measurement.md) | Select Electron as the desktop shell | Accepted |
| [0007](0007-use-vrm-10-avatars.md) | Use VRM 1.0 avatars with a licence-verified, qualified default | Accepted (amended 2026-10-10: not assumed CC0) |
| [0008](0008-build-phoneme-to-viseme-performance-engine.md) | Build a phoneme-to-viseme performance engine | Accepted (amended 2026-10-10) |
| [0009](0009-use-kokoro-with-thin-onnx-runner.md) | Use Kokoro through a thin ONNX runner | Accepted (amended 2026-10-10) |
| [0010](0010-capture-audio-in-core-with-pipewire.md) | Capture microphone audio in the core with PipeWire | Accepted |
| [0011](0011-separate-chat-and-agent-model-ports.md) | Separate chat-model and agent-backend ports | Accepted |
| [0012](0012-build-small-agent-loop-and-append-only-history.md) | Build a small agent loop with append-only history | Accepted |
| [0013](0013-enforce-capability-permissions-in-tool-handlers.md) | Enforce capability permissions inside tool handlers | Accepted |
| [0014](0014-use-jsonc-pydantic-and-live-reconfiguration.md) | Use JSONC, pydantic schemas, and live plugin reconfiguration | Accepted |
| [0015](0015-store-events-in-append-only-sqlite.md) | Store events in append-only SQLite | Accepted |
| [0016](0016-distribute-a-deb-without-auto-update.md) | Distribute an Ubuntu deb without automatic updates | Accepted |
| [0017](0017-make-pcm-frames-the-performance-segment-authority.md) | Make PCM frames the authority for every performance segment | Accepted |
| [0018](0018-give-articulation-one-owner-and-declare-phone-inventories.md) | Give articulation one owner and declare each provider's phone inventory | Accepted |
| [0019](0019-own-asynchronous-operations-by-intent.md) | Own asynchronous operations by intent, from start to final effect | Accepted |
| [0020](0020-pace-frames-by-target-phase-and-report-deficits.md) | Pace frames by target phase and report deficits honestly | Accepted |
| [0021](0021-run-a-bounded-lab-track-with-separate-acceptance-levels.md) | Run a bounded speech/avatar lab track with separate acceptance levels | Accepted |
| [0022](0022-hold-a-liquid-smooth-experience-contract.md) | Hold a liquid-smooth experience contract across every surface | Accepted |

ADRs 0017–0021 record the decisions taken from the
[engineering review, rounds 1–5](../reviews/2026-10-09-engineering-review-rounds-1-5.md); the
[reconciliation](../reviews/2026-10-10-engineering-review-reconciliation.md) maps every finding to
its ADR, document and owning task.
