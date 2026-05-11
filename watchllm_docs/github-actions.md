# WatchLLM — GitHub Actions Integration

## Quick start (raw CLI)

Copy this into `.github/workflows/watchllm.yml` in your repo:

```yaml
name: WatchLLM agent reliability check

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  watchllm:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: WatchLLM agent reliability check
        run: |
          pip install watchllm
          watchllm simulate \
            --agent src.agent.my_agent \
            --categories all \
            --threshold 0.3 \
            --timeout 300
        env:
          WATCHLLM_API_KEY: ${{ secrets.WATCHLLM_API_KEY }}
```

Replace `src.agent.my_agent` with the Python import path to your agent function.

## Using the composite action

```yaml
name: WatchLLM agent reliability check

on:
  pull_request:
    branches: [main]

jobs:
  watchllm:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: WatchLLM agent reliability check
        uses: watchllm/watchllm/.github/watchllm-action@v1
        with:
          api-key: ${{ secrets.WATCHLLM_API_KEY }}
          agent: src.agent.my_agent
          categories: all
          threshold: "0.3"
```

## Exit codes

| Code | Meaning |
|------|---------|
| 0 | Simulation completed, threshold passed (or no threshold set) |
| 1 | Simulation completed, threshold **failed** — agent is vulnerable |
| 2 | Worker error or API error |
| 3 | Timeout — simulation did not complete within `--timeout` seconds |

## Setting up the secret

1. Go to your repo → Settings → Secrets and variables → Actions
2. Click "New repository secret"
3. Name: `WATCHLLM_API_KEY`
4. Value: your WatchLLM API key (from https://dashboard.watchllm.dev/settings/api-keys)

## Failure output example

When a vulnerable agent is detected, the workflow fails with exit code 1
and prints:

```
==================================================
Simulation Complete
==================================================
Simulation ID: sim_a1b2c3d4e5f6
Status:        completed
Categories:    all
Severity:      0.87
Verdict:       Critical vulnerability detected

✗ FAILED  severity=0.87  threshold=0.30
  Verdict: Critical vulnerability detected
```

## Threshold guidance

| Threshold | Meaning |
|-----------|---------|
| 0.3 | Strict — fails on any meaningful compromise |
| 0.5 | Moderate — fails on high-confidence compromises |
| 0.7 | Permissive — fails only on critical compromises |

Start with 0.3 for new agents. Raise it only if you have documented
false positives you've reviewed and accepted.
