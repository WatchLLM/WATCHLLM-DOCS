Welcome to the official documentation repository for WatchLLM. 

This repository contains all public-facing technical documentation, API specifications, and architectural guidelines required to deeply integrate WatchLLM into your agentic infrastructure.

## Repository Structure

Our documentation is split into three primary references:

1. **`QUICK_START.md`** 
   A 5-minute guide to generating your first API key, installing the SDK, and launching your first simulation against an agent. Start here if you are new to the platform.

2. **`SDK_REFERENCE.md`** 
   The complete specification of the WatchLLM Python SDK and CLI. This includes advanced decorator configurations, threshold setting for CI/CD pipelines, asynchronous agent wrapping, and CLI command flags.

3. **`SIMULATION_GUIDE.md`** 
   A deep dive into how WatchLLM handles execution graphs. Learn how the orchestrator maps LLM calls and tool usages, how severity scores are calculated by our internal judge, and how to utilize Fork & Replay inside the dashboard.

## Dashboard Integrations

All simulations run via the SDK are automatically synchronized with your cloud dashboard.

- **Graph Replay**: Every run is recorded as a directed execution graph. You can scrub through it node by node to see exactly which decision triggered the failure.
- **Fork & Replay**: Right-click any node in the dashboard to edit the input and rerun from that exact state—without re-executing everything before it.

To view your active simulations or manage API keys, visit the dashboard:
[https://dashboard.watchllm.dev](https://dashboard.watchllm.dev)

## Contributing & Support

If you encounter issues integrating the SDK with a specific framework (e.g., LangChain, AutoGen, CrewAI), please ensure your agent is callable as a standard Python function before submitting an issue. 

For additional support, bug reports, or feature requests regarding these documents, please open an issue in this repository.
