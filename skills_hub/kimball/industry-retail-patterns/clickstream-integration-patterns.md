# Clickstream Integration Patterns for E-Commerce

## Overview

E-commerce businesses must integrate traditional retail dimensional models with unique clickstream data capturing every web visitor gesture. This reference provides comprehensive guidance on clickstream dimensional modeling and integration with conventional retail business processes.

---

## Clickstream Data Challenges

### Challenge 1: Identifying Visitor Origin

How did the visitor arrive at your website?

**Possible Origins:**

**1. Default Home Page**
- Visitor's browser configured with your URL as home page
- Unlikely unless you're a portal or intranet
- No easy way to determine from log if you're set as home page

**2. Search Engine Referral**
- Visitor found site via Yahoo!, Google, Bing search
- May be paid placement or organic search result
- Referrer field in log identifies search engine
- Search specification usually captured

**3. Browser Bookmark**
- Visitor previously bookmarked your site
- Requires prior visit where trust/interest threshold crossed
- Common source for repeat visitors
- Difficult to detect definitively (appears as direct navigation)

**4. Clickthrough from Another Site**
- Deliberate click on link (text or graphical)
- May be paid banner ad or free referral
- Referring site identifiable in log's referrer field
- Critical for verifying marketing program efficacy
- Important for auditing clickthrough advertising charges

**Capturing Referrer Data:**
- Store in referral dimension
- Enables analysis: "Which referrers drive most sales?"
- Supports marketing ROI analysis

---

### Challenge 2: Identifying the Session

**Problem:** HTTP protocol is stateless - no inherent concept of session (login/logout)

**Goal:** Assign unique session ID to group all visitor actions in single visit

**Methods (from least to most reliable):**

**Method 1: Time-Contiguous Log Entries from Same Host**
- Group log entries with same IP address within short time window (e.g., 1 hour)
- Assumes same IP = same visitor = same session

**Limitations:**
- Dynamically assigned IP addresses reused immediately by different visitors
- Different IP addresses used within same session (mobile switching networks)
- Breaks down for high-traffic sites
- Fails for visitors behind firewalls (all appear as single IP)

**Advantages:**
- No cookies required (works for cookie-blocking visitors)
- No special web server features needed
- Many commercial log analysis products use this method

**Method 2: Session-Level Cookie**
- Web server places temporary cookie in visitor's browser
- Cookie persists for browser session only (deleted when browser closed)
- Cookie value serves as session ID

**Advantages:**
- Reliable session identification during single browser session
- Works across dynamically assigned IPs
- Available to any application requesting session cookie

**Limitations:**
- Cannot identify returning visitor in subsequent sessions
- Requires visitor to accept cookies
- No cross-device tracking (different cookie per device)

**Method 3: SSL (Secure Sockets Layer) Login**
- SSL may include explicit login action
- Exchange of encryption keys enables session tracking

**Limitations:**
- Entire session must use high-overhead SSL
- Security advisories may deter visitors
- Each host requires unique security certificate
- Overkill for non-sensitive browsing

**Method 4: Hidden Field Session ID (Dynamic Pages)**
- Session ID embedded in hidden field on each dynamically generated page
- Returned to server as query string on subsequent URL

Example URL: `http://site.com/product?sessionid=ABC123&productid=456`

**Advantages:**
- Works without cookies
- Precise session tracking

**Limitations:**
- Requires control over all page generation
- Session thread breaks if visitor clicks links not supporting session ID
- Single session may appear as multiple sessions
- Difficult if multiple vendors supply content

**Method 5: Persistent Cookie (Most Reliable)**
- Cookie not deleted when browser session ends
- Survives browser restart
- Enables identification of returning visitors across sessions

**Advantages:**
- Most reliable session tracking
- Identifies returning visitors
- Tracks behavior across multiple visits
- Can be read only by site that created it

**Limitations:**
- Visitor may refuse cookies
- Visitor may manually delete cookies
- Identifies computer, not person (shared computer problem)

**Persistent Cookie Advanced: Super Session**
- Groups of websites store common ID tag in persistent cookies
- Enables combining separate sessions across collaborating sites
- Creates "super session" view of visitor behavior

**Recommended Approach:**
Set BOTH persistent cookie (cross-session) AND session-level cookie (current session)
- Best of both worlds
- Fallback to time-contiguous host grouping if cookies blocked

---

### Challenge 3: Identifying the Visitor

**Goal:** Know WHO is visiting, not just THAT someone is visiting

**Challenges:**

**1. Visitors Want Anonymity**
- May not trust website, internet, or their computer
- Reluctant to provide personal identification
- Concerned about credit card information
- Privacy-conscious

**2. Visitors May Provide Inaccurate Information**
- Fake names, addresses, emails
- Desire to avoid spam/marketing

**3. Cannot Identify Individual on Shared Computer**
- Persistent cookie identifies computer, not person
- Family members share home computer
- Employees share office workstation
- Cannot distinguish which family member/employee is browsing

**4. Individuals Use Multiple Devices**
- Same person accesses site from office computer, home computer, tablet, phone
- Different cookie on each device
- Visitor appears as 4+ different people

**5. Cross-Device Tracking Difficult**
- Requires login to associate devices
- Or sophisticated fingerprinting techniques (IP address, browser, OS patterns)

**Approaches to Visitor Identification:**

**Approach 1: Anonymous Visitor (Cookie-Based)**
- Assign unique visitor ID via persistent cookie
- Track behavior without knowing identity
- Sufficient for many analyses (page popularity, navigation patterns)

**Approach 2: Self-Identified Visitor (Login)**
- Visitor creates account and logs in
- Explicit identification
- Can track behavior tied to known identity
- Enables personalization

**Approach 3: Inferred Identity (Purchase/Form)**
- Visitor remains anonymous until purchase or form submission
- Identity captured at conversion event
- Retroactively associate prior anonymous session with identity

**Approach 4: Third-Party Identity (Facebook Login, Google Sign-In)**
- Leverage authentication from external provider
- More trustworthy (users already trust provider)
- Easier for visitor (no new account)

**Customer Dimension Considerations:**
- Include "Anonymous Visitor" rows for unidentified visitors
- Link cookie ID to customer dimension when identity revealed
- Handle multiple cookies per customer (multi-device)
- Handle shared cookie (household)

---

## Clickstream Unique Dimensions

### Page Dimension

**Grain:** Individual page type/function (NOT every dynamic page instance)

**Why This Grain:**
- Dynamic pages could generate astronomical rows (each unique instance)
- Want rows for each "interesting distinguishable type" of page
- Static pages get own row; dynamic pages grouped by function/type

**Essential Attributes:**

| Attribute | Description | Example Values |
|-----------|-------------|----------------|
| Page Key (PK) | Surrogate key | 1, 2, 3, ... |
| Page Natural Key (NK) | Operational page identifier | "prod_detail_page", "checkout_step2" |
| Page Name | Descriptive name | "Product Detail Page", "Checkout - Shipping" |
| Page Type | Classification | "Landing Page", "Product Page", "Checkout Page", "Thank You Page" |
| Page Category | Content classification | "Product Catalog", "Customer Service", "Company Info" |
| Site Section | Major section | "Shopping", "My Account", "Help" |
| Page Template | Technical template | "product_detail_template_v3" |
| Page Function | Primary purpose | "Browse", "Search", "Purchase", "Support" |
| Page URL Pattern | URL pattern (for dynamic pages) | "/product/*", "/category/*/subcategory/*" |

**Example Rows:**

| Page Key | Page Name | Page Type | Page Category | Site Section | Page Function |
|---------|-----------|-----------|---------------|--------------|---------------|
| 1 | Home Page | Landing Page | Home | Shopping | Browse |
| 2 | Product Detail | Product Page | Product Catalog | Shopping | Browse |
| 3 | Search Results | Search Results | Product Catalog | Shopping | Search |
| 4 | Shopping Cart | Cart Page | Checkout Process | Shopping | Purchase |
| 5 | Checkout - Shipping | Checkout Page | Checkout Process | Shopping | Purchase |
| 6 | Order Confirmation | Thank You Page | Checkout Process | Shopping | Purchase |
| 7 | Account Login | Login Page | My Account | My Account | Login |

**Handling Dynamic Pages:**

**Problem:** Product detail page dynamically generated per product - could be 60,000 unique pages

**Solution:** Create ONE page dimension row for "Product Detail Page" type
- Page dimension describes page FUNCTION, not specific content
- Specific product identified by product dimension foreign key in fact table

**Collaboration Required:**
- Web developers embed descriptive codes/attributes in web log optional fields
- Data governance defines page classification scheme
- DW/BI architects map codes to page dimension attributes

**Slowly Changing Dimension Strategy:**
- **Type 1 (Overwrite):** Page definition changes immaterially (minor tweaks)
- **Type 2 (New Row):** Page definition changes materially (major redesign)
- Decision based on whether historical analysis requires distinguishing old vs new design

---

### Event Dimension

**Grain:** Type of page event

**Purpose:** Describe WHAT happened on page at specific point in time

**Essential Attributes:**

| Attribute | Description | Example Values |
|-----------|-------------|----------------|
| Event Key (PK) | Surrogate key | 1, 2, 3, ... |
| Event Name | Descriptive name | "Open Page", "Refresh Page", "Click Link", "Enter Data" |
| Event Type | Classification | "Page View", "Interaction", "Error" |
| Event Category | Grouping | "Navigation", "Data Entry", "Purchase Action" |
| Event Action | Specific action | "Click", "Submit", "Download" |
| Event Label | Additional context | "Add to Cart", "Remove from Cart", "Apply Coupon" |

**Example Rows:**

| Event Key | Event Name | Event Type | Event Category | Event Action |
|-----------|-----------|------------|----------------|--------------|
| 1 | Open Page | Page View | Navigation | View |
| 2 | Refresh Page | Page View | Navigation | Refresh |
| 3 | Click Link | Interaction | Navigation | Click |
| 4 | Enter Data | Interaction | Data Entry | Input |
| 5 | Submit Form | Interaction | Data Entry | Submit |
| 6 | Add to Cart | Interaction | Purchase Action | Add |
| 7 | Error - Page Not Found | Error | Error | Error |

**Small Dimension:**
- Typically 10-50 rows
- Relatively static (new event types added occasionally)

---

### Session Dimension

**Grain:** Classification of visitor session outcome/context

**Purpose:** Describe session-level characteristics that group sessions for analysis

**Essential Attributes:**

| Attribute | Description | Example Values |
|-----------|-------------|----------------|
| Session Key (PK) | Surrogate key | 1, 2, 3, ... |
| Local Context | Immediate session context | "Requesting Product Info", "Comparing Products", "Adding to Cart" |
| Overall Session Context | Session goal | "Ordering Product", "Customer Service Inquiry", "Browsing" |
| Success Status | Did visitor achieve goal? | "Completed Purchase", "Abandoned Cart", "Information Only" |
| Session Outcome | Final result | "Sale", "No Sale", "Support Request" |
| Customer Status | Customer classification (at time of session) | "New Visitor", "Returning Customer", "VIP", "At-Risk" |
| Device Type | Device used | "Desktop", "Mobile", "Tablet" |
| Browser Type | Browser used | "Chrome", "Safari", "Firefox" |
| OS Type | Operating system | "Windows", "macOS", "iOS", "Android" |

**Example Rows:**

| Session Key | Local Context | Overall Session Context | Success Status | Session Outcome | Customer Status |
|------------|---------------|------------------------|----------------|-----------------|-----------------|
| 1 | Requesting Product Info | Ordering Product | Completed Purchase | Sale | New Visitor |
| 2 | Requesting Product Info | Browsing | Information Only | No Sale | Returning Customer |
| 3 | Comparing Products | Ordering Product | Abandoned Cart | No Sale | Returning Customer |
| 4 | Adding to Cart | Ordering Product | Completed Purchase | Sale | VIP |

**Context Determination:**
- **Local context:** May be derivable from current page (checkout page → "Adding to Cart")
- **Overall context:** Requires processing complete session at ETL time
- **Customer status:** Derived from auxiliary business processes, embedded here for direct clickstream analysis

**Customer Status Considerations:**
- Placement in session dimension allows status to change over short periods
- Do NOT put in customer dimension (would require frequent Type 2 SCD rows)
- If large number of statuses, consider separate customer status mini-dimension

**Analysis Enabled:**
- How many customers requested product info before ordering?
- How many looked at product info and never ordered?
- Where did customers stop in abandoned sessions?
- Conversion rate by customer status

---

### Referral Dimension

**Grain:** How visitor arrived at current page

**Purpose:** Understand traffic sources and marketing channel effectiveness

**Essential Attributes:**

| Attribute | Description | Example Values |
|-----------|-------------|----------------|
| Referral Key (PK) | Surrogate key | 1, 2, 3, ... |
| Referrer Type | Classification | "Search Engine", "Paid Ad", "Partner Site", "Direct", "Email", "Social Media" |
| Referrer URL | Full URL of referring page | "http://google.com/search?q=..." |
| Referrer Domain | Domain only | "google.com", "facebook.com", "partner.com" |
| Referrer Site Name | Friendly name | "Google", "Facebook", "Partner XYZ" |
| Search Specification | Search query (if from search) | "blue widgets", "cheap laptops" |
| Search Keywords | Parsed keywords | "blue", "widgets" |
| Campaign ID | Marketing campaign identifier | "SUMMER2013", "EMAIL_BLAST_Q2" |
| Ad Group | Ad grouping | "Remarketing", "Brand Terms", "Product Category" |
| Medium | Marketing medium | "CPC", "Organic", "Email", "Social" |
| Source | Specific source | "google", "bing", "facebook", "newsletter" |

**Example Rows:**

| Referral Key | Referrer Type | Referrer Domain | Referrer Site Name | Search Specification | Campaign ID | Medium | Source |
|--------------|---------------|-----------------|-------------------|---------------------|-------------|--------|--------|
| 0 | Direct | None | Direct Navigation | None | None | Direct | Direct |
| 1 | Search Engine | google.com | Google | blue widgets | None | Organic | google |
| 2 | Paid Ad | google.com | Google Ads | cheap laptops | SUMMER2013 | CPC | google |
| 3 | Social Media | facebook.com | Facebook | None | FB_CAMPAIGN_Q2 | Social | facebook |
| 4 | Email | None | Newsletter | None | EMAIL_BLAST_Q2 | Email | newsletter |
| 5 | Partner Site | partner.com | Partner XYZ | None | AFFILIATE_001 | Referral | partner |

**Search Specification Handling:**
- Raw search specifications very complex and idiosyncratic
- May not be worth storing in full detail (difficult to query)
- Store simplified/cleaned specification
- Extract key search terms into separate attribute

**No Referrer Handling:**
- Create "Direct Navigation" row (surrogate key 0)
- Indicates visitor typed URL directly or used bookmark
- Avoid NULL referrer keys in fact table

**Analysis Enabled:**
- Which referrers drive most traffic? Most sales?
- What search terms lead to conversions?
- Marketing channel ROI analysis
- Paid vs organic search performance

---

## Clickstream Fact Tables

### Fact Table 1: Session-Level Clickstream

**Grain:** "One row per completed customer session"

**Why This Grain:**
- More manageable size than page-level events
- Example: 100M page events → 20M sessions (if 5 pages/session average)
- Supports session-level analysis without overwhelming row count

**Dimensions:**
- Calendar Date (GMT) - Universal synchronized time
- Calendar Date (Local Wall Clock) - Visitor's local time
- Time of Day (GMT) - Session start time (universal)
- Time of Day (Local Wall Clock) - Session start time (visitor perspective)
- Customer
- Page (Entry Page) - First page of session
- Session
- Referral

**Facts:**
- Session Seconds - Total time visitor spent (numeric, additive)
- Pages Visited - Count of pages viewed in session (numeric, additive)
- Orders Placed - Count of orders completed (numeric, additive)
- Units Ordered - Total units purchased (numeric, additive)
- Order Dollars - Total revenue (numeric, additive)

**Example Schema:**

```
Clickstream Session Fact Table
├─ Date Key (GMT) → Date Dimension
├─ Date Key (Local) → Date Dimension (role-playing view)
├─ Time Stamp (GMT) - Date/time stamp
├─ Time Stamp (Local) - Date/time stamp
├─ Customer Key → Customer Dimension
├─ Entry Page Key → Page Dimension
├─ Session Key → Session Dimension
├─ Referral Key → Referral Dimension
├─ Session Seconds (fact)
├─ Pages Visited (fact)
├─ Orders Placed (fact)
├─ Units Ordered (fact)
└─ Order Dollars (fact)
```

**Two Date/Time Roles:**

**Role 1: GMT (Universal Synchronized Time)**
- Enables synchronization across servers, time zones, geographies
- Required for comparing events across multiple web servers
- Allows integration with non-web systems
- Industry standard: Use GMT or UTC

**Role 2: Local Wall Clock (Visitor's Time)**
- Visitor's local time when session occurred
- Enables analysis by visitor's time-of-day patterns
- "Show sales by visitor's local hour" (when THEY think it is)
- Avoids complicated time zone calculations in BI applications

**Time Zone Handling:**
- Could store time zone in customer dimension, but conversions are complicated
- Daylight savings time shifts by city (London vs Sydney differ by 2 hours at some times)
- Simpler to store both GMT and local wall clock times directly
- Application just constrains/groups on appropriate role

**Role-Playing Date Dimensions:**
Views on single underlying date dimension table:

```sql
CREATE VIEW date_gmt AS
  SELECT day_number AS gmt_day_number,
         month AS gmt_month,
         fiscal_period AS gmt_fiscal_period,
         ...
  FROM date_dimension;

CREATE VIEW date_local AS
  SELECT day_number AS local_day_number,
         month AS local_month,
         fiscal_period AS local_fiscal_period,
         ...
  FROM date_dimension;
```

**Time of Day Handling:**
- Use full date/time stamp (not time-of-day dimension)
- Time-of-day dimension would be huge (86,400 rows for second grain)
- Few meaningful attributes for time-of-day dimension
- Date/time stamp enables precise time arithmetic (calculate gaps between sessions)

**Entry Page in Session Fact:**
- Page dimension represents ENTRY page (first page of session)
- Answers: "How did customer hop onto your bus?"
- Combined with referral dimension: Complete picture of session initiation
- Could also add exit page dimension (last page before leaving)

**Session Seconds Fact:**
- Total seconds visitor spent during session
- Problem: Cannot always detect when visitor left
  - Visitor types new URL (not detected by web server log)
  - Visitor gets up and leaves for 1 hour (no activity)
  - Visitor closes browser (no exit event)
- Solution: ETL assigns nominal small value to last step where exit not detected
- Prevents unrealistic distortion of average session time

**Missing Dimensions (Intentionally):**
- Product dimension not included (session spans multiple products typically)
- Promotion dimension not included if product-specific (multivalued in session)
- Could include market conditions dimension (applies to all products)

---

### Fact Table 2: Page Event-Level Clickstream

**Grain:** "One row per page event in each customer session"

**Why This Grain:**
- Lowest practical granularity for data warehouse
- Enables analysis of individual visitor behavior and specific pages
- Micro-events (individual JPGs, GIFs) typically discarded (unless you're Yahoo!/eBay scale)

**Warning:** Could become astronomically large
- 100M page events per day common for large sites
- 1B+ micro-events per day if tracking graphical elements

**Dimensions:**
- Calendar Date (GMT)
- Calendar Date (Local Wall Clock)
- Time of Day (GMT stamp)
- Time of Day (Local stamp)
- Customer
- Page - Specific page viewed
- Event - What happened on page
- Session - Session classification
- Session ID (Degenerate Dimension) - Groups events within session
- Product - If page involves specific product
- Referral - How visitor arrived at THIS page
- Promotion - If product-specific promotion
- Step (Overall Session) - Position in overall session
- Step (Purchase Subsession) - Position in purchase flow
- Step (Abandonment Subsession) - Position in abandonment flow

**Facts:**
- Page Seconds - Time until next page event (additive)
- Units Ordered - Units purchased (mostly NULL except purchase event)
- Order Dollars - Revenue (mostly NULL except purchase event)

**Example Schema:**

```
Clickstream Page Event Fact Table
├─ Date Key (GMT) → Date Dimension
├─ Date Key (Local) → Date Dimension (role-playing)
├─ Time Stamp (GMT) - Date/time
├─ Time Stamp (Local) - Date/time
├─ Customer Key → Customer Dimension
├─ Page Key → Page Dimension
├─ Event Key → Event Dimension
├─ Session Key → Session Dimension
├─ Session ID (DD) - Degenerate dimension
├─ Product Key → Product Dimension
├─ Referral Key → Referral Dimension
├─ Promotion Key → Promotion Dimension
├─ Overall Step Key → Step Dimension (role 1)
├─ Purchase Step Key → Step Dimension (role 2)
├─ Abandonment Step Key → Step Dimension (role 3)
├─ Page Seconds (fact)
├─ Units Ordered (fact, often NULL)
└─ Order Dollars (fact, often NULL)
```

**Session ID Degenerate Dimension:**
- Unique identifier with no semantic content
- Groups page events belonging to same session
- Does NOT join to dimension table (that's what makes it "degenerate")
- Typical dimensional modeling construct for grouping
- Not needed in session-level fact table, but serves as "parent key" to link to page-level

**Session Dimension vs Session ID:**
- **Session dimension:** Describes classes/categories of sessions (10-100 rows)
- **Session ID:** Unique identifier for each individual session (millions of values)
- Session dimension = high-level classification; Session ID = granular grouping

**Product Dimension:**
- Included because page events often relate to specific product
- Product detail page → specific product
- Add to cart event → specific product
- Purchase event → specific product
- Browse events may not involve specific product (product key = 0 "No Product")

**Promotion Dimension:**
- Product-specific promotions now appropriate (unlike session-level table)
- Page event involving specific product → can capture product's promotion

**Page Seconds vs Session Seconds:**
- **Page seconds:** Time from this page event to next page event
- **Session seconds:** Total time for entire session
- Different facts, named differently to avoid inappropriate aggregation
- Page seconds for session should sum to session seconds (conformed facts)

**Units Ordered and Order Dollars:**
- Mostly zero or NULL for non-purchase events
- Critical to include because ties revenue directly to behavior
- Enables: "What page paths lead to highest revenue?"
- NULL facts handled efficiently by DBMS (may consume zero space)
- NULL facts handled correctly by SQL aggregation (SUM, AVG ignore NULLs)

**Step Dimension - Three Roles:**

The step dimension (originally from Chapter 8: CRM) provides position within sequential process.

**Role 1: Overall Session Step**
- Position of page event within entire session (1st page, 2nd page, ... last page)
- Attributes: Step number, steps remaining, midpoint indicator

**Role 2: Purchase Subsession Step**
- Position within successful purchase sequence
- Filtered to events leading to purchase
- Constraint to "step number = 1" → Returns first page of successful purchases
- Enables: "What pages start successful purchase paths?"

**Role 3: Abandonment Subsession Step**
- Position within unsuccessful session sequence
- Filtered to events NOT leading to purchase
- Constraint to "steps remaining = 0" → Returns last page before abandonment
- Enables: "Where do visitors abandon? What's the last page they see?"

**Step Dimension Analysis Examples:**

**Query: Entry pages for successful purchases**
```sql
SELECT p.page_name, COUNT(*) as entry_count
FROM clickstream_page_event_fact f
JOIN page_dimension p ON f.page_key = p.page_key
JOIN step_dimension s ON f.purchase_step_key = s.step_key
WHERE s.step_number = 1  -- First step of purchase subsession
GROUP BY p.page_name
ORDER BY entry_count DESC;
```

**Query: Abandonment exit pages**
```sql
SELECT p.page_name, COUNT(*) as abandonment_count
FROM clickstream_page_event_fact f
JOIN page_dimension p ON f.page_key = p.page_key
JOIN step_dimension s ON f.abandonment_step_key = s.step_key
WHERE s.steps_remaining = 0  -- Last step before abandonment
GROUP BY p.page_name
ORDER BY abandonment_count DESC;
```

---

### Fact Table 3: Aggregate Clickstream (Performance)

**Grain:** "One row per month, demographic type, entry page, session outcome"

**Why Build Aggregate:**
- Frequent queries summarize millions of session or page event rows
- Aggregate provides 100x+ performance improvement
- Common query: "Show monthly visits and revenue by customer demographics"

**Dimensions:**
- Month (shrunken rollup from Date)
- Demographic Type (shrunken rollup from Customer)
- Entry Page (possibly shrunken)
- Session Outcome (shrunken rollup from Session)

**Facts:**
- Session Count
- Total Session Seconds
- Total Pages Visited
- Total Orders Placed
- Total Units Ordered
- Total Order Dollars

**Aggregate Table Size:**
- Built from session-level fact table (20M sessions)
- Results in drastically smaller table (<1% of original)
- Example: 200K rows (3 years × 12 months × 50 demographics × 100 pages × 4 outcomes)

**Shrunken Rollup Dimensions:**
- Month dimension: Subset of date dimension attributes (month, quarter, year)
- Demographic dimension: Subset of customer dimension (age band, gender, income range)
- Conformed to base dimensions (compatible, can be linked)

**Building Aggregate:**
```sql
INSERT INTO clickstream_aggregate_fact
SELECT
    d.year_month_key,
    c.demographic_type_key,
    f.entry_page_key,
    s.session_outcome_key,
    COUNT(*) as session_count,
    SUM(f.session_seconds) as total_session_seconds,
    SUM(f.pages_visited) as total_pages_visited,
    SUM(f.orders_placed) as total_orders_placed,
    SUM(f.units_ordered) as total_units_ordered,
    SUM(f.order_dollars) as total_order_dollars
FROM clickstream_session_fact f
JOIN date_dimension d ON f.date_key_gmt = d.date_key
JOIN customer_dimension c ON f.customer_key = c.customer_key
JOIN session_dimension s ON f.session_key = s.session_key
GROUP BY d.year_month_key, c.demographic_type_key,
         f.entry_page_key, s.session_outcome_key;
```

**Query Performance:**
- Queries hitting aggregate table run 100x faster
- Small table fits in memory
- Indexes more effective
- BI tool query rewrites can transparently use aggregate

---

## Google Analytics Integration

### What is Google Analytics?

**Google Analytics (GA):** External data warehouse service provided by Google

**How It Works:**
1. Add GA tracking code (GATC) to each web page
2. JavaScript code snippet in HTML <head> section
3. When visitor accesses page (with JavaScript enabled), data sent to Google
4. Google Analytics service collects and processes data
5. Data available via dashboard or can be exported

**Data Collected:**
- Virtually all clickstream data described in this chapter
- Exception: Personally Identifiable Information (PII) forbidden by GA terms of service
- Dimensions and measures terminology used correctly by Google

**Integration with Google Services:**
- Google Adwords integration for ad campaign tracking
- Conversion tracking (sales)
- Goal tracking (custom events)

**Usage:**
- Used by 50%+ of most popular websites
- Free for most use cases
- Premium version available for enterprise

**Data Warehouse Implications:**

**Option 1: Use GA Dashboard Directly**
- View data in Google's BI dashboard
- No local data warehouse needed for clickstream
- Limited to GA's predefined reports and dimensions

**Option 2: Export GA Data to Local Warehouse**
- GA provides standard and custom reports
- Export data to build local clickstream fact tables
- Integrate GA data with other business processes
- Provides full flexibility for custom analysis

**Advantages of GA:**
- No need to build/maintain clickstream collection infrastructure
- Google's expertise in web analytics
- Sophisticated analysis tools included
- Free (for most usage levels)

**Disadvantages of GA:**
- Data resides externally (privacy concerns)
- PII restrictions limit customer linkage
- Dependent on Google's data retention policies
- Limited customization of data collection

---

## Integrating Clickstream into Retail Bus Matrix

### Web Retailer Bus Matrix

Clickstream business process integrated with traditional retail processes:

**Business Processes (Rows):**
1. Supplier Shipments
2. Parts Inventory
3. Product Build
4. **Web Visitor Clickstream** ← Clickstream process
5. Sales Transactions (all channels)
6. Customer Communications
7. Service Calls
8. Cost Allocations

**Conformed Dimensions (Columns):**
- Date/Time - Shared across all processes
- Product - Shared with sales, inventory, build
- Customer - Shared with sales, communications, service
- Media - Shared with sales (channel), communications
- Promotion - Shared with sales, communications
- **Page** - Unique to clickstream
- **Event** - Unique to clickstream
- **Session** - Unique to clickstream
- **Referral** - Unique to clickstream

**Integration Benefits:**

**1. Conformed Dimensions Enable Drill-Across**
- Query clickstream and sales separately
- Combine results using shared dimensions (date, product, customer)
- Example: "Show web sessions for customers who purchased in store"

**2. Unique Clickstream Dimensions Provide New Analysis**
- Constrain sales by session characteristics
- "Show store purchases by customers whose web sessions ended in abandonment"
- "Show phone sales for customers referred by Google search"

**3. Bridge Between Web and Physical Channels**
- Track customer across channels (web browsing → store purchase)
- Measure cross-channel influence
- Example: "Customer researches online, purchases in store"

**4. Unified Customer View**
- Same customer dimension across web, store, phone channels
- Single view of customer behavior across touchpoints
- Enables omnichannel analysis

---

## Multi-Channel Profitability with Clickstream

### Extending Profitability to Include Web Costs

**Challenge:** Allocate website system costs to individual sales transactions

**Grain:** "One row per product line item on sales ticket" (all channels: store, web, phone)

**Channels:**
- Store Sales
- Web Sales
- Telesales

**Additional Costs for Web Channel:**
- Website Development Cost
- Website Hosting Cost
- Website Maintenance Cost
- Content Management Cost
- Payment Gateway Fees

**Allocation Approaches:**

**Approach 1: Allocate by Pages Devoted to Product**
- Count pages featuring each product category
- Allocate website cost proportionally
- Simple, but doesn't reflect actual usage

**Approach 2: Allocate by Page Visits**
- Track actual page views per product category (from clickstream)
- Allocate cost based on actual traffic
- Better reflects resource consumption

**Approach 3: Allocate by Purchases**
- Allocate cost only to products purchased via web
- Non-web purchases receive zero web cost
- Most directly ties cost to revenue

**Approach 4: Activity-Based Allocation**
- Complex allocation based on multiple factors
- Page views, bandwidth consumed, database queries, session time
- Most accurate, but most complex

**Example Profitability Fact with Web Costs:**

```
Units Sold
Gross Revenue (list price × quantity)
- Manufacturer Allowance
- Marketing Promotion
- Markdown
= Net Revenue
- Product Cost
- Product Storage Cost
= Gross Profit
- Outbound Shipping Cost
- Customer Service Cost
- Website System Cost (allocated)      ← Web-specific
- Payment Gateway Fees (if web)        ← Web-specific
- Marketing Cost (channel-specific)
- Overhead Cost
= Net Profit
```

**Analysis Enabled:**
- Profitability by channel (web vs store vs phone)
- True cost of web sales (including infrastructure)
- ROI of website investment
- Product profitability by channel

---

## Common Clickstream Patterns

### Pattern 1: Funnel Analysis

**Business Question:** "Where do visitors drop off in checkout process?"

**Design:**
- Page dimension classifies pages by checkout step
- Session dimension captures completion status
- Step dimension provides position in funnel

**Query:**
```sql
SELECT
    p.page_name,
    COUNT(DISTINCT f.session_id) as sessions_reaching_page,
    SUM(f.units_ordered) as orders_completed
FROM clickstream_page_event_fact f
JOIN page_dimension p ON f.page_key = p.page_key
WHERE p.page_category = 'Checkout Process'
GROUP BY p.page_name, p.page_sequence
ORDER BY p.page_sequence;
```

**Result:**
- Shopping Cart: 10,000 sessions, 0 orders
- Shipping Info: 7,000 sessions, 0 orders (30% drop-off)
- Payment Info: 5,000 sessions, 0 orders (29% drop-off)
- Order Review: 4,000 sessions, 0 orders (20% drop-off)
- Confirmation: 3,500 sessions, 3,500 orders (12.5% drop-off)
- Overall conversion: 35% (3,500 / 10,000)

### Pattern 2: Path Analysis

**Business Question:** "What navigation paths lead to purchase?"

**Design:**
- Session ID degenerate dimension groups page events
- Step dimension provides sequence
- Purchase subsession step dimension filters to successful paths

**Query:**
```sql
SELECT
    f.session_id,
    s.step_number,
    p.page_name
FROM clickstream_page_event_fact f
JOIN page_dimension p ON f.page_key = p.page_key
JOIN step_dimension s ON f.purchase_step_key = s.step_key
JOIN session_dimension sess ON f.session_key = sess.session_key
WHERE sess.success_status = 'Completed Purchase'
  AND f.session_id IN (SELECT TOP 100 session_id FROM ...)
ORDER BY f.session_id, s.step_number;
```

**Analysis:**
- Identify common paths to purchase
- Detect inefficient paths (many pages before purchase)
- Optimize navigation based on successful paths

### Pattern 3: Campaign Attribution

**Business Question:** "Which marketing campaigns drive most valuable sessions?"

**Design:**
- Referral dimension captures campaign ID
- Session dimension captures outcome
- Facts include revenue

**Query:**
```sql
SELECT
    r.campaign_id,
    r.medium,
    r.source,
    COUNT(DISTINCT f.session_id) as total_sessions,
    SUM(f.order_dollars) as total_revenue,
    AVG(f.order_dollars) as revenue_per_session
FROM clickstream_session_fact f
JOIN referral_dimension r ON f.referral_key = r.referral_key
WHERE r.campaign_id IS NOT NULL
GROUP BY r.campaign_id, r.medium, r.source
ORDER BY total_revenue DESC;
```

---

## Implementation Checklist

**Clickstream Collection:**
- [ ] Decision made on session identification method (persistent cookie recommended)
- [ ] Visitor identification approach defined (anonymous, login, inferred)
- [ ] Web server logs configured to capture referrer, user agent, session ID
- [ ] GA tracking code deployed (if using Google Analytics)

**Page Dimension:**
- [ ] Page classification scheme defined (collaboration with web developers)
- [ ] Descriptive codes embedded in web logs
- [ ] Dynamic pages grouped by function/type (not individual instances)
- [ ] Page hierarchy attributes defined (site section, category, type)

**Session Dimension:**
- [ ] Session outcome classifications defined
- [ ] ETL processes entire session to determine overall context
- [ ] Customer status labels sourced and embedded

**Referral Dimension:**
- [ ] Referrer type classifications defined
- [ ] Search specification parsing logic implemented
- [ ] Campaign tracking parameters defined
- [ ] "Direct Navigation" row included

**Session-Level Fact Table:**
- [ ] Grain clearly defined (one row per session)
- [ ] GMT and local wall clock date/times captured
- [ ] Entry page captured
- [ ] Session duration calculated correctly (handle unknown exit times)

**Page Event-Level Fact Table:**
- [ ] Grain clearly defined (one row per page event)
- [ ] Session ID degenerate dimension groups events
- [ ] Step dimension (three roles) implemented
- [ ] Product dimension included for product-specific pages
- [ ] Page seconds calculated (time to next event)
- [ ] Revenue facts included despite being mostly NULL

**Integration:**
- [ ] Clickstream business process added to bus matrix
- [ ] Conformed dimensions (date, product, customer) consistent across processes
- [ ] Clickstream data linkable to sales transaction data
- [ ] Multi-channel customer view implemented

---

## Summary

Clickstream integration for e-commerce requires:

1. **Unique dimensions** (page, event, session, referral) capture web-specific context
2. **Session identification** challenges require persistent cookies or sophisticated grouping
3. **Two-level fact tables** (session + page event) balance size vs granularity
4. **Step dimension** (three roles) enables powerful sequential analysis
5. **GMT + local time** dual date/times enable global synchronization + local perspective
6. **Conformed dimensions** (date, product, customer) integrate clickstream with traditional retail
7. **Aggregate fact tables** provide performance for common queries
8. **Google Analytics** can serve as external data warehouse or data source
9. **Multi-channel profitability** requires allocating web infrastructure costs

E-commerce dimensional models extend traditional retail patterns with web-specific dimensions while maintaining integration through conformed dimensions.
