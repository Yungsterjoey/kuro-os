# KURO OS — Directory Tree

This is a public-facing map of the repository layout. File contents are
*not* included; this document describes what each file or directory is
for.

```
kuro-os/
├── server.cjs                     Express HTTP server v7.0.3, mounts every route
├── package.json                   v9.0.0; CommonJS throughout
├── vite.config.js                 Frontend build configuration
├── index.html                     SPA entry
├── public/
│   └── index.html                 kuroglass.net static landing page
├── dist/                          Built React SPA (artifact)
│
├── layers/                        Cognitive pipeline + supporting layers
│   ├── auth/                      Auth, DB schema, OTP, OAuth (Google)
│   ├── liveedit/                  Streaming controller (SSE via axios)
│   ├── shadow/
│   │   └── mnemosyneCache.js      L11 — memory persistence
│   ├── vision/                    GPU mutex, vision orchestrator, vision routes
│   ├── tools/                     Context router, VFS tools, policies
│   ├── vfs/                       Virtual filesystem
│   ├── search/                    Search layer
│   ├── web/                       Web layer (HTTP fetch tools)
│   ├── git/                       Git integration tools
│   ├── runner/                    Code execution runner (sandbox-bound)
│   ├── preempt/                   Preemption control + routes
│   ├── observability/             Metrics + tracing
│   ├── security/                  Security hardening primitives
│   ├── stripe/                    Stripe billing integration
│   ├── iron_dome.js               L0 — rate limiting, IP banning
│   ├── guest_gate.js              L1 — anonymous session gate
│   ├── memory.js                  L2 — session memory retrieval
│   ├── context_reactor.js         L3 — dynamic context injection
│   ├── bloodhound.js              L4 — debug/trace mode
│   ├── iff_gate.js                L5a — friend-or-foe / intent gate
│   ├── semantic_router.js         L5b — intent routing + temperature
│   ├── voter_layer.js             L6 — multi-model consensus
│   ├── thinking_stream.js         L7 — extended reasoning + think-block filter
│   ├── synthesis_layer.js         Model synthesis pipeline
│   ├── frontier_assist.js         L8 — Anthropic API fallback
│   ├── output_enhancer.js         L9a — artifact extraction
│   ├── maat_refiner.js            L9b — output purification (purify())
│   ├── audit_chain.js             L10 — tamper-evident log
│   ├── agent_orchestrator.js      Agent routing + skill gates
│   ├── fire_control.js            Safety circuit breaker
│   ├── edubba_archive.js          recall() / inscribe() semantic memory
│   ├── mcp_connectors.js          MCP file/terminal/session connectors
│   ├── sms_forward.cjs            SMS forward: AU number → VN number
│   ├── sandbox_routes.cjs         Code execution sidecar routes
│   ├── capability_router.cjs      Capability-based routing
│   ├── sovereignty_dashboard.js   System health dashboard
│   ├── self_heal.js               Auto-recovery watchdog
│   ├── model_warmer.js            Model warmup on start
│   ├── flight_computer.js         Mission control logic
│   ├── smash_protocol.js          SMASH safety protocol
│   ├── kuro_drive.js              Drive integration
│   ├── kuro_lab.js                Lab environment
│   ├── harvester.js               Data harvesting
│   ├── artifact_renderer.js       Artifact rendering
│   ├── reactor_telemetry.js       Telemetry sink
│   ├── request_validator.js       Input validation
│   └── web_search.js              Web search integration
│
├── neuro/                         NeuroKURO circadian engine (see separate repo)
│   ├── circadian_model.js         Core phase reconstruction
│   ├── circadian_model.test.js    Test suite
│   ├── circadian_model_math.md    Mathematical derivation
│   ├── circadian_validation.js    Validation harness
│   ├── neuro_routes.cjs           API routes (/api/neuro/*)
│   ├── msf.js                     Mid-sleep function computation
│   ├── sandd_validation.js        SANDD dataset (N=368, MAE=0.31h)
│   ├── mmash_validation.js        MMASH dataset (N=20, MAE=0.29h)
│   ├── VALIDATION_SUMMARY.md      Ground truth results — never edited
│   ├── RESEARCH_BRIEF.md          Research context
│   ├── paper_draft.md             Manuscript draft
│   └── README.md
│
├── modules/                       Feature modules
│   ├── pay/                       KURO::PAY (see separate repo)
│   │   ├── index.cjs              Router assembly + x402 mounting
│   │   ├── x402_card_bridge.cjs   EMVCo QR parser + Solana USDC bridge
│   │   ├── vietqr_parser.cjs      EMVCo TLV parser, CRC16-CCITT validation
│   │   ├── stripe_connector.cjs   AUD card capture
│   │   ├── pay_ledger.cjs         Ledger
│   │   ├── pay_routes.cjs         Aggregated route mount
│   │   ├── shim_v1_routes.cjs     v1 API compatibility shim
│   │   ├── rails/                 Per-rail adapters
│   │   │   ├── _interface.cjs     Rail interface contract
│   │   │   ├── _connector_dispatch.cjs  Connector selection
│   │   │   ├── vietqr.cjs         VND
│   │   │   ├── promptpay.cjs      THB
│   │   │   ├── duitnow.cjs        MYR
│   │   │   ├── qris.cjs           IDR
│   │   │   └── qrph.cjs           PHP
│   │   ├── routes/
│   │   │   ├── accounts.cjs       /api/pay/accounts
│   │   │   ├── ops.cjs            /api/pay/ops
│   │   │   ├── insights.cjs       /api/pay/insights
│   │   │   ├── intelligence.cjs   /api/pay/intelligence
│   │   │   ├── vaults.cjs         /api/pay/vaults
│   │   │   ├── webhooks.cjs       /api/pay/webhook (raw body)
│   │   │   ├── audit_routes.cjs   /api/pay/audit
│   │   │   └── monitoring.cjs     /api/pay/monitoring
│   │   ├── intelligence/          Pay-specific AI subsystems
│   │   │   ├── worker.cjs
│   │   │   ├── admin_assistant.cjs
│   │   │   ├── merchant_normalizer.cjs
│   │   │   ├── oracle.cjs         Predictive pricing
│   │   │   ├── prompt_safety.cjs
│   │   │   ├── receipt_search.cjs
│   │   │   ├── ticket_triager.cjs
│   │   │   └── admin_tools.cjs
│   │   ├── scheduler/
│   │   │   └── commission_payout_hourly.cjs
│   │   └── admin/
│   │       └── assistant_routes.cjs
│   ├── grab/                      KURO::GRAB (Grab API + WS replay)
│   │   ├── grab_auth.cjs
│   │   ├── grab_client.cjs
│   │   ├── grab_config.json
│   │   ├── grab_routes.cjs
│   │   ├── grab_ws.cjs
│   │   └── har_import.cjs
│   └── wager/                     KURO::WAGER (trading engines)
│       ├── db.cjs
│       ├── engine.cjs
│       ├── fusion.cjs
│       ├── index.cjs
│       ├── quantum.cjs
│       ├── router.cjs
│       └── tesla.cjs
│
└── src/                           React frontend
    ├── App.jsx                    OS root
    ├── components/
    │   ├── apps/                  App windows (KuroPay, Messages, AuthGate, …)
    │   └── os/                    OS chrome (taskbar, dock, window manager)
    └── stores/                    Zustand state stores (one per app)
```
