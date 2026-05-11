# WatchLLM — Engineering Rules

## Language & Runtime
- Workers (api, orchestrator, chaos): TypeScript strict
- Frontend: TypeScript strict, React, Next.js App Router
- Python SDK: Python 3.10+, typed with mypy

## Directory Structure
```
watchllm/
├── apps/
│   ├── web/              # Next.js frontend
│   └── workers/
│       ├── api/          # Hono API worker
│       ├── orchestrator/ # Simulation orchestrator worker
│       └── chaos/        # Individual attack run worker
├── packages/
│   ├── sdk-python/       # Python SDK + CLI
│   └── types/            # Shared TypeScript types
├── migrations/           # D1 SQL migrations
└── docs/                 # Product docs
```

## API Design Rules
- All routes: /api/v1/{resource}
- Response shape always:
```typescript
  { data: T, error: null }              // success
  { data: null, error: { message, code } } // failure
```
- HTTP status codes: 200 success, 201 created, 400 bad input,
  401 unauth, 403 forbidden, 404 not found, 429 rate limited, 500 server
- No custom error codes. Standard HTTP status codes only.
- All timestamps: Unix seconds (integer), not ISO strings
- All IDs: nanoid (21 chars), prefixed by resource type
    - users: usr_
    - projects: prj_
    - agents: agt_
    - simulations: sim_
    - runs: run_
    - api keys: key_

## TypeScript Rules
- strict: true in tsconfig, no exceptions
- No `any`. If you need escape hatch, use `unknown` + type guard.
- No non-null assertions (!) unless after explicit null check
- All Worker bindings typed in Env interface
- Zod for all request validation at API boundary

## Database Rules
- All SQL in migration files, never inline in application code
- No ORM. Raw D1 SQL only.
- All queries parameterized, never string-concatenated
- Transaction pattern for multi-table writes
- Index every foreign key column

## Naming Conventions
- Files: kebab-case (user-router.ts, chaos-worker.ts)
- Functions: camelCase (registerAgent, buildSimulation)
- Constants: SCREAMING_SNAKE (MAX_RUNS_PER_TIER)
- DB columns: snake_case (created_at, user_id)
- Env vars: SCREAMING_SNAKE (STRIPE_SECRET_KEY)

## Security Rules
- All API key operations: hash with bcrypt, never store plaintext
- Full API key shown exactly once (at creation), never again
- All webhook endpoints: verify signature before processing
- CORS: locked to watchllm.dev in production
- Rate limiting on every simulation launch endpoint
- No secrets in source code, ever — Doppler only

## Error Handling
- Never swallow errors silently
- All Worker errors caught at top level, logged to Sentry
- User-facing errors: clear message, no stack traces
- Internal errors logged with context (user_id, simulation_id, etc.)

## What Needs Tests
- SDK decorator behavior (unit)
- API key hash/verify (unit)
- Severity score calculation (unit)
- Simulation lifecycle state machine (integration)
- Everything else: test manually during development

## Cardinal Rules
1. No logic in React components — components render, hooks handle logic
2. No raw SQL outside /migrations files
3. No secrets in code
4. No `any` in TypeScript
5. No frontend fetches that bypass the API worker
6. No Cloudflare binding accessed outside the Worker that owns it