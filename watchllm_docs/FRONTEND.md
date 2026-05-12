# WatchLLM — Frontend Specification

## Tech
- Next.js 14+ (App Router)
- Deployed on Cloudflare Pages
- Tailwind CSS (utility only, no component library)
- No shadcn. No MUI. No Chakra. Build raw.
- TypeScript strict mode

## Design System

### Colors
```
background:     #080808
surface:        #0f0f0f
surface-raised: #161616
border:         rgba(255, 255, 255, 0.08)
border-hover:   rgba(255, 255, 255, 0.15)
accent:         #00C896
accent-dim:     rgba(0, 200, 150, 0.15)
text-primary:   #f0f0f0
text-secondary: #888888
text-tertiary:  #555555
danger:         #ff4444
warning:        #f59e0b
success:        #00C896
```

### Typography
- Font: Geist (already in Next.js), fallback: Inter
- Hero: 64px / weight 600 / line-height 1.1
- H1: 48px / weight 600
- H2: 32px / weight 500
- H3: 20px / weight 500
- Body: 16px / weight 400 / line-height 1.6
- Small: 14px / weight 400
- Mono: Geist Mono for code, metrics, IDs

### Components
- No border-radius above 8px except pills (9999px)
- Buttons: 36px height, 16px horizontal padding, 7px border-radius
- Cards: 1px solid border, 8px border-radius, surface background
- Inputs: 40px height, 8px border-radius, 1px border
- Tables over lists for structured data

### Animation
- CSS transitions only: 150ms ease for hover states
- No Framer Motion unless explicitly added later
- Subtle only: opacity, border-color, background-color
- No transforms that feel heavy

## Page Map

### Public (no auth)
```
/                 Landing page
/pricing          Pricing (anchor from landing)
/docs             Documentation (placeholder initially)
/sign-in          GitHub OAuth sign in
/sign-up          GitHub OAuth sign up (same flow)
```

### Dashboard (auth required)
```
/dashboard                    Overview: recent simulations, quick stats
/dashboard/simulations        All simulations list
/dashboard/simulations/[id]   Single simulation: progress + graph replay
/dashboard/agents             Registered agents
/dashboard/projects           Projects
/dashboard/failures           Failure library
/dashboard/settings           API keys, account, billing
```

## Landing Page Structure

Sections in exact order:

### 1. NAV
Sticky. Height 56px. Backdrop blur on scroll.
Left: WatchLLM wordmark
Center: Product · Docs · Pricing
Right: Sign in (ghost) · Get started (accent filled)
Collapses to hamburger below 768px.

### 2. HERO
Eyebrow: pill tag "Agent Reliability Platform" with pulsing green dot
H1 (two lines):
"Your agent works in dev.
WatchLLM makes it work in prod."
Subtext: One sentence, 16px, secondary color.
CTAs: "Start testing free" (primary) + "Read the docs" (ghost)
Trust row: "No credit card · Deploy in 5 min · Works with any framework"
Hero visual: animated mock simulation terminal (JSX component, not image)
- Shows: attack categories loading in, progress bar, severity scores
- Dark terminal window, monospace font, real data shapes
- Subtle CSS animation, nothing heavy

### 3. PROBLEM (3 columns)
"Agents fail silently" · "Logs don't replay" · "Every debug costs money"
Each: SVG icon, 18px title, one sentence body.

### 4. FEATURES (3 alternating sections)
Each: visual on one side, copy on the other.
Built as JSX components, not images. Subtle animation.

Feature 1 — Stress Testing
Copy: "Break it before users do"
Visual: animated grid of attack categories lighting up

Feature 2 — Graph Replay  
Copy: "Rewind to the exact moment it went wrong"
Visual: node graph with failure point highlighted, scrubber below

Feature 3 — Fork & Replay
Copy: "Fix once. Don't rerun everything."
Visual: branching diagram, original (failed) vs fix (passed)

### 5. CODE BLOCK
Real SDK usage:
```python
from watchllm import test

@test(
    categories=["prompt_injection", "tool_abuse", "hallucination"],
    threshold="severity < 0.3"
)
def my_agent(user_input: str):
    ...
```
Syntax highlighted. Copy button. Dark background.
Caption: "Three lines. Any framework. CI/CD ready."

### 6. METRICS ROW
"20+" attack categories
"< 5 min" to first simulation  
"100%" run coverage via graph replay
"0" cold reruns with fork & replay
Numbers in accent color. Monospace font.

### 7. PRICING
Three tiers: Free · Pro · Team
Pro card: accent border, "Most popular" label.

### 8. FOOTER
Left: logo + tagline
Center: Product · Docs · GitHub · Status  
Right: "Built for engineers who ship agents"
Bottom: copyright · Privacy · Terms

## Dashboard UX Rules
- Sidebar: 220px wide, collapsible, icons + labels
- Nav items: Dashboard, Simulations, Agents, Projects, Failures, Settings
- Active state: accent left border + accent text
- Table rows: 48px height, hover state on entire row
- Status badges: pill shape, semantic colors
    - queued: gray
    - running: amber (pulsing dot)
    - completed: green
    - failed: red
- Empty states: meaningful copy (not "No data found")
    - Simulations: "Run your first simulation to see results here."
    - Agents: "No agents registered. Install the SDK to get started."
    - Failures: "No failures yet. Run a simulation to find out."
- Simulation detail page: split view — graph on left, inspector on right
    - Graph: nodes as circles, edges as lines, failure nodes red
    - Inspector: click any node to see full input/output/metadata
    - Timeline scrubber below graph
