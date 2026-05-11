# WatchLLM — Monetization

## Tiers

### Free
- 5 simulations/month
- 3 attack categories (prompt_injection, tool_abuse, hallucination)
- 7-day run history
- No graph replay
- No fork & replay
- 1 user, 1 project
- CI/CD integration included (exit codes work, no dashboard)

### Pro — $29/month
- 100 simulations/month
- All 8 attack categories
- 90-day run history
- Full graph replay
- Fork & replay
- 1 user, unlimited projects
- Priority simulation queue

### Team — $99/month
- 500 simulations/month
- All 8 attack categories
- 365-day run history
- Full graph replay
- Fork & replay
- 10 users, unlimited projects
- Priority queue
- Slack notifications

## Payment Stack
- Stripe for all payments (international)
- First $1,000 in revenue: zero transaction fees (GitHub Student Pack)
- Stripe Billing for subscriptions
- Webhooks: /api/v1/webhooks/stripe for lifecycle events

## Gating Logic
- Categories gated at API level: chaos worker checks tier before running
- Replay gated at API level: GET /simulations/{id}/replay returns 403 on free
- Fork gated at API level: POST /simulations/{id}/fork returns 403 on free
- UI shows upgrade prompts at: replay attempt, fork attempt,
  5th simulation of month

## The Conversion Moment
Engineer runs a free simulation. It finds a failure. They want to:
1. See the graph replay → "Upgrade to Pro to replay this run"
2. Fork from the failure node → "Upgrade to Pro to fork this run"

That is the funnel. Free tier must find real failures or it doesn't convert.
The attack quality is what drives conversion, not the feature gates.

## Revenue Projections (Conservative)
Month 3: 10 Pro users = $290/month
Month 6: 50 Pro users = $1,450/month
Month 12: 200 Pro + 10 Team = $6,780/month

These are the numbers that get you to Germany at 18 with runway.
