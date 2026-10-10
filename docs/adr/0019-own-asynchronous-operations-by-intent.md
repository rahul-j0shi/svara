# 0019 — Own asynchronous operations by intent, from start to final effect
Status: Accepted

Context: Several boundaries already had local generation guards, yet the review reproduced stale work
taking effect because ownership started too late or ended too early:
a delayed startup avatar fetch replaced and disposed a later manual selection (SV-18); a stale
startup rejection overwrote the status of the successful newer avatar (SV-19); a candidate whose
preprocessing threw was never disposed (SV-20); the managed-hints focus bridge executed twenty
obsolete opens, each calling `focus()`, after the user asked to close (SV-23); and cancelling a
Python await does not stop native inference or free its slot, while measured Kokoro runs take
0.6–2.2 s, not "a few hundred ms" (SV-05). The renderer's existing in-loader fence and disposal fence
passed and are kept. See
[01 §5](../revamp/01-architecture.md#5-concurrency-model-and-the-python-314-question).

Decision — one small principle, specialized implementations, no new framework:
- **Identity at intent.** An operation's generation is assigned when the user/system intent arrives
  (selection, open/close, utterance), before its first await, and is checked after every await that
  precedes a state change.
- **Every effect is owned:** scene installation, status text, object URLs, focus calls, native window
  hints, audio sources and native job slots publish only while their operation is current. A stale
  completion is a typed *stale* result, logged with its identity, never a user-facing failure.
- **Prepare / commit / rollback** for replaceable resources (avatars, models): a candidate is owned
  from load completion; optimization, qualification and framing happen before the previous resource
  is released; any pre-commit failure or staleness disposes the candidate exactly once; after commit,
  ownership moves to the renderer. Observer-callback failures after commit do not undo the commit.
- **Current-state requests are coalesced, not queued.** Focus/interaction state is a desired state:
  at most one active helper operation and one latest pending state. If an in-flight native helper
  changed hints, reconcile to the latest desired state afterwards. Existing sender checks remain.
- **Native work keeps its slot until native exit.** One active job per model, a cap on submitted (not
  only active) work, results fenced by turn generation, and the slot released only when the native
  call actually returns. Per-run ONNX Runtime termination (`RunOptions.terminate`) is an optional
  experiment if bounded completion misses an existing target, never a replacement for fencing.
- Keep the accepted small kernel (ADR-0004). This principle is applied per boundary, not by adding a
  scheduler abstraction or dispatch modes.

Consequences:
- Tests drive explicit deferred-promise/fake-helper schedules (late success, late failure, failure
  before commit, disposal during load, open→close before/during/after helper completion, repeated
  cancel/replace) and assert bounded queues, final state and single disposal.
- Mock schedules prove ordering; real desktop focus, GPU memory and native exit timing remain
  separate platform/lab measurements.
- ADR-0005's "results from a cancelled turn are discarded" stays, with the added slot-ownership rule.

Evidence:
- Review rounds 4–5 controlled reproductions using the actual renderer, spike app fetch chain and
  Electron main-process source with mocked dependencies.
- Implementation: lab units V-B.2, V-E.1, V-C.1 (V-C.2 optional); product owners T2.3, T2.4, T3.6,
  T3.15 ([16](../revamp/16-lab-track-and-review-findings.md)).
