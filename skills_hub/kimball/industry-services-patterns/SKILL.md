---
name: industry-services-patterns
description: Apply comprehensive dimensional modeling patterns for service industries including telecommunications billing and call detail records, transportation voyage/route tracking with multiple granularities, education applicant pipelines and factless fact tables for course registrations, and healthcare claims billing with multivalued diagnoses and electronic medical records. Use when designing data warehouses for telecom, airlines, shipping, universities, hospitals, or similar service-based organizations.
---

# Industry-Specific Patterns - Services

## When to Use This Skill

Use this skill when designing dimensional models for service industry organizations:

**Telecommunications:** Call detail records, customer billing, network route analysis, service calls
**Transportation:** Airlines, cargo shipping, travel services, package delivery, car rentals
**Education:** Applicant tracking, course registration, facility utilization, student attendance, research grants
**Healthcare:** Claims billing and payments, electronic medical records, clinical lab results, facility/equipment utilization

These industries share common characteristics:
- Complex event tracking with multiple milestones
- Many-to-many relationships requiring bridge tables or factless facts
- Accumulating snapshots for short-lived pipelines
- Extensive use of dimension role-playing (multiple dates, physicians, airports)
- Highly regulated with standardized coding systems

## Telecommunications Patterns

### Call Detail Records (CDR)

**Business Process:** Track individual telephone calls, text messages, and data sessions for billing and network analysis.

**Grain:** One row per call/message/data session

<telecom_schema>
**Dimensions:**
- Date (call origination date)
- Time of day (call start time)
- Calling phone number (from service line dimension)
- Called phone number (to service line dimension or external number)
- Cell tower / network location (origin and destination)
- Service plan
- Call type (voice, SMS, data, roaming)
- Network carrier (if inter-carrier)

**Facts:**
- Call duration (minutes/seconds)
- Data volume (MB/GB for data sessions)
- Number of messages
- Call charges
- Roaming charges
- Dropped call indicator
</telecom_schema>

**Design Considerations:**

1. **Volume:** CDR data is extremely high volume. Billions of rows annually for large carriers.
   - Partition by date (monthly or daily)
   - Consider aggregate tables for common queries
   - Implement data retention policies

2. **Service line as dimension:** The phone number/service line is a dimension, not a degenerate dimension
   - Contains attributes: customer, sales organization, rate plan, activation date
   - Can change over time (portability, plan changes)
   - Use slowly changing dimension Type 2 for rate plan changes

3. **Geographic location:** Cell tower or switch location
   - Abstract location dimension with latitude/longitude
   - Supports geospatial analysis and visualization
   - Avoid single master location table outrigger (performance penalty)

### Customer Billing Process

**Business Process:** Monthly billing statement generation per service line.

**Grain:** One row per service line per bill (monthly)

Based on the [Chapter 11](#chapter-11) case study, the initial design had flaws:

**Common Design Mistakes (Fixed in Review):**

❌ **Bill dimension with nearly as many rows as fact table**
✅ **Bill number as degenerate dimension**

❌ **Snowflaked sales channel hierarchy**
✅ **Collapsed hierarchy into sales organization dimension**

❌ **Rate plan type code as textual fact**
✅ **Rate plan type as dimension attribute rollup**

❌ **Year-to-date metric in fact table**
✅ **Calculate YTD on-the-fly using date dimension constraints**

❌ **Inconsistent use of surrogate keys**
✅ **Surrogate keys for all dimension primary keys**

**Correct Design:**

```
Fact Table: Monthly Service Line Billing
Grain: One row per service line per bill per month

Dimensions:
- Bill date (role-playing date dimension)
- Service line (millions of rows - rapidly growing)
- Customer (mini-dimension or outrigger from service line)
- Sales organization (includes sales channel hierarchy attributes)
- Rate plan (includes rate plan type rollup)

Degenerate Dimensions:
- Bill number

Facts:
- Number of voice minutes
- Number of text messages
- Data usage (MB)
- Monthly service charges
- Overage charges
- Taxes and fees
- Total billed amount
```

**Key Patterns:**

1. **Service line vs customer:** Although each service line has one customer, treat them as separate dimensions
   - Service line dimension: millions of rows, attributes change (rate plan, status)
   - Customer dimension: can be mini-dimension or outrigger
   - Avoids schema with only two dimensions (bill date and massive service line)

2. **Surrogate keys everywhere:** Even for date dimension (though date is exception elsewhere)

3. **Operational codes with descriptive decodes:** Never present codes alone to business users

### Network Route Analysis

**Business Process:** Analyze call routing through network switches and paths.

**Grain:** One row per network hop in a call's route

Similar to transportation voyages (discussed below), a phone call travels from origin to destination through intermediate switches. This can be modeled as segments (hops) through the network with origin and destination network nodes.

See [Transportation Patterns](#transportation-patterns) for voyage schema concepts that apply to network routing.

---

## Transportation Patterns

Transportation industries deal with **voyages** - movement from point A to point B, potentially with intermediate stops.

### Multiple Fact Table Granularities

Transportation schemas frequently require **multiple fact tables at different grains**, each with grain-specific metrics:

<airline_granularities>
**Airline Example (Chapter 12):**

1. **Leg Level:** Aircraft takeoff at one airport, landing at another (no intermediate stops)
   - Facts: Flight duration, departure/arrival delays, fuel weights
   - Use: Operations, capacity planning

2. **Segment Level:** Single flight number on single aircraft (may include multiple legs)
   - Facts: Passenger revenue, mileage credit, base fare, taxes
   - Use: Marketing, revenue analysis
   - **Primary grain for passenger revenue**

3. **Trip Level:** Complete passenger journey origin to destination (may include multiple segments/aircraft changes)
   - Facts: Total trip revenue, number of segments, average fare
   - Use: Customer demand analysis, sales analytics

4. **Itinerary Level:** Entire airline ticket/reservation (may include round-trip or multi-city)
   - Facts: Total ticket revenue, commission
   - Use: Sales channel, booking analysis
</airline_granularities>

**Grain Selection Strategy:**

Start with the lowest grain that has **meaningful business metrics**. For airlines, that's the segment level (revenue is measured there). Leg-level data can be added later with conformed dimensions.

### Airline Segment Flight Activity Schema

**Grain:** One row per boarding pass collected (segment level)

```
Dimensions:
- Flight date (role-playing date dimension)
- Departure time, Arrival time (role-playing time-of-day dimensions)
- Origin airport, Destination airport (role-playing airport dimension)
- Trip origin, Trip destination (role-playing airport dimension)
- Passenger
- Passenger profile mini-dimension (frequent flyer tier, home airport, club membership, lifetime tier)
- Aircraft
- Class of service flown
- Fare basis
- Sales channel

Degenerate Dimensions:
- Itinerary number
- Ticket number
- Flight number
- Segment sequence number

Facts:
- Base fare revenue
- Passenger facility charges
- Airport and government taxes
- Ancillary fees
- Segment miles flown
- Segment miles awarded
```

**Key Patterns:**

1. **Extensive role-playing:** Multiple date, time, and airport dimensions link to single underlying physical tables via views
   - Cosmetically relabel column names in views for understandability
   - Example: "origin_airport_code" vs "destination_airport_code"

2. **Passenger mini-dimension:** Type 4 SCD pattern for frequently analyzed, slowly changing attributes
   - Frequent flyer elite tier, home airport, club membership, lifetime mileage tier
   - Avoids Type 2 on massive passenger dimension (tens/hundreds of millions of rows)
   - Marketing analysts often query mini-dimension without touching full passenger dimension

3. **Trip origin/destination:** Added to segment grain to answer "Where are passengers going?"
   - Segments mask true trip nature (most start/end at same airport)
   - Determined during ETL using 4-hour stopover definition
   - Enables trip-level analysis while maintaining segment grain

4. **Ticket-level dimensionality inherited by segments:** Sales channel applies to entire ticket but each segment inherits it

### Combining Correlated Dimensions

In most cases, role-playing dimensions should be separate. However, when data volumes are **extremely small** or there's a **need for combined attributes**, consider combining:

<combined_dimensions>
**Example 1: Class of Service (Purchased vs Flown)**

Separate approach:
- Class purchased dimension (4 rows: first, business, premium economy, economy)
- Class flown dimension (4 rows)
- Upgrade indicator dimension (3 rows: upgrade, downgrade, no change)
- Result: 3 foreign keys in fact table

Combined approach:
- Single class of service dimension with Cartesian product
- Rows: 16 (4 purchased × 4 flown)
- Includes derived "class change indicator" attribute
- Result: 1 foreign key in fact table

**When to combine:** Data volumes extremely small (< 100 rows) AND derived attributes needed

**Example 2: Origin/Destination City-Pair Route**

Challenge: Business users need attributes that depend on BOTH origin and destination:
- Distance between city-pair airports
- Type of city pair (domestic, trans-Atlantic)
- Nondirectional route name (SFO-DEN, not SFO-DEN vs DEN-SFO)

Options:
1. **Separate origin/destination dimensions + city-pair route dimension** (recommended)
   - 3 foreign keys in fact table
   - City-pair route dimension contains: directional route name, nondirectional route name, type, distance
   - Avoids combining dozens of origin/destination attributes

2. **Combined origin-destination dimension**
   - Theoretical: Cartesian product of all origin × destination airports
   - Reality: Much smaller (airlines don't fly every possible route)
   - Still large with ~24 attributes per airport × 2 + route attributes
</combined_dimensions>

**Rule:** Combine role-playing dimensions ONLY when:
- Row counts extremely small (< 100 rows), OR
- Additional attributes depend on combined underlying roles for context/meaning

Treat as **exception, not rule.**

### Cargo Shipper Schema

**Grain:** One row per container per bill-of-lading per leg of trip

```
Dimensions:
- Leg departure date, Leg arrival date
- Origin port, Destination port (role-playing port dimension)
- Trip origin port, Trip destination port (role-playing)
- Ship mode (shipping company type, vessel)
- Container (size, refrigeration requirement, power requirement)
- Commodity (harmonized commodity codes - conformed master dimension)
- Consignor, Foreign transporter, Foreign consolidator, Shipper, Domestic consolidator, Domestic transporter, Consignee
  (All roles played by master business entity dimension)

Degenerate Dimensions:
- Bill-of-lading number

Facts:
- Fees and tariffs (applicable to leg)
- Weight
- Volume
```

**Key Patterns:**

1. **Harmonized commodity codes:** U.S. Customs and other agencies use standardized codes
   - Treat as conformed dimension across organizations
   - Describes anything that can be shipped

2. **Business entity role-playing:** Single business entity dimension plays 7 roles

3. **Multimodal support:** Leg may be by ship or truck

### Travel Services Hotel/Car Rental

**Hotel Stay Grain:** One row per entire stay

**Car Rental Grain:** One row per entire rental episode

```
Hotel Stay Dimensions:
- Check-in date, Check-out date
- Hotel property
- Room type
- Customer
- Rate plan

Facts:
- Number of nights
- Room charges
- Ancillary charges (parking, meals, etc.)
- Total charges

Car Rental Dimensions (similar):
- Pickup date/location, Return date/location
- Vehicle type
- Customer
- Rate plan

Facts:
- Rental days
- Mileage
- Fuel charges
- Total charges
```

**Key Patterns:**

1. **Shared conformed dimensions with flight activity:** Date, customer
2. **Complete episode grain:** Not daily, but entire stay/rental
3. **Travel services company vs hotel chain:** Hotel chain would have much richer property, services, and charge detail

---

## Education Patterns

### Applicant Pipeline (Accumulating Snapshot)

**Business Process:** Track prospective students through admissions milestones.

**Grain:** One row per prospective student (applicant)

**Fact Table Type:** Accumulating snapshot (rows are revisited and updated as applicant progresses)

<applicant_pipeline>
**Standard Milestone Dates:**
- Initial inquiry date
- Campus visit date
- Application submitted date
- Application file completed date
- Admissions decision notification date
- Enrolled date (or withdrawn date)

Each date is a role-playing dimension with default surrogate key for TBD dates.

**Dimensions:**
- All milestone dates (6+ role-playing date dimensions)
- Applicant (geography, GPA, test scores, AP credits, high school, gender, ethnicity, preliminary major, application source)
- Intended academic program
- Admissions recruiter
- Decision status

**Facts:**
- Applicant status counts (at each milestone)
- Estimated probability of applying
- Estimated probability of enrolling if accepted
- Days between milestones (lags)
- Financial aid offered
</applicant_pipeline>

**Accumulating Snapshot Characteristics:**

1. **Single row = complete history** of applicant workflow
2. **Multiple dates** represent standard pipeline milestones
3. **Rows are updated** as applicant progresses; foreign keys AND facts may change
4. **Status counts** and **elapsed durations** are typical facts
5. **Appropriate for short-lived processes** with defined beginning, end, and standard milestones

**Alternative Approaches:**

- **Periodic snapshots:** Preserve applicant counts/statuses at critical calendar dates (early decision notification)
- **Transaction fact table:** One row per transaction per applicant for period-to-period comparisons
- **Combination:** Accumulating snapshot + periodic snapshots for critical milestones

**Other Education Pipelines:**

- **Research grant proposals:** Track from preliminary proposal → approval → award receipt
- **Student retention:** Track student from enrollment → degree completion (or withdrawal)

### Factless Fact Tables

**Factless fact tables** have no measured facts - they record relationships between dimensions at points in time.

<factless_types>
**Two Types:**

1. **Event Coverage:** Recording that an event occurred
   - Example: Student attendance, admission event participation

2. **Eligibility/Coverage:** Recording what COULD happen, whether or not it did
   - Example: Course offerings by facility/time, student course eligibility
</factless_types>

#### Admission Events

**Grain:** One row per prospective student per admission event

```
Dimensions:
- Event date
- Admission event (high school visit, college fair, alumni interview, campus overnight)
- Prospective student
- Admissions recruiter
- Geographic location

Facts:
- [None] or artificial count = 1
```

**Analysis Questions:**
- Which events are most attended?
- Which geographic regions generate most engagement?
- Which recruiters conduct most effective events?

#### Course Registration

**Grain:** One row per student per registered course per term

```
Dimensions:
- Term (conforms to calendar date dimension)
- Student (expanded from applicant dimension)
- Course (academic department, course number, credits, level)
- Faculty/Instructor
- Facility (building, room)
- Time block (days of week, start time, end time)

Facts:
- [None] or registration_count = 1 (artificial)
- [Optional if available] Tuition revenue, earned credit hours, grade score
```

**Term Dimension:**

- Term grain (Fall 2024, Spring 2025) rather than daily
- Must conform to calendar date dimension
- Each calendar date identifies: term, term + academic year, academic year
- Column labels and values identical where common

**Student Dimension SCD Strategies:**

Three approaches for tracking student attribute changes:

1. **Type 4 mini-dimension:** Separate profile dimension for frequently changing attributes
   - Declared major, class level (freshman/sophomore/junior/senior), enrollment status
   - Avoids bloating student dimension with Type 2 history
   - Good for retention rate analysis by major/class

2. **Type 7 dual keys:** Preserve both registration-time and current student profiles
   - Student surrogate key → Type 2 dimension (student as of registration)
   - Student durable identifier → Current row view (student now)
   - Supports filtering by current characteristics, grouping by historical

3. **Type 1 overwrite:** If historical student profile not critical for registration analysis

**Artificial Count Metric:**

Without numeric facts, SQL for counting is asymmetric:

```sql
-- Counts any key (all give same answer)
SELECT faculty, COUNT(term_key)
FROM course_registration_fact
GROUP BY faculty;
```

**Better:** Add artificial fact = 1 for clarity and consistency:

```sql
SELECT faculty, SUM(registration_count)
FROM course_registration_fact
GROUP BY faculty;
```

**Benefits of artificial count:**
- More expressive SQL
- Easier for BI tools to construct queries
- Required for aggregate tables (need real column to roll up)
- Required for OLAP cubes (dimension keys not revealed in cube)

**Multiple Course Instructors:**

If courses co-taught, options:
1. **Alter grain:** One row per instructor per course per student per term (unnatural, error-prone)
2. **Bridge table:** Instructor group key → instructor group bridge → instructor dimension
3. **Concatenate instructors:** Delimited text attribute (limits analysis by instructor attributes)
4. **Primary instructor:** Foreign key to primary instructor dimension only

#### Facility Utilization (Coverage Factless)

**Grain:** One row per facility per time block per day per term **regardless of usage**

```
Dimensions:
- Date (daily calendar date during term)
- Term
- Time block (hourly blocks, or specific periods like 8-9am, 9-10am)
- Facility (building, room, type, capacity, square footage, amenities)
- Utilization status (Available or Utilized)
- Owner organization (who owns the time block)
- User organization (who uses the facility - may differ from owner)

Facts:
- Utilization_flag = 1 (utilized) or 0 (available)
```

**Analysis Questions:**
- Which facilities used most heavily?
- Average occupancy rate by time of day?
- Does utilization drop on Fridays?
- Underutilized facilities?

**Key Pattern:** Insert rows for ALL facility/time combinations, not just when used. This enables "what didn't happen" analysis.

#### Student Attendance

**Grain:** One row per student who attended class per date

```
Dimensions:
- Attendance date (daily)
- Course
- Student
- Faculty
- Facility

Facts:
- [None] or attendance_count = 1
```

**Explicit Rows for Non-Attendance:**

If tracking students registered but NOT attending, add rows with attendance_flag = 0.

**Viable when:**
- Non-events have same dimensionality as events
- Non-events are relatively small percentage of total
- Business value in tracking absences

**Not viable for:** Promoted products not purchased (too many non-events)

**What Didn't Happen with OLAP Cubes:**

Multidimensional OLAP databases excel at handling sparsity:
- Minimize storage overhead for explicit zeros
- Event and non-event data available for analysis
- Less complex than relational star schema approaches for sparse data

#### Course Registration Periodic Snapshot

**Grain:** One row per student per course per term per snapshot date

**Snapshot Dates:** Preregistration, term start, drop/add deadline, term end

Captures registration counts at critical academic calendar milestones for period-to-period comparison.

---

## Healthcare Patterns

### Claims Billing and Payment (Accumulating Snapshot)

**Business Process:** Track medical claims from charge creation through full payment.

**Grain:** One row per line item on medical claim

**Fact Table Type:** Accumulating snapshot (updated as claim progresses through payment workflow)

<claims_pipeline>
**Standard Milestone Dates:**
- Treatment date
- Primary insurance billing date
- Secondary insurance billing date
- Responsible party billing date
- Last primary insurance payment date
- Last secondary insurance payment date
- Last responsible party payment date
- Zero balance date

All dates are role-playing dimensions with TBD defaults for future milestones.

**Dimensions:**
- All milestone dates (8 role-playing date dimensions)
- Patient
- Responsible party (guarantor)
- Physician
- Physician organization
- Procedure (HCPCS/CPT codes)
- Facility (office, clinic, outpatient, hospital)
- Diagnosis group (bridge to multivalued diagnoses)
- Primary insurance organization (payer)
- Secondary insurance organization (role-playing payer)

Degenerate Dimensions:
- Master patient bill ID

**Facts:**
- Billed amount
- Primary insurance paid amount
- Secondary insurance paid amount
- Responsible party paid amount
- Total paid amount (calculated)
- Amount sent to collections
- Amount written off
- Amount remaining (calculated)
- Length of stay (inpatient)
- Days from billing to primary insurance payment (lag)
- Days from billing to secondary insurance payment (lag)
- Days from billing to responsible party payment (lag)
- Days to zero balance (lag)
</claims_pipeline>

**Accumulating Snapshot Update Pattern:**

1. **Initial row creation:** When charge received from physician/facility, bill generated
   - Future milestone dates point to TBD date dimension row
   - Future payment amounts are NULL or 0

2. **Updates as payments received:**
   - Same fact table row revisited and **destructively updated**
   - Appropriate date keys updated to actual dates
   - Payment facts updated with amounts
   - Lag facts calculated

3. **Final state:** All dates populated, zero balance reached

**Physical Database Considerations:**

- **Partition on treatment date:** Preserves clustering (treatment date never changes)
- **Physical reorganization:** After rows stabilize (stop changing), recover disk space and improve performance
- **Expect rows to stabilize** within defined timeframe after treatment

**Complementary Transaction Schemas:**

Accumulating snapshot handles standard workflow but cannot fully describe messy scenarios:
- Multiple payments for single line item
- Single payment applies to multiple claims

**Solution:** Build transaction fact tables in addition to accumulating snapshot:
- **Charge transactions:** One row per charge event
- **Payment transactions:** One row per payment received
- Accumulating snapshot provides unified framework for standard analysis
- Transaction tables provide complete audit trail for complex scenarios

### Multivalued Diagnoses

**Challenge:** Patient may have 1 to 20+ simultaneous diagnoses at time of procedure.

**Why not multiple diagnosis FKs in fact table?**

❌ Don't know how many diagnosis slots needed
❌ Diagnoses don't behave like independent roles
❌ Queries don't know which slot to constrain
❌ Inefficient BI applications

**Solution:** Bridge table pattern

<diagnosis_bridge>
```
Fact Table:
- Diagnosis_group_key (FK to diagnosis group bridge)
- [Other dimensions and facts]

Diagnosis Group Bridge Table:
- Diagnosis_group_key (FK from fact table)
- Diagnosis_key (FK to diagnosis dimension)
- [Optional: Primary_diagnosis_indicator]

Diagnosis Dimension:
- Diagnosis_key (PK)
- ICD code (International Classification of Diseases)
- Diagnosis description
- [Other diagnosis attributes]
```

**Bridge Table Row Pattern:**

- If patient has 3 diagnoses → 3 rows in bridge with same diagnosis_group_key
- Each row links to different diagnosis_key
- Fact table has single diagnosis_group_key

**Weighting Factors:** Typically NOT used for diagnoses
- Virtually impossible to weight impact of each diagnosis on treatment/bill
- Exception: Primary diagnosis indicator
- Analysis focuses on **impact questions** (may result in over-counting)
  - Example: "Total billed amount for procedures involving congestive heart failure diagnosis"
</diagnosis_bridge>

**Alternative: Diagnosis Group Dimension**

If modeling tools require proper FK-to-PK relationships:

```
Fact Table → Diagnosis Group Dimension → Diagnosis Group Bridge → Diagnosis Dimension
```

- Diagnosis group dimension PK = diagnosis_group_key
- Enables conventional many-to-one joins from fact table
- May include labels for diagnosis clusters ("Kimball Syndrome")

**Creating Diagnosis Groups:**

Two approaches:

1. **Unique group per patient encounter:**
   - Row count could be astronomical
   - Many identical groups

2. **Portfolio of reusable diagnosis groups (recommended):**
   - Lookup during ETL: If existing group found, use it; if not, create new
   - Reduces redundancy
   - See Chapter 19 for bridge table administration guidance

**Evolving Diagnosis Groups (Inpatient):**

For hospital stays where diagnoses evolve over time:
- Add begin_date and end_date stamps to bridge table rows
- Complicates updates but provides change tracking
- See Chapter 7 for temporal bridge table patterns

### Supertypes and Subtypes for Charges

**Challenge:** Inpatient hospital charges differ from outpatient professional charges.

**Supertype/Subtype Pattern:**

**Shared Core Dimensions (Supertype):**
- Date dimensions
- Patient
- Responsible party
- Facility
- Primary/secondary payer
- Master patient bill ID

**Inpatient-Specific Dimensions (Subtype):**
- **Admitting physician** (role-playing physician dimension)
- **Attending physician** (role-playing physician dimension)
- **Admitting physician organization** (role-playing)
- **Attending physician organization** (role-playing)
- **Admitting diagnosis group** (multivalued, determined at admission)
- **Discharge diagnosis group** (multivalued, determined at discharge)

**Complex Surgical Teams:**

For heart transplant or other complex surgeries with specialist teams:
- Primary responsible physician FK in fact table
- Other physicians/staff linked via physician group key → bridge table

**Implementation Options:**

1. **Separate fact tables:** Inpatient charges, outpatient charges
   - Share conformed dimensions
   - Subtype-specific dimensions only in relevant fact table

2. **Consolidated fact table:** All charges with role-specific FKs
   - Nulls for inapplicable dimensions
   - May complicate queries

**Recommendation:** Separate fact tables with conformed dimensions (bus architecture)

### Electronic Medical Records (EMR/EHR)

**Challenge:** Extreme variability and volume
- Numeric data (lab results, vital signs)
- Freeform text (clinical notes, physician orders)
- Images (X-rays, CT scans, photos)

**EMR/EHR may become classic big data use case** (see Chapter 21).

#### Measurement Type Dimension for Sparse Facts

**Use Case:** Lab test results with highly variable measurements per test

**Traditional Positional Design:**

```
Lab Test Fact Table:
- [Dimensions]
- Hemoglobin_result
- White_blood_cell_count
- Glucose_level
- ... [potentially hundreds of measurement columns]
```

**Problems:**
- Sparse (many NULLs if test doesn't measure all values)
- Inflexible (ALTER TABLE to add new measurement types)

**Measurement Type Dimension Approach:**

```
Lab Test Fact Table:
Grain: One row per measurement per test event

Dimensions:
- Test date
- Patient
- Ordering physician
- Lab facility
- Test type
- Measurement type (WHAT the numeric value represents)

Facts:
- Measurement value (generic numeric)

Measurement Type Dimension:
- Measurement_type_key
- Measurement name
- Unit of measure
- Additivity restrictions (additive, semi-additive, non-additive)
- Normal range low/high
- Test category
```

**Benefits:**
- **Flexible:** Add new measurement types by adding dimension rows (no ALTER TABLE)
- **Eliminates NULLs:** Row exists only if measurement exists
- **Supports heterogeneous measurements** in single framework

**Trade-offs:**
- **Generates more fact rows:** If test has 10 measurements → 10 fact rows instead of 1
- **Complicates BI applications:**
  - Combining two measurements from same event requires fetching two rows
  - SQL arithmetic functions work within row, not across rows
  - Must prevent mixing incompatible measurements (all in single "amount" column)
- **Better for OLAP cubes:** Cubes more tolerant of calculations across measurement types

**When to Use:**
- **Extremely sparse situations** (clinical lab, manufacturing test environments)
- Density low (< 10-20% of possible measurements per event)

**When NOT to Use:**
- **Dense facts:** As density grows, too many fact rows generated
- Return to classic positional fact table design with fixed columns

#### Freeform Text Comments

**Examples:** Clinical notes, physician orders, discharge summaries

**Storage Options:**

<text_comment_handling>
**Option 1: Separate Comments Dimension (Recommended for low cardinality)**

Use when: Many fact rows share same comment (e.g., "No Comment")

```
Fact Table:
- Comment_key (FK to comments dimension)
- [Other dimensions and facts]

Comments Dimension:
- Comment_key (PK)
- Comment_text (VARCHAR/CLOB)
- Comment_type
- Comment_date
```

**Option 2: Transaction Event Dimension (For high cardinality)**

Use when: Nearly unique comment for every fact row

```
Transaction Event Dimension:
- Transaction_event_key (PK)
- Transaction_date
- Transaction_type
- Clinical_notes_text (VARCHAR/CLOB)
```

**DO NOT:**
❌ Store text in fact table directly (even as "degenerate dimension")
❌ Waste space and negatively impact query performance
❌ Degenerate dimensions are for operational control numbers, NOT bulky text
</text_comment_handling>

**Query Performance:**

Queries involving both text comments and fact metrics perform poorly (join two voluminous tables).

**Usage Pattern:**
- Business users drill into text after highly selective fact table filters applied
- Text provides context for investigation, not primary analytic target

**Parsing Text into Dimensions:**

If text contains structured information, parse into dimension attributes:
- Example: "Patient allergic to penicillin" → Allergy dimension
- Improves analytics but requires business rules and ongoing maintenance

#### Images

**Storage Options:**

1. **JPEG filename in fact table:**
   - Advantages: Other programs can access images, easier maintenance
   - Disadvantages: Separate file database must stay synchronized

2. **Embedded BLOB in database:**
   - Advantages: Single repository, automatic synchronization
   - Disadvantages: Database bloat, difficult external access

**Hybrid Approach:**
- Store filename/path in fact table or dimension
- Store actual image in content management system or cloud storage
- Maintain referential integrity through filename

**Metadata Capture:**

Regardless of storage method, capture image metadata as dimensions:
- Image type (X-ray, CT scan, MRI, photo)
- Body part/region
- Image date
- Imaging equipment
- Resolution
- Technician

### Facility/Equipment Inventory Utilization

**Business Process:** Track utilization and availability of hospital beds, operating rooms, diagnostic equipment.

**Fact Table Type Options:**

<facility_utilization>
**Option 1: Periodic Snapshot**

Grain: One row per bed/facility per snapshot time

```
Dimensions:
- Snapshot date
- Snapshot time-of-day (midnight, start of shift, hourly)
- Bed/Facility
- Patient (if occupied)
- Attending physician (if occupied)
- Assigned nurse
- Bed status (occupied, available, maintenance, cleaning)

Facts:
- Occupancy_indicator (1 or 0)
- Duration_in_status (minutes)
```

**When to use:** Regular snapshots throughout day (every hour, every shift)

**Option 2: Transaction Fact Table**

Grain: One row per movement into/out of bed/facility

```
Dimensions:
- Transaction date
- Transaction time-of-day
- Bed/Facility
- Patient
- Movement type (filled, vacated, transfer in, transfer out)

Facts:
- [Factless or occupancy_event = 1]
```

**When to use:** Changes are frequent and unpredictable

**Option 3: Timespan Fact Table**

Grain: One row per period bed/facility in specific status

```
Dimensions:
- Bed/Facility
- Patient (if occupied)
- Status (occupied, available, maintenance)

Facts:
- Effective_datetime (begin)
- Expiration_datetime (end)
- Duration (calculated)
```

**When to use:** Changes infrequent (rehabilitation, eldercare environments)
</facility_utilization>

**Operating Room Utilization:**

More complex statuses than beds:
- Pre-operation setup
- Surgery in progress
- Post-operation cleanup
- Downtime/maintenance
- Available

Facts may include duration in each status.

### Conformed Dimensions in Healthcare

**Critical Conformed Dimensions:**

1. **Patient (most important):** 360-degree view of patient
   - Challenge: Lack of reliable national ID in U.S., inconsistent patient identifiers across facilities
   - HIPAA privacy and security requirements
   - EMR/EHR systems improving patient identification

2. **Date:** All healthcare processes use common date dimension

3. **Physician:** Shared across claims, procedures, admissions, lab tests

4. **Facility:** Offices, clinics, outpatient facilities, hospitals

5. **Diagnosis:** ICD (International Classification of Diseases) standard
   - Mandated by insurance industry and government
   - Structured and predictable
   - Natural conformed dimension

6. **Procedure:** HCPCS (Healthcare Common Procedure Coding System) based on AMA's CPT (Current Procedural Terminology)
   - Medical, surgical, diagnostic services
   - Supplies and devices
   - Dentists use CDT (Current Dental Terminology) from ADA

7. **Medication:** Drug names, NDC codes, formularies

8. **Lab Test:** Test codes, descriptions

9. **Equipment:** Diagnostic and surgical equipment

10. **Payer:** Health plans, insurance organizations (primary and secondary)

11. **Employer:** For employer-sponsored health plans

12. **Responsible Party:** Guarantor for patient bills

**Easier to Conform Than Expected:**

Despite healthcare complexity, diagnosis and procedure dimensions are highly structured due to regulatory mandates (ICD, HCPCS/CPT).

### Dealing with Late Arriving Data

**Healthcare Challenge:** Legacy systems often produce late arriving data

**Scenarios:**
- Procedure data received weeks/months after occurrence
- Back-dated patient profile updates effective months ago

**Impact on DW/BI:**
- More delayed = more challenging ETL
- Late arriving facts (historical key lookup)
- Late arriving dimensions (Type 2 history corrections)

**Healthcare Reality:**
- Late arrivals may be **dominant mode**, not outlier cases
- More effective source data capture systems reducing frequency over time

See Chapter 19 (ETL Subsystems) for late arriving fact and dimension handling patterns.

---

## Cross-Industry Service Patterns Summary

### Common Service Industry Characteristics

<service_commonalities>
1. **Accumulating Snapshots for Pipelines**
   - Telecom: Service installation/provisioning
   - Transportation: Order fulfillment, package tracking
   - Education: Applicant tracking, research grant proposals
   - Healthcare: Claims billing and payment

2. **Extensive Dimension Role-Playing**
   - Telecom: Multiple dates, phone numbers, locations
   - Transportation: Multiple dates, times, airports/ports
   - Education: Multiple milestone dates
   - Healthcare: Multiple dates, physicians, payers

3. **Factless Fact Tables**
   - Telecom: Service coverage/availability
   - Transportation: Scheduled routes/flights
   - Education: Course registrations, event attendance, facility utilization
   - Healthcare: Facility/equipment availability

4. **Multivalued Dimensions with Bridge Tables**
   - Telecom: Service bundles, feature packages
   - Transportation: Co-flight passengers, cargo items
   - Education: Multiple instructors, research investigators
   - Healthcare: Diagnoses, procedures, physicians (surgical teams)

5. **High Transaction Volumes**
   - Telecom: Billions of CDR rows annually
   - Transportation: Millions of passenger segments
   - Education: Hundreds of thousands of registrations
   - Healthcare: Millions of claims and procedures

6. **Multiple Fact Table Granularities**
   - Telecom: Call detail, daily summary, monthly billing
   - Transportation: Leg, segment, trip, itinerary
   - Education: Daily attendance, term registration, academic year summary
   - Healthcare: Individual charges, claim line items, monthly summaries

7. **Regulatory/Standard Code Systems**
   - Telecom: Area codes, carrier codes, network identifiers
   - Transportation: Airport codes (IATA), country codes, aircraft types
   - Education: CIP codes (Classification of Instructional Programs), degree codes
   - Healthcare: ICD (diagnosis), HCPCS/CPT (procedure), NDC (drug codes)
</service_commonalities>

### Design Review Checklist (From Telecom Chapter 11)

When conducting dimensional model design reviews, look for these common flaws:

<design_review_checklist>
**Grain and Granularity:**
- [ ] Grain clearly declared and understood by all team members?
- [ ] Grain at lowest level possible for selected business process?
- [ ] All facts consistent with single grain (no year-to-date or aggregated facts)?

**Dimension Design:**
- [ ] Each dimension attribute takes single value for given dimension row?
- [ ] Hierarchies collapsed/denormalized within dimension (not snowflaked)?
- [ ] Surrogate keys used for all dimension primary keys (except possibly date)?
- [ ] All operational codes accompanied by descriptive decodes?
- [ ] No textual facts in fact table (moved to dimensions)?

**Fact Table Design:**
- [ ] No more than ~20 foreign keys (if more, look to combine/collapse dimensions)?
- [ ] At least one foreign key to explicit date dimension?
- [ ] Degenerate dimensions used for transaction numbers (not separate dimension tables)?
- [ ] Transaction dimension doesn't have nearly as many rows as fact table?

**Time Handling:**
- [ ] No fixed time series buckets (month1, month2, ...)?
- [ ] Meaningful role names for role-playing date dimensions?

**Conformity:**
- [ ] Commitment to conformed dimensions across business processes?
- [ ] Shared dimensions reused (not recreated with different structures)?

**Performance and Usability:**
- [ ] No snowflaking of dimension hierarchies?
- [ ] No abstract generic dimensions (e.g., generalized location)?
- [ ] Consistent naming conventions?
- [ ] Business-friendly attribute names (not codes alone)?
</design_review_checklist>

**Design Review Process Tips:**

**Before Review:**
- Invite right players (modeling team, BI developers, business representatives)
- Designate facilitator
- Agree on scope
- Block 2 days on calendars (no floating in/out)
- Reserve right space (white board, flip charts, markers)
- Assign homework (top 5 concerns/opportunities)

**During Review:**
- Check attitudes at door (change is possible)
- Ban technology unless needed (no laptops/smartphones)
- Exhibit strong facilitation
- Ensure common understanding of current model
- Designate scribe
- Start with big picture (bus matrix → grain → dimensions)
- Business acceptance is critical measure
- Sketch sample data rows
- Close with recap and assignments

**After Review:**
- Assign responsibility for open issues
- Don't let work gather dust (evaluate cost/benefit, develop action plans)
- Anticipate future reviews (12-24 months)

### Retrofitting Existing Data Structures

**Challenges:**

1. **Adding Type 2 SCD attribute to existing dimension:**
   - Requires adding dimension rows for historical changes
   - Fact table rows must be recast to reference appropriate dimension row
   - Surprisingly more effort than expected

2. **Converting to conformed dimensions:**
   - Minimum: Reprocess all fact table rows to reference conformed dimension keys
   - More challenging if granularity or major structural issues exist

3. **BI application impacts:**
   - Views provide some buffering but typically not adequate
   - Unwanted ripples in existing reports and analytics

**Cost/Benefit Evaluation:**

- **Make improvements despite pain:** If value justifies cost
- **Decommission and rebuild:** Fresh slate approach
- **Ignore suboptimal structures:** If costs outweigh benefits
- **Opportunistic remodeling:** Leverage source system conversion or BI tool migration as catalyst

---

## Implementation Guidelines

### Telecommunications Implementation

**Priority Sequence:**

1. **Month 1-2:** Customer billing process
   - Service line dimension (including customer, rate plan)
   - Sales organization dimension
   - Monthly billing fact table

2. **Month 3-4:** Call detail records (if needed)
   - Call type dimension
   - Network location dimension
   - CDR transaction fact table
   - Partition strategy and retention policies

3. **Month 5-6:** Service provisioning pipeline
   - Accumulating snapshot for service activations/changes

**Critical Success Factors:**
- Surrogate keys everywhere (no operational keys as PKs)
- Descriptive decodes for all codes
- Bill number as degenerate dimension (not separate table)
- Collapsed hierarchies (not snowflaked)

### Transportation Implementation

**Priority Sequence:**

1. **Month 1-3:** Segment-level flight activity
   - Passenger dimension + mini-dimension
   - Airport, aircraft, class of service, fare basis dimensions
   - Flight segment fact table

2. **Month 4-5:** Add trip origin/destination for demand analysis

3. **Month 6+:** Leg-level operational metrics (if needed)
   - Reuse conformed dimensions from segment schema

**Critical Success Factors:**
- Start with meaningful revenue grain (segment), not most granular (leg)
- Extensive role-playing dimensions (date, time, airport)
- Mini-dimension for frequently analyzed passenger attributes
- Trip origin/destination at segment grain (not separate trip table unless performance issues)

### Education Implementation

**Priority Sequence:**

1. **Month 1-2:** Course registration factless fact table
   - Student dimension
   - Course dimension
   - Faculty dimension
   - Term dimension (conforms to date)

2. **Month 3-4:** Applicant pipeline accumulating snapshot
   - Applicant dimension
   - Multiple milestone date dimensions
   - Status and probability facts

3. **Month 5-6:** Additional factless tables as needed
   - Admission events
   - Student attendance
   - Facility utilization

**Critical Success Factors:**
- Artificial count metric (= 1) for factless tables
- Student dimension SCD strategy decision (Type 4 mini-dim or Type 7 dual keys)
- Term dimension conformance to calendar date
- Bridge tables for multi-instructor courses (if needed)

### Healthcare Implementation

**Priority Sequence:**

1. **Month 1-3:** Claims billing accumulating snapshot
   - Patient dimension (360-degree view)
   - Physician, facility, payer dimensions
   - Diagnosis bridge table setup
   - Procedure dimension (HCPCS/CPT codes)

2. **Month 4-6:** Charge and payment transaction tables
   - Complement accumulating snapshot
   - Complete audit trail for complex scenarios

3. **Month 7+:** EMR/EHR integration
   - Lab results (potentially measurement type dimension)
   - Clinical notes handling
   - Image metadata capture

**Critical Success Factors:**
- Patient master data management (MDM) for consistent identification
- Leverage regulatory code systems (ICD, HCPCS/CPT) as conformed dimensions
- Diagnosis bridge table WITHOUT weighting factors (typically)
- Accumulating snapshot for standard workflow + transactions for complex scenarios
- Text comments in separate dimension (not in fact table)
- Late arriving data handling processes (common in healthcare)

---

## Common Mistakes to Avoid

### All Service Industries

❌ **Fixed time buckets in fact table rows** (month1, month2, ..., month12)
✅ **Separate fact table rows per time period**

❌ **Snowflaked dimension hierarchies** (normalized into separate tables)
✅ **Collapsed hierarchies within single dimension table**

❌ **Transaction control numbers as separate dimensions** (nearly as many rows as fact table)
✅ **Transaction numbers as degenerate dimensions**

❌ **Textual facts in fact table** (codes, flags, comments)
✅ **Text in dimension tables with descriptive decodes**

❌ **Abstract generic dimensions** (single location table for all addresses)
✅ **Geographic attributes within specific dimensions**

❌ **Aggregated facts at detail grain** (year-to-date in transaction fact table)
✅ **Atomic facts only; calculate aggregations on-the-fly**

❌ **Operational keys as dimension primary keys**
✅ **Surrogate keys for all dimensions (except possibly date)**

❌ **Codes without descriptive decodes**
✅ **Both operational code AND business-friendly description**

### Telecommunications

❌ **Bill dimension with row per bill** (nearly as many as fact table)
✅ **Bill number as degenerate dimension**

❌ **Service line, customer, rate plan collapsed into single mega-dimension**
✅ **Separate dimensions (even though one customer per service line)**

❌ **Year-to-date revenue in fact table**
✅ **Calculate YTD via date dimension constraints or BI tool**

### Transportation

❌ **Starting with most granular level** (leg) without business metrics
✅ **Start with lowest grain that has meaningful metrics** (segment for revenue)

❌ **Separate trip fact table for trip-level analysis**
✅ **Add trip origin/destination to segment table** (unless severe performance issues)

❌ **Combining all role-playing dimensions** (origin/destination into one table)
✅ **Separate role-playing dimensions unless volumes extremely small**

### Education

❌ **Relying on COUNT(key) in SQL without artificial fact**
✅ **Include registration_count = 1 for clarity and aggregate table support**

❌ **Trying to handle multi-instructor by altering grain** (row per instructor per course)
✅ **Use bridge table or primary instructor pattern**

❌ **Term dimension not conforming to calendar date**
✅ **Ensure term attributes identical to calendar date where common**

### Healthcare

❌ **Diagnosis as single foreign key** (can't handle 20 diagnoses)
✅ **Diagnosis group key → bridge table → diagnosis dimension**

❌ **Weighting factors in diagnosis bridge** (impossible to determine realistic weights)
✅ **Omit weighting factors; accept impact analysis over-counting**

❌ **Freeform text in fact table** (even as degenerate dimension)
✅ **Text in separate comments dimension or transaction event dimension**

❌ **Combining inpatient and outpatient in single fact table** (many NULLs)
✅ **Separate fact tables with conformed dimensions**

❌ **Not planning for late arriving data** (common in healthcare)
✅ **Implement late arriving fact and dimension handling processes**

---

## Supporting Files Reference

For additional depth, see these supporting reference files:

1. **`telecommunications-call-detail-patterns.md`**: CDR schema deep dive, volume management, geographic location dimensions

2. **`transportation-voyage-schemas.md`**: Leg/segment/trip granularities, role-playing dimension combinations, cargo shipper details

3. **`education-factless-fact-tables.md`**: Complete guide to event vs coverage factless tables, artificial count metrics, OLAP cube handling

4. **`healthcare-emr-patterns.md`**: Measurement type dimension details, text comment strategies, image handling, EMR/EHR integration

5. **`multivalued-dimension-bridge-tables.md`**: Bridge table design, weighting factors, group management, SQL query patterns

6. **`accumulating-snapshot-patterns.md`**: Applicant/grant/claims pipeline details, update strategies, complementary transaction tables

---

## Quick Reference Tables

### Fact Table Type Selection

| Business Process | Transaction | Periodic Snapshot | Accumulating Snapshot |
|------------------|-------------|-------------------|-----------------------|
| Phone calls | ✓ (CDR) | — | — |
| Monthly billing | ✓ | ✓ (if monthly metrics) | — |
| Flight segments | ✓ | — | — |
| Course registration | ✓ (Factless) | ✓ (at key dates) | — |
| Student applicants | — | ✓ (optional) | ✓ (primary) |
| Medical claims | ✓ (charges, payments) | — | ✓ (primary) |
| Hospital bed utilization | ✓ (movements) | ✓ (hourly status) | ✓ (timespans) |

### Role-Playing Dimension Intensity

| Industry | Role-Playing Usage | Common Role-Players |
|----------|-------------------|---------------------|
| Telecommunications | Moderate | Dates, phone numbers, locations |
| Transportation | Very High | Dates, times, airports/ports (4-6 roles) |
| Education | High | Dates (6+ milestone dates) |
| Healthcare | Very High | Dates (8 milestones), physicians (3+ roles), payers (primary/secondary) |

### Regulatory Code Systems as Conformed Dimensions

| Industry | Code System | Description | Conformance Potential |
|----------|-------------|-------------|----------------------|
| Telecommunications | Area codes, carrier codes | Phone number allocation | High |
| Transportation | IATA airport codes, aircraft types | International standards | Very High |
| Education | CIP codes | Classification of Instructional Programs | High |
| Healthcare | ICD | International Classification of Diseases | Very High |
| Healthcare | HCPCS/CPT | Healthcare procedure codes | Very High |
| Healthcare | NDC | National Drug Codes | Very High |

---

## Skill Integration Points

This skill integrates with:

- **Four-Step Dimensional Modeling (Skill 3):** Apply to service industry business processes
- **Fact Table Design Patterns (Skill 5):** Accumulating snapshots, factless fact tables
- **Dimension Table Design Patterns (Skill 4):** Role-playing dimensions, bridge tables, mini-dimensions
- **Enterprise Bus Matrix Design (Skill 2):** Identify service industry conformed dimensions
- **Incremental ETL Processing (Skill 10):** Accumulating snapshot updates, late arriving data
- **BI Application Design (Skill 12):** Drill-across queries using conformed service dimensions
