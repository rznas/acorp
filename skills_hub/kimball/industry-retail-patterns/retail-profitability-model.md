# Retail Profitability Model

## Overview

Profitability analysis extends the sales transaction fact table to allocate all activity and infrastructure costs down to each individual sales transaction. This enables measurement of true profitability across all sales channels (store, web, phone) and all dimensions (product, customer, time, channel, promotion).

---

## Profitability vs Sales Transaction

### Sales Transaction Fact Table

**Purpose:** Record what was sold

**Grain:** One row per product line item on sales transaction

**Typical Facts:**
- Sales Quantity
- Extended Sales Dollars
- Extended Cost Dollars (if readily available)

**Focus:** Top-line revenue and basic gross profit

### Profitability Fact Table

**Purpose:** Record complete profit and loss (P&L) for each sale

**Grain:** One row per product line item on sales transaction (SAME as sales)

**Facts:** Complete P&L structure from revenue to net profit

**Focus:** All components of profitability including allocated costs

**Key Difference:**
- Sales transaction: Captures basic transaction data
- Profitability: Adds all costs (activity-based, infrastructure, allocated) to calculate true profit

---

## Grain Declaration

**Grain:** "One row per product line item sold on a sales ticket to a customer at a point in time"

**Applies to ALL sales channels:**
- Store Sales (POS transactions)
- Web Sales (e-commerce orders)
- Telesales (phone orders)
- Mobile App Sales
- Any other sales channel

**Why Same Grain as Sales Transaction:**
- Enables comparison across channels at atomic level
- Can aggregate up to any level (product, customer, time, channel)
- Avoids controversial allocation to session level (clickstream)
- Links to existing sales transaction data

---

## Dimensions

**Same dimensions as sales transaction fact table:**

- **Date** - Transaction date
- **Time** - Transaction time (or time-of-day dimension)
- **Customer** - Who purchased
- **Channel** - How purchased (store/web/phone)
- **Product** - What was purchased
- **Promotion** - Promotional conditions
- **Store** - Which store (if store channel)
- **Transaction Number** (Degenerate Dimension) - Groups line items

**Additional possible dimensions:**
- **Employee** - Sales associate (if applicable to channel)
- **Payment Method** - How customer paid
- **Shipping Method** - For delivery channels

**No new dimensions specific to profitability** - Uses same dimensional framework

---

## Profit & Loss Fact Structure

The P&L is organized as a series of revenue calculations and cost subtractions, progressively arriving at more comprehensive measures of profit.

### Complete P&L Structure

```
Units Sold (quantity)

REVENUE:
  Gross Revenue (catalog/list price × quantity)
  - Manufacturer Allowance
  - Marketing Promotion (price reduction)
  - Markdown
  = Net Revenue (actual price paid by customer × quantity)

PRODUCT COSTS:
  - Product Manufacturing Cost (if manufactured)
    OR Product Acquisition Cost (if purchased from supplier)
  - Product Storage Cost
  = Gross Profit

ACTIVITY-BASED COSTS:
  - Outbound Shipping Cost
  - Customer Service Cost
  - Website System Cost (allocated for web channel)
  - Payment Processing Cost
  - Returns Processing Cost

ALLOCATED COSTS:
  - Marketing Cost (allocated)
  - Overhead Cost (allocated)
  - Facility Cost (allocated)

  = Net Profit
```

### Fact Table Example

| Fact | Data Type | Additivity | Description |
|------|-----------|------------|-------------|
| units_sold | INTEGER | Additive | Quantity purchased |
| gross_revenue | DECIMAL(12,2) | Additive | List price × quantity |
| manufacturer_allowance | DECIMAL(12,2) | Additive | Vendor rebates/allowances |
| marketing_promotion | DECIMAL(12,2) | Additive | Price reductions from promotions |
| markdown | DECIMAL(12,2) | Additive | Clearance/markdown reductions |
| net_revenue | DECIMAL(12,2) | Additive | Actual customer payment |
| product_cost | DECIMAL(12,2) | Additive | Manufacturing or acquisition cost |
| storage_cost | DECIMAL(12,2) | Additive | Warehousing/inventory holding cost |
| gross_profit | DECIMAL(12,2) | Additive | Net revenue - product cost - storage |
| shipping_cost | DECIMAL(12,2) | Additive | Outbound delivery cost |
| customer_service_cost | DECIMAL(12,2) | Additive | Support cost (allocated) |
| website_cost | DECIMAL(12,2) | Additive | Web infrastructure (allocated if web) |
| payment_processing_cost | DECIMAL(12,2) | Additive | Credit card fees, etc. |
| marketing_cost | DECIMAL(12,2) | Additive | Marketing spend (allocated) |
| overhead_cost | DECIMAL(12,2) | Additive | General overhead (allocated) |
| net_profit | DECIMAL(12,2) | Additive | Final profit after all costs |

### Calculated Facts: Store Physically or Compute in View?

**Calculated Facts in P&L:**
- Net Revenue = Gross Revenue - Allowances - Promotion - Markdown
- Gross Profit = Net Revenue - Product Cost - Storage Cost
- Net Profit = Gross Profit - All Activity Costs - All Allocated Costs

**Decision Criteria:**

**Store Physically if:**
- Users access physical table directly (not through view)
- BI tools access table directly
- Want to ensure consistent calculation across all users/applications
- Calculation is complex or error-prone if done by users

**Compute in View if:**
- ALL users access data through views only
- No ad-hoc query tool access to physical table
- DBA enforces view-only access with no exceptions
- Want to minimize storage

**Recommendation: Store Physically**

**Why:**
- Ensures consistent calculation (eliminates user error)
- Users cannot calculate gross profit incorrectly
- Minor storage cost (a few extra columns)
- Eliminates risk of different applications calculating differently
- Provides clear, labeled metrics (gross_profit more obvious than SUM(a-b-c))

**Exception:**
If view-only access strictly enforced and no exceptions, views acceptable

---

## Cost Sourcing and Quality

### Cost Data Availability Spectrum

Different costs available at different granularities and quality levels:

#### High-Quality, Activity-Based Costs

**Characteristics:**
- Available at granular level (per transaction, per line item)
- Vary dynamically based on actual activity
- High confidence in accuracy
- Direct causal relationship to transaction

**Examples:**
- **Outbound Shipping Cost:** Actual carrier charges per shipment
- **Payment Processing Cost:** Credit card merchant fees (% of transaction + per-transaction fee)
- **Product Cost:** Standard cost from ERP system
- **Returns Processing Cost:** Actual cost to process return (if tracked)

**Data Source:** Operational systems (shipping system, payment gateway, ERP)

**Allocation:** Direct attribution to transaction (no allocation needed)

#### Medium-Quality, Estimated Costs

**Characteristics:**
- Available at moderate granularity (monthly, by region, by product category)
- Some estimation/averaging required
- Reasonable confidence in accuracy
- Indirect relationship to transaction

**Examples:**
- **Customer Service Cost:** Total monthly support cost / transaction count
- **Storage Cost:** Warehouse cost / product volume stored
- **Website Cost:** Monthly hosting cost / page views (or products sold)
- **Marketing Cost:** Campaign spend / attributed sales

**Data Source:** Finance systems, cost accounting

**Allocation:** Rule-based allocation (volume, time, attribution)

#### Low-Quality, Pro-Forma Costs

**Characteristics:**
- Available only at aggregate level (annual, company-wide)
- Heavy estimation/allocation required
- Low confidence in granular accuracy
- Very indirect relationship to transaction

**Examples:**
- **Overhead Cost:** Annual corporate overhead / total company revenue
- **Facility Cost:** Store rent + utilities / store transaction count
- **IT Cost:** Annual IT budget / company revenue

**Data Source:** Finance (general ledger)

**Allocation:** Simple allocation formulas (percentage of revenue, equal allocation)

### Cost Quality Documentation

**Critical:** Document quality level and allocation methodology for each cost

**Why:**
- Users need to understand limitations
- Cannot claim precision where none exists
- Enables informed decision-making
- Supports continuous improvement

**Example Metadata:**

| Cost Component | Granularity | Quality Level | Allocation Method | Update Frequency |
|----------------|-------------|---------------|-------------------|------------------|
| Product Cost | Line Item | High | Direct from ERP | Daily |
| Shipping Cost | Shipment | High | Direct from carrier | Daily |
| Payment Processing | Transaction | High | Calculated (% + fixed fee) | Daily |
| Customer Service | Monthly Total | Medium | Allocated by transaction count | Monthly |
| Website Cost | Monthly Total | Medium | Allocated by web sales revenue | Monthly |
| Marketing Cost | Campaign Total | Medium | Attribution model | Weekly |
| Overhead | Annual Total | Low | Allocated by revenue % | Annually |

### Improving Cost Quality Over Time

**Approach:** Publish profitability with best available costs, improve incrementally

**Phase 1: Initial Deployment**
- Use readily available costs (even if low quality)
- Simple allocation formulas for unavailable costs
- Document quality levels clearly
- Deliver value immediately

**Phase 2: Incremental Improvement**
- As better cost data sources identified, incorporate them
- Refine allocation formulas based on analysis
- Move from annual to monthly to daily granularity
- Notify users when cost quality improves

**Phase 3: Activity-Based Costing**
- Implement true ABC for critical cost drivers
- Track actual resource consumption
- Highest accuracy for most important costs

**Important:** DW/BI team is NOT responsible for implementing ABC across enterprise
- Use best cost data currently available
- Improve as finance systems evolve
- Focus on delivering value, not perfecting costs

---

## Multi-Channel Profitability

### Channel Dimension

**Essential Attributes:**
- Channel Name: "Store Sales", "Web Sales", "Telesales", "Mobile App"
- Channel Type: "Physical", "Digital", "Hybrid"
- Channel Category: "Direct", "Partner"

**Analysis Enabled:**
- Profitability by channel (which channel most profitable?)
- Cost structure by channel (web has lower labor, higher tech costs)
- Cross-channel customer behavior (research online, buy in store)

### Channel-Specific Costs

Different channels incur different costs:

**Store Sales:**
- Store rent and utilities
- Store labor (cashiers, stockers)
- POS system costs
- Minimal shipping (customer carries out)

**Web Sales:**
- Website development/maintenance
- Hosting and infrastructure
- Payment gateway fees
- Outbound shipping (100% of orders)
- Higher returns rate (blind purchase)
- Lower labor (automated checkout)

**Telesales:**
- Call center labor
- Phone system costs
- Outbound shipping (100% of orders)
- Potential for upselling (human interaction)

**Cost Allocation Approach:**

**Option 1: Conditional Cost Inclusion**
Store costs in fact table; populate based on channel:
- Store sales: Store cost populated, web cost = 0
- Web sales: Web cost populated, store cost = 0

**Option 2: Channel-Specific Cost Allocation Rules**
Allocate shared costs differently by channel:
- Marketing cost for TV ad → allocate to all channels
- Marketing cost for Google Ads → allocate only to web channel

---

## Website Cost Allocation (E-Commerce)

Website infrastructure costs are classic "infrastructure costs" - difficult to allocate but essential for profitability.

### Website Cost Components

**Development Costs:**
- Initial website build
- Feature enhancements
- A/B testing infrastructure
- Integration with inventory/fulfillment

**Hosting Costs:**
- Web server hosting
- Content delivery network (CDN)
- Database hosting
- Bandwidth

**Maintenance Costs:**
- Content updates
- Bug fixes
- Security patches
- Performance optimization

**Third-Party Costs:**
- Payment gateway fees (allocated directly)
- Analytics services (Google Analytics, etc.)
- Marketing platforms
- Customer reviews/ratings platforms

### Allocation Methodologies

#### Methodology 1: Allocate by Pages Devoted to Product

**Approach:**
- Count pages featuring each product category
- Allocate total website cost proportionally

**Example:**
- 1,000 total pages on website
- 300 pages feature "Electronics" category (30%)
- Allocate 30% of website cost to electronics sales

**Advantages:**
- Simple to calculate
- Stable over time (pages don't change daily)

**Disadvantages:**
- Doesn't reflect actual usage
- Static product pages may receive little traffic

#### Methodology 2: Allocate by Page Views

**Approach:**
- Track actual page views per product category (from clickstream)
- Allocate cost based on actual traffic

**Example:**
- 10M total page views last month
- 4M page views for electronics product pages (40%)
- Allocate 40% of monthly website cost to electronics sales

**Advantages:**
- Reflects actual resource consumption (server load, bandwidth)
- Rewards high-traffic products
- Dynamic (changes with visitor behavior)

**Disadvantages:**
- High-traffic products with low conversion subsidize low-traffic products
- Doesn't directly tie cost to revenue

#### Methodology 3: Allocate by Revenue

**Approach:**
- Allocate website cost proportionally to product category revenue

**Example:**
- $10M total web revenue last month
- $4M electronics revenue (40%)
- Allocate 40% of website cost to electronics

**Advantages:**
- Simple, stable
- High-revenue products bear more infrastructure cost

**Disadvantages:**
- Circular logic (profitability drives allocation, allocation drives profitability)
- Doesn't reflect actual resource consumption

#### Methodology 4: Allocate by Purchases

**Approach:**
- Allocate cost only to products actually purchased via web
- Non-web purchases receive zero web cost
- Within web purchases, allocate by revenue or transaction count

**Example:**
- Product sold in store: $0 web cost allocated
- Product sold on web: Pro-rata share of total website cost

**Advantages:**
- Directly ties cost to revenue generation
- Clearly separates web vs non-web profitability

**Disadvantages:**
- Products viewed but not purchased bear no cost (ROPO effect: research online, purchase offline)

#### Methodology 5: Activity-Based Allocation (Most Complex)

**Approach:**
- Track multiple activity drivers:
  - Page views (content consumption)
  - Database queries (search, filtering)
  - Bandwidth consumed (images, videos)
  - Session duration (server connection time)
  - Checkout transactions (payment processing)

**Example:**
- Base cost allocated by page views (40%)
- Database cost allocated by search queries (20%)
- Bandwidth cost allocated by MB transferred (15%)
- Transaction cost allocated by checkout events (25%)

**Advantages:**
- Most accurate reflection of resource consumption
- Captures different cost drivers appropriately

**Disadvantages:**
- Complex to implement and maintain
- Requires detailed tracking of resource usage
- May not be worth added complexity

### Recommended Approach

**Start Simple, Refine Over Time:**

**Phase 1:** Allocate by revenue (easiest to implement)
**Phase 2:** Allocate by page views (requires clickstream data)
**Phase 3:** Hybrid allocation (page views + purchases)
**Phase 4:** Activity-based (if precision required and worth effort)

**Key Principle:** Publish profitability now with simple allocation, improve over time

---

## Profitability Analysis Queries

### Query 1: Channel Profitability

**Business Question:** "Which sales channel is most profitable?"

```sql
SELECT
    ch.channel_name,
    SUM(p.units_sold) as total_units,
    SUM(p.net_revenue) as total_revenue,
    SUM(p.gross_profit) as total_gross_profit,
    SUM(p.net_profit) as total_net_profit,
    AVG(p.net_profit) as avg_profit_per_line,
    SUM(p.net_profit) / SUM(p.net_revenue) as net_margin_pct
FROM profitability_fact p
JOIN channel_dimension ch ON p.channel_key = ch.channel_key
JOIN date_dimension d ON p.date_key = d.date_key
WHERE d.calendar_year = 2013
GROUP BY ch.channel_name
ORDER BY total_net_profit DESC;
```

**Sample Results:**

| Channel | Total Units | Total Revenue | Gross Profit | Net Profit | Net Margin % |
|---------|-------------|---------------|--------------|------------|--------------|
| Store Sales | 5.2M | $312M | $124M | $28M | 9.0% |
| Web Sales | 2.1M | $156M | $78M | $18M | 11.5% |
| Telesales | 0.8M | $52M | $26M | $4M | 7.7% |

**Insights:**
- Web sales have highest margin (11.5%) despite lower volume
- Store sales highest absolute profit ($28M) due to volume
- Telesales lowest margin (7.7%) - high labor cost

### Query 2: Product Profitability by Channel

**Business Question:** "Which products are most profitable in each channel?"

```sql
SELECT
    ch.channel_name,
    pr.category_name,
    pr.brand_name,
    SUM(p.units_sold) as total_units,
    SUM(p.net_revenue) as total_revenue,
    SUM(p.net_profit) as total_profit,
    SUM(p.net_profit) / SUM(p.net_revenue) as profit_margin_pct
FROM profitability_fact p
JOIN channel_dimension ch ON p.channel_key = ch.channel_key
JOIN product_dimension pr ON p.product_key = pr.product_key
JOIN date_dimension d ON p.date_key = d.date_key
WHERE d.calendar_year = 2013
GROUP BY ch.channel_name, pr.category_name, pr.brand_name
HAVING SUM(p.net_revenue) > 100000  -- Significant revenue only
ORDER BY ch.channel_name, total_profit DESC;
```

**Insights:**
- Same product may have different profitability in different channels
- High-margin products may not be high-volume
- Some products profitable in web, unprofitable in store (due to allocated costs)

### Query 3: Customer Segment Profitability

**Business Question:** "Which customer segments are most profitable?"

```sql
SELECT
    c.customer_segment,
    ch.channel_name,
    SUM(p.net_revenue) as total_revenue,
    SUM(p.net_profit) as total_profit,
    COUNT(DISTINCT p.transaction_number) as transaction_count,
    COUNT(DISTINCT c.customer_key) as customer_count,
    SUM(p.net_profit) / COUNT(DISTINCT c.customer_key) as profit_per_customer
FROM profitability_fact p
JOIN customer_dimension c ON p.customer_key = c.customer_key
JOIN channel_dimension ch ON p.channel_key = ch.channel_key
WHERE c.customer_segment IS NOT NULL
GROUP BY c.customer_segment, ch.channel_name
ORDER BY profit_per_customer DESC;
```

**Insights:**
- VIP customers may be highly profitable overall but use expensive channels
- Budget customers may be low-margin but use efficient web channel
- Customer segment profitability varies by channel preference

### Query 4: Promotion Effectiveness (Profitability-Based)

**Business Question:** "Were promotions actually profitable?"

```sql
SELECT
    pm.promotion_name,
    SUM(p.units_sold) as units_sold,
    SUM(p.net_revenue) as revenue,
    SUM(p.gross_profit) as gross_profit,
    SUM(p.marketing_cost) as allocated_promo_cost,
    SUM(p.net_profit) as net_profit,
    SUM(p.net_profit) / NULLIF(SUM(p.marketing_cost), 0) as roi_ratio
FROM profitability_fact p
JOIN promotion_dimension pm ON p.promotion_key = pm.promotion_key
WHERE pm.promotion_key <> 0  -- Exclude "No Promotion"
GROUP BY pm.promotion_name
ORDER BY net_profit DESC;
```

**Insights:**
- Promotion may increase revenue but decrease profit (over-discounting)
- ROI ratio shows profit generated per dollar of promotion spend
- Some promotions drive volume but destroy margin

### Query 5: Time-Based Profitability Trends

**Business Question:** "How has profitability trended over time?"

```sql
SELECT
    d.year_month,
    ch.channel_name,
    SUM(p.net_revenue) as revenue,
    SUM(p.gross_profit) as gross_profit,
    SUM(p.net_profit) as net_profit,
    SUM(p.gross_profit) / SUM(p.net_revenue) as gross_margin_pct,
    SUM(p.net_profit) / SUM(p.net_revenue) as net_margin_pct
FROM profitability_fact p
JOIN date_dimension d ON p.date_key = d.date_key
JOIN channel_dimension ch ON p.channel_key = ch.channel_key
WHERE d.calendar_year IN (2012, 2013)
GROUP BY d.year_month, ch.channel_name
ORDER BY d.year_month, ch.channel_name;
```

**Insights:**
- Identify margin erosion over time
- Seasonal profitability patterns (Q4 may have lower margins due to holiday promotions)
- Channel margin trends (web margin improving, store margin declining)

---

## Cost Components Deep Dive

### Product Cost

**Sources:**
- **Manufacturing Cost:** If company manufactures products
- **Acquisition Cost:** If company purchases from suppliers
- **Standard Cost:** From ERP/accounting system

**Quality:** Usually high (available at SKU level)

**Allocation:** Direct - each line item has known product cost

**Challenges:**
- Standard cost vs actual cost (standard may not reflect current reality)
- Cost fluctuations (seasonality, market conditions)
- Promotional allowances from vendors (reduce effective cost)

### Storage Cost

**Components:**
- Warehouse rent/mortgage
- Warehouse utilities
- Warehouse labor
- Inventory holding cost (capital tied up)
- Shrinkage/spoilage

**Allocation Methods:**
- Per cubic foot stored
- Per pallet stored
- Per inventory turn
- Percentage of product cost

**Quality:** Medium (requires allocation)

**Challenge:** Products with different storage requirements (frozen, refrigerated, shelf-stable)

### Shipping Cost

**Components:**
- Carrier charges (UPS, FedEx, etc.)
- Packaging materials
- Warehouse labor (pick/pack)
- Shipping insurance

**Quality:** High for individual shipments (carrier provides actual cost)

**Allocation:**
- Direct for delivered products (web, telesales)
- Zero for customer carry-out (store sales)
- Split for mixed shipments (allocate by weight, volume, or value)

### Customer Service Cost

**Components:**
- Call center labor
- Support technology (CRM, phone system)
- Chat/email support
- Self-service portal maintenance

**Allocation Methods:**
- Per customer contact (track contacts per customer)
- Per transaction (total cost / transaction count)
- Per customer (total cost / active customers)

**Quality:** Medium (requires allocation to transaction level)

**Challenge:** Some customers generate much higher service cost than others

### Marketing Cost

**Components:**
- Advertising (TV, print, digital)
- Promotions (discounts, coupons)
- Marketing technology (email platform, analytics)
- Marketing labor

**Allocation Methods:**
- **Attribution:** Track which sales attributed to which campaigns
- **Revenue percentage:** Allocate marketing spend proportional to revenue
- **Channel-specific:** Allocate digital marketing only to web sales
- **Product-specific:** Allocate product advertising only to that product

**Quality:** Medium to low (attribution models imperfect)

**Challenge:** Marketing affects multiple products, channels, time periods

---

## Gross Margin vs Net Margin

### Gross Margin (Gross Profit %)

**Formula:**
```
Gross Margin % = Gross Profit / Gross Revenue × 100
```

Where:
- Gross Profit = Net Revenue - Product Cost - Storage Cost

**What It Measures:**
- Product markup over direct costs
- Pricing power
- Product mix effect

**Use Cases:**
- Compare product categories (which have higher markup?)
- Assess pricing strategies
- Quick profitability check

**Does NOT Include:**
- Activity-based costs (shipping, service)
- Allocated costs (marketing, overhead)

### Net Margin (Net Profit %)

**Formula:**
```
Net Margin % = Net Profit / Net Revenue × 100
```

Where:
- Net Profit = Gross Profit - All Costs (activity + allocated)

**What It Measures:**
- True profitability after all costs
- Channel efficiency
- Complete cost structure

**Use Cases:**
- Compare channels (which is most efficient?)
- Assess true profitability
- Pricing decisions considering full cost

**Includes:**
- ALL costs (direct, activity, allocated)

### Comparing Margins

**Example:**

| Metric | Store Sales | Web Sales |
|--------|-------------|-----------|
| Net Revenue | $1,000,000 | $1,000,000 |
| Product Cost | $600,000 | $600,000 |
| Storage Cost | $50,000 | $50,000 |
| **Gross Profit** | **$350,000** | **$350,000** |
| **Gross Margin %** | **35.0%** | **35.0%** |
| Shipping Cost | $10,000 | $80,000 |
| Customer Service | $30,000 | $20,000 |
| Store/Web Cost | $100,000 | $50,000 |
| Marketing Cost | $50,000 | $40,000 |
| Overhead Cost | $70,000 | $60,000 |
| **Net Profit** | **$90,000** | **$100,000** |
| **Net Margin %** | **9.0%** | **10.0%** |

**Insights:**
- Same gross margin (35%) - same products, same markup
- Different net margins (9% vs 10%) - channel cost structure differs
- Web has higher shipping but lower labor/facility costs
- Web is more profitable on net basis despite higher shipping

---

## Implementation Checklist

**Fact Table Structure:**
- [ ] Grain defined: One row per product line item (all channels)
- [ ] All revenue components captured (gross, allowances, promotions, markdowns)
- [ ] All product costs captured (manufacturing/acquisition, storage)
- [ ] All activity costs captured (shipping, service, payment processing)
- [ ] All allocated costs captured (marketing, overhead, facility)
- [ ] Calculated facts stored physically (net revenue, gross profit, net profit)

**Cost Sourcing:**
- [ ] High-quality costs identified (direct from operational systems)
- [ ] Medium-quality costs identified (allocation method defined)
- [ ] Low-quality costs identified (simple allocation formula)
- [ ] Cost quality documented in metadata
- [ ] Allocation methodologies documented
- [ ] Update frequencies defined

**Channel Handling:**
- [ ] Channel dimension includes all sales channels
- [ ] Channel-specific costs handled correctly (web cost only for web sales)
- [ ] Cross-channel analysis supported

**Website Cost Allocation:**
- [ ] Allocation methodology selected (revenue, page views, purchases, hybrid)
- [ ] Allocation formula implemented in ETL
- [ ] Allocation methodology documented

**Analysis Support:**
- [ ] BI applications can analyze profitability by all dimensions
- [ ] Standard reports: Channel profitability, product profitability, customer profitability
- [ ] "Why" analysis supported (drill into P&L components)

**Cost Improvement:**
- [ ] Process for incorporating better cost data over time
- [ ] Users notified when cost quality improves
- [ ] Continuous improvement roadmap defined

---

## Summary

Retail profitability dimensional modeling:

1. **Same grain as sales transaction** - enables atomic-level profitability analysis
2. **Complete P&L structure** - from gross revenue to net profit
3. **Multi-channel support** - compare profitability across store, web, phone
4. **Cost quality spectrum** - high-quality direct costs to low-quality allocated costs
5. **Website cost allocation** - multiple methodologies, start simple, refine over time
6. **Store calculated facts** - ensure consistent net revenue, gross profit, net profit calculation
7. **Document cost quality** - users need to understand limitations and allocation methods
8. **Continuous improvement** - publish with available costs, improve incrementally
9. **Rich dimensional framework** - answers "what" is profitable and "why"

With P&L embedded in dimensional model, you have immense analytic power to understand profitability across every conceivable dimension slice and dice.
