# Lifecycle & advanced setups reference

- [Launch checklist](#launch-checklist)
- [Lifecycle: end / pause / reset comparison](#lifecycle-end--pause--reset-comparison)
- [Holdouts](#holdouts)
- [Freezing enrollment for long-term metrics](#freezing-enrollment-for-long-term-metrics)
- [No-code web experiments](#no-code-web-experiments)
- [Running experiments without PostHog feature flags](#running-experiments-without-posthog-feature-flags)

## Launch checklist
Before launch: clear goal metric; clear hypothesis; secondary metrics; counter (guardrail) metrics; predefined duration based on sample size; code only includes eligible/affected users (check eligibility *before* the flag); QA on control and test variants.
24–48h after launch: variant volumes as expected; logging correct and in correct ratios; no increase in crashes/errors.
End of duration: validate/invalidate hypothesis; document and share results; end experiment, ship winner, delete losing-variant code.

Only-include-affected-users rule: filter ineligible users *before* checking the flag, so unaffected users never get counted (e.g. return early if the user already completed the action, then check the flag).

## Lifecycle: end / pause / reset comparison

| Status | Feature flag | Users see | Exposure tracking | Results | Can resume |
|---|---|---|---|---|---|
| Running | Enabled | Multiple variants | Continues | Updating | N/A |
| Pausing | Disabled | Control variant | Stops | Fixed | Yes |
| Ending | Rolls chosen variant to all | Chosen variant | Continues (chosen only) | Fixed | No |
| Resetting | Unchanged | Multiple variants | Continues | Cleared | Yes (relaunch) |

- **End:** pick winner → roll to 100% → results frozen. Creator gets in-app outcome notification (significant or inconclusive), unless the ender is the creator. If already shipped to 100%, ending just marks complete without changing the flag.
- **Pause:** disables the flag (users revert to control), halts collection; **Resume** continues from where it left off.
- **Reset:** clears results to draft without changing what users see; data kept but not applied unless you change the start date after relaunching. Use after fixing setup bugs or changing metric definitions.
- After ending: document conclusions in the description; remove losing-variant code; delete the flag once the winner is hardcoded (active flags bill even at 100%, and flags linked to non-running experiments can be deleted without losing historical data).

## Holdouts
Randomly assigned users excluded from experiments, for measuring long-term effects (e.g. is the new onboarding still better after 3 months). Recommend **1–10%** (large enough for significance, small enough to not drain the experiment pool).
1. **Experiments → Holdout groups → New holdout**; name, description, percentage.
2. Assign to a **draft** experiment via **Manage distribution** on the Variants tab. Locked once any using experiment launches.
3. Appears as another variant in results (conversion, delta %, credible interval, win probability).
- Holdout users get flag variant `holdout-{holdout_id}` (id is in the holdout URL). Holdout conditions evaluate **before** regular flag conditions. In code, treat any variant starting with `holdout-` as the default experience.

## Freezing enrollment for long-term metrics
When you've hit sample size but metrics (e.g. 90-day revenue, long-term retention) need months to mature, freeze the participant set and ship the winner immediately:
1. Note the cutoff timestamp ("now").
2. **Edit exposure criteria** → if using default, switch to Custom event `$feature_flag_called` → add a **HogQL** property filter: `timestamp < '2026-03-23 15:44:00'` (your cutoff).
3. Roll out the winning variant. New users get it but won't enter results (their exposures are after the cutoff). Existing groups stay frozen and keep maturing.

## No-code web experiments
Beta. Good for simple copy/layout/CSS edits on server-rendered sites (e.g. Next.js); risky for SPAs (React/Vue/Angular re-render and may overwrite changes), and for complex multi-element changes.
Setup:
1. Enable the **No code web experiments** feature preview and set `disable_web_experiments: false` in the PostHog snippet config (otherwise toolbar experiments won't run live).
2. Go to the **toolbar** tab, add your site as an authorized domain, **Launch** the toolbar on your site.
3. Toolbar → **Experiment** tab → **New experiment** → name + variants → select variant to modify → **Add element** → edit **Text** (text only), **CSS** (styling), or **HTML** (replaces whole element). Use a shared CSS class selector (e.g. `.sign-up-button`) to change an element across many pages. `<script>`, `<iframe>`, `<object>`, `<embed>`, `javascript:`, and `data:text/html` are blocked.
4. **Save experiment** → it appears in the Experiments tab → add metrics, configure exposure criteria, preview variants (Variants tab), launch.
Tips: avoid hydration-error confusion by evaluating the flag on a prior page or via server-side bootstrapping to remove the visible change delay.

## Running experiments without PostHog feature flags
For teams using a third-party flag library who still want PostHog analysis:
1. Create the experiment normally — you still name a flag key (used only to attribute events; `control` name is fixed, other variant names are free).
2. Your own system assigns variants.
3. Set a **custom exposure event** (e.g. `$pageview`, `button_clicked`) in exposure criteria instead of `$feature_flag_called`.
4. Add `$feature/<flag-key>` = variant to **both** the exposure event and **every metric event** (PostHog SDKs do this automatically for native flags; here you do it manually).
PostHog then records first exposure per variant, counts only post-exposure metric events, handles multiple-exposure rules, and runs SRM the same way. Common issues: events missing the `$feature/<flag-key>` property; variant/flag-key name mismatches (case-sensitive); metric events firing before exposure; inconsistent per-user variant assignment.
