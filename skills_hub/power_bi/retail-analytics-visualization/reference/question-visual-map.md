# Retail Question -> Power BI Visual Map

Each entry: business question -> source fact/grain/additivity -> recommended Power BI visual (confirmed standard visual) -> measure to use (see dax-measure-library.md). Ratios use DIVIDE(SUM,SUM); balances use period-end/average.

## Contents
- Product performance (Q1-Q5)
- Store performance (Q6-Q9)
- Promotion effectiveness (Q10-Q14)
- Customer behavior (Q15-Q19)
- Time-based trends (Q20-Q23)
- Inventory (Q24-Q28)
- E-commerce / clickstream (Q29-Q33)
- Profitability (Q34-Q37)
- Notes on Power BI visual constraints

---

## Product performance

**Q1. What are the top 10 products by revenue?**
POS sales (transaction; revenue additive). Visual: **Bar chart** (horizontal, SKU on axis, Top N filter, descending). Measure: `Total Revenue`. Enhancement: conditional data bars; small multiples by Store via the visual's small-multiples field.

**Q2. How do product categories contribute to total sales?**
POS sales (additive). Visual: **Treemap** (Department -> Category -> Brand hierarchy, size = revenue). Alternative: **Donut chart** if <10 categories. Measure: `Total Revenue`.

**Q3. What is the product sales lifecycle (introduction -> growth -> maturity -> decline)?**
POS sales aggregated by month + Product + Date. Visual: **Line chart** (one line per product) or a Line chart with the Product field in small multiples. Measure: `Total Revenue`.

**Q4. Which brands rank highest each quarter, and how does rank shift?**
POS sales (additive). Visual: **Ribbon chart** (Date quarter on axis, Brand legend) - shows rank change over time. Measure: `Total Revenue`.

**Q5. How do products distribute by price tier vs units sold?**
POS sales + Product price-tier attribute. Visual: **Scatter chart** (X = current standard price, Y = units, play axis = month) or **Column chart** with price-tier bins on the axis. Measures: `Total Units`, plus Product[Price Tier].

## Store performance

**Q6. Which stores have the highest sales?**
POS sales (additive) + Store geography. Visual: **Filled map (choropleth)** shaded by revenue. Alternative when geography is irrelevant: **Column chart**. Measure: `Total Revenue`.

**Q7. How do stores compare across regions on multiple metrics?**
POS sales + Store organizational hierarchy. Visual: **Matrix** (rows = Region/District/Store, columns = Revenue, Gross Margin %, Transactions) with conditional formatting (data bars / color scales). Measures: `Total Revenue`, `Gross Margin %`, `Transaction Count`.

**Q8. Are sales-per-square-foot meeting targets by store?**
POS sales (additive) + Store[Selling Square Footage] attribute. Visual: **Bar chart** with a target reference line. Measures: `Total Selling Sq Ft` = `SUMX(VALUES('Store'[Store Key]), CALCULATE(MAX('Store'[Selling Sq Ft])))` then `Sales per Sq Ft` = `DIVIDE([Total Revenue], [Total Selling Sq Ft], 0)`. Note: never `SUM('Store'[Selling Sq Ft])` directly - square footage lives on the Store *dimension*, so a raw SUM adds up rows of the filtered Store table rather than the distinct stores actually in context (and ignores how the active relationship filters Store). `SUMX(VALUES(Store Key), MAX(...))` aggregates the attribute once per distinct store in the current filter context, which is the safe pattern.

**Q9. How does same-store (comparable) sales growth compare across stores?**
POS sales (additive) restricted to stores open in both periods. Visual: **Column chart** or **Matrix**. Measure: `Same-Store Sales %` (see dax-measure-library.md).

## Promotion effectiveness

**Q10. Which promotions drove the most incremental sales (lift)?**
POS sales + Promotion (causal). Visual: **Column chart** comparing promoted vs baseline sales. Alternative: **Waterfall chart** showing incremental contribution. Measure: `Promotion Lift %`.

**Q11. What is the relationship between discount depth and sales lift?**
POS sales + Promotion[Price Reduction Percent]. Visual: **Scatter chart** (X = discount %, Y = lift %, bubble size = units). Measures: `Promotion Lift %`, `Total Units`.

**Q12. Which promotional mechanism combinations work best?**
POS sales + Promotion (price/ad/display/coupon attributes). Visual: **Matrix** (rows = mechanism combination, value = avg units per transaction) or **Column chart**. Measure: `Avg Units per Transaction` filtered to Promotion <> "No Promotion".

**Q13. Did the promotion just shift sales across time (pre/during/post)?**
POS sales + Date windows. Visual: **Column chart** with a Pre / During / Post category on the axis. Measure: `Total Units` sliced by period; compare to baseline.

**Q14. Did a brand promotion cannibalize other brands in the category?**
POS sales + Product[Category]. Visual: **Bar chart** of unit change by brand within the category, or **Waterfall chart** (category net change). Measure: `Units Change vs Prior Period`.

## Customer behavior

**Q15. What is the distribution of customer purchase frequency?**
POS sales aggregated to purchases per customer (additive). Visual: **Column chart** with purchase-count bins on the axis (binning groups customers). Measure: `Distinct Customers`.

**Q16. Which customers are high recency/frequency/monetary (RFM)?**
POS sales + Customer. Visual: **Scatter chart** (X = recency days, Y = frequency, bubble size = monetary value). Measures: `Days Since Last Purchase`, `Purchase Count`, `Customer Revenue`.

**Q17. Which customer segments are most profitable?**
Profitability fact + Customer segment. Visual: **Bar chart** or **Treemap** by segment. Measure: `Net Profit`.

**Q18. What is the average basket (transaction) size?**
POS sales (additive). Visual: **Card** for the headline, **Line chart** for trend. Measure: `Average Basket Size` (see dax-measure-library.md).

**Q19. Which customer segments drive the most transactions?**
POS sales + Customer segment. Visual: **Clustered column chart**. Measure: `Transaction Count`.

## Time-based trends

**Q20. How have sales trended over the past 12 months?**
POS sales (additive) + Date. Visual: **Line chart** (Date on X) with a target reference line. Measure: `Total Revenue`. Tip: add the Line chart's built-in analytics line; use a Key influencers chart separately for "why" questions.

**Q21. What is the day-of-week / month-of-year seasonality pattern?**
POS sales + Date text attributes (Day of Week Name, Month Name). Visual: **Column chart** by Day of Week, or a Line chart with Day-of-Week in small multiples. Measure: `Total Revenue`.

**Q22. How do holiday vs non-holiday days compare?**
POS sales + Date[Holiday Indicator] (text). Visual: **Clustered column chart** (Holiday / Non-Holiday). Measure: `Average Daily Revenue`.

**Q23. How does cumulative year-to-date revenue build over the year?**
POS sales (additive). Visual: **Area chart** (emphasizes cumulative total). Measure: `YTD Revenue`.

## Inventory (SEMI-ADDITIVE - never SUM across dates)

**Q24. What is current on-hand inventory by product and store?**
Inventory periodic snapshot (semi-additive). Visual: **Matrix** (rows = Product, columns = Store, value = on-hand qty) or **Treemap** sized by inventory value. Measure: `Ending On Hand Qty`. Critical: period-end, not SUM.

**Q25. Which products are below minimum stock?**
Inventory snapshot. Visual: **Table** with conditional formatting (red < min, yellow near min). Measure: `Ending On Hand Qty` vs Product[Reorder Point].

**Q26. What is current inventory vs target range?**
Inventory snapshot. Visual: **Gauge chart** (value = on-hand, min/max thresholds) or **KPI**. Measure: `Ending On Hand Qty`.

**Q27. Which products have stockout risk vs excess inventory?**
Inventory snapshot + POS sales velocity. Visual: **Scatter chart** (X = days of supply, Y = sales velocity) for quadrant analysis. Measures: `Days of Supply`, `Sales Velocity`.

**Q28. How did inventory change (beginning -> receipts -> sales -> adjustments -> ending)?**
Inventory movements (additive deltas). Visual: **Waterfall chart**. Measures: receipt, sales, adjustment deltas.

## E-commerce / clickstream

**Q29. How many visitors convert at each stage (visit -> add-to-cart -> checkout -> purchase)?**
Clickstream page-event/session facts. Visual: **Funnel chart** with stage counts. Measure: distinct sessions per stage; `Conversion Rate %`.

**Q30. What is the cart abandonment trend over time?**
Clickstream session (Session[Success Status]). Visual: **Combo chart** (column = abandoned count, line = abandonment %). Measures: `Abandoned Sessions`, `Abandonment Rate %`.

**Q31. Which marketing channels/campaigns drive the most valuable sessions?**
Clickstream session + Referral dimension. Visual: **Bar chart** ranked by revenue per session, or **Matrix** (Medium x Source). Measures: `Order Dollars`, `Revenue per Session`.

**Q32. Where do visitors drop off in checkout?**
Clickstream page events + Page[Page Category = Checkout]. Visual: **Funnel chart** by checkout step. Measure: `Sessions Reaching Page`.

**Q33. How does online conversion vary by device type?**
Clickstream session + Session[Device Type]. Visual: **Clustered column chart**. Measure: `Conversion Rate %`.

## Profitability

**Q34. What are gross margin trends by product category?**
POS/profitability fact (profit & revenue additive; margin % non-additive). Visual: **Combo chart** (column = gross profit, line = gross margin %). Measures: `Total Gross Profit`, `Gross Margin %`. Critical: `DIVIDE(SUM(profit), SUM(revenue))`.

**Q35. Which channels/categories/products are most and least profitable?**
Profitability fact (additive). Visual: **Waterfall chart** of profit contribution by segment, or **Decomposition tree** (Channel -> Category -> Product). Measure: `Net Profit`.

**Q36. What is the full P&L bridge from gross revenue to net profit?**
Profitability fact. Visual: **Waterfall chart** (gross revenue minus allowances/markdown/costs = net profit). Measures: P&L component sums.

**Q37. What is GMROI (gross margin return on inventory investment) by category?**
Profitability + inventory (semi-additive average inventory). Visual: **Bar chart** ranked by GMROI with target reference line. Measure: `GMROI` (see dax-measure-library.md).

---

## Notes on Power BI visual constraints

- Power BI has no dedicated "histogram" visual; create distributions with a **Column chart** over binned groups (Power BI binning) or a calculated bin attribute.
- "Small multiples" is a formatting option available on **Bar, Column, and Line charts**, not a separate visual.
- For "why did X change" exploratory questions use the **Key influencers chart** or **Decomposition tree** (both AI visuals) rather than a static chart.
- Maps (Filled/Shape/Azure) require geographic fields categorized correctly (Country, State, City, Postal Code, or lat/long).
