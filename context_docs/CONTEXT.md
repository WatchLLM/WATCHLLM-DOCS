# WatchLLM — Agent Context File

## What this project is
WatchLLM is an AI agent reliability platform. It stress tests, 
replays, and debugs AI agents before they hit production.
Core features: prompt injection testing, execution graph replay, 
fork & replay from any node, severity scoring.

## Stack
- api worker: Hono on Cloudflare Workers
- orchestrator worker: CF Queue consumer, fans out chaos jobs
- chaos worker: Cloudflare Workers (queue consumer, single attack run)
- dashboard: Next.js App Router on CF Pages
- shared types: packages/types (@watchllm/types)
- sdk: Python (watchllm on PyPI)
- db: Cloudflare D1 (SQLite)
- storage: Cloudflare R2 (gzipped trace JSON)
- queue: Cloudflare Queues
- auth: Clerk Pro (GitHub OAuth)
- billing: Stripe
- secrets: Doppler

## Rules you must follow absolutely
- No `any` types. strict TypeScript only.
- No string concatenation for SQL. D1 parameterized queries only.
- No console.log in production paths.
- All errors must be typed and handled, never swallowed.
- All async operations must have timeout handling (max 30s).
- All functions must be under 50 lines. Extract helpers.
- All DB queries in migration files, never inline schema.

## Auth
- Dashboard: Clerk Pro (free via existing plan)
  - GitHub OAuth handled entirely by Clerk
  - Clerk middleware protects all dashboard routes
  - Clerk webhook (POST /api/v1/webhooks/clerk) syncs user to D1 on first sign-in
  - Never store session tokens in D1 — Clerk owns identity
- SDK/CLI: API key via X-WatchLLM-Api-Key header (no change)

## Workers (3 total)
- apps/api        → HTTP routes, Clerk auth middleware, D1 CRUD, CF Queue producer
- apps/orchestrator → CF Queue consumer, fans out chaos jobs by category  
- apps/chaos      → single attack run per category, writes R2 + D1

## Environment Variables
CLERK_SECRET_KEY        (api-worker)
CLERK_PUBLISHABLE_KEY   (dashboard)
CLERK_WEBHOOK_SECRET    (api-worker, for /webhooks/clerk)

## Shared types location
packages/types/src/ — import from "@watchllm/types", never redefine locally.

## Response format contract
Every API route returns: { data: T, error: null } | { data: null, error: { code, message } }
Never deviate from this shape.

## Current month/day
Month 4, Days 17–18 — Public landing page + docs (COMPLETE). Month 4 DONE.

## Today's interface contract
[PASTE THE CONTRACT FOR TODAY'S TASK HERE BEFORE GENERATING CODE]

## Session Protocol (read this at the start of EVERY session)

Before generating any code, you must:
1. Read DAILY_TASKS_NOTES.md — understand what was built yesterday,
   what broke, what decisions were made
2. Read the relevant spec file for today's task (ARCHITECTURE.md, 
   SDK_SPEC.md, SIMULATION_SPEC.md, FRONTEND.md)
3. Confirm today's day/month from DAILY_TASKS.md
4. State back what you understand today's gate to be
5. Only then write any code

After generating any code, you must:
1. Re-read what you generated against the interface contract
2. Flag any deviation from the contract before I review it

You have no memory between sessions. DAILY_TASKS_NOTES.md is your memory.
Treat it as ground truth. If it says something is broken, it is broken.
If it says a decision was made, that decision stands unless I explicitly 
override it.