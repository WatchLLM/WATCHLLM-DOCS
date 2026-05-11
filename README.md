<p align="center">
  <img src="https://raw.githubusercontent.com/WatchLLM/.github/refs/heads/main/edited%20(1).png" width="800" alt="WatchLLM Banner" />
</p>

<h3 align="center">
  AI agent reliability testing infrastructure. Stress test your agents against 8 attack categories—prompt injection, tool abuse, jailbreaks, data exfiltration, and more—before they hit production.
</h3>

<p align="center">
  <strong>Observability, evaluation, and adversarial testing platform for LLM agents.</strong><br>
  Ship reliable, secure, and debuggable agentic systems.
</p>

<p align="center">
  <a href="https://watchllm.dev">Website</a> • 
  <a href="https://watchllm.dev/docs">Documentation</a>
</p>

<div align="center">
  
  [![GitHub Stars](https://img.shields.io/github/stars/kaadipranav/watchllm.dev?style=flat-square&logo=github)](https://github.com/kaadipranav/watchllm.dev)
  [![License](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
  [![TypeScript](https://img.shields.io/badge/TypeScript-Ready-blue.svg)](https://www.typescriptlang.org/)
  [![Python](https://img.shields.io/badge/Python-Ready-blue.svg)](https://www.python.org/)

</div>

----

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
