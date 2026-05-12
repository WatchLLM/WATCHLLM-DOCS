# JS/Node SDK Implementation Plan

Based on the existing Python SDK (from BUILDPLAN.md and project knowledge), here's a comprehensive plan to implement a JS/Node SDK for WatchLLM. This mirrors the Python SDK's features: client library, decorators, CLI, async support, and integration with the WatchLLM API. The plan follows the same architecture, ensuring consistency across languages.

## Overall Goals
- Provide a TypeScript-first Node.js SDK (ESM + CommonJS compatible).
- Support popular JS agent frameworks (LangChain.js, Vercel AI SDK, etc.).
- Enable CI/CD integration via CLI and decorators.
- Publish to npm as `watchllm` (scoped or not, depending on naming).
- Maintain API compatibility with Python SDK for core features.

## Detailed Implementation Plan

1. **Review Existing Specs and Python SDK**
   - Reference BUILDPLAN.md for monorepo structure and shared types.
   - Study Python SDK in `packages/sdk/` for API design, error handling, and CLI commands.
   - Ensure JS SDK uses same endpoints, auth (API key), and response formats.

2. **Design JS SDK API**
   - **Client Class**: `WatchLLMClient` with methods like `simulate(agentFn, categories, options)`.
   - **Decorators**: `@watchllm.test()` for wrapping agent functions, supporting async agents.
   - **CLI**: `watchllm simulate --agent file.js --categories all --threshold 0.3`.
   - **Async Support**: Native Promises/async-await, no sync blocking.
   - **Error Handling**: Typed errors (e.g., `WatchLLMThresholdError`), rate limiting, timeouts.
   - **Config**: Environment variables or `.watchllm/config.json` for API keys.

3. **Create WATCHLLM-SDK-JS Repo**
   - New public GitHub repo: `WatchLLM/WATCHLLM-SDK-JS`.
   - Initialize with Node.js, TypeScript, Jest for testing.
   - Package.json: `"name": "watchllm"`, `"bin": {"watchllm": "dist/cli.js"}`.
   - Use shared types from core repo if possible (or define locally).

4. **Implement Core Client**
   - HTTP client using `node-fetch` or native `fetch` (Node 18+).
   - Methods: `simulate`, `getStatus`, `getReplay`.
   - Handle auth, retries, timeouts (30s max).
   - Typed responses matching API contracts.

5. **Implement @watchllm.test Decorator**
   - Decorator for functions: intercepts calls, runs simulations, throws on threshold exceed.
   - Support async: `@watchllm.test({categories: ["prompt_injection"], threshold: 0.3})`.
   - Options: categories, threshold, wait, projectId.

6. **Build CLI Tool**
   - Use `commander.js` or `yargs` for parsing.
   - Commands: `simulate`, `replay`, `status`, `auth login`, `doctor`.
   - Support for JS/TS agents: `--agent path/to/agent.js`.
   - Exit codes: 0 (pass), 1 (fail), 2 (error).

7. **Add Examples for JS Frameworks**
   - LangChain.js: Wrap `chain.call()` with decorator.
   - Vercel AI SDK: Integrate with streaming responses.
   - Raw OpenAI: Direct API calls.
   - Examples in `/examples` with READMEs.

8. **Write Tests and Ensure Compatibility**
   - Unit tests for client, decorators, CLI.
   - Integration tests against staging API.
   - Mock agents for testing vulnerabilities.
   - Ensure same attack categories and severity scoring as Python.

9. **Publish to npm and Release**
   - Build with `tsc`, publish via `npm publish`.
   - Create GitHub release v0.1.0 with changelog.
   - Update docs with JS SDK reference.
   - Add npm badge to repo README.

## Timeline and Dependencies
- **Week 1**: Repo setup, core client, basic simulate.
- **Week 2**: Decorators, CLI, examples.
- **Week 3**: Testing, compatibility checks.
- **Week 4**: Publish, docs update.

This plan ensures the JS SDK feels native to Node.js developers while maintaining feature parity with Python. If shared types evolve, update both SDKs accordingly.

## Current Todos
- Review existing Python SDK implementation and specs from BUILDPLAN.md and SDK_SPEC.md
- Design JS SDK API mirroring Python SDK: client, decorators, CLI, async support
- Create WATCHLLM-SDK-JS repo with Node.js/TypeScript setup
- Implement core client for API interactions
- Implement @watchllm.test decorator for JS agents
- Build CLI tool with commands like simulate, replay, status
- Add examples for popular JS frameworks (e.g., LangChain.js, Vercel AI SDK)
- Write tests and ensure compatibility with WatchLLM API
- Publish to npm and create GitHub release