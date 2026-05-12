# Quick Start Guide

Welcome to WatchLLM! This guide will get you up and running with agent reliability testing in under 10 minutes. We'll cover installation, authentication, and your first simulation.

## Prerequisites

- Python 3.9 or higher
- A GitHub account (for authentication)
- Basic familiarity with command-line tools

If you're new to Python or agents, don't worry—we'll explain each step.

## Step 1: Install the SDK

WatchLLM provides a Python SDK for testing your AI agents. Install it via pip:

```bash
pip install watchllm
```

This installs the `watchllm` command-line tool and Python library.

**Troubleshooting:**
- If you get permission errors, use `pip install --user watchllm` or a virtual environment.
- On Windows, ensure Python is in your PATH.

## Step 2: Authenticate with WatchLLM

WatchLLM uses API keys for secure access. Create your first key through our dashboard:

1. Visit [https://dashboard.watchllm.dev](https://dashboard.watchllm.dev)
2. Sign in with your GitHub account
3. Navigate to Settings > API Keys
4. Click "Create API Key" and give it a name (e.g., "My First Key")
5. Copy the key immediately—it won't be shown again!

Store your key securely. Never commit it to version control.

### Set Your API Key

**Option 1: Environment Variable (Recommended for Production)**
```bash
# Linux/Mac
export WATCHLLM_API_KEY="wllm_your_key_here"

# Windows PowerShell
$env:WATCHLLM_API_KEY = "wllm_your_key_here"
```

**Option 2: Interactive Login (Easier for Local Development)**
```bash
watchllm auth login
```
This prompts for your key and saves it securely in `~/.watchllm/config`.

**Verification:**
```bash
watchllm doctor
```
This checks your setup and key validity.

## Step 3: Prepare Your Agent

WatchLLM tests AI agents that can be called as Python functions. Your agent should:

- Accept a string input
- Return a string output
- Be importable in your Python environment

Example agent (save as `my_agent.py`):

```python
def my_agent(user_input: str) -> str:
    # Your agent logic here
    # For example, a simple echo agent:
    return f"You said: {user_input}"
```

**Common Agent Frameworks:**
- **Raw OpenAI/Claude:** Wrap API calls in a function
- **LangChain:** Use `chain.run()` as your agent
- **AutoGen:** Wrap the assistant's response method
- **CrewAI:** Wrap the crew's execute method

If your agent uses tools or has complex state, ensure it's callable as a single function.

## Step 4: Run Your First Simulation

Run a basic prompt injection test:

```bash
watchllm simulate --agent my_agent.my_agent --categories prompt_injection
```

**What this does:**
- Loads your agent function
- Runs 10 adversarial prompts designed to compromise AI agents
- Scores each run for vulnerability (0.0 = safe, 1.0 = critical)
- Saves results to your dashboard

**Expected Output:**
```
Simulation ID: sim_abc123
Status: completed
Severity: 0.15
Verdict: Low risk detected
```

**Run All Categories:**
```bash
watchllm simulate --agent my_agent.my_agent --categories all
```
Tests against 8 attack types: prompt injection, tool abuse, hallucination, context poisoning, infinite loops, data exfiltration, role confusion, and jailbreaks.

## Step 5: View Results in Dashboard

1. Visit [https://dashboard.watchllm.dev/simulations](https://dashboard.watchllm.dev/simulations)
2. Find your simulation by ID or timestamp
3. Click to view the execution graph

The graph shows:
- Each step your agent took
- Which attacks succeeded or failed
- Detailed input/output for each node

## Step 6: Integrate into Your Workflow

### Python Code Integration
For automated testing in CI/CD:

```python
import watchllm

@watchllm.test(categories=["prompt_injection", "tool_abuse"], threshold=0.3)
def my_agent(user_input: str) -> str:
    # Your agent code
    return response
```

This runs tests on every call and raises `WatchLLMThresholdError` if vulnerabilities exceed the threshold.

### CI/CD Integration
Add to your GitHub Actions workflow:

```yaml
- name: Test Agent Security
  run: |
    pip install watchllm
    watchllm simulate -a my_agent -c all -t 0.3
```

Fails the build if severity > 0.3.

## Common Issues and Solutions

### "Module not found"
Ensure your agent file is in the Python path. Use `--agent my_agent:my_agent` for relative imports.

### "Authentication failed"
- Verify your API key is set correctly
- Check the key hasn't been revoked in the dashboard
- Run `watchllm doctor` for diagnostics

### Simulations hang or timeout
- Ensure your agent responds within 30 seconds
- Check for infinite loops in your agent logic
- Network issues: verify internet connectivity

### Low severity scores on vulnerable agents
- Test with known vulnerable prompts manually first
- Ensure your agent actually processes the input (not just echoes)

## Next Steps

- **Explore Advanced Features:** Learn about fork & replay for debugging
- **Upgrade Your Plan:** Free tier includes 5 simulations/month; Pro offers unlimited
- **Join the Community:** Report issues or request features on our GitHub repo

For detailed API references, see [SDK Reference](SDK_REFERENCE.md). For advanced simulation concepts, see [Simulation Guide](SIMULATION_GUIDE.md).

Need help? Our docs are designed to be comprehensive—search this page or check the FAQ in our dashboard.