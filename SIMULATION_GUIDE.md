# Simulation Guide

This guide explains how WatchLLM simulations work, how to interpret results, and how to use advanced features like fork & replay for debugging.

## How Simulations Work

WatchLLM stress-tests your AI agents by running them against adversarial inputs designed to expose vulnerabilities. Each simulation produces an execution graph that captures every step of your agent's behavior.

### Attack Categories

Simulations test against one or more attack categories:

- **Prompt Injection**: Attempts to override your agent's instructions
- **Tool Abuse**: Tries to make your agent misuse external tools
- **Hallucination**: Induces confident but incorrect responses
- **Context Poisoning**: False information injected into context
- **Infinite Loop**: Causes endless response cycles
- **Data Exfiltration**: Probes for sensitive data leaks
- **Role Confusion**: Tricks agent into unauthorized roles
- **Jailbreak**: Breaks safety restrictions

### Execution Graph

Every simulation generates a directed graph of your agent's execution:

- **Nodes**: Represent individual steps (LLM calls, tool usages, decisions)
- **Edges**: Show flow between steps
- **Metadata**: Timestamps, latencies, token counts, input/output

Example graph structure:
```
Start → LLM Call (attack input) → Tool Usage → Decision → LLM Call (response) → End
```

### Severity Scoring

Each run receives a severity score (0.0-1.0):

- **0.0-0.2**: Low risk - Minor issues or false positives
- **0.3-0.6**: Medium risk - Potential vulnerabilities requiring attention
- **0.7-1.0**: High risk - Critical vulnerabilities that could compromise production

Scoring combines:
- **Rule-based filters**: Detect obvious patterns (destructive commands, PII leaks)
- **LLM judge**: AI evaluates semantic compromise using context

## Using the Dashboard

The dashboard at [dashboard.watchllm.dev](https://dashboard.watchllm.dev) provides visual tools for analyzing simulations.

### Authentication

Sign in with GitHub OAuth. Dashboard sessions are separate from API keys.

### Simulations List

- View all your simulations chronologically
- Filter by status, severity, project
- See overall stats: total runs, average severity, pass/fail rate

### Simulation Detail View

Click any simulation to see:

- **Metadata**: ID, timestamp, categories tested, overall severity
- **Graph**: Interactive execution visualization
- **Verdict**: AI-generated summary of findings

### Graph Interaction

- **Zoom/Pan**: Mouse wheel and drag
- **Node Inspection**: Click nodes to see full input/output
- **Timeline Scrubber**: Slide to replay execution step-by-step
- **Flagged Nodes**: Red borders indicate vulnerability points

## Fork & Replay

Fork & replay lets you debug by editing inputs and re-running from any point in the execution.

### When to Use Fork & Replay

- **Debug failures**: Understand why a specific path led to compromise
- **Test fixes**: Modify inputs to verify patches work
- **Explore alternatives**: See how different responses affect outcomes

### How to Fork

1. Open a completed simulation in the dashboard
2. Find the node where you want to intervene
3. Right-click the node → "Fork from here"
4. Edit the input in the modal
5. Click "Run Fork"

### Fork Results

- **New Simulation**: Creates a separate sim with forked execution
- **Compare View**: Side-by-side graphs showing original vs. fork
- **Lineage Tracking**: See parent/child relationships between simulations

### Example Use Case

Your agent fails a prompt injection attack. The graph shows it at node 3, where it calls a dangerous tool.

**Fork Strategy:**
1. Right-click node 2 (before the bad decision)
2. Edit input to add safety checks
3. Run fork → Verify the attack now fails

## Interpreting Results

### Understanding Severity

Severity isn't binary—it's contextual:

- **High severity on critical paths**: Fix immediately
- **Medium severity on edge cases**: Monitor and mitigate
- **Low severity on rare inputs**: Log for awareness

### Common Patterns

- **Flagged tool calls**: Agent misusing APIs
- **PII in responses**: Data exfiltration risks
- **Role adoption**: Context poisoning success
- **Infinite loops**: Missing termination conditions

### False Positives

WatchLLM aims for accuracy, but false positives can occur:

- **Over-cautious judge**: Safe responses scored medium due to wording
- **Rule matches**: Benign text triggering patterns (e.g., "DROP TABLE" in SQL docs)

Review flagged nodes manually to confirm real issues.

## Advanced Features

### Projects

Group related simulations:

- Create projects in Settings
- Assign simulations via SDK: `client.simulate(agent, project_id="prj_xxx")`
- View project-specific analytics

### API Keys Management

- Create multiple keys for different environments
- Revoke compromised keys immediately
- Track usage per key

### Billing & Limits

- **Free Tier**: 5 sims/month, basic features
- **Pro**: 100 sims/month, full replay, fork & replay
- **Team**: 500 sims/month, collaboration features

Upgrade in Settings > Billing.

## Best Practices

### Testing Strategy

- **Start small**: Test individual categories before "all"
- **Iterate**: Fix high-severity issues first
- **Automate**: Use decorators in CI/CD for continuous testing

### Agent Preparation

- **Consistent interface**: Ensure agents are callable as (str) -> str
- **Error handling**: Agents should handle malformed inputs gracefully
- **Logging**: Add debug logs to correlate with graph nodes

### Security Considerations

- **API keys**: Never commit to code; use environment variables
- **Network**: Simulations run on WatchLLM infrastructure, not yours
- **Data privacy**: Inputs/outputs are stored securely but review before production use

## Troubleshooting Simulations

### Common Issues

- **Timeout errors**: Agent takes >30s; optimize or increase limits
- **Import failures**: Ensure agent module is in Python path
- **High false positive rate**: Review agent context; may need custom scoring

### Debugging with Graphs

- **Node inspection**: Check exact inputs causing failures
- **Latency spikes**: Identify slow operations
- **Token usage**: Monitor for unexpected consumption

### Support

- **Docs first**: These guides cover most use cases
- **Dashboard help**: Tooltips and inline explanations
- **GitHub issues**: For bugs or feature requests

For SDK usage, see [SDK Reference](SDK_REFERENCE.md). For setup, see [Quick Start](QUICK_START.md).