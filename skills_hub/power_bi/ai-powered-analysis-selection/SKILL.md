---
name: ai-powered-analysis-selection
description: Selects and configures Power BI AI visuals (Key influencers, Decomposition tree, Anomaly detection, Smart narrative, Q&A) for exploratory analysis and automated insight discovery. Use when stakeholders ask "why" or "what drives" questions, need root-cause analysis, want to find anomalies or spikes in time series, need auto-generated text summaries, or want ad-hoc natural-language querying of a semantic model. Triggers include "why did X change", "what factors influence X", "what's driving X", "find unusual patterns", "summarize this page", "let users ask questions", and discovering insights they don't know to ask for.
---

# AI-Powered Analysis Selection

Power BI ships five AI-powered visuals for exploratory analysis. Standard charts (column, line, pie) answer questions you already know to ask. AI visuals answer **"why"** questions and surface insights the stakeholder did not know to ask. Use this skill to pick the right AI visual and prepare the model so it actually works.

Prerequisite: this skill assumes the dimensional model is understood (grain, facts, dimensions, relationships). If the model is unclear, apply `visualizing-dimensional-models` first.

## The five AI visuals at a glance

| AI visual | Answers | Core inputs | Requires time series? |
|-----------|---------|-------------|----------------------|
| **Key influencers** | "What factors influence [metric]? Which matters most?" | Analyze (target) + Explain by (factors) | No |
| **Decomposition tree** | "Why is [metric] high/low? Where is it concentrated?" | Analyze (measure) + Explain by (dimensions) | No |
| **Anomaly detection** | "When did [metric] deviate from normal?" | Enabled on a line chart with a date axis | Yes |
| **Smart narrative** | "Summarize this page/visual in words." | Existing visuals on the page | No |
| **Q&A visual** | "Let users ask anything in natural language." | Well-modeled semantic model | No |

Pick the visual with the decision tree in [ai-visual-decision-tree.md](ai-visual-decision-tree.md). Match it to a concrete stakeholder question with the 20+ mappings in [question-to-visual-mappings.md](question-to-visual-mappings.md).

## When to use which (quick guidance)

- **Ranked causes, with statistical weighting** → Key influencers. It runs logistic/linear regression and only reports factors that pass a Wald test (p < 0.05), so it filters out chance findings on small groups.
- **Free-form drill to locate concentration** → Decomposition tree. Use AI Splits (the light-bulb "High value" / "Low value") to let it choose the next dimension that maximizes/minimizes the measure.
- **Spikes and dips over time, with explanations** → Anomaly detection on a line chart. It flags points outside the expected range and explains them across model fields.
- **Auto-written takeaways that refresh with filters** → Smart narrative.
- **Self-service exploration by business users** → Q&A visual.

Key influencers and Decomposition tree are complementary: Key influencers ranks *which factors matter* across the whole population; Decomposition tree lets you *navigate a specific path* (Region → Store → Product) to see where a number concentrates. Offer both for deep "why" investigations.

## Data preparation checklist (do this before placing any AI visual)

AI visuals are only as good as the model. Copy this checklist and verify each item:

```
AI-Visual Readiness:
- [ ] Relationships exist between every fact and dimension used (no isolated tables)
- [ ] Explanatory fields are at the SAME grain as the Analyze target (1:1 or many:1)
- [ ] Tables and columns are named for their content (Customers, not CustomerSummary)
- [ ] Date and number columns have correct data types (not strings)
- [ ] Year / Month / Day / ID columns set to "Don't summarize"
- [ ] Date and geography columns have a Data Category assigned
- [ ] Target metric for Key influencers has enough data (>=100 of the analyzed state, >=10 of comparison state)
- [ ] Target metric for Key influencers has <=10 unique categorical values (or switch to Continuous)
- [ ] For Q&A / Smart narrative: synonyms added for common business terms
```

### Why each item matters

- **Relationships**: Without them, Power BI can't join tables, so Key influencers, Decomposition tree, and Q&A produce nothing or errors. This is the single most common failure.
- **Same grain for Explain by**: Key influencers analyzes at the table level of the target. A factor at a finer grain (e.g., device, when a customer has many devices) raises *"a field in Explain by isn't uniquely related"*. Fix by aggregating to a count, or pivoting the values into columns.
- **>10 unique target values**: More categories means fewer observations each, weakening pattern detection. Group into bands (High/Medium/Low) or switch the visual's Analysis card to **Continuous**.
- **Data Category**: Drives Q&A visual-type selection (geography → map, date → line-chart axis) and language biases ("when" maps to a date column).

For Q&A specifically, follow the full optimization workflow in [q-and-a-optimization.md](q-and-a-optimization.md).

## Configuration essentials per visual

### Key influencers
- **Analyze** = one target (the metric you want to explain). **Explain by** = candidate factors.
- Categorical target (Churned Yes/No, Rating Low) → asks "what influences it to be [value]?". Numeric target → choose **Categorical** vs **Continuous** Analysis Type on the Format > Analysis card; Continuous asks "what influences it to increase/decrease?".
- **Expand by** is only for a measure/summarized-column target — it sets the analysis grain without adding the field as an influencer (e.g., add the row ID to Expand by).
- Use the **Top segments** tab to find combinations of factors (a decision-tree-derived subgroup), not just single factors.
- Enable **Counts** (Analysis card) so a small-but-strong influencer isn't over-prioritized — a thin ring means little data backs it.

### Decomposition tree
- **Analyze** must be a measure or aggregate; **Explain by** holds the drill dimensions.
- The user (or you) picks each split with **+**; the order is free. **AI Splits** (light bulb) auto-pick the next dimension for highest/lowest value.
- Set AI split mode to **Relative** (Analysis formatting) to surface values that *stand out* vs the column, rather than just the largest **Absolute** value.
- **Lock** the first N levels so report consumers explore freely below a fixed top path. You can't have unlocked levels before locked ones.
- Limits: max 50 levels, top 10 per level, 5,000 data points at once.

### Anomaly detection
- Turn on via **Analytics** tab > **Anomalies** on a **line chart** that has a date/time field on the Axis.
- Select a flagged point to get a natural-language explanation; drop fields into the anomaly **Explain by** well to constrain which dimensions explain it. Strength = ratio of the filtered deviation to total deviation.
- Tune **Sensitivity** (higher = flags smaller deviations).
- Strict requirements: line chart only, single value, time axis, >=4 data points; no legend/multiple/secondary values; drill-down not supported.

### Smart narrative
- Add for a page (select canvas → Smart narrative icon) or a single visual (right-click → **Summarize**). It auto-detects trends and computes values (e.g., 72% growth).
- Add **dynamic values** by typing a natural-language measure (no DAX needed) and format them (currency, decimals).
- It refreshes and cross-filters with the page. Generates up to 4 summaries per visual, 16 per page.
- Cannot summarize tables, matrices, R/Python/custom visuals — pair it with charts.

### Q&A visual
- Place the Q&A visual; users type questions and get an auto-chosen chart. They can append a chart type ("...as a column chart") to force the visual.
- Use the **gear / Tooling** pane to teach unrecognized terms, manage suggestions, and review questions users asked.
- Designers can convert a Q&A result into a standard visual via the convert icon.
- Q&A interprets best on a normalized, well-named model with synonyms — see [q-and-a-optimization.md](q-and-a-optimization.md).

## Validating an AI-visual recommendation

Before finalizing, run this loop:

1. Confirm the question is genuinely a "why / what-drives / when-unusual / explore" question — not a known comparison or trend (those use standard charts; see `mapping-questions-to-charts`).
2. Pick the visual via [ai-visual-decision-tree.md](ai-visual-decision-tree.md).
3. Run the data-preparation checklist above. If any item fails, fix the model first.
4. Configure inputs per the section above.
5. If the visual errors or returns nothing, diagnose with the troubleshooting table in [ai-visual-decision-tree.md](ai-visual-decision-tree.md), fix, and re-check.

## Reference DAX for AI-visual targets

AI visuals analyze measures and aggregates. Use additivity-correct measures (apply `understanding-fact-additivity` for semi-additive/non-additive cases) as Key-influencers or Decomposition-tree targets:

```dax
-- Additive target for a decomposition tree (drill Region -> Store -> Product)
Total Revenue = SUM ( Sales[Revenue] )

-- Non-additive ratio target for Key influencers "what influences margin %"
Gross Margin % = DIVIDE ( SUM ( Sales[Profit] ), SUM ( Sales[Revenue] ), 0 )

-- Churn flag as a categorical Key-influencers target (Analyze = this column)
-- (calculated column on the customer table, one row per customer)
Churned = IF ( Customer[Status] = "Cancelled", "Yes", "No" )

-- Support-ticket count aggregated to customer grain so it is a valid Explain-by factor
Support Ticket Count = COUNTROWS ( RELATEDTABLE ( SupportTickets ) )
```

## Related skills

- `visualizing-dimensional-models` — prerequisite; grain and relationships drive AI-visual readiness.
- `mapping-questions-to-charts` — classify whether a question even needs an AI visual.
- `understanding-fact-additivity` — build correct measure targets for Key influencers / Decomposition tree.
- `dashboard-composition` — where Smart narrative and AI visuals fit on an analytical dashboard.

---
*Provenance: Implements SKILL 8 (AI-Powered Analysis Selection), Layer 3, of `power_bi_user/planning/00-POWERBI-VISUALIZATION-SKILLS-MASTER-PLAN.md`.*
