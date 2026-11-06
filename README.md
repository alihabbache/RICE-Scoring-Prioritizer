# RICE Scoring Prioritizer

A single-page tool that ranks product features using the **RICE** framework, so prioritisation is driven by numbers instead of opinions.

**Live demo:** https://alihabbache.github.io/RICE-Scoring-Prioritizer/

> **Status: proof of concept.** The "subagents" are a visual metaphor. The page animates three agents and logs their activity, but all scoring is plain arithmetic running locally in your browser. It does not query analytics, call an LLM, or read any real market data. See [How it works](#how-it-works) and [Limitations](#limitations).

## What it does

1. You add one or more features, each with a few inputs.
2. You click **Run RICE Analysis**.
3. The page walks through three "subagents" and an orchestrator, then shows a ranked backlog with each feature's RICE score and its score relative to the top item.

```
RICE = (Reach × Impact × Confidence%) ÷ Effort
```

A higher score means a higher priority.

## Using it

Fill in the **RICE Orchestrator** form, click **+ Add to Queue**, repeat for each feature, then click **Run RICE Analysis**. Use **Reset All** to start over.

| Field | Required | Description |
|---|---|---|
| Feature Name | Yes | Label shown in the results |
| Description | No | Optional context, shown (truncated) under the name in the results |
| Monthly Active Users | Yes | Total active users |
| % Users Affected | Yes | Share of users the feature touches (0–100) |
| Market Impact Level | Yes | Massive (3×), High (2×), Medium (1×), Low (0.5×), Minimal (0.25×) |
| Confidence | Yes | High (100%), Medium (80%), Low (50%), Very Low (30%) |
| Engineering Complexity | Yes | Trivial, Small, Medium, Large, Epic |
| Cross-team Dependencies | Yes | None, 1 team, 2–3 teams, 4+ teams |

## How it works

| Step | Component | What it actually does |
|---|---|---|
| ① | Orchestrator | Collects the feature queue |
| ② | Subagent A, Reach Estimator | **Reach** = `MAU × % affected ÷ 100`, rounded |
| ② | Subagent B, Impact Synthesizer | Takes **Impact** and **Confidence** from your dropdown choices |
| ② | Subagent C, Effort Sizer | **Effort** (person-months) = complexity base × dependency multiplier |
| ③ | Orchestrator | Applies the RICE formula to every feature and sorts by score |
| ④ | Results | Ranked table with Reach, Impact, Confidence, Effort, Score and a bar showing score vs. the top item |

### Effort model

Effort is estimated from two lookups:

| Complexity | Base (person-months) |
|---|---|
| Trivial | 0.25 |
| Small | 0.5 |
| Medium | 1.5 |
| Large | 4 |
| Epic | 10 |

| Cross-team dependencies | Multiplier |
|---|---|
| None | ×1.00 |
| 1 team | ×1.15 |
| 2–3 teams | ×1.35 |
| 4+ teams | ×1.60 |

**Example:** 50,000 MAU, 35% affected, High impact (2×), Medium confidence (80%), Medium complexity (1.5) with 1 team dependency (×1.15):

- Reach = 17,500
- Effort = 1.5 × 1.15 = 1.725, rounded to 1.72 person-months
- RICE = (17,500 × 2 × 0.8) ÷ 1.72 ≈ **16,279**

## Run locally

No build step, dependencies, or server needed.

```bash
git clone https://github.com/alihabbache/RICE-Scoring-Prioritizer.git
cd RICE-Scoring-Prioritizer
open index.html        # macOS; or just double-click the file
```

## Project structure

```
.
├── index.html   # the whole app: markup, styles and scoring logic
└── README.md
```

## Privacy and security

- Runs entirely client-side. There are no network requests, accounts, cookies, or analytics.
- Nothing is persisted. Features live in memory and are lost on refresh.
- User input is HTML-escaped before it is rendered into the results table.
- To report a security issue, use [GitHub private vulnerability reporting](https://github.com/alihabbache/RICE-Scoring-Prioritizer/security/advisories/new) rather than a public issue.

## Limitations

- **Simulated agents.** The delays and log lines are cosmetic. Results depend only on the values you enter.
- **Subjective inputs.** The score is only as good as your Reach, Impact and Confidence estimates. The dropdowns use fixed scales, and the effort lookup values are rough heuristics, not calibrated data.
- **No persistence or export.** There is no save, import, or CSV export yet.
- **Reach is a snapshot.** It is not time-bound (RICE conventionally measures reach per time period, such as per quarter).

## Ideas for next steps

- Export results to CSV or Markdown
- Save and restore the queue (e.g. `localStorage`)
- Allow editing queued features
- Wire the subagents to real data sources (analytics, research, engineering estimates)

## About RICE

RICE is a prioritisation framework popularised by Intercom: **R**each, **I**mpact, **C**onfidence, **E**ffort. It gives a comparable score across very different ideas, which makes trade-offs easier to discuss with stakeholders.
