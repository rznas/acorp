---
name: running-experiments
description: Guides product managers through designing, running, and reading PostHog experiments (A/B tests) on self-hosted PostHog. Use when a PM asks "does this change move the metric", "is the result statistically significant", "how long should I run the test", "what sample size do I need", "how do I split traffic", or wants to set primary/secondary metrics, configure exposures, choose Bayesian vs frequentist stats, use CUPED variance reduction, run holdouts for long-term effects, build no-code web experiments, run experiments without PostHog feature flags, detect sample ratio mismatch, or decide whether to ship, extend, or kill a variant. Covers the full experiment lifecycle: hypothesis, variants, rollout, metrics, analysis, ending.
---

# Running experiments (A/B tests) in PostHog

This skill helps a PM spec, launch, and interpret PostHog experiments. It does not teach what an A/B test is — it tells you where things live in PostHog, how to configure them, and what decision each output supports.

Experiments are billed with feature flag requests (first 1M flag requests/month free). Each experiment is normally backed by a multivariate feature flag.

> Self-hosted note: Experiments themselves are core and run on self-hosted PostHog. AI/Max analysis and session-replay summarization (see "AI helpers") rely on hosted LLM features and may require extra config or be unavailable on self-hosted. Where docs are silent, assume availability may differ on self-hosted.

## PM question → feature map

| PM question | Where to go / what to use |
|---|---|
| Does this change move my key metric? | Set it as the **primary metric**; read the delta chart |
| Is the result real or noise? | Significance / chance-to-win / p-value in the delta chart |
| How long must I run it? | **Running time calculator** (gear icon next to time estimate) |
| What sample size do I need? | Same calculator → "Recommended sample size" |
| What % of users should see the new thing? | **Rollout %** + **variant split** (Variant rollout step) |
| Did the change break something else? | **Secondary / counter metrics** |
| Is my traffic split actually 50/50? | **Exposures panel** + **Sample ratio mismatch (SRM)** check |
| Will this still be good in 3 months? | **Holdouts** (Holdout groups tab) |
| Can I detect a smaller effect with less traffic? | Enable **CUPED** variance reduction |
| Test copy/layout without engineering? | **No-code web experiment** via the toolbar |
| We use LaunchDarkly etc., not PostHog flags | **Run experiments without feature flags** |
| Should I ship, extend, or stop? | Read result + end via **End experiment** |

## Designing an experiment (PM workflow)

1. **Write a hypothesis** that names the goal metric, the expected direction, and supporting metrics. Good example: "A tutorial video in onboarding will increase successful app interactions and reduce churn (primary)."
2. **Pick variants.** Control + 1–9 test variants. Fewer variants = faster significance. Default and recommended: equal split.
3. **Decide rollout %** (how many users enter the experiment) separate from **split** (how entered users divide across variants).
4. **Define metrics upfront** (primary = ship/kill driver; secondary = context/guardrails). Defining after launch risks biasing analysis.
5. **Calculate running time / sample size** before launch (manual mode for drafts).
6. **QA with a small rollout** (e.g. 5%) for a few days before going to full traffic — a bad full launch can poison the user pool and force a restart.
7. Save as draft → launch from the experiment detail page.

Create at **Experiments → New experiment**. The 3-step wizard: Description (name, hypothesis, flag key) → Variant rollout (variants, split, rollout %, participant type) → Analytics (inclusion criteria, metrics).

Use the launch checklist in [reference/best-practices-checklist.md](reference/best-practices-checklist.md).

## Metrics: choosing the right type

Only events **after a user's exposure** count. Four metric types:

| Type | Answers | Example |
|---|---|---|
| **Funnel** | Conversion rate through steps | exposure → purchase |
| **Mean** | Per-user count / sum / average | revenue per user, events per user |
| **Ratio** | One metric divided by another | revenue per order, items per session |
| **Retention** | Do users return within a window | return to use feature within 7 days |

Key PM gotchas:
- In experiment funnels the **first step is always the exposure event** — only add steps that happen *after* exposure.
- "**Sum**" computes the **mean of per-user totals**, not a raw grand total. For raw totals use a Trends insight outside the experiment.
- Set a **conversion window** matching your natural conversion cycle (e.g. 7 days). Without one, a 45-day-later conversion counts the same as a 5-minute one.
- **Outlier handling (Winsorization):** clamp extreme values at percentile bounds for skewed metrics like revenue. No users are dropped.
- **Shared metrics:** define company-wide metrics (conversion, revenue, churn) once and reuse across experiments.

More metric detail: [reference/metrics-and-exposures.md](reference/metrics-and-exposures.md).

## Traffic, exposures, and assignment

- **Rollout %** = portion of users included. **Split** = how included users divide across variants. Both in the Variant rollout step or **Manage distribution** on a running experiment.
- Assignment is by `distinctId` and **stable** across sessions/devices. Increasing rollout only pulls in *new* users — it never reassigns existing ones.
- **Exposure** = a user encounters the experiment. With PostHog SDKs, calling `getFeatureFlag()` auto-fires a `$feature_flag_called` event; no extra work. You can narrow exposure to a **custom event** (e.g. `viewed_checkout_page`) so only users who actually reached the change are counted.
- **Participant type:** user-level by default; if you have groups, you can run group-targeted experiments (every group member gets the same variant).
- **Anti-patterns** after launch: changing the split or adding variants reassigns users and biases analysis. Only ever *increase rollout*. Details: [reference/metrics-and-exposures.md](reference/metrics-and-exposures.md).

## Sample size & running time

Open the calculator via the gear icon next to the running-time estimate on the experiment page.

- **Manual mode** (drafts): enter metric type, baseline value, MDE, and expected daily exposures → get recommended sample size + days.
- **Automatic mode** (launched, ≥1 day & ≥100 exposures): uses real traffic to project remaining time and progress.
- **Minimum detectable effect (MDE):** smallest change you want to reliably catch. Default 30%. Lower MDE = more traffic / longer runtime. Choose based on what change is worth detecting for the business.
- "**Complete**" means you have power for your MDE — it does **not** mean you have a significant result. Still check significance.
- **Predetermine duration** to avoid the peeking problem. Or enable **sequential testing** (frequentist) for always-valid p-values when you want to monitor continuously.

## Reading results & making the decision

Each metric shows a **delta chart** (each variant vs. gray control baseline) with a 95% credible interval (Bayesian) or confidence interval (frequentist).

- **Green** = winning, **red** = losing, **no color** = not significant. An ↑/↓ arrow shows direction when significant.
- **Significant when the interval doesn't cross 0%.** Narrower interval = more precision.
- Hover/click a variant for: significance status (won/lost/not significant), measured value, exposures, **chance to win** (Bayesian) or **p-value** (frequentist), delta %, and the interval.
- **Time series view** (click an interval bar): check stability over several days. Flip-flopping = need more data or a problem.

Decision guidance:
- **Clear winner:** all primary metrics green + no negative secondary/guardrail metrics → ship. Weigh practical (not just statistical) significance.
- **Mixed:** prioritize the primary metric; weigh trade-offs (e.g. +5% conversion vs −2% AOV); segment in Product analytics / Session replay.
- **No significance:** check sample size, verify variants actually differ, consider the effect may be too small. A null result is still a learning.
- **Multiple metrics warning:** at 95% confidence, each metric has a 5% false-positive chance (≈23% across 5 metrics, ≈40% across 10). Trust a *coherent pattern* across related metrics; be skeptical of a lone significant metric, especially one unrelated to the hypothesis. If a surprise looks compelling, re-test it as a primary in a follow-up.

Deeper reading guide + Bayesian-vs-frequentist choice: [reference/statistics-methodology.md](reference/statistics-methodology.md).

## Bayesian vs frequentist (quick pick)

- **Bayesian (default):** "There's a 96% chance variant B is better." Best for intuitive ship/no-ship framing. Significant when chance-to-win > confidence level (or < 1 − level).
- **Frequentist:** p-values and confidence intervals; answers "can we be confident this isn't chance?" Supports **sequential testing** for continuous monitoring.
- Set per experiment in **Statistics** settings, or a project default in **Experiments → Settings**. Confidence levels: 90 / 95 (default) / 99%.

## CUPED variance reduction

Enable when **pre-exposure behavior predicts the metric** (revenue, usage, activity counts). CUPED adjusts metric values using each user's baseline to cut noise → narrower intervals → detect smaller effects or reach the same decision with less traffic. It does **not** change randomization or who's included. Little effect when users lack history. Has a compute cost (control via look-back window). Set project default or override per experiment in **Settings**.

## Holdouts (long-term effects)

A holdout is a randomly assigned set of users (recommend **1–10%**) excluded from experiments, used to measure long-term impact (e.g. is the new onboarding still good after 3 months). Create under **Experiments → Holdout groups**, assign to a draft experiment in **Manage distribution**. It appears as another variant in results (conversion, delta %, interval, win probability). Holdout users get a variant `holdout-{id}` from the flag. Locked once a using experiment launches.

To **freeze enrollment** but keep measuring matured long-term metrics after shipping: add a HogQL exposure filter `timestamp < 'cutoff'`, then roll out the winner — new users won't enter results. See [reference/lifecycle-and-advanced.md](reference/lifecycle-and-advanced.md).

## No-code web experiments (PM self-serve)

Best for **simple copy/layout/CSS changes on server-rendered sites**; risky for SPAs (React/Vue/Angular re-render and may overwrite changes). Currently beta — enable the feature preview and set `disable_web_experiments: false` in the snippet. Build by launching the **toolbar** on your site → Experiment tab → New experiment → select element → edit Text/CSS/HTML. Saved experiments appear in PostHog to add metrics and launch. Details: [reference/lifecycle-and-advanced.md](reference/lifecycle-and-advanced.md).

## Running without PostHog feature flags

If you use a third-party flag system, still create the experiment (you only name a flag key for attribution), set a **custom exposure event**, and ensure every exposure + metric event carries the `$feature/<flag-key>` property with the variant value. PostHog handles attribution, SRM, and results the same way. Details: [reference/lifecycle-and-advanced.md](reference/lifecycle-and-advanced.md).

## Managing lifecycle

- **End:** pick the winning variant, roll it to 100%, freeze results. The creator gets an in-app outcome notification.
- **Pause:** disables the flag (users see control), stops collection, results frozen, resumable.
- **Reset:** clears results to draft without changing what users see; data is kept but not applied until you relaunch.
- After ending: document conclusions in the description, remove losing-variant code, then delete the flag (active flags still bill even at 100%).

Full comparison table: [reference/lifecycle-and-advanced.md](reference/lifecycle-and-advanced.md).

## Quality checks before trusting results

- **Sample ratio mismatch (SRM):** auto chi-squared check after 100 exposures. Yellow warning (p < 0.001) = distribution doesn't match your split → likely a bias (flag implementation, missing targeting properties, ad-blockers, slow variant, bots). Investigate before concluding.
- **Multiple-exposure handling:** default = exclude users seen on multiple variants (cleanest); alternative = use first-seen variant.
- **Exposure dedup:** one exposure per user per variant; duplicate `$feature_flag_called` events don't inflate counts.

## AI helpers

- **PostHog AI / Max** can create experiments from natural language ("Set up an A/B test with a 70/30 split for a new checkout flow"), summarize results, interpret significance, and summarize session replays per variant ("Should I ship this?"). See `analyze-experiments-ai.mdx`.
- **MCP server** lets a coding agent create/get/update/delete experiments and fetch results from the editor (`experiment-create`, `experiment-results-get`, etc.). See `create-experiments-mcp.mdx`.
- For setup and PM value of these assistants, see the **using-posthog-ai-assistant** skill.

> Self-hosted note: PostHog AI/Max and replay summarization depend on hosted LLM features; confirm availability and configuration on your self-hosted instance before relying on them.

## Related skills

- **using-feature-flags** — the flags that power experiment variants, rollouts, and targeting
- **using-posthog-ai-assistant** — Max/MCP for creating and interpreting experiments
- **analyzing-product-analytics** — trends, funnels, and cohorts for deeper segmentation of results
- **using-session-replay** — watch how each variant behaved
