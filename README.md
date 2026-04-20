# KURO OS

**Sovereign AI Operating System**

KURO OS is a self-hosted intelligence platform that runs end-to-end on
GPU-accelerated hardware with no third-party cloud inference. It exposes a
desktop-class graphical environment, a streaming reasoning pipeline, and a
modular set of products (payments, scheduling, trading, biological state
modelling) over a single Express server.

It is the operating system Henry George Lowe-Sevilla and KURO Technologies
use as the substrate for every other product in the KURO portfolio.

---

## What it is

| Layer | Purpose |
|---|---|
| **Hardware** | RTX 5090 32GB + TensorDock V100 satellite |
| **Inference** | Local Ollama. `huihui-ai/qwen3.5-abliterated:9b` primary, Anthropic frontier as fallback |
| **Server** | Node.js + Express (CommonJS), SQLite via `better-sqlite3` (WAL) |
| **Frontend** | React 18 + Vite, served as a desktop-style OS shell |
| **Process supervisor** | PM2 (`kuro-core`, `kuro-sandbox`) |

Everything is single-tenant. There is no central account server.
Authentication, billing, audit, and memory all live on the operator's box.

---

## Architecture (one-pager)

```
                    ┌────────────────────────────────────────┐
                    │   React 18 OS Shell (App.jsx + apps/)  │
                    └───────────────────┬────────────────────┘
                                        │   POST /api/stream
                                        ▼
   ┌────────────────────────────────────────────────────────────────┐
   │                    L0  iron_dome     - rate / abuse            │
   │                    L1  guest_gate    - anon session            │
   │                    L2  memory        - session history         │
   │                    L3  context_reactor - time/profile/tools    │
   │                    L4  bloodhound    - debug / trace           │
   │                    L5  iff_gate + semantic_router              │
   │                    L6  voter_layer   - multi-model consensus   │
   │                    L7  thinking_stream - extended reasoning    │
   │                    L8  frontier_assist - Anthropic fallback    │
   │                    L9  output_enhancer + maat_refiner          │
   │                    L10 audit_chain   - tamper-evident log      │
   │                    L11 mnemosyneCache - memory persistence     │
   └────────────────────────────────────────┬───────────────────────┘
                                            │
                ┌───────────────────────────┼───────────────────────────┐
                ▼                           ▼                           ▼
        modules/pay/  (KURO::PAY)    neuro/  (NeuroKURO)        modules/wager/
        x402 + EMVCo QR              circadian phase v1.0        trading engines
        7 SEA rails                  MAE 0.31h                   Quantum / Tesla
```

See [ARCHITECTURE.md](ARCHITECTURE.md) for the layer pipeline in prose and
[TREE.md](TREE.md) for a directory map.

---

## Features

- **L0–L11 streaming reasoning pipeline.** Every prompt traverses eleven
  layers, each loaded under try/catch so a single failed layer degrades
  rather than crashing the request.
- **Local-first inference.** Default routing goes to a local Ollama model.
  Anthropic frontier is used only as a fallback inside `frontier_assist`.
- **Tamper-evident audit log.** `audit_chain` produces hash-linked records
  of every reasoning event for forensic replay.
- **Capability-based tool routing.** `capability_router` and
  `agent_orchestrator` gate which tools a session may call based on profile
  (`gov` / `enterprise` / `lab`).
- **Desktop-style frontend.** React shell renders applications in windows
  with their own state stores; `KuroPayApp`, `MessagesApp`, `AuthGateApp`
  and others compose against the same backend.
- **Sovereignty dashboard.** A live system-health view of every layer plus
  PM2, GPU, and disk telemetry.
- **Modular products.** `modules/pay`, `modules/grab`, `modules/wager`,
  and `neuro/` mount under their own route prefixes and can be enabled or
  disabled per deployment.

---

## Status

- Server version: **v7.0.3** (1-GPU commercial build).
- Package version: **9.0.0**.
- KURO::PAY: scaffolded; needs live Wise + Solana keys.
- NeuroKURO: validated, patent-pending, manuscript under review.
- KURO::FLUX / KURO::HUNT: spec complete, not yet deployed.

---

## Source

KURO OS source is **not** open. This repository contains the public
specification, architecture documentation, and example payloads. The full
implementation is available to design partners and licensees under a
commercial agreement.

**Contact:** henry@kuroglass.net

---

## License

See [LICENSE](LICENSE). Source-available on request.
