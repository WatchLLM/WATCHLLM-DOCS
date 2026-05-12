# WatchLLM — SDK & CLI Specification

## Python SDK

### Installation
```bash
pip install watchllm
```

### The @watchllm.test() Decorator
```python
import watchllm

@watchllm.test(
    categories=["prompt_injection", "tool_abuse"],  # required
    threshold="severity < 0.3",                     # optional, for CI
    project_id="prj_xxxx",                          # optional
    wait=True                                        # wait for results
)
def my_agent(user_input: str) -> str:
    # your agent code here
    return response
```

Parameters:
- `categories`: list of attack category IDs. Required.
  Pass ["all"] to run every category.
- `threshold`: expression for CI pass/fail.
  "severity < 0.3" means fail if any category scores >= 0.3.
  Supports: <, >, <=, >=, ==, and, or
- `project_id`: which project to associate with. Defaults to default project.
- `wait`: if True, blocks until simulation completes. Default False.
- `timeout`: seconds to wait if wait=True. Default 300.

What it does internally:
1. Registers the agent (POST /api/v1/agents) on first call
2. Launches a simulation (POST /api/v1/simulations)
3. If wait=True, polls status until terminal state
4. If threshold set and wait=True: evaluates threshold, raises
   WatchLLMThresholdError if failed

### Authentication
SDK reads API key from:
1. `WATCHLLM_API_KEY` environment variable
2. `~/.watchllm/config` file (created by `watchllm auth login`)

### Framework Compatibility
The decorator works by wrapping the function and exposing its signature
to WatchLLM's attack runner. Works with any framework as long as the
agent is callable as a Python function.

Tested with: LangChain agents, CrewAI crews, raw OpenAI, AutoGen.
For async agents: use `@watchllm.test_async(...)` variant.

---

## CLI

### watchllm auth login
Interactive flow. Opens browser to GitHub OAuth.
Saves API key to ~/.watchllm/config.

### watchllm simulate
```bash
watchllm simulate \
  --agent my_module.my_agent \
  --categories prompt_injection,tool_abuse \
  --threshold "severity < 0.3"
```
Launches a simulation and polls for results.
Exits 0 on pass, 1 on fail (for CI/CD).

### watchllm replay
```bash
watchllm replay --simulation sim_xxxx
```
Prints a text-based tree of the execution graph to terminal.
Shows node types, latency, and any flagged nodes.

### watchllm status
```bash
watchllm status --simulation sim_xxxx
```
Prints current status, completion %, severity scores per category.

### watchllm doctor
```bash
watchllm doctor
```
Checks: API key valid, agent reachable, network connectivity.
Prints a pass/fail checklist.

---

## CI/CD Integration

### GitHub Actions
```yaml
- name: WatchLLM agent reliability check
  run: |
    pip install watchllm
    watchllm simulate \
      --agent src.agent.my_agent \
      --categories all \
      --threshold "severity < 0.3"
  env:
    WATCHLLM_API_KEY: ${{ secrets.WATCHLLM_API_KEY }}
```

Exit codes:
- 0: simulation completed, threshold passed (or no threshold set)
- 1: simulation completed, threshold failed
- 2: simulation failed (worker error)
- 3: timeout