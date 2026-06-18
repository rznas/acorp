# Statistics methodology reference

- [Bayesian vs frequentist: when to use which](#bayesian-vs-frequentist-when-to-use-which)
- [Data quality validation gates](#data-quality-validation-gates)
- [How significance is decided](#how-significance-is-decided)
- [Confidence levels](#confidence-levels)
- [Sequential testing (frequentist, beta)](#sequential-testing-frequentist-beta)
- [Sample size formula](#sample-size-formula)
- [CUPED variance reduction](#cuped-variance-reduction)
- [Multiple variants](#multiple-variants)
- [Multiple metrics & false positives](#multiple-metrics--false-positives)
- [Sample ratio mismatch (SRM)](#sample-ratio-mismatch-srm)

PostHog supports both engines on the same three metric families (funnel, mean, ratio). Both run a 5-step pipeline: aggregate into sufficient statistics → validate data quality → calculate effect size + variance → test → generate results.

## Bayesian vs frequentist: when to use which
- **Bayesian (default):** reports **chance to win** ("96% chance variant B increases conversion") and a 95% **credible interval** ("95% probability the true effect lies in this range"). Uses non-informative priors (mean 0, large variance) so the data drives the result. Most intuitive for ship/no-ship decisions.
- **Frequentist:** reports a **p-value** and a 95% **confidence interval** ("if repeated many times, 95% of intervals contain the true effect"). Uses Welch's two-sided t-test (handles unequal variances). Choose it for classic NHST reporting and when you want **sequential testing**.

Both default to relative differences (e.g. control 10% → treatment 12% shows as +20%).

## Data quality validation gates
Analysis is blocked (with an error, not a misleading result) unless:
- **All metrics:** ≥ 50 exposures per variant.
- **Funnels:** ≥ 5 conversions per variant, and normal-approximation validity `n·p > 5` and `n·(1−p) > 5`.
- **Mean/ratio:** control mean must be non-zero (needed for relative-difference math).

## How significance is decided
- **Bayesian:** significant ("decisive") when `chance_to_win > confidence_level` or `< (1 − confidence_level)`. At 95%, that's chance-to-win above 95% or below 5%.
- **Frequentist:** significant when `p_value < alpha` (alpha = 0.05 at 95%).
- **Visually (both):** the credible/confidence interval **does not cross 0%**. Green = win, red = loss, no color = not significant.

## Confidence levels
90% / 95% (default) / 99%, set per experiment in **Statistics** settings or as a project default in **Experiments → Settings**. Lower (90%) shows significance sooner but more false positives; higher (99%) needs stronger evidence and more data.

## Sequential testing (frequentist, beta)
Solves the "peeking problem" — checking dashboards repeatedly inflates false positives. When enabled, PostHog replaces the fixed-horizon t-test with always-valid p-values / confidence sequences (Waudby-Smith et al. 2023) that stay bounded by alpha no matter how often you look. Trade-off: slightly wider intervals; tightest near the **tuning parameter** sample size (default 5,000 — set near your expected total sample size). Enable per experiment (Statistics settings) or as a project default. Use when monitoring continuously; if you strictly wait for the predetermined duration, the fixed-horizon test is more efficient.

## Sample size formula
`N = (16 × variance) / d²` per variant, then × number of variants.
- 16 ≈ 80% power at 95% confidence.
- d = absolute effect size = MDE × baseline value.
- Variance by type: funnel `p(1−p)`; count `2 × baseline`; sum/avg `0.25 × baseline²`.

Example: 10% conversion (p=0.1), 20% MDE → variance 0.09, d = 0.02, per-variant N = (16×0.09)/0.02² = 3,600; two variants → 7,200 total exposures.

## CUPED variance reduction
Optional. Adjusts each user's metric value using pre-exposure behavior to cut noise. Does not change randomization, inclusion, or which post-exposure events count. Strongest when pre-exposure behavior correlates with the metric (revenue, usage volume, activity). Narrows intervals → detect smaller effects or same decision with less traffic. Weak/no effect when users lack history. Compute cost scales with the look-back window.

## Multiple variants
Each variant is compared to control independently; chance-to-win / p-value computed per comparison. **No multiple-comparison correction** is applied — each comparison stands alone.

## Multiple metrics & false positives
At 95% confidence each metric has a 5% false-positive chance: ≈23% across 5 metrics, ≈40% across 10. PostHog does not adjust for this. Practical guidance:
- Tie every metric to the hypothesis with clear success criteria before launch; designate one primary decision-driver.
- Read the **full pattern**: a coherent story across related metrics is strong evidence; a lone significant metric (especially unrelated to the hypothesis) deserves skepticism.
- If a surprising metric lights up, don't ship on it — re-run with that metric as primary; ship only if it replicates.

## Sample ratio mismatch (SRM)
Auto chi-squared check comparing observed distribution vs configured split, once the experiment has ≥ 100 total exposures. Flagged at p < 0.001 (yellow warning); green = within normal variation. SRM means something is biasing assignment, which can invalidate results. Investigate: flag implementation across all code paths (call `identify()` before evaluating flags on frontends), missing targeting properties (prefer simple % rollout), ad-blockers/network drops (consider a reverse proxy), slower/crashing variant causing pre-exposure drop-off, bot traffic, race conditions, test-account filtering.
