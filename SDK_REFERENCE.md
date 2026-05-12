# SDK Reference

This reference covers the complete WatchLLM Python SDK and CLI. Use it to integrate WatchLLM into your agent development and testing workflows.

## Installation

```bash
pip install watchllm
```

Requires Python 3.9+. Installs the `watchllm` CLI and `watchllm` Python package.

## Authentication

All operations require authentication via API key.

### Environment Variable (CI/CD)
```bash
export WATCHLLM_API_KEY="wllm_your_api_key"
```

### Interactive Login (Local Dev)
```bash
watchllm auth login
```
Prompts for your key and stores it in `~/.watchllm/config`.

### Verification
```bash
watchllm doctor
```
Checks API connectivity, key validity, and agent reachability.

## Python SDK

### Core Client

```python
from watchllm import WatchLLMClient

client = WatchLLMClient()  # Uses env var or config file
```

#### Methods

**simulate(agent_fn, categories, threshold=None, wait=True, project_id=None)**

Runs a simulation against your agent.

- `agent_fn`: Callable (str -> str) - Your agent function
- `categories`: List[str] or "all" - Attack categories to test
- `threshold`: float (0.0-1.0) - Fail if severity exceeds this
- `wait`: bool - Wait for completion (default True)
- `project_id`: str - Optional project grouping

Returns: `SimulationResult` dataclass

```python
@dataclass
class SimulationResult:
    id: str          # sim_xxx
    status: str      # queued | running | completed | failed
    severity: float  # 0.0-1.0 overall score
    verdict: str     # Human-readable summary
    categories: dict # {category: severity} for each tested
```

**Example:**
```python
result = client.simulate(my_agent, ["prompt_injection", "tool_abuse"])
print(f"Severity: {result.severity}")
```

### Decorators

#### @watchllm.test()

Wraps an agent function with automatic testing.

```python
import watchllm

@watchllm.test(
    categories=["prompt_injection"],  # Required
    threshold=0.3,                   # Optional: raise error if exceeded
    wait=True,                       # Optional: wait for completion
    project_id="prj_xxx"             # Optional: group simulations
)
def my_agent(user_input: str) -> str:
    return "Response"
```

- On each call, triggers a simulation
- If `wait=True`, blocks until completion
- If `threshold` set and severity > threshold, raises `WatchLLMThresholdError`
- Returns your agent's normal response

**WatchLLMThresholdError:**
```python
try:
    response = my_agent("Hello")
except watchllm.WatchLLMThresholdError as e:
    print(f"Vulnerability detected: {e.severity}")
    print(f"Verdict: {e.verdict}")
    # Handle error (e.g., log, alert)
```

#### @watchllm.test_async()

Async version for async agents.

```python
@watchllm.test_async(categories=["all"], threshold=0.5)
async def my_async_agent(user_input: str) -> str:
    # Async agent logic
    return await some_async_call(user_input)
```

### Error Handling

**Common Exceptions:**
- `WatchLLMThresholdError`: Severity exceeded threshold
- `AuthenticationError`: Invalid API key
- `RateLimitError`: Free tier limit reached
- `TimeoutError`: Simulation took too long
- `NetworkError`: Connectivity issues

**Best Practices:**
```python
try:
    result = client.simulate(agent, categories)
except watchllm.AuthenticationError:
    # Re-authenticate
except watchllm.RateLimitError:
    # Upgrade plan or wait
except Exception as e:
    # Log and retry
```

## CLI Commands

### watchllm simulate

Runs a simulation from the command line.

```bash
watchllm simulate --agent module.function --categories category1,category2
```

**Options:**
- `--agent, -a`: Python import path (e.g., `my_module.my_agent`)
- `--categories, -c`: Comma-separated list or "all"
- `--threshold, -t`: Fail if severity > threshold
- `--project-id, -p`: Project ID for grouping
- `--wait, -w`: Wait for completion (default true)
- `--async`: For async agents (use `asyncio.run()`)

**Examples:**
```bash
# Basic test
watchllm simulate -a my_agent.main -c prompt_injection

# All categories with threshold
watchllm simulate -a agents.chatbot -c all -t 0.3

# Async agent
watchllm simulate -a my_async_agent -c tool_abuse --async
```

**Exit Codes:**
- 0: Success (severity ≤ threshold)
- 1: Threshold exceeded
- 2: Worker error
- 3: Timeout/network error

### watchllm replay

Displays execution graph in terminal.

```bash
watchllm replay --simulation sim_abc123
```

Prints a text-based tree of nodes, latencies, and severity indicators.

### watchllm status

Checks simulation progress.

```bash
watchllm status --simulation sim_abc123
```

Shows completion percentage and live severity scores.

### watchllm auth login

Interactive key setup.

```bash
watchllm auth login
```

### watchllm doctor

Diagnoses setup issues.

```bash
watchllm doctor
```

Checks:
- API key validity
- Network connectivity
- Agent importability (if --agent specified)

## Attack Categories

WatchLLM tests against 8 categories:

- **prompt_injection**: Adversarial prompts trying to override instructions
- **tool_abuse**: Attempts to misuse external tools or APIs
- **hallucination**: Inducing confident but incorrect responses
- **context_poisoning**: False information injected into agent context
- **infinite_loop**: Inputs causing endless cycles
- **data_exfiltration**: Probing for sensitive data leaks
- **role_confusion**: Tricking agent into adopting unauthorized roles
- **jailbreak**: Breaking safety restrictions

Use "all" to test every category.

## Rate Limits & Tiers

- **Free**: 5 simulations/month, 3 categories, no graph replay
- **Pro**: 100 simulations/month, all categories, full features
- **Team**: 500 simulations/month, team collaboration

Upgrade via [dashboard.watchllm.dev/upgrade](https://dashboard.watchllm.dev/upgrade).

## Integration Examples

### LangChain Agent
```python
from langchain import LLMChain, PromptTemplate
from watchllm import test

template = "You are a helpful assistant. {user_input}"
chain = LLMChain(llm=llm, prompt=PromptTemplate(template=template))

@test(categories=["prompt_injection"])
def langchain_agent(user_input: str) -> str:
    return chain.run(user_input=user_input)
```

### AutoGen Assistant
```python
from autogen import AssistantAgent
from watchllm import test

agent = AssistantAgent("helper", llm_config=llm_config)

@test(categories=["tool_abuse"])
def autogen_agent(user_input: str) -> str:
    response = agent.generate_reply(messages=[{"role": "user", "content": user_input}])
    return response["content"]
```

### CI/CD Pipeline
```yaml
# .github/workflows/test.yml
name: Agent Security Tests
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v4
        with: { python-version: '3.9' }
      - run: pip install -r requirements.txt watchllm
      - run: watchllm simulate -a my_agent -c all -t 0.3
        env:
          WATCHLLM_API_KEY: ${{ secrets.WATCHLLM_API_KEY }}
```

## Troubleshooting

### Import Errors
- Ensure agent function is at module level
- Use absolute imports: `my_package.agents.main_agent`

### Performance
- Simulations take 1-5 minutes depending on categories
- Free tier: 3 categories max, 30s timeout per run
- Pro/Team: All categories, 60s timeout

### Debugging Failures
- Use `watchllm replay` for terminal graph view
- Visit dashboard for full interactive graph
- Check agent logs for custom error handling

For conceptual guides, see [Simulation Guide](SIMULATION_GUIDE.md). For getting started, see [Quick Start](QUICK_START.md).