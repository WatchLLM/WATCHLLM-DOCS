# WatchLLM — Product Definition

## What It Is
WatchLLM is an agent reliability platform. It lets engineers stress test,
replay, and debug AI agents before and after production failures.

It is not a logger. It is not an observability dashboard. It is not a
LangSmith competitor. Those tools watch agents passively. WatchLLM breaks
them on purpose, then gives you the tools to understand and fix what broke.

## The Three Core Features

### 1. Stress Testing
Run a battery of attack scenarios against any agent before it ships.
Attack categories include: prompt injection, tool abuse, hallucination
induction, context poisoning, infinite loop triggering, jailbreak attempts,
data exfiltration probing, and role confusion.

Each run produces a severity score (0.0–1.0) per attack category and an
overall reliability score. CI/CD integration lets you fail a pipeline if
severity exceeds a threshold.

### 2. Graph Replay
Every agent run — whether from a stress test or production — is recorded
as a directed graph of execution nodes. Each node captures: type (llm_call,
tool_call, decision, memory_read, memory_write), input, output, timestamp,
latency, token count, and cost.

The dashboard lets engineers scrub through this graph chronologically.
Any node is inspectable. The exact moment a failure occurred is visible.

### 3. Fork & Replay
From any node in any recorded run, an engineer can fork a new run that
starts from that exact state. Change the input, change the prompt, change
the tool response — rerun from that node forward without re-executing
everything before it.

This eliminates the cost and time of full agent reruns for debugging.

## What It Is Not
- Not a general observability platform (no metrics dashboards, no uptime monitoring)
- Not a prompt management tool
- Not a model evaluation/evals platform (though it uses LLM-as-judge internally)
- Not a LangChain/CrewAI wrapper

## The User
AI engineers shipping agents to production. They use LangChain, CrewAI,
AutoGen, or raw OpenAI SDK. They have a working agent in dev that breaks
in prod in ways they can't reproduce. They are willing to pay to not be
woken up at 3am by a rogue agent deleting a database.

## Positioning
Category: Agent reliability platform
Tagline: Agent reliability, from first run to production.
One-liner: Your agent passed every test. WatchLLM shows you what it does
when things go wrong — before your users find out.

## Competitive Moat
No single tool has all four: stress testing + graph replay + fork & replay +
run versioning. Balagan Agent has chaos injection only. agent-replay has CLI
replay only. LangSmith/Langfuse have post-mortem logs only. WatchLLM is the
only unified platform.
