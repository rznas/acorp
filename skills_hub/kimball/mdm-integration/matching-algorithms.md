# Matching Algorithms for Master Data Management

This reference provides detailed technical guidance on matching algorithms used in MDM deduplication and cross-system record linkage.

---

## Exact Matching

### Single Key Exact Match

**Use When**: A reliable unique identifier exists across sources (SSN, email, account number).

**Algorithm**:
```
For each record R1 in Source A:
    For each record R2 in Source B:
        If R1.key_field = R2.key_field:
            Mark R1 and R2 as matched
```

**Optimization**: Hash join or indexed lookup (don't use nested loops for large datasets).

**SQL Implementation**:
```sql
-- Create match table
INSERT INTO Customer_Matches
SELECT
    a.source_customer_id AS source_a_id,
    b.source_customer_id AS source_b_id,
    'Email_Exact_Match' AS match_method,
    1.0 AS match_confidence
FROM staging_customer_a a
INNER JOIN staging_customer_b b
    ON LOWER(TRIM(a.email)) = LOWER(TRIM(b.email))
WHERE a.email IS NOT NULL
    AND b.email IS NOT NULL;
```

**Challenges**:
- Key field may have typos or variations
- NULL values don't match
- Keys may not exist across all systems

---

## Fuzzy Matching Algorithms

### 1. String Similarity Algorithms

#### Levenshtein Distance (Edit Distance)

**Definition**: Minimum number of single-character edits (insertions, deletions, substitutions) to transform string A into string B.

**Example**:
```
String A: "Smith"
String B: "Smyth"

Levenshtein Distance = 1 (substitute i → y)

String A: "John"
String B: "Jon"

Levenshtein Distance = 1 (delete h)
```

**Similarity Score**:
```
Similarity = 1 - (Levenshtein Distance / MAX(Length(A), Length(B)))

"Smith" vs "Smyth":
Similarity = 1 - (1 / 5) = 0.80 (80% similar)
```

**SQL Implementation** (if database supports):
```sql
-- PostgreSQL with fuzzystrmatch extension
SELECT
    customer_name_a,
    customer_name_b,
    levenshtein(customer_name_a, customer_name_b) AS edit_distance,
    1.0 - (levenshtein(customer_name_a, customer_name_b)::FLOAT /
           GREATEST(LENGTH(customer_name_a), LENGTH(customer_name_b))) AS similarity
FROM candidate_pairs
WHERE levenshtein(customer_name_a, customer_name_b) <= 3;  -- Max 3 edits
```

**When to Use**: Names with misspellings, typos, or minor variations.

**Limitations**: Computationally expensive for large datasets (O(n*m) complexity).

#### Jaro-Winkler Similarity

**Definition**: Measures similarity based on common characters and transpositions, with bonus for matching prefixes.

**Formula**:
```
Jaro Similarity = (1/3) * (m/|A| + m/|B| + (m-t)/m)

Where:
m = matching characters (within distance threshold)
t = transpositions (matching characters in different positions)
|A|, |B| = lengths of strings

Jaro-Winkler adds prefix bonus:
JW = Jaro + (prefix_length * 0.1 * (1 - Jaro))
```

**Example**:
```
"Martha" vs "Marhta"
Matching chars: M, a, r, h, t, a (6)
Transpositions: h and t swapped (1)

Jaro = (1/3) * (6/6 + 6/6 + (6-1)/6) = 0.944
Jaro-Winkler = 0.944 + (3 * 0.1 * (1-0.944)) = 0.961
```

**When to Use**: Name matching where prefix similarity is important (first names, last names).

**Advantage**: Faster than Levenshtein, good for names.

#### Soundex and Metaphone

**Soundex**: Phonetic algorithm encoding names by sound.

**Example**:
```
"Smith" → S530
"Smyth" → S530
"Schmidt" → S530

All encode to same value, enabling phonetic match.
```

**Metaphone**: Improved phonetic algorithm.

**Example**:
```
"Catherine" → K0RN
"Kathryn" → K0RN
```

**SQL Implementation**:
```sql
-- PostgreSQL with fuzzystrmatch extension
SELECT
    a.customer_name,
    b.customer_name,
    soundex(a.customer_name) AS soundex_a,
    soundex(b.customer_name) AS soundex_b
FROM customer_a a
JOIN customer_b b
    ON soundex(a.customer_name) = soundex(b.customer_name)
WHERE a.customer_name <> b.customer_name;  -- Different spelling, same sound
```

**When to Use**: Matching names with varied spellings but similar pronunciation.

**Limitations**: English-language specific, loses accuracy with non-English names.

---

### 2. Token-Based Matching

#### Jaccard Similarity (Set Similarity)

**Definition**: Measures overlap between two sets of tokens.

**Formula**:
```
Jaccard(A, B) = |A ∩ B| / |A ∪ B|

Intersection of sets / Union of sets
```

**Example - Address Matching**:
```
Address A: "123 Main Street Boston MA"
Tokens A: {123, Main, Street, Boston, MA}

Address B: "123 Main St Boston MA"
Tokens B: {123, Main, St, Boston, MA}

Intersection: {123, Main, Boston, MA} = 4 tokens
Union: {123, Main, Street, St, Boston, MA} = 6 tokens

Jaccard = 4/6 = 0.667
```

**SQL Implementation**:
```sql
-- Tokenize and compare
WITH tokens AS (
    SELECT
        customer_id,
        UNNEST(STRING_TO_ARRAY(LOWER(address), ' ')) AS token
    FROM customer
)
SELECT
    a.customer_id AS customer_a,
    b.customer_id AS customer_b,
    COUNT(DISTINCT a.token) FILTER (WHERE a.token = b.token) AS intersection,
    COUNT(DISTINCT a.token) + COUNT(DISTINCT b.token) -
        COUNT(DISTINCT a.token) FILTER (WHERE a.token = b.token) AS union_size,
    COUNT(DISTINCT a.token) FILTER (WHERE a.token = b.token)::FLOAT /
        (COUNT(DISTINCT a.token) + COUNT(DISTINCT b.token) -
         COUNT(DISTINCT a.token) FILTER (WHERE a.token = b.token)) AS jaccard
FROM tokens a
JOIN tokens b ON a.customer_id < b.customer_id
GROUP BY a.customer_id, b.customer_id
HAVING jaccard >= 0.6;
```

**When to Use**: Addresses, company names, product descriptions with word reordering or abbreviations.

#### Cosine Similarity (Vector Similarity)

**Definition**: Measures cosine of angle between two vectors (TF-IDF weighted tokens).

**When to Use**: Text fields with varied lengths (company descriptions, addresses).

**Advantage**: Less sensitive to string length differences than Jaccard.

---

### 3. Composite Scoring

Most real-world MDM uses composite scores combining multiple fields.

#### Weighted Attribute Matching

**Approach**: Assign weights to each attribute based on discriminating power.

**Example - Customer Matching**:
```
Attributes and Weights:
- Last Name: 30% (high discriminating power)
- First Name: 20%
- Address: 25%
- Phone: 15%
- Date of Birth: 10%

Scoring:
Customer A: John Smith, 123 Main St, 617-555-1234, DOB 1975-03-15
Customer B: Jon Smith, 123 Main Street, 617-555-1234, DOB 03/15/1975

Last Name Match: "Smith" = "Smith" → 100% → 0.30 * 1.0 = 0.30
First Name Match: "John" vs "Jon" (Jaro-Winkler 0.93) → 0.20 * 0.93 = 0.186
Address Match: (Jaccard 0.80) → 0.25 * 0.80 = 0.20
Phone Match: Exact match → 0.15 * 1.0 = 0.15
DOB Match: Same date, different format → 0.10 * 1.0 = 0.10

Total Match Score: 0.936 (93.6%)
```

**SQL Implementation**:
```sql
WITH match_scores AS (
    SELECT
        a.customer_id AS customer_a,
        b.customer_id AS customer_b,
        -- Last name match (30% weight)
        CASE
            WHEN LOWER(a.last_name) = LOWER(b.last_name) THEN 0.30
            WHEN jaro_winkler(a.last_name, b.last_name) >= 0.85 THEN 0.30 * jaro_winkler(a.last_name, b.last_name)
            ELSE 0
        END AS last_name_score,
        -- First name match (20% weight)
        CASE
            WHEN LOWER(a.first_name) = LOWER(b.first_name) THEN 0.20
            WHEN jaro_winkler(a.first_name, b.first_name) >= 0.80 THEN 0.20 * jaro_winkler(a.first_name, b.first_name)
            ELSE 0
        END AS first_name_score,
        -- Address match (25% weight)
        0.25 * jaccard_similarity(a.address, b.address) AS address_score,
        -- Phone match (15% weight)
        CASE
            WHEN REGEXP_REPLACE(a.phone, '[^0-9]', '', 'g') =
                 REGEXP_REPLACE(b.phone, '[^0-9]', '', 'g') THEN 0.15
            ELSE 0
        END AS phone_score,
        -- DOB match (10% weight)
        CASE
            WHEN a.date_of_birth = b.date_of_birth THEN 0.10
            ELSE 0
        END AS dob_score
    FROM customer_a a
    CROSS JOIN customer_b b
    WHERE a.customer_id <> b.customer_id
)
SELECT
    customer_a,
    customer_b,
    last_name_score + first_name_score + address_score +
        phone_score + dob_score AS total_match_score
FROM match_scores
WHERE (last_name_score + first_name_score + address_score +
       phone_score + dob_score) >= 0.75  -- 75% threshold
ORDER BY total_match_score DESC;
```

#### Threshold-Based Classification

**Match Classification**:
```
Score >= 0.90: High Confidence Match (automatic merge)
0.75 <= Score < 0.90: Medium Confidence (manual review)
Score < 0.75: Non-Match (separate records)
```

**Tuning Thresholds**:
- Analyze sample of known matches and non-matches
- Calculate precision and recall at various thresholds
- Balance false positives (incorrect merges) vs false negatives (missed duplicates)
- Adjust thresholds based on business tolerance

---

## Blocking Strategies

**Challenge**: Comparing every record to every other record is O(n²) complexity—infeasible for millions of records.

**Solution**: Blocking reduces candidate pairs by pre-filtering.

### Standard Blocking

**Approach**: Only compare records sharing a blocking key.

**Example**:
```
Blocking Key: First 3 letters of Last Name + First letter of First Name

Block "SMIJ": Compare only:
- Smith, John
- Smith, Jane
- Smyth, Jack

Don't compare "Smith, John" to "Johnson, Jane" (different blocks)
```

**SQL Implementation**:
```sql
-- Create blocking keys
CREATE TABLE customer_blocks AS
SELECT
    customer_id,
    SUBSTR(last_name, 1, 3) || SUBSTR(first_name, 1, 1) AS block_key,
    last_name,
    first_name,
    address,
    phone
FROM customer;

CREATE INDEX idx_block_key ON customer_blocks(block_key);

-- Compare only within blocks
SELECT
    a.customer_id AS customer_a,
    b.customer_id AS customer_b,
    calculate_match_score(a, b) AS match_score
FROM customer_blocks a
JOIN customer_blocks b
    ON a.block_key = b.block_key
    AND a.customer_id < b.customer_id  -- Avoid duplicate pairs
WHERE calculate_match_score(a, b) >= 0.75;
```

**Trade-off**: Reduces comparisons dramatically but may miss matches across block boundaries.

### Multi-Pass Blocking

**Approach**: Use multiple blocking strategies to capture missed matches.

**Example**:
```
Pass 1: Block on Soundex(Last Name) + State
Pass 2: Block on Phone Area Code + First Initial
Pass 3: Block on ZIP Code + First 3 chars of Last Name
```

Each pass captures matches the others might miss.

### Sorted Neighborhood Blocking

**Approach**: Sort records by composite key, compare only within sliding window.

**Example**:
```
Sort by: Last Name + First Name + DOB

Window size: 5 records

Compare record 1 to records 2-5
Compare record 2 to records 3-6
Compare record 3 to records 4-7
...
```

**Advantage**: Captures near-matches without arbitrary blocking keys.

---

## Machine Learning Approaches

### Supervised Learning for Matching

**Training Data**: Manually labeled pairs (match / non-match).

**Features**:
- Levenshtein distance of last name
- Jaro-Winkler score of first name
- Jaccard similarity of address
- Phone number exact match (binary)
- DOB exact match (binary)
- Email domain match (binary)
- Latitude/longitude distance

**Algorithms**:
- Logistic Regression
- Random Forest
- Gradient Boosted Trees (XGBoost)
- Neural Networks

**Advantage**: Learns optimal weights and thresholds from data.

**Example** (Conceptual):
```
Training Set:
(Levenshtein_LN=0, JW_FN=1.0, Jaccard_Addr=0.95, Phone_Match=1, DOB_Match=1) → MATCH
(Levenshtein_LN=2, JW_FN=0.85, Jaccard_Addr=0.70, Phone_Match=1, DOB_Match=0) → MATCH
(Levenshtein_LN=5, JW_FN=0.50, Jaccard_Addr=0.30, Phone_Match=0, DOB_Match=0) → NON-MATCH

Model learns: Phone match + high address similarity = likely match, even if name slightly different
```

### Active Learning

**Approach**: Model identifies most uncertain pairs for manual review, learns from labels.

**Workflow**:
1. Train initial model on small labeled set
2. Score all candidate pairs
3. Route pairs with 0.4 < score < 0.6 (uncertain) to stewards
4. Stewards label as match/non-match
5. Retrain model with new labels
6. Repeat until accuracy acceptable

**Advantage**: Minimizes manual labeling effort while maximizing model improvement.

---

## Address Matching Specifics

### Address Parsing

**Before Matching**: Decompose addresses into standardized components.

**Example**:
```
Input: "123 Main Street Apt 4B Boston MA 02108"

Parsed:
- Street Number: "123"
- Street Name: "Main"
- Street Type: "Street" (standardized)
- Unit Type: "Apt"
- Unit Number: "4B"
- City: "Boston"
- State: "MA"
- ZIP: "02108"
```

### Address Standardization

**Apply USPS (or country-specific) standards**:
```
"St" → "Street"
"Rd" → "Road"
"Ave" → "Avenue"
"Ste" → "Suite"
"#" → "Apt"
```

### Geocoding and Distance-Based Matching

**Approach**: Convert addresses to latitude/longitude, match by proximity.

**Example**:
```
Address A: "123 Main St, Boston MA" → (42.3601, -71.0589)
Address B: "125 Main Street, Boston MA" → (42.3602, -71.0587)

Distance: 20 meters → Likely same building or very close

Address C: "123 Main St, Cambridge MA" → (42.3736, -71.1097)

Distance: 5.8 km → Different location (not a match)
```

**When to Use**: Addresses with typos, abbreviations, or missing unit numbers.

---

## Commercial vs Open Source Tools

### Open Source Matching Libraries

**Python**:
- `fuzzywuzzy` / `rapidfuzz`: Levenshtein, Jaro-Winkler
- `dedupe`: Machine learning-based deduplication
- `recordlinkage`: Comprehensive record linkage toolkit

**R**:
- `stringdist`: String distance algorithms
- `RecordLinkage`: Probabilistic record linkage

**SQL Extensions**:
- PostgreSQL `fuzzystrmatch`: Soundex, Levenshtein, Metaphone

### Commercial MDM Platforms

**Full-featured MDM with advanced matching**:
- **Informatica MDM**: AI-driven matching, survivorship, data quality
- **IBM InfoSphere MDM**: Entity resolution, hierarchy management
- **SAP Master Data Governance**: Integrated with SAP ecosystem
- **Microsoft Master Data Services**: Windows/.NET integration
- **Talend MDM**: Open-source core with commercial extensions
- **Reltio**: Cloud-native MDM with ML matching

**Advantages**:
- Pre-built matching algorithms tuned for customer/product data
- Reference data libraries (company registries, postal codes)
- Steward UI for manual review and merge/unmerge
- Survivorship workflow automation
- Data quality monitoring dashboards

**Disadvantages**:
- Expensive licensing and implementation
- Vendor lock-in
- May be overkill for simpler use cases

---

## Performance Optimization

### Indexing Strategies

```sql
-- Index blocking keys
CREATE INDEX idx_customer_block ON customer(soundex(last_name), state);

-- Index for exact match lookups
CREATE INDEX idx_customer_email ON customer(LOWER(TRIM(email)));

-- Covering index for match scoring
CREATE INDEX idx_customer_match_attrs ON customer(customer_id, last_name, first_name, address, phone);
```

### Parallelization

**Approach**: Partition matching workload across parallel workers.

**Example**:
```sql
-- Partition by block key modulo
SELECT customer_id, MOD(HASHTEXT(block_key), 10) AS partition
FROM customer_blocks;

-- Run 10 parallel matching jobs, each processing one partition
```

### Incremental Matching

**Challenge**: Re-matching entire dataset daily is wasteful.

**Solution**: Match only new/changed records against existing master.

**Workflow**:
1. Identify changed records via CDC (Change Data Capture)
2. Match changed records against existing golden records
3. If match found: Update existing golden record (survivorship)
4. If no match: Create new golden record
5. Update cross-reference table

---

## Summary

Effective matching requires combining multiple algorithms, tuning thresholds based on data characteristics, and using blocking strategies to achieve acceptable performance. Start with simple exact match and rule-based fuzzy matching, measure accuracy, then evolve to machine learning approaches if needed. Always maintain transparency in matching decisions through match confidence scores and data lineage tracking.
