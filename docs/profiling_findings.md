# Data Profiling Findings — Olist E-Commerce

**Layer profiled:** `homecredit_stg` &nbsp;|&nbsp; **Phase:** Profiling (`sql/01*`) &nbsp;|&nbsp; **Grain focus:** 

---

## Scope & relationship to other docs

This file is the **systematic characterization** of each table under the 8-point framework. It records what the data *is* — baselines, distributions, cardinality, coverage — not only what is wrong. It is the evidence trail behind the modeling decisions in `strategy.md`.

| Doc | Answers | Contains |
|---|---|---|
| `strategy.md` | *What did we decide and why?* | Architecture & modeling decisions (stable) |
| `profiling_findings.md` *(this file)* | *What is the data actually like?* | 8-point observations per table (accumulating) |
| `data_quality_log` | *What is broken and what did we do about it?* | Defect register: issue, severity, status, resolution |

**Rule:** when profiling surfaces a genuine defect, write one line in the finding's *So what* and link to its `data_quality_log` entry (`DQ-###`). Do **not** duplicate the defect detail here.

---

## Finding template

Every finding follows the fixed shape:

- **Numbers** — the raw metric / query output.
- **Reading** — what it says literally.
- **So what** — implication for the mart, dashboard, or analysis (link `DQ-###` if it's a defect).
- **Limits** — what this metric does *not* tell us; caveats.

---

## The 8 points

| # | Point | Question |
|---|---|---|
| 1 | **Volume** | Row count per table — does it match expectation? (baseline for later comparison) |
| 2 | **Uniqueness / PK** | Is the primary key actually unique? *(grain trap: `customer_id` is per-order, not per-person)* |
| 3 | **Completeness / Null** | Null rate per column — especially join keys and delivery-logic columns |
| 4 | **Cardinality** | Distinct-value count — spot near-constant columns or fake identifiers |
| 5 | **Range / Distribution** | min/max/quantiles for numeric & date (`APPROX_QUANTILES`) — negative, future, or impossible dates? |
| 6 | **Domain validity** | Do categorical values sit in the allowed set? (`order_status`, `review_score` 1–5) |
| 7 | **Referential integrity** | Orphans across tables — does every FK resolve? *(critical for the star schema)* |
| 8 | **Anomaly / business rule** | Business-rule violations (delivered but no delivery date, empty order_items…) *(keep all statuses — avoid survivor bias)* |

---

## Table tiers & status

| Table | Tier | Treatment | Status |
|---|---|---|---|
| `application_train` | A | Full 8-point | ☑ |
| `???` | — | **Deferred** (payment flow = future project) | ⏸ |

> Mark ☑ when the section below is filled and committed.

---

## Coverage matrix

Rows = tables · Columns = the 8 points · **☐** applicable, not yet done · **☑** done · **–** not applicable

| Table | Tier | 1 Vol | 2 PK | 3 Null | 4 Card | 5 Range | 6 Domain | 7 RefInt | 8 Rule |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| `application_train` | A | ☑ | ☑ | ☐ | ☐ | ☐ | ☐ | ☐ | ☑ |

**Why the blanks:** 

---

## Shared evidence

Raw output from batched queries, stored once and referenced by ID from the table sections below. Findings cite the source (e.g. `source: E1`) instead of repeating the numbers.

### E1 — Row counts, all tables
`sql/01a_structure.sql` · `Olist_stg.__TABLES__`

|TABLE|Tier|Value|
|---|---|---:|
|order_payments|	-	|103,886|
|order_items|	A	|112,650|
|order_reviews|	A	|99,224|
|orders|	A	|99,441|
|customers|	B	|99,441|
|products|	B	|32,951|
|sellers|	B	|3,095|
|geolocation|	C	|1,000,163|
|product_category_name|C|72|

*Anchor ≈ 99.4k across the three order-related tables. Deviations from this anchor are structural (one-to-many relationships or coverage gaps), not random.*

### E2 — PK uniqueness & key nulls
`sql/01b_keys.sql` · `Olist_stg.__TABLES__`

|TABLE|Tier|Size|PK|
|---|---|---|---:|
|order_items|A|112650|112650|

### E3 — Null rates, all columns
`sql/01c_nulls.sql` · *pending*

|TABLE: orders|tbl|column_name|null_count|null_pct|
|---|---|---|---|---:|
|1|orders|order_delivered_customer_date|2965|2.98|

### E4 _ Cardinality
|tbl|column_name|card|
|---|---|---:|
|orders|order_status|8|

### E5 _ Range
|tbl|first_order|last_order|delivered_before_purchase|
|---|---|---|---:|
|orders|2016-09-04 21:15:19.000000 UTC|2018-10-17 17:30:18.000000 UTC|0|

### E6 _ Domain Validity
|tbl|order_status|count|pct|
|---|---|---|---:|
|orders|canceled|625|0.63|

## E7 _ Referential integrity
| Relationship | Direction | Metric | Count |
|---|---|---|---:|
| order_items → orders | forward | orphan items (item with no order) | 0 |

#### product_names_relationship
|total_categories|translated_cat|
|---|---:|
|73|71|

#### geolocation_relationship
|relationship|total_zip|matched_zip|
|---|---|---:|
|customers-geo|14994|14837|

## E8 _ Anomaly / business rule
|approved_before_purchase|carrier_before_approved|delivered_before_carrier|delivered_before_purchase|delivered_no_carrier|delivered_no_approval|delivered_no_purchase|
|---|---|---|---|---|---|---:|
|0|1359|23|0|1|14|0|

# Tier A — full 8-point

## <TABLE>

> PK: <PK> · join keys: <> · key dates drive <> / <>

### 1. Volume
- **Numbers:** 
- **Reading:** 
- **So what:** 
- **Limits:** 

### 2. Uniqueness / PK

### 3. Completeness / Null

### 4. Cardinality

### 5. Range / Distribution

### 6. Domain validity

### 7. Referential integrity

### 8. Anomaly / business rule
---

## Open hypotheses to confirm

## Confirmed data characteristics
