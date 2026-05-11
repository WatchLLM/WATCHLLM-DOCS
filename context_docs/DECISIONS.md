# WatchLLM — Decision Log
> Permanent record. Never delete entries. Add new ones at the top.
> If an agent's output contradicts anything here, reject it immediately.

---

## 2026-05-06 — Month 4: Dodo Payments instead of Stripe
**Decision:** Use Dodo Payments for billing, not Stripe.
**Why:** Dodo Payments is a Merchant of Record — they handle VAT, global tax compliance, and international payouts automatically. No need to register for VAT in every country. Webhook signature verification uses Standard Webhooks spec (HMAC SHA256), not Stripe's svix library. Key events: `subscription.active` (upgrade), `subscription.cancelled` / `subscription.expired` (downgrade), `subscription.on_hold` (payment failed, grace period).
**What this rules out forever:** Using Stripe, Paddle, or any other payment processor. All billing references in code must use Dodo Payments API. The webhook handler is `POST /api/v1/webhooks/dodo`, not `/webhooks/stripe`.

## 2026-05-06 — Month 4: Azure OpenAI as primary LLM judge, CF AI as free fallback
**Decision:** Use Azure OpenAI `gpt-4o-mini` as the primary LLM judge for Pro/Team users. Keep Cloudflare AI `llama-2-7b` as the fallback for free tier users and when Azure credits are exhausted.
**Why:** CF AI's llama-2-7b misses SQL injection and prompt leaks (both scored 0.00 in Month 1 testing). Azure gpt-4o-mini has dramatically better detection quality. $1000 Azure credits at ~$0.15/1M tokens = ~6.6B tokens, enough for millions of simulations. Free tier uses CF AI only — zero Azure spend on non-paying users. When credits run out, all tiers fall back to CF AI automatically.
**What this rules out forever:** Using CF AI as the sole judge for Pro/Team users. The `judgeScore()` function must accept a `tier` parameter and route accordingly. Never spend Azure credits on free tier users.

## 2026-05-06 — Month 3: Fork-of-a-fork is transparent to reconstructState
**Decision:** `reconstructState` only looks at the trace's own nodes and their `state_snapshot` fields. The `forked_from` field is provenance metadata — it has no effect on reconstruction. A fork of a fork works identically to a fork of a root simulation.
**Why:** The cumulative snapshot design (Day 4) means each node is self-contained. There is no need to traverse the lineage chain during reconstruction.
**What this rules out forever:** Any implementation that walks `forked_from` during `reconstructState`. The function signature is `(trace, nodeId, newInput)` — it only needs the trace it was given.

## 2026-05-06 — Month 3: Flagged nodes fork identically to unflagged nodes
**Decision:** `flagged: true` on a `TraceNode` is a display hint for the dashboard. It has no effect on `reconstructState`, the fork API, or the chaos worker.
**Why:** Flagging is a severity annotation applied after scoring. The fork mechanic operates on state snapshots, which are independent of the flagging decision.
**What this rules out forever:** Any fork validation that checks `flagged` status. Engineers should be able to fork from any node, including the one that failed.

## 2026-05-06 — Month 3: context_poisoning produces memory_write node
**Decision:** `context_poisoning` attack category produces a 4-node trace: `simulation_start → llm_call → memory_write → judge_eval`. The `memory_write` node's `state_snapshot.node_input` includes `written_value` so fork can replay with a different memory value.
**Why:** Context poisoning attacks work by writing malicious content to agent memory. The `memory_write` node captures what was written, making it forkable — engineers can replay with a safe value to verify the fix.
**What this rules out forever:** Treating `memory_write` as a non-forkable node type. All node types are forkable.

## 2026-05-06 — Month 3: What surprised us technically about fork & replay
**Observation (not a rule, but important context for Month 4):**
The hardest part of fork & replay was not the API or the UI — it was the state snapshot design on Day 4. The key insight that made everything else simple: snapshots are **cumulative**, not delta. Each node stores the full conversation history up to that point, not just what changed. This made `reconstructState` a direct O(1) lookup instead of a chain walk. If we had stored deltas, Days 7 and 15 would have been significantly harder.
The second surprise: the `forked_from` field on `TraceGraph` is purely provenance metadata. It does not need to be read during reconstruction. This separation of concerns (provenance vs state) kept the fork execution path clean.

## 2026-05-06 — Month 3: Beta signup endpoint is unauthenticated
**Decision:** `POST /api/v1/beta` requires no API key. It is a public endpoint.
**Why:** Beta signups come from people who don't have an account yet. Requiring auth would defeat the purpose.
**What this rules out forever:** Adding API key auth to the beta signup endpoint. Rate limiting (if needed) should be done by IP or email domain, not by API key.

## 2026-05-05 — Month 3: Use @clerk/react not @clerk/nextjs for ClerkProvider
**Decision:** Import `ClerkProvider` from `@clerk/react`, not `@clerk/nextjs`. Wrap it in a `'use client'` component (`ClerkClientProvider.tsx`) before using in Next.js App Router layout.
**Why:** `@clerk/nextjs`'s `ClerkProvider` uses Server Actions internally, which are incompatible with `output: 'export'`. `@clerk/react`'s `ClerkProvider` is a pure React context provider with no server dependencies.
**What this rules out forever:** Using `@clerk/nextjs`'s `ClerkProvider` in the dashboard. All Clerk hooks (`useAuth`, `useUser`, etc.) should be imported from `@clerk/react`.

## 2026-05-05 — Month 3: Static Export Stays, Hash Routing for Sign-in
**Decision:** Dashboard remains `output: 'export'` + CF Pages. Sign-in page uses `<SignIn routing="hash" />`.
**Why:** OpenNext Cloudflare requires edge runtime for middleware. Next 16's `proxy.ts` always runs on Node.js runtime — `export const runtime = 'edge'` is rejected. Static export + CF Pages is the only working path on Windows without WSL.
**What this rules out forever:** Switching to OpenNext from Windows. If OpenNext is ever needed (e.g., for SSR auth), it must be built in WSL or CI.

## 2026-05-04 — Month 2: Flagging Uses Combined Score
**Decision:** `buildNodes()` uses `Math.max(ruleScore, judgeScore)` for the `isCompromised` check, not `judgeScore` alone.
**Why:** When the LLM judge fails (returns 0.0) but the rule scorer fires (0.9), nothing was getting flagged. The final severity is always `Math.max(ruleScore, judgeScore)` — the flagging logic must match.
**What this rules out forever:** Using only the judge result for flagging decisions. Any severity-based logic must use the combined score.

## 2026-05-04 — Month 2: Dashboard is Static Export
**Decision:** Dashboard uses `output: 'export'` + CF Pages. API key baked at build time via `NEXT_PUBLIC_WATCHLLM_API_KEY`.
**Why:** OpenNext has a Windows pipe bug (`write EOF`) on deploy. Static export + `wrangler pages deploy` works cleanly on Windows.
**What this rules out forever:** Deploying OpenNext from Windows directly. Month 3 must use WSL or CI for the OpenNext build when Clerk SSR middleware is needed.

## 2026-05-04 — Month 2: Cache Completed Traces Only
**Decision:** KV trace cache (`trace:{simId}`, TTL 3600s) stores only completed `TraceGraph` payloads. Never cache `{ status: 'pending' }` or error envelopes.
**Why:** Caching pending responses would cause the dashboard to show "trace not available" permanently for a simulation that later completes.
**What this rules out forever:** Caching any non-terminal trace state.

## 2026-05-04 — Month 2: Decorator Tests Use _check_threshold Directly
**Decision:** Decorator sign-off tests call `_check_threshold()` with known `SimulationResult` objects rather than asserting live severity values from decorator calls.
**Why:** The decorator always hits the mock agent endpoint with the attack payload. Severity depends on which attack is selected (seed = simId + category) — non-deterministic across runs.
**What this rules out forever:** Writing tests that assert a specific severity value from a live `@watchllm.test()` call.

## 2026-04-29 — Month 1 Learnings: LLM Judge Conservatism
**Decision:** CF AI's free model (@cf/meta/llama-2-7b-chat-int8) is conservative and doesn't always detect subtle compromises
**Why:** The free model focuses on obvious patterns. It correctly detected PII (0.70) and shell commands (0.90) but missed SQL injection and prompt leaks (both scored 0.00). The rule-based scorer compensates by catching most obvious patterns.
**What this rules out forever:** Relying solely on LLM judge for severity scoring. Rule-based scorer must remain the primary defense. In Month 2-3, we may experiment with different CF AI models or add more sophisticated rule patterns.

## 2026-04-29 — Month 1 Learnings: Test LLM Models Earlier
**Decision:** Should have tested multiple CF AI models during Week 2 (Days 6-10) to understand detection capabilities
**Why:** Understanding the LLM judge's limitations early would have allowed us to tune the rule-based scorer more aggressively or choose a different model. However, the current system works well enough for Month 1.
**What this rules out forever:** Assuming any LLM model will work without testing. Always test multiple models early in the development cycle to understand their strengths and weaknesses.

---

## [Decision format]
**Date:**
**Decision:**
**Why:**
**What this rules out forever:**

---

## 2025-XX-XX — Auth: Clerk Pro for dashboard
**Decision:** Using Clerk Pro for dashboard auth
**Why:** Free via existing plan, eliminates OAuth boilerplate
**What this rules out:** Supabase Auth, NextAuth, rolling our own session management

## 2025-XX-XX — 3 Workers, not 2
**Decision:** api-worker / orchestrator-worker / chaos-worker
**Why:** Orchestrator cleanly separates fan-out logic from HTTP concerns
**What this rules out:** Running fan-out inside the api-worker, 
collapsing orchestrator and chaos into one worker

## 2025-XX-XX — Monorepo
**Decision:** Single repo, npm workspaces
**Why:** Shared types, one git history, solo founder
**What this rules out:** Separate repos per service

## 2025-XX-XX — No any types, ever
**Decision:** strict TypeScript, zero any
**Why:** Agents produce undetectable bugs under loose typing
**What this rules out:** Quick fixes with any, casting through unknown