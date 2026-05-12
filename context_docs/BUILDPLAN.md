# WatchLLM — Monorepo Build Plan
> Single repo. Everything in one place. One context, one history, one deploy pipeline.

---

## Why Monorepo

| Concern | Monorepo answer |
|---|---|
| Shared types drift | `packages/types` — defined once, imported everywhere |
| Context switching | One terminal, one `git log`, one PR |
| CI complexity | One GitHub Actions file fans out to all apps |
| Solo founder overhead | Multiple repos = 4x the boilerplate, 0x the benefit |
| Type safety across services | `TraceGraph`, `SimulationResult`, attack enums are identical everywhere |

Split into multiple repos only when: different teams own different services, or deployment cadences are wildly different. You have neither.

---

## Repository Structure

```
watchllm/                          ← root (github.com/you/watchllm)
│
├── apps/
│   ├── api/                       ← Hono CF Worker — Clerk middleware, CRUD, queue producer
│   │   ├── src/
│   │   │   ├── index.ts           ← Hono app entrypoint
│   │   │   ├── routes/
│   │   │   │   ├── simulations.ts
│   │   │   │   ├── auth.ts
│   │   │   │   ├── billing.ts
│   │   │   │   └── webhooks.ts
│   │   │   ├── middleware/
│   │   │   │   ├── auth.ts        ← API key validation
│   │   │   │   └── ratelimit.ts
│   │   │   └── lib/
│   │   │       ├── d1.ts          ← D1 query helpers
│   │   │       ├── r2.ts          ← R2 read/write
│   │   │       └── stripe.ts
│   │   ├── wrangler.toml
│   │   ├── tsconfig.json
│   │   └── package.json
│   │
│   ├── orchestrator/              ← CF Worker — queue consumer, fans out chaos jobs
│   │   ├── src/
│   │   │   └── index.ts           ← reads sim job, enqueues N chaos jobs by category
│   │   ├── wrangler.toml
│   │   ├── tsconfig.json
│   │   └── package.json
│   │
│   ├── chaos/                     ← CF Worker — attack runner, queue consumer
│   │   ├── src/
│   │   │   ├── index.ts           ← Queue consumer entrypoint
│   │   │   ├── attacks/
│   │   │   │   ├── prompt_injection.ts
│   │   │   │   ├── tool_abuse.ts
│   │   │   │   ├── hallucination.ts
│   │   │   │   ├── context_poisoning.ts
│   │   │   │   ├── infinite_loop.ts
│   │   │   │   ├── data_exfiltration.ts
│   │   │   │   ├── role_confusion.ts
│   │   │   │   └── jailbreak.ts
│   │   │   ├── scorer/
│   │   │   │   ├── rules.ts       ← rule-based pre-filter
│   │   │   │   └── judge.ts       ← CF AI LLM judge
│   │   │   └── lib/
│   │   │       ├── agent.ts       ← callAgent() with timeout/error handling
│   │   │       ├── trace.ts       ← buildTrace(), saveTrace()
│   │   │       └── state.ts       ← reconstructState() for fork jobs
│   │   ├── wrangler.toml
│   │   ├── tsconfig.json
│   │   └── package.json
│   │
│   └── dashboard/                 ← Next.js (App Router) on CF Pages
│       ├── app/
│       │   ├── layout.tsx
│       │   ├── page.tsx           ← redirects to /simulations
│       │   ├── login/
│       │   ├── simulations/
│       │   │   ├── page.tsx       ← simulation list
│       │   │   └── [id]/
│       │   │       ├── page.tsx   ← detail + graph
│       │   │       └── compare/
│       │   │           └── [fork_id]/page.tsx
│       │   ├── settings/
│       │   │   ├── api-keys/
│       │   │   └── billing/
│       │   └── upgrade/
│       ├── components/
│       │   ├── graph/
│       │   │   ├── TraceGraph.tsx ← React Flow wrapper
│       │   │   ├── NodePanel.tsx  ← inspection sidebar
│       │   │   └── Scrubber.tsx   ← timeline control
│       │   └── ui/                ← shared primitives (Button, Badge, etc.)
│       ├── lib/
│       │   └── api.ts             ← typed fetch client for the api Worker
│       ├── next.config.js
│       ├── tsconfig.json
│       └── package.json
│
├── packages/
│   ├── types/                     ← shared TypeScript — used by api, chaos, dashboard
│   │   ├── src/
│   │   │   ├── trace.ts           ← TraceGraph, TraceNode, TraceEdge
│   │   │   ├── simulation.ts      ← SimulationResult, SimRun, AttackCategory
│   │   │   ├── api.ts             ← request/response shapes for every endpoint
│   │   │   └── index.ts
│   │   ├── tsconfig.json
│   │   └── package.json           ← name: "@watchllm/types"
│   │
│   └── sdk/                       ← Python SDK — published to PyPI as "watchllm"
│       ├── watchllm/
│       │   ├── __init__.py
│       │   ├── client.py          ← WatchLLMClient
│       │   ├── decorators.py      ← @watchllm.test(), @watchllm.test_async()
│       │   └── cli.py             ← watchllm simulate / replay / status / auth
│       ├── tests/
│       ├── pyproject.toml
│       └── README.md
│
├── site/                          ← static landing page (watchllm.dev)
│   └── index.html
│
├── infra/
│   ├── migrations/                ← D1 SQL migration files (numbered)
│   │   ├── 001_initial.sql
│   │   ├── 002_add_forks.sql
│   │   └── 003_add_projects.sql
│   ├── schema.sql                 ← canonical current schema (generated)
│   └── doppler.yaml               ← secret mapping (no values, just keys)
│
├── .github/
│   └── workflows/
│       ├── deploy-api.yml         ← wrangler deploy on merge to main
│       ├── deploy-chaos.yml
│       ├── deploy-dashboard.yml
│       └── sdk-publish.yml        ← PyPI publish on tag
│
├── package.json                   ← workspace root (npm workspaces)
├── tsconfig.base.json             ← shared TS config extended by all apps
├── .gitignore
└── README.md
```

---

## Toolchain Decisions

| Tool | Choice | Why |
|---|---|---|
| Monorepo manager | npm workspaces (native) | Zero extra tooling, works with any Node |
| TypeScript | strict mode, no `any` | Enforced by `tsconfig.base.json` |
| Linting | eslint + prettier | Standard, agent-friendly |
| Type checking in CI | `tsc --noEmit` | Catches drift before deploy |
| API framework | Hono | Tiny, edge-native, excellent TS support |
| Dashboard framework | Next.js (App Router) | CF Pages adapter exists, React ecosystem |
| Graph rendering | @xyflow/react (React Flow v12) | Free, maintained, handles large graphs |
| Auth | Clerk Pro | Free via existing plan, Worker SDK available, handles GitHub OAuth |
| Billing | Stripe | Webhooks, portal, subscription management |
| Secrets | Doppler | Single source of truth, syncs to CF |
| Error tracking | Sentry | CF Workers SDK available |
| Python SDK tooling | uv + pyproject.toml | Fast, modern Python packaging |

---

## Shared Types — the critical contract

Everything that crosses a service boundary lives in `packages/types`. This is non-negotiable.

```
TraceGraph          → chaos writes it, api reads it, dashboard renders it
SimulationResult    → api returns it, SDK consumes it, CLI prints it
AttackCategory      → chaos uses it, api validates it, dashboard filters by it
ForkState           → chaos reconstructs it, api validates fork requests
ApiResponse<T>      → every endpoint returns { data: T, error: null | ApiError }
```

When you change a type, TypeScript catches every consumer that breaks — before you deploy anything.

---

## D1 Schema (canonical)

```sql
-- migrations/001_initial.sql

CREATE TABLE users (
  id          TEXT PRIMARY KEY,          -- "usr_" + nanoid
  github_id   TEXT UNIQUE,
  email       TEXT,
  username    TEXT,
  tier        TEXT DEFAULT 'free',       -- free | pro | team
  stripe_customer_id   TEXT,
  stripe_subscription_id TEXT,
  created_at  INTEGER DEFAULT (unixepoch())
);

CREATE TABLE api_keys (
  id          TEXT PRIMARY KEY,          -- "key_" + nanoid
  user_id     TEXT NOT NULL REFERENCES users(id),
  prefix      TEXT NOT NULL,             -- first 8 chars, shown in UI
  hash        TEXT NOT NULL,             -- bcrypt hash of full key
  name        TEXT,
  last_used_at INTEGER,
  revoked_at  INTEGER,
  created_at  INTEGER DEFAULT (unixepoch())
);

CREATE TABLE projects (
  id          TEXT PRIMARY KEY,          -- "prj_" + nanoid
  user_id     TEXT NOT NULL REFERENCES users(id),
  name        TEXT NOT NULL,
  created_at  INTEGER DEFAULT (unixepoch())
);

CREATE TABLE simulations (
  id              TEXT PRIMARY KEY,      -- "sim_" + nanoid
  user_id         TEXT NOT NULL REFERENCES users(id),
  project_id      TEXT REFERENCES projects(id),
  parent_sim_id   TEXT REFERENCES simulations(id),  -- set if this is a fork
  fork_from_node  TEXT,                  -- node ID in parent trace
  agent_endpoint  TEXT NOT NULL,
  categories      TEXT NOT NULL,         -- JSON array of AttackCategory
  status          TEXT DEFAULT 'queued', -- queued | running | completed | failed
  severity        REAL,                  -- 0.0 - 1.0, null until completed
  verdict         TEXT,
  created_at      INTEGER DEFAULT (unixepoch()),
  completed_at    INTEGER,
  expires_at      INTEGER                -- set based on tier retention
);

CREATE INDEX idx_simulations_user    ON simulations(user_id, created_at DESC);
CREATE INDEX idx_simulations_project ON simulations(project_id, created_at DESC);
CREATE INDEX idx_simulations_parent  ON simulations(parent_sim_id);
```

---

## R2 Trace Path Convention

```
traces/
└── {sim_id}/
    └── runs/
        └── {run_id}/
            └── graph.json.gz     ← gzipped TraceGraph JSON
```

Cache key in KV: `trace:{sim_id}` — TTL 1 hour, invalidated on write.

---

## API Surface (complete)

```
# Auth
# Dashboard authentication is handled by Clerk (GitHub OAuth + session via Clerk middleware).
# The API worker only needs the Clerk webhook to sync user identity into D1.
POST /api/v1/webhooks/clerk    → Clerk webhook handler (sync user to D1)

# User
GET  /api/v1/me                → current user + tier info

# API Keys
POST /api/v1/api-keys          → create key (returns full key once)
GET  /api/v1/api-keys          → list keys (prefix only)
DELETE /api/v1/api-keys/:id    → revoke key

# Projects
POST /api/v1/projects          → create project
GET  /api/v1/projects          → list projects

# Simulations
POST /api/v1/simulations       → create + enqueue (returns immediately)
GET  /api/v1/simulations       → list (paginated, filterable)
GET  /api/v1/simulations/:id   → detail + severity
GET  /api/v1/simulations/:id/trace  → full TraceGraph (Pro/Team only)
POST /api/v1/simulations/:id/fork   → fork from node (Pro/Team only)
GET  /api/v1/simulations/:id/forks  → list child simulations

# Billing
GET  /api/v1/billing/portal    → Stripe billing portal redirect
POST /api/v1/webhooks/stripe   → Stripe webhook handler
```

All routes return `{ data: T, error: null }` or `{ data: null, error: { code, message, upgrade_url? } }`.

---

## CF Queue Message Shapes

```typescript
// Fresh simulation job
type SimulationJob = {
  type: 'simulation'
  sim_id: string
  user_id: string
  agent_endpoint: string
  categories: AttackCategory[]
}

// Fork job
type ForkJob = {
  type: 'fork'
  sim_id: string               // the NEW fork's sim_id
  parent_sim_id: string
  fork_from_node: string       // node ID in parent trace
  new_input: unknown
  user_id: string
  agent_endpoint: string
  categories: AttackCategory[]
}
```

The chaos worker reads `job.type` on every message to route correctly.

---

## Environment Variables (Doppler → CF)

```
# api worker
DATABASE_URL          ← D1 binding (not a string, set in wrangler.toml)
R2_BUCKET             ← R2 binding
KV_NAMESPACE          ← KV binding
QUEUE                 ← Queue binding (producer)
STRIPE_SECRET_KEY
STRIPE_WEBHOOK_SECRET
CLERK_SECRET_KEY
CLERK_PUBLISHABLE_KEY
CLERK_WEBHOOK_SECRET
SENTRY_DSN

# orchestrator worker
DATABASE_URL
QUEUE                 ← Queue binding (consumer)
SENTRY_DSN

# chaos worker
DATABASE_URL
R2_BUCKET
AI                    ← CF AI binding
QUEUE                 ← Queue binding (consumer)
SENTRY_DSN

# dashboard (Next.js / CF Pages)
NEXT_PUBLIC_API_URL   ← https://api.watchllm.dev
SENTRY_DSN
```

Bindings (D1, R2, KV, Queue, AI) are declared in `wrangler.toml`, not as strings. Secret strings go in Doppler, synced to CF via `doppler run -- wrangler deploy`.

---

## GitHub Actions — Deploy Pipeline

```yaml
# .github/workflows/deploy-api.yml
name: Deploy API
on:
  push:
    branches: [main]
    paths: ['apps/api/**', 'packages/types/**']

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm ci
      - run: npm run typecheck --workspace=packages/types
      - run: npm run typecheck --workspace=apps/api
      - run: npx wrangler deploy
        working-directory: apps/api
        env:
          CLOUDFLARE_API_TOKEN: ${{ secrets.CF_API_TOKEN }}
```

Same pattern for chaos and dashboard. Only deploys when files in that app or shared types change.

---

## Local Dev Setup (Day 1)

```bash
# 1. Clone and install
git clone https://github.com/you/watchllm
cd watchllm
npm install          # installs all workspaces

# 2. Authenticate
wrangler login
doppler login

# 3. Create D1 and run migrations
wrangler d1 create watchllm-db
wrangler d1 execute watchllm-db --local --file=infra/migrations/001_initial.sql

# 4. Start api worker locally
cd apps/api
wrangler dev --local

# 5. Start dashboard locally (separate terminal)
cd apps/dashboard
npm run dev

# 6. Type check everything
cd ../..
npm run typecheck    # runs tsc --noEmit across all workspaces
```

---

## Root package.json

```json
{
  "name": "watchllm",
  "private": true,
  "workspaces": [
    "apps/*",
    "packages/*"
  ],
  "scripts": {
    "typecheck": "tsc -b apps/api apps/chaos apps/dashboard packages/types",
    "lint": "eslint apps/*/src packages/*/src",
    "format": "prettier --write .",
    "db:migrate:local": "wrangler d1 execute watchllm-db --local --file",
    "db:migrate:prod": "wrangler d1 execute watchllm-db --file"
  },
  "devDependencies": {
    "typescript": "^5.4.0",
    "eslint": "^9.0.0",
    "prettier": "^3.0.0"
  }
}
```

---

## tsconfig.base.json

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "lib": ["ES2022"],
    "skipLibCheck": true
  }
}
```

Every app's `tsconfig.json` extends this: `"extends": "../../tsconfig.base.json"`.

---

## Month-by-Month What Gets Built When

| Month | Apps touched | Packages touched |
|---|---|---|
| 1 | `apps/api` (CRUD, Clerk middleware, queue) · `apps/orchestrator` (stub) · `apps/chaos` (attack loop, scorer, trace) | `packages/types` (TraceGraph, SimulationResult) · `packages/sdk` (client, CLI) |
| 2 | `apps/dashboard` (scaffold, graph, node panel) · `apps/chaos` (graph nodes in trace) | `packages/types` (TraceNode, TraceEdge extended) |
| 3 | `apps/api` (fork endpoint, OAuth, billing placeholder) · `apps/chaos` (fork execution, state reconstruction) · `apps/dashboard` (compare view, paywall UI) | `packages/types` (ForkState, ForkJob) · `packages/sdk` (decorator) |
| 4 | `apps/api` (Stripe webhooks, tier gates) · `apps/chaos` (all 8 attacks) · `apps/dashboard` (upgrade flow, history) · `site/` (landing page) | `packages/sdk` (CLI polish, PyPI publish) |

---

## Non-negotiable Rules

1. **`packages/types` is the contract.** If a type changes, fix every consumer before merging.
2. **No `any`.** `tsconfig.base.json` enforces `strict: true`. Agents must satisfy the type checker.
3. **No inline SQL strings.** All queries go through typed D1 helpers in `lib/d1.ts`.
4. **Migrations are append-only.** Never edit a migration that has run in production. Write a new one.
5. **Secrets never touch the repo.** Doppler only. `doppler run -- wrangler deploy` for all deploys.
6. **One PR per day's work.** Gate: CI green + day's gate criteria passing = merge. Otherwise: extend.
7. **`tsc --noEmit` must pass before any deploy.** It is a hard CI gate, not a suggestion.
8. **Clerk handles auth, D1 handles everything else.** Never store session tokens or auth state in D1. Clerk owns the identity layer. D1 owns the product data (tier, api_keys, simulations, projects).

---

*This document is the architectural source of truth. If a decision here conflicts with a task breakdown, this wins. Update this file when architectural decisions change — not after.*