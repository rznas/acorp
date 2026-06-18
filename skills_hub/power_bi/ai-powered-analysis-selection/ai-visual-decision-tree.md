# AI Visual Selection Decision Tree

## Contents
- Decision tree (text form)
- Disambiguating overlapping cases
- Per-visual fit and limits summary
- Troubleshooting table (errors → fixes)

## Decision tree (text form)

Start from the stakeholder's intent.

```
Is the question about discovering/explaining, not just reporting a known number?
│
├─ NO  → Use a standard visual (see mapping-questions-to-charts). Stop.
│
└─ YES
   │
   ├─ Does the question name a TIME deviation?
   │  ("when did it spike/drop", "unusual pattern over time", "off from normal")
   │  → ANOMALY DETECTION on a line chart (date axis, >=4 points).
   │
   ├─ Does the question ask WHICH FACTORS matter / their relative importance?
   │  ("what influences X", "what drives X", "why are some X high vs low",
   │   "which factor matters most")
   │  → KEY INFLUENCERS  (Analyze = target, Explain by = factors)
   │     └─ Need combinations of factors, not single ones? → its Top segments tab.
   │
   ├─ Does the question ask WHERE a number concentrates / free-form drill to a cause?
   │  ("why is profit down — break it down", "where are stockouts",
   │   "drill into the biggest contributor")
   │  → DECOMPOSITION TREE  (Analyze = measure, Explain by = dimensions)
   │     └─ Want it to auto-pick the next dimension? → AI Splits (High/Low value).
   │
   ├─ Does the consumer want a WRITTEN summary that updates with filters?
   │  ("summarize this page", "executive narrative", "explain these charts")
   │  → SMART NARRATIVE  (page-level or right-click a visual → Summarize)
   │
   └─ Should END USERS ask their own ad-hoc questions in natural language?
      ("let users explore", "self-service Q&A", "type any question")
      → Q&A VISUAL  (requires a well-modeled, well-named semantic model)
```

## Disambiguating overlapping cases

These pairs are the most common confusions:

- **Key influencers vs Decomposition tree.** Both answer "why". Choose **Key influencers** when you want a *ranked, statistically weighted* list of factors across the whole population ("what matters most"). Choose **Decomposition tree** when you want to *navigate a chosen hierarchy* and see where a measure concentrates ("show me Region → Store → Product and where it's worst"). For a thorough investigation, offer both: influencers names the factors, the tree lets the user walk a path.

- **Anomaly detection vs Key influencers for "why did it change".** If the change is a *point in time* on a trend ("why did revenue spike on Aug 30"), use **Anomaly detection** — it pins the date, shows the expected range, and explains it. If the question is about *sustained drivers* ("what generally drives revenue down"), use **Key influencers** or a **Decomposition tree**.

- **Smart narrative vs Q&A.** **Smart narrative** *writes* insights for a fixed page (push). **Q&A** lets the user *ask* (pull). Use narrative for executive read-outs; use Q&A for analyst exploration.

- **Key influencers numeric target.** If the target is numeric with many distinct values, do not ask "what influences price to be 156,214". Switch the Analysis card to **Continuous** ("what influences price to increase/decrease"). Keep **Categorical** only for low-cardinality targets (survey 1–10, Rating Low/Med/High).

## Per-visual fit and limits summary

| Visual | Best for | Hard requirements | Notable limits |
|--------|----------|-------------------|----------------|
| Key influencers | Ranked drivers of one metric; segment discovery | Explain-by factors at target's grain; >=100 obs of analyzed state, >=10 of comparison | No DirectQuery; no Live Connection to AAS/SSAS; runs on 10,000-row sample; categorical target unsupported when implicit measures discouraged (calc groups) |
| Decomposition tree | Free-form drill, root-cause concentration, AI-chosen splits | Analyze must be a measure/aggregate | 50 levels max, top 10 per level, 5,000 points; AI splits not on Azure AS / Report Server / Publish-to-Web |
| Anomaly detection | Spikes/dips in one time series + explanations | Line chart, single value, date/time axis, >=4 points | No legend/multiple/secondary values; no drill-down; no DirectQuery over SAP, no Live Connection to AAS/SSAS |
| Smart narrative | Auto text summaries that cross-filter | At least one summarizable visual on page | Can't summarize tables/matrices/R/Python/custom; no Publish-to-Web; no dashboard pin; 4 per visual / 16 per page |
| Q&A visual | Self-service natural-language querying | Well-named, normalized model with relationships | Depends entirely on model quality; ambiguous synonyms reduce accuracy |

## Troubleshooting table (errors → fixes)

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Key influencers: "no influencers or segments found" | Target also placed in Explain by; or factors too sparse; or no meaningful correlation | Remove target from Explain by; group sparse categories; switch numeric target to Continuous |
| Key influencers: "not enough data to run the analysis" | Too few observations for the analyzed state | Get >=100 of the analyzed state and >=10 of the comparison state, or broaden the filter |
| Key influencers: "field in Explain by isn't uniquely related" | Explanatory field is at a finer grain than the target table (1:many) | Aggregate it to a count, or pivot the values into per-value columns; recreate relationships |
| Key influencers: "measures weren't included" warning | Target table has no unique identifier | Add an index/primary-key column (Power Query) to the target table |
| Key influencers: ">10 unique values" warning | Categorical target has too many categories | Bin into High/Medium/Low, or switch Analysis card to Continuous |
| Decomposition tree shows nothing | Analyze is not a measure/aggregate, or no Explain-by fields | Use a measure in Analyze; add dimensions to Explain by |
| AI split light bulb missing / disabled | Unsupported source (Azure AS, Report Server, Publish-to-Web) or complex measure in Analyze | Use a supported source; simplify the Analyze measure |
| Anomaly detection greyed out / errors | Not a line chart, no date axis, legend/multiple values present, or <4 points | Use a single-value line chart with a date axis and >=4 points; remove legend/secondary values |
| Smart narrative empty (hidden-summary symbol) | Current filters yield a flat/no-result series | Expected; summary repopulates when data/filters change |
| Q&A underlines terms in red (unrecognized) | Model term differs from user vocabulary | Add synonyms; rename tables/columns to content; teach the term in Q&A Tooling |
| Q&A "total sales by year" returns a grand total of years | Year/ID column is being summed | Set Year/Month/Day/ID Summarization to "Don't summarize" |
| Q&A picks wrong chart for geography/date | Missing Data Category | Set Data Category (City/Country, etc.) and correct date data types |
