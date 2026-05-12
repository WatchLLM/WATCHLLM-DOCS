# WatchLLM — Strategic Implementation Guide

## Product-Market Validation

### Market Size & Growth
- **AI Agent Security/Red-Teaming Market**: $3.59B (2025) → $43.8B (2034), 35% CAGR
- **AI Agent Platform Market**: 41.5% CAGR, adding $31.46B 2025-2030
- **Prompt Injection**: 37% of all LLM security breaches (2024), fastest-growing attack vector at 39.1% CAGR
- **API-Based SaaS Deployment**: 20.1% market share, fastest-growing segment at 37.4% CAGR

### Regulatory Drivers (Non-Optional Spending)
- EU AI Act (enforced Aug 2024): Mandatory adversarial testing for high-risk AI
- US Executive Order on AI: Pre-deployment red-teaming disclosures required
- FDA, OCC, Federal Reserve, NIST: All issued adversarial testing guidance
- **44% of 2025 revenues compliance-driven** — predictable procurement cycles

### Competitive Reality Check
- **LangSmith/Langfuse**: Passive observability only (post-mortem logs)
- **Confident AI/DeepEval**: Evaluation-first, no active adversarial testing
- **Promptfoo**: Red-teaming capable, no graph replay or fork/replay mechanics
- **OpenAI acquired Promptfoo (late 2024)**: Validates category legitimacy
- **Cisco acquired Robust Intelligence**: Major M&A activity confirms enterprise budgets exist

### Strategic Position
- **No direct competitor** with unified: stress testing + graph replay + fork & replay
- Targeting largest solution segment (LLM Red-Teaming Platforms, 34.2% share)
- **Risk**: Execution speed, not market existence. Well-funded competitors moving fast.

---

## Development Methodology: Vertical Slice Architecture

### Anti-Pattern: Horizontal Phases
Do NOT build: Month 1 = all stress testing, Month 2 = all graph replay, etc.
Result: Month 4 integration hell, architectural flaws discovered too late.

### Correct Pattern: Monthly Vertical Slices
Each month delivers a complete, working, end-to-end product that expands.

#### Month 1: Core Loop (Working MVP)
**Scope:**
- One attack category only: `prompt_injection`
- Minimal graph storage: raw JSON to R2 (no visualization)
- CLI-only interface (no web dashboard)
- Manual verification of results

**Success Criteria:**
```bash
$ watchllm simulate
> severity: 0.7
> trace saved to R2
```

**Daily Rhythm:**
- Week 1-2: Infrastructure/schema (you define contracts, agents implement)
- Week 3: Integration and manual testing (you heavily involved)
- Week 4: Bug fixes and documentation

#### Month 2: Graph & Inspection
**Scope:**
- Add execution graph structure to traces (nodes, edges, types)
- Simple web view: read-only replay (no forking yet)
- Add 2-3 more attack categories: `tool_abuse`, `hallucination`
- SDK decorator: `@watchllm.test()` working

**Success Criteria:**
- Load simulation in dashboard
- See nodes rendered as circles/lines
- Click node to inspect input/output/metadata

#### Month 3: Fork & Replay (Differentiation)
**Scope:**
- State reconstruction from any node in trace
- Fork simulation with modified input/prompt/tool response
- Side-by-side comparison: original (failed) vs fork (passed)

**Success Criteria:**
- Click "fork from here" on any node
- Edit input, rerun from that state
- See both traces in comparison view

#### Month 4: Scale & Polish
**Scope:**
- All 8 attack categories implemented
- CI/CD integration (GitHub Actions, exit codes)
- Billing/tier gates (Stripe webhooks, feature flags)
- Performance optimization (gzip, caching, query optimization)

**Critical Rule:** If Month N's slice doesn't work, extend Month N. Do NOT start Month N+1.

---

## AI-Assisted Code Quality Framework

### Your Role: Architectural Constraint Setter
You are not the typist. You are the **acceptance criteria** and **contract validator**.

### Pre-Generation Protocol
Before any agent writes code, you must define:

| Element | Your Responsibility | Example |
|---------|---------------------|---------|
| Interface Contracts | Function signatures, data structures | `POST /api/v1/simulations` returns `{ data: T, error: null \| object }` |
| Test Assertions | Expected behavior, severity thresholds | `severity > 0.7` = `compromised: true` |
| Complexity Limits | Max lines, max branching, max nesting | Functions max 50 lines, max 3 nested ifs |
| Safety Rules | Non-negotiable security patterns | No SQL concat, no `any` types, no swallowed errors |

### Post-Generation Review: Three-Layer Check

**Layer 1: Logic Review**
- Does output match approved contract?
- Red flags: Magic numbers, undefined behavior, implicit assumptions

**Layer 2: Safety Review**
- Injection risks? SQL concat, `eval()`, `new Function()`
- User input in strings without sanitization?
- Secrets in code?

**Layer 3: Idiom Review**
- TypeScript: strict mode, no `any`, proper error typing
- Workers: edge-native patterns, no Node.js assumptions
- SQL: parameterized queries only (no string templates)

### Agent Prompt Engineering (System Prompt)
```
You are a senior TypeScript engineer writing for Cloudflare Workers.
Follow these rules absolutely:
- No 'any' types. Use strict TypeScript with explicit interfaces.
- No string concatenation for SQL. Parameterized queries via D1 only.
- No console.log in production paths. Use structured logging with context.
- All errors must be typed and handled, never swallowed with empty catch.
- All async operations must have timeout handling (max 30s default).
- All external calls must implement circuit breaker pattern.
- All functions must be under 50 lines. Extract helpers aggressively.
- All database queries must be in migration files, never inline.
```

### Automated Quality Gates
**Required tooling in CI:**
- `tsc --noEmit` (type checking)
- `eslint` with strict rules
- `prettier` (formatting)
- `mypy` for Python SDK
- **Zero warnings policy**: Agents must fix all warnings, not just errors

### Staging Infrastructure
```
staging.watchllm.dev
  ↓ (auto-deploy on PR)
Integration tests run (GitHub Actions)
  ↓
You review test results
  ↓
Manual approval → production (watchllm.dev)
```

### Human Override Authority
When agent code **technically passes** but **feels wrong**:

1. **Immediate**: `git revert` + demand rewrite
2. **Document**: "Rejected: [specific principle violated]"
3. **Track**: Build rejection log to train your taste

**Example rejections:**
- "Works but violates single responsibility (function does 3 things)"
- "Passes tests but has implicit coupling to R2 structure"
- "Correct output but uses 4 nested callbacks instead of async/await"

---

## Daily Operational Rhythm

### Morning (30 min): State Check
```
1. Run existing test suite (did yesterday's code survive?)
2. Review yesterday's agent output against contracts
3. Identify integration blockers
```

### Midday (2 hrs): Specification & Generation
```
1. Define today's interface contracts (you)
2. Write test assertions that must pass (you)
3. Prompt agents with contracts + constraints (you)
4. Agents generate implementation
```

### Afternoon (1 hr): Review & Validation
```
1. Three-layer review (logic, safety, idiom)
2. Run static analysis (tsc, eslint, mypy)
3. Execute integration tests manually
4. Deploy to staging if passing
```

### Evening (30 min): Documentation
```
1. Update decision log (what worked, what didn't)
2. Refine tomorrow's contracts based on learnings
3. Check staging health
```

### Weekend: Deep Architecture Review
```
1. Review accumulated codebase for structural drift
2. Refactor contracts if assumptions changed
3. Plan next week's vertical slice scope
```

---

## Risk Mitigation: When to Stop

### Daily Stopping Rule
If anything is wrong at end of day: **stop immediately**. Do not "prompt through" the night.

### Monthly Go/No-Go Gates

| Month | Gate Criteria | If Failed |
|-------|---------------|-----------|
| 1 | CLI runs one attack, produces severity score, saves trace | Extend Month 1, cut scope further |
| 2 | Dashboard renders graph, node inspection works | Extend Month 2, defer SDK polish |
| 3 | Fork creates new run from any node, side-by-side view works | Extend Month 3, simplify state reconstruction |
| 4 | All 8 categories, CI/CD passes, billing gates functional | Launch or extend for polish |

### Scope Cut Hierarchy (Emergency Use)
If behind schedule, cut in this order:
1. **First**: Reduce attack categories (ship with 3, not 8)
2. **Second**: Simplify graph visualization (text tree vs interactive nodes)
3. **Third**: Defer fork & replay to post-launch (still valuable without it)
4. **Never cut**: Core stress testing loop, severity scoring, trace storage

---

## Success Metrics by Phase

### Month 1: Validation
- [ ] `watchllm simulate` executes without errors
- [ ] Produces severity score 0.0-1.0
- [ ] Saves trace to R2 (gzip JSON)
- [ ] Manual verification of 5 test runs

### Month 2: Usability
- [ ] Dashboard loads simulation list
- [ ] Graph renders nodes and edges
- [ ] Click node shows input/output
- [ ] SDK decorator works with `@watchllm.test()`

### Month 3: Differentiation
- [ ] Fork button creates new simulation from any node
- [ ] State reconstruction matches original context
- [ ] Side-by-side comparison view functional
- [ ] 3+ attack categories running

### Month 4: Launch Readiness
- [ ] All 8 attack categories implemented
- [ ] GitHub Action exits 0/1 based on threshold
- [ ] Stripe billing gates replay/fork on free tier
- [ ] 10+ beta users with positive feedback

---

## Final Strategic Notes

### The Real Timeline Expectation
- **Months 1-2**: Accept most working code, build taste
- **Months 3-4**: Start rejecting substandard agent output
- **Months 5-6**: Anticipate problems in contracts before agents code

By Month 4, you are not "a non-coder using agents" — you are a **product architect with very fast typists**.

### The Conversion Funnel (Revisited)
Your free tier must find **real failures** or it doesn't convert. The attack quality is the funnel driver, not feature gates.

### Domain as Commitment Device
`watchllm.dev` is secured. This is your public commitment. Ship Month 1 vertical slice to staging on this domain within 30 days.

---

*Document generated from strategic planning session. Replaces all previous timeline estimates. Vertical slice methodology is non-negotiable.*