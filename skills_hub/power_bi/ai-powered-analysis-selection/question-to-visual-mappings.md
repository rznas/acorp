# Question → AI Visual Mappings

Concrete stakeholder questions mapped to the specific Power BI AI visual, the field-well setup, and a starter DAX measure where a measure target is needed. Only the five AI visuals plus standard charts confirmed in the Power BI visualization-types doc are used.

## Contents
- Key influencers mappings (1–7)
- Decomposition tree mappings (8–13)
- Anomaly detection mappings (14–17)
- Smart narrative mappings (18–20)
- Q&A visual mappings (21–24)

---

## Key influencers mappings

**1. "What factors influence customer churn?"**
Key influencers. Analyze = `Churned` (Yes/No, customer grain). Explain by = Contract Type, Tenure, Region, Support Ticket Count. Categorical analysis.
```dax
Support Ticket Count = COUNTROWS ( RELATEDTABLE ( SupportTickets ) )
```

**2. "Why do some customers leave low product ratings?"**
Key influencers. Analyze = `Rating`, select state **Low**. Explain by = Role in Org, Subscription Type, Theme, Company Size.

**3. "What drives customer satisfaction scores up or down?"**
Key influencers, **Continuous** analysis (many distinct score values). Analyze = `CSAT Score`. Explain by = Channel, Tenure, Product.

**4. "What influences a house/asset price to increase?"**
Key influencers, Continuous. Analyze = `Sale Price`. Explain by = Year Built, Kitchen Quality, Year Remodeled.

**5. "Which combination of attributes produces our worst-rated segment?"**
Key influencers → **Top segments** tab. Analyze = `Rating` = Low. Explain by = Tenure, Support Ticket Count, Role. Read the discovered segments (e.g., tenure > 29 months AND tickets > 4).

**6. "What factors push gross margin % down?"**
Key influencers, Continuous, on a non-additive target.
```dax
Gross Margin % = DIVIDE ( SUM ( Sales[Profit] ), SUM ( Sales[Revenue] ), 0 )
```
Analyze = `Gross Margin %` with row ID in **Expand by** to set grain. Explain by = Category, Channel, Promotion.

**7. "Does the number of support tickets affect the score a customer gives?"**
Key influencers. Aggregate the ticket ID to customer grain (count) and place in Explain by (see #1 measure). Analyze = `Rating`.

---

## Decomposition tree mappings

**8. "Why is profit down — break it down by where it's concentrated."**
Decomposition tree. Analyze = `Total Profit`. Explain by = Region, Store, Product.
```dax
Total Profit = SUM ( Sales[Profit] )
```

**9. "Where are stockouts/backorders occurring?"**
Decomposition tree. Analyze = `% on Backorder`. Explain by = Plant, Product Type, Month. Use **AI Split → High value** to auto-find the worst node.
```dax
Pct On Backorder = DIVIDE ( SUM ( Inventory[BackorderQty] ), SUM ( Inventory[OnHandQty] ), 0 )
```

**10. "Which publisher/platform/genre drives the most sales — let me explore freely?"**
Decomposition tree. Analyze = `Total Sales`. Explain by = Publisher, Platform, Game Genre (drill in any order).

**11. "Find the value that stands out, not just the biggest."**
Decomposition tree with AI split set to **Relative** mode (surfaces values disproportionate to the column rather than absolute maxima).

**12. "Show executives where revenue concentrates but lock the top two levels."**
Decomposition tree. Analyze = `Total Revenue`; **Lock** Region and Channel levels so consumers explore only below them.

**13. "Why did backorders peak in April specifically?"**
Decomposition tree cross-filtered by a bar chart on Month → select April. AI Splits recalculate and may pick a different top dimension (e.g., Product Type = Advanced Surgical).

---

## Anomaly detection mappings

**14. "When did revenue deviate from its normal pattern?"**
Anomaly detection enabled on a **line chart**: Axis = Date, Value = `Total Revenue`. Select a flagged point to read the expected range vs actual.

**15. "Was the Aug 30 revenue spike caused by a specific seller or city?"**
Anomaly detection on the revenue line chart; drag Seller and City into the anomaly **Explain by** well. Read strength % per dimension (e.g., seller "Fabrikam" 99%).

**16. "Are there unusual swings in daily inventory levels?"**
Anomaly detection on a line chart of daily inventory change over date. Increase Sensitivity if subtle deviations should surface.

**17. "Did web traffic spike unexpectedly this quarter?"**
Anomaly detection on a line chart: Axis = Date, Value = `Sessions`. Requires >=4 points and no legend/secondary values.

---

## Smart narrative mappings

**18. "Auto-generate an executive summary of this dashboard page."**
Smart narrative, page-level (select canvas → Smart narrative icon). It detects trends and computes growth (e.g., revenue and visits both up 72%).

**19. "Add a short written takeaway under this scatter chart of transactions."**
Smart narrative on the visual: right-click → **Summarize** → pin. It states which city has the highest revenue per transaction and the expected ranges.

**20. "Include a returns figure in the summary without writing DAX."**
Smart narrative **dynamic value**: type the value name in natural language, then format as currency. The narrative cross-filters with the page.

---

## Q&A visual mappings

**21. "Let store managers ask their own questions about the data."**
Q&A visual. Users type e.g. "top geo states by sum of revenue"; Power BI picks the visual (a filled map for geography).

**22. "Show sales trend by month for the electronics category" (typed by a user)."**
Q&A visual returns a line chart; appending "as a column chart" forces a column chart.

**23. "Users keep asking for 'net sales' but the column is 'Revenue' — make Q&A understand."**
Q&A visual + add the synonym "net sales" to the Revenue column (Model view → Properties → Synonyms); also fix via Q&A **Tooling**.

**24. "Build a quick standard chart from a natural-language question."**
Designer uses the Q&A visual, then the **convert icon** to turn the answer into a standard visual (e.g., a clustered column chart) that can be pinned.
