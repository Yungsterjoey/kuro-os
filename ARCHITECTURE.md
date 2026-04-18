# KURO OS — Architecture

This document describes the runtime architecture of KURO OS in prose. No
implementation is included; the source lives in a private repository
available to design partners.

---

## 1. Process model

A single Express HTTP server (`kuro-core`) listens on port 3000. A
sidecar process (`kuro-sandbox`) listens on `127.0.0.1:3101` and is the
only host where untrusted code execution is allowed. Both processes are
supervised by PM2 and never auto-restarted by the agent — restarts are
always operator-initiated.

State is held in a SQLite database opened in WAL mode for write
concurrency. Every connection sets `journal_mode = WAL` on open. Schema
migrations are additive: tables are created with `IF NOT EXISTS`, columns
are added under `try/catch` guards, and destructive `ALTER`s are
forbidden in the migration path.

The frontend is a React 18 + Vite SPA built into `dist/` and served as a
desktop-style "OS" shell. Each application window owns its Zustand
store; the shell does not arbitrate between apps.

---

## 2. The L0–L11 layer pipeline

The hot path of the system is `POST /api/stream`. A request traverses
twelve layers in order. Every layer is loaded under `try/catch`; a layer
that fails to load produces a degraded but functional pipeline rather
than a crashed server.

### L0 — iron_dome

Rate limiting and IP banning. Operates on the raw request before any
session is established. Repeat abuse promotes an IP from soft-throttle
to hard-block, with a TTL.

### L1 — guest_gate

Anonymous session provisioning for unauthenticated traffic. Guest
sessions are ephemeral and never granted write capability against the
ledger, audit chain, or memory store.

### L2 — memory

Session memory retrieval. The previous turns of the conversation are
re-hydrated for the model. A separate persistence layer (L11) writes
back at the end of the request.

### L3 — context_reactor

Dynamic context injection. Time of day, operator profile, and the set of
tools currently authorised for the session are folded into the prompt.
This layer is responsible for ensuring the model sees *operational
context*, not just user input.

### L4 — bloodhound

Debug and trace mode. When enabled, every downstream layer emits a
structured trace event, surfacing as a debug stream alongside the
primary response. Off by default in production.

### L5 — iff_gate + semantic_router

Intent classification. `iff_gate` answers a friend-or-foe question (is
this prompt allowed at all?); `semantic_router` then picks a downstream
*temperature* and *route* — fast lookup, deliberative reasoning, tool
use, or a domain-specific module like KURO::PAY.

### L6 — voter_layer

Optional multi-model consensus. When the route requires high confidence,
several model heads are sampled in parallel and a voting policy selects
the winning continuation. Disabled by default for latency reasons.

### L7 — thinking_stream

Extended reasoning. Produces a think-block that is streamed to the UI
but filtered from the user-visible response unless explicitly requested.

### L8 — frontier_assist

Anthropic API fallback. If the local model declines or under-performs on
a given route, this layer dispatches to a frontier API provider and
re-enters the pipeline at L9.

### L9 — output_enhancer + maat_refiner

Artifact extraction (`output_enhancer`) and output purification
(`maat_refiner.purify()`). Code blocks, diagrams, and downloadable
artifacts are split out of the prose stream and offered to the frontend
as separately-renderable objects.

### L10 — audit_chain

Tamper-evident log. Each request produces a hash-linked record of
inputs, intermediate decisions, model identity, and output. This is the
forensic backbone of the operating system.

### L11 — mnemosyneCache

Memory persistence. The session memory updated in L2 is durably written
back here. `edubba_archive.recall()` and `inscribe()` provide semantic
recall against this store across sessions.

---

## 3. Authentication and authorisation

A single middleware, `requireAuth`, gates every private route. It
populates `req.user.userId` (never `req.user.id`), and downstream code
is required to use that identifier. OTP, OAuth (Google), and magic-link
flows all converge on the same session table. Guest sessions are
provisioned at L1 and never escalated.

Capabilities are profile-driven. Each operator profile (`gov`,
`enterprise`, `lab`) declares which agents and tools may be invoked.
The `capability_router` enforces the boundary; the
`agent_orchestrator` performs the routing within it.

---

## 4. Streaming

Server-sent events are produced with `axios` in `responseType: 'stream'`
mode. The Node Web Streams API is **not** used for SSE — empirical
backpressure issues motivated the choice of `axios` as the canonical
streaming HTTP client.

---

## 5. Module mounting

Domain modules (`modules/pay`, `modules/grab`, `modules/wager`, and
`neuro/`) export a router-assembly function that the main server mounts
under a stable prefix. A failed module mount is caught and logged; the
rest of the system stays online. This decouples product launches from
core-server release cadence.

---

## 6. Failure model

The codebase is intentionally tolerant of partial failure:

- A layer that fails to **load** is skipped; the pipeline runs in
  degraded mode and surfaces the degradation in the audit chain.
- A layer that fails to **execute** raises through to L10, which records
  the failure and returns a structured error to the frontend.
- A failed module mount removes its routes but leaves the rest of the
  server running.
- PM2 restarts are *never* automatic. They are always printed for the
  operator.

The shape of the system reflects a single-operator philosophy: the
operating system never takes destructive action on the operator's
behalf without explicit confirmation.

---

## 7. Performance and token economy

The pipeline is architected around algorithmic efficiency targets
(O(n log n) or better on the hot path), strict output formatting
(no conversational filler in agent responses), and aggressive context
pruning. Memoisation is the default for any recursive structure.
Numerical work uses log-sum-exp where probabilities multiply.

These constraints are codified in the operator's `CLAUDE.md` and are
enforced at code-review time rather than at runtime.
