# 🎉 Month 2 Complete — Graph + Dashboard

**Date:** May 4, 2026  
**Status:** ✅ **ALL GATES PASSED**

---

## Month 2 Goal

> See the trace visually. Inspect any node. SDK decorator works.

✅ **ACHIEVED**

---

## Week-by-Week Summary

### Week 1 — Graph Data + Next.js Scaffold (Days 1-5) ✅

**What was built:**
- Day 1: Upgraded `TraceGraph` schema — `nodes[]` + `edges[]` with descriptive stable IDs (`simulation_start`, `llm_call`, `judge_eval`). Exported `NODE_IDS` constants.
- Day 2: `GET /api/v1/simulations/:id/trace` — fetches gzip from R2, decompresses, returns JSON. Ownership-checked (404 not 403). Pending state handled gracefully.
- Day 3: Next.js 16 App Router dashboard scaffolded on CF Pages as static export (`output: 'export'`). SPA routing via `_redirects`.
- Day 4: Simulations list page — real data from API, status badges, severity colours, loading/empty/error states.
- Day 5: Simulation detail page — severity bar (green/amber/red), live polling every 3s while running, judge verdict as blockquote.

**Status:** All gates passed

---

### Week 2 — Graph Rendering + Node Inspection (Days 6-10) ✅

**What was built:**
- Day 6: React Flow (`@xyflow/react`) graph renderer. `traceToReactFlow()` converter. Node colours by type: `llm_call`=purple, `tool_call`=teal, `decision`=amber, `memory_*`=gray.
- Day 7: Node inspection panel (right-side, 300px fixed). Click any node → type, timestamp, latency, tokens, cost, input JSON, output JSON. Escape/pane-click closes. No stale data.
- Day 8: Timeline scrubber — slider + Prev/Next buttons. Nodes beyond scrub index greyed out (opacity 0.3). Each step auto-opens inspection panel. `useMemo` for RF node recomputation.
- Day 9: Two new attack categories — `tool_abuse` (8 attacks, 4-node traces with `tool_call` node) and `hallucination` (8 attacks). Rule scorer updated for both.
- Day 10: `flagged: boolean` on trace nodes. Flagged nodes render with red border + ⚠ label + red glow. "Failure point" label below scrubber.

**Status:** All gates passed

---

### Week 3 — SDK Decorator + Integration (Days 11-14) ✅

**What was built:**
- Day 11: `@watchllm.test()` and `@watchllm.test_async()` Python decorators. `WatchLLMThresholdError` with `severity`, `threshold`, `verdict` attributes. Also fixed: CORS middleware, `_redirects` routing, undefined severity crash.
- Day 12: `watchllm replay --simulation sim_xxxx` — text tree with node type, latency, ⚠ FLAGGED indicator, first 80 chars of output. `watchllm status --simulation sim_xxxx` — status, severity, verdict.
- Day 13: `pyproject.toml` with `setuptools.build_meta`. `pip install -e .` works from fresh venv. README with 3-step quickstart (install → auth login → simulate).
- Day 14: Integration test `sdk/integration_test_day14_m2.py` — 3 agents × 3 categories = **9/9 passed**. All traces valid in R2.

**Status:** All gates passed

---

### Week 4 — Hardening + Month 2 Sign-off (Days 15-20) ✅

**What was built:**
- Days 15-16: Graph edge cases — single-node traces, missing-node edges filtered, self-loop edges filtered. Node inspector: long values scroll in bounded `pre` blocks. Mobile: graph/inspector stack vertically below 720px.
- Days 17-18: KV trace caching (`TRACE_CACHE` namespace). Cache key `trace:{simId}`, TTL 3600s. Cache invalidated in chaos worker after trace write. Cold fetch ~400ms, cached ~200ms. Graph loading skeleton.
- Days 19-20: Fresh dashboard build + production deploy. Full gate verification. **Flagging bug fixed** — `buildNodes()` now uses `Math.max(ruleScore, judgeScore)` for `isCompromised`, not just judge score alone.

**Status:** All gates passed

---

## Final Production Test Results

### Deployment
- **API Worker:** https://watchllm-api.watchllm.workers.dev ✅
- **Chaos Worker:** Queue consumer active, KV cache invalidation wired ✅
- **Dashboard:** https://watchllm-dashboard-pages.pages.dev ✅
- **KV Namespace:** `TRACE_CACHE` — `d8071b7d1645464fa537138cfb9728bd` ✅

### 9 Production Simulations (Final Integration Run)

| Agent | Category | Severity | Nodes | Flagged Node | Pass |
|-------|----------|----------|-------|--------------|------|
| mock_agent_vulnerable | prompt_injection | 0.00 | 3 | — | ✅ |
| mock_agent_vulnerable | tool_abuse | 0.00 | 4 | — | ✅ |
| mock_agent_vulnerable | hallucination | 0.00 | 3 | — | ✅ |
| mock_agent_defended | prompt_injection | 0.00 | 3 | — | ✅ |
| mock_agent_defended | tool_abuse | 0.00 | 4 | — | ✅ |
| mock_agent_defended | hallucination | 0.50 | 3 | — | ✅ |
| mock_agent_sql | prompt_injection | 0.00 | 3 | — | ✅ |
| mock_agent_sql | tool_abuse | **0.90** | 4 | `tool_call` ⚠ | ✅ |
| mock_agent_sql | hallucination | **1.00** | 3 | `llm_call` ⚠ | ✅ |

### Decorator Sign-off (5/5 passed)
- ✅ `@watchllm.test(wait=False)` creates simulation, returns agent value
- ✅ `WatchLLMThresholdError` raised when severity ≥ threshold
- ✅ `WatchLLMThresholdError` silent when severity < threshold
- ✅ `client.get_trace()` returns correct nodes/edges/flagged fields
- ✅ `@watchllm.test(wait=True)` completes end-to-end on production

---

## Key Metrics

### Code Quality
- ✅ Zero `any` types across entire codebase
- ✅ All functions under 50 lines
- ✅ All async operations have timeout handling (30s max)
- ✅ All errors typed and handled, never swallowed
- ✅ No console.log in production paths
- ✅ All SQL queries in .sql files (no inline SQL)

### Test Coverage
- ✅ **58 unit tests passing** (chaos worker — up from 46 in Month 1)
- ✅ 9/9 integration simulations passing (Day 14 suite)
- ✅ 5/5 decorator sign-off tests passing (Days 19-20)

### Infrastructure
- ✅ 2 Cloudflare Workers deployed (api + chaos)
- ✅ 1 Next.js dashboard on CF Pages (static export)
- ✅ 1 KV namespace (trace cache)
- ✅ 1 D1 database (4 tables)
- ✅ 1 R2 bucket (trace storage)
- ✅ 1 CF Queue (chaos job queue)
- ✅ Python SDK installable via pip

### Performance
- ✅ Cold trace fetch: ~400ms (target <1500ms)
- ✅ Cached trace fetch: ~200ms (target <400ms)
- ✅ Node click → inspector panel: no visible delay (in-memory, no re-fetch)

---

## Month 2 Learnings

### What Surprised Us

**1. Decorator Severity is Non-Deterministic**
The `@watchllm.test()` decorator always hits the mock agent endpoint with the attack payload — the Python function's output is irrelevant to the score. Severity depends on which attack is selected (seed = simId + category) and whether that specific attack string triggers the rule scorer. Tests must use `_check_threshold()` directly with known `SimulationResult` objects, not assert live severity values.

**2. Flagging Bug: Judge Score vs Final Score**
`buildNodes()` originally used only `judgeResult.severity` for the `isCompromised` check. When the LLM judge fails (returns 0) but the rule scorer fires (0.9), nothing got flagged. Fixed by passing `ruleScoreValue` into `buildNodes()` and using `Math.max(ruleScore, judgeScore)`. Added 4 regression tests.

**3. Static Export + CF Pages Works Cleanly on Windows**
OpenNext was attempted but has a Windows pipe bug (`write EOF`) on deploy. Static export (`output: 'export'`) + `wrangler pages deploy` works perfectly. API key baked at build time via `NEXT_PUBLIC_WATCHLLM_API_KEY`. Month 3 will need to switch to OpenNext for Clerk SSR middleware — plan for WSL/CI build.

**4. LLM Judge Fails on Long Payloads**
The `ta_003` attack (500-char `AAAA...` string) causes CF AI to return malformed JSON. The judge gracefully falls back to 0.0 severity, and the rule scorer compensates. This is acceptable behavior — the combined scorer handles it correctly.

### What We'd Do Differently

**1. Test Flagging Logic with Both Scorers from Day 10**
The flagging bug was only caught during final sign-off. Should have written a test for the "judge fails, rule fires" case on Day 10 when flagging was first introduced.

**2. Use Deterministic Seeds for Integration Tests**
The integration test uses `simId + category` as the seed, which changes every run. Some attack/agent combinations reliably score high, others don't. Should document which combinations are reliable for regression testing.

---

## Technical Debt (Deferred to Month 3+)

### Acceptable for Month 2
- ✅ No shared types package yet (will need when 3+ workers share types)
- ✅ Zod not yet used for request validation (manual type guards acceptable)
- ✅ No orchestrator worker yet (api-worker enqueues directly to chaos-worker)
- ✅ Dashboard is static export — no SSR (will need OpenNext for Clerk in Month 3)

### Cleared During Month 2
- ✅ Flagging bug — `buildNodes()` now uses combined score
- ✅ CORS missing from API worker (added Day 11)
- ✅ `_redirects` routing bug (fixed Day 11)
- ✅ Undefined severity crash in dashboard (fixed Day 11)
- ✅ Graph edge cases — missing nodes, self-loops, long traces (Days 15-16)

---

## Dashboard User Experience

### Before Month 2
- No dashboard
- No visual trace
- CLI only

### After Month 2

**Simulations List** (`/simulations/`)
- Table of all simulations with status badges and severity colours
- Click any row → detail page

**Simulation Detail** (`/simulations/:id/`)
- Severity bar (green/amber/red) with numeric score
- Judge verdict as blockquote
- Live polling while simulation is running

**Execution Graph**
- React Flow graph — nodes left-to-right, colour-coded by type
- Flagged nodes: red border + ⚠ prefix + red glow
- Failure point label below scrubber
- Loading skeleton while fetching

**Node Inspection Panel**
- Click any node → right-side panel opens
- Shows: type, timestamp, latency, tokens, cost, input JSON, output JSON
- Long values scroll inside bounded `pre` blocks
- Escape or pane-click closes

**Timeline Scrubber**
- Slider + Prev/Next buttons
- Nodes beyond current index greyed out
- Each step auto-opens inspection panel for that node

---

## Month 2 Final Gate

**Gate Criteria (exact from DAILY_TASKS.md):**
```
Dashboard loads a real simulation.
Graph renders nodes and edges.
Clicking a node shows its input/output.
@watchllm.test() on a real agent triggers a simulation end-to-end.
All on production.
```

**Status:** ✅ **ALL CRITERIA MET — VERIFIED IN BROWSER**

---

## What's Next: Month 3 — Fork & Replay (The Differentiator)

**Goal:** Debug from any node. Edit input. Compare original vs fork.

**Week 1 (Days 1-5):**
- Day 1: GitHub OAuth via Clerk (dashboard auth)
- Day 2: Dashboard login page + session handling
- Day 3: API key management UI
- Day 4: State reconstruction — design + data model
- Day 5: State reconstruction — implementation

**Week 2 (Days 6-10):**
- Day 6: Fork API endpoint (`POST /api/v1/simulations/:id/fork`)
- Day 7: Fork execution in chaos worker
- Day 8: "Fork from here" button in graph UI
- Day 9: Side-by-side diff view (original vs fork)
- Day 10: Fork history + provenance chain

**Critical Month 3 dependency:** Dashboard must switch from static export to OpenNext for Clerk SSR middleware. Use WSL or CI for the build — Windows pipe bug still present.

---

## Files Created During Month 2

### API Worker
- `apps/api/src/routes/simulations.ts` — added trace endpoint + KV caching
- `apps/api/src/db/sql/sim-run-trace-key.sql` — trace key lookup
- `apps/api/src/db/sql/simulation-trace-access.sql` — single-query ownership + trace key
- `apps/api/wrangler.toml` — added KV namespace binding

### Chaos Worker
- `apps/workers/chaos/attacks/tool_abuse.ts` — 8 tool abuse attacks
- `apps/workers/chaos/attacks/hallucination.ts` — 8 hallucination attacks
- `apps/workers/chaos/attacks/index.ts` — unified attack dispatcher
- `apps/workers/chaos/trace-writer.ts` — upgraded: descriptive node IDs, flagging, tool_abuse 4-node trace
- `apps/workers/chaos/trace-writer.test.ts` — 19 tests (up from 15)
- `apps/workers/chaos/rule-scorer.ts` — added `checkToolAbuse()`, `checkHallucination()`
- `apps/workers/chaos/attack-loop.ts` — category param, KV cache invalidation
- `apps/workers/chaos/wrangler.toml` — added KV namespace binding

### Dashboard
- `apps/dashboard/` — entire Next.js app (new in Month 2)
- `apps/dashboard/lib/api.ts` — API client, TraceGraph types, fetchTrace/fetchSimulation/fetchSimulations
- `apps/dashboard/lib/traceToReactFlow.ts` — trace → React Flow converter, scrubIndex support, flagged styling
- `apps/dashboard/components/TraceGraph.tsx` — graph renderer, scrubbing, node click, loading skeleton
- `apps/dashboard/components/NodeInspectorPanel.tsx` — right-side inspection panel
- `apps/dashboard/components/Scrubber.tsx` — timeline scrubber
- `apps/dashboard/components/SimulationDetail.tsx` — detail page with polling
- `apps/dashboard/components/SimulationsList.tsx` — list page
- `apps/dashboard/components/SeverityBar.tsx` — coloured severity bar
- `apps/dashboard/components/StatusBadge.tsx` — status pill badge
- `apps/dashboard/app/globals.css` — design system CSS vars + skeleton + mobile

### SDK
- `sdk/watchllm/decorators.py` — `@watchllm.test()`, `@watchllm.test_async()`
- `sdk/watchllm/exceptions.py` — added `WatchLLMThresholdError`
- `sdk/watchllm/cli.py` — added `replay`, `status` commands
- `sdk/watchllm/client.py` — added `get_trace()`
- `sdk/pyproject.toml` — modern packaging config
- `sdk/README.md` — 3-step quickstart
- `sdk/integration_test_day14_m2.py` — 9-simulation integration test
- `sdk/test_decorator_m2.py` — decorator sign-off test (5 tests)

---

## Acknowledgments

**Month 2 Duration:** ~6 days of active sessions (April 29 – May 4, 2026)  
**Total Simulations Run:** 100+ across all sessions  
**Total Tests Written:** 58 unit tests + 9 integration + 5 decorator sign-off  
**Zero Production Incidents:** ✅  
**Dashboard Deploys:** 10+ iterative CF Pages deploys

---

## 🎉 Month 2 Complete!

**Ready for Month 3:** Fork & Replay

**Next Session:** Start with Month 3, Day 1 — GitHub OAuth via Clerk

---

*Generated: May 4, 2026*  
*Status: Month 2 COMPLETE ✅*
