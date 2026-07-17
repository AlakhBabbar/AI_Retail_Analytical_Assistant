# Knowledge Base: Retail Sales Analytics (sales_data)

This document defines every metric, business rule, and dimension used in the
`sales_data` table. It is written for retrieval — each section is a
self-contained fact block so a RAG system can pull the right definition
without needing surrounding context.

---

## Table Grain

One row = one product sold in one region during one week.
Primary key: `transaction_id`.
Time grain: weekly, anchored to `week_start` (always a Monday).

---

## Metrics

### Promotion Lift

Promotion Lift =
(Current Promotion Revenue - Baseline Revenue) / Baseline Revenue

- **Current Promotion Revenue**: SUM of `revenue` for the product/region
  during the week(s) the promotion was active (`promotion_active = True`).
- **Baseline Revenue** is a **derived value, not a stored column**. Compute
  it as the average weekly `revenue` for the same product/region over the
  4 weeks immediately preceding the promotion's start date.
- Always compute lift against a matched baseline (same product, same
  region) — never compare across different regions or product lines.
- Positive lift = promotion increased revenue vs. its own pre-promotion
  baseline. Negative lift = revenue fell relative to baseline.

### Inventory Reduction

Inventory Reduction = Inventory Before - Inventory After

- Measures units removed from stock for a given store/product in a given
  week.

### KPI Definitions

| KPI                     | Formula                                              |
|--------------------------|-------------------------------------------------------|
| Average Revenue per Store | SUM(revenue) / COUNT(DISTINCT store_id)             |
| Average Discount %        | SUM(discount_amount) / SUM(revenue + discount_amount) |
| Average Customer Rating   | AVG(customer_rating)                                 |
| Sell-Through Rate         | SUM(units_sold) / SUM(inventory_before)              |

---

## Business Rules

### Revenue Calculation

Revenue = Units Sold × Selling Price (discounted)

- `revenue` is the **final net amount actually collected**, i.e. already
  after any discount has been applied.
- Gross Revenue (pre-discount) = `revenue + discount_amount`.
- Selling Price is not stored directly; if needed, derive the *discounted*
  unit price as `revenue / units_sold`, or the *list* unit price as
  `(revenue + discount_amount) / units_sold`.

### Discount Amount

- `discount_amount` is non-zero only when `promotion_active = True`.
- It is always expressed in currency (₹), never as a percentage, regardless
  of `promotion_type`.

### Promotion Activity

- `promotion_active = True` means a promotion was live for that product, at
  that store, during that week.
- `promotion_name` / `promotion_type` are blank when `promotion_active =
  False`.
- The same `promotion_name` may span multiple weeks, regions, or products.

---

## Promotion Types

| Type                  | Definition                                                                 |
|------------------------|----------------------------------------------------------------------------|
| **BOGO**                | "Buy One Get One" — every second unit purchased is free (~50% discount on paired units). |
| **Flat Discount**       | A fixed currency amount (₹) deducted per unit or per transaction.          |
| **Percentage Discount** | A fixed percentage deducted from the listed selling price.                 |
| **Bundle Offer**        | Multiple units/products sold together at a combined price lower than buying individually. Defined for completeness; not currently present in `sales_data.csv`. |

- Promotion type never changes how `revenue` is calculated — `revenue` is
  always the net amount collected, regardless of discount mechanism.

---

## Regional Definitions

All regions present in `sales_data.csv`:

| Region  | States Included                                            |
|---------|--------------------------------------------------------------|
| **North**   | Delhi, Punjab, Uttar Pradesh                              |
| **South**   | Karnataka, Tamil Nadu, Telangana                          |
| **East**    | West Bengal, Odisha, Bihar                                |
| **West**    | Maharashtra, Gujarat                                      |
| **Central** | Madhya Pradesh, Chhattisgarh                              |

- Region is the primary grouping level for campaign performance analysis
  (e.g., "did a campaign improve sales in the South region").
- `state` and `city` roll up into `region` in that order: city → state →
  region.

---

## Column Glossary

| Column              | Type    | Description                                                            |
|---------------------|---------|--------------------------------------------------------------------------|
| transaction_id       | INT     | Unique identifier for the row/transaction.                              |
| week_start           | DATE    | Monday date marking the start of the reporting week.                    |
| region               | TEXT    | Top-level geography (North/South/East/West/Central).                    |
| state                | TEXT    | State within the region.                                                |
| city                 | TEXT    | City within the state.                                                  |
| store_id             | TEXT    | Unique store identifier.                                                |
| product_id           | TEXT    | Unique product identifier (SKU).                                        |
| product_name         | TEXT    | Human-readable product name.                                            |
| category             | TEXT    | Product category (e.g., Soft Drinks, Snacks, Dairy).                    |
| brand                | TEXT    | Product brand/manufacturer.                                             |
| promotion_name       | TEXT    | Name of the active campaign, blank if none.                             |
| promotion_type       | TEXT    | BOGO / Flat Discount / Percentage Discount / Bundle Offer, blank if none.|
| promotion_active     | BOOLEAN | Whether a promotion was live for this row.                              |
| units_sold           | INTEGER | Number of units sold in the week.                                       |
| revenue              | FLOAT   | Net revenue collected (post-discount).                                  |
| discount_amount      | FLOAT   | Currency value of the discount applied, 0 if no promotion.              |
| inventory_before      | INTEGER | Stock level at the start of the week.                                   |
| inventory_after       | INTEGER | Stock level at the end of the week.                                     |
| customer_rating       | FLOAT   | Average customer satisfaction rating (1.0–5.0) for that product/week.   |

---

## Analytical Conventions

- **"Last month"** refers to the most recently completed calendar month
  relative to the query's reference date, based on `week_start`.
- **Campaign impact questions** ("did campaign X improve sales in region Y")
  should compare: (a) the region/product during the campaign period against
  (b) its own pre-campaign baseline, and ideally (c) a non-campaign region
  over the same period as a control, to rule out seasonality or
  market-wide trends.
- **"Sales"** is ambiguous by default — unless a question specifies
  otherwise, use `revenue` as the primary sales metric, with `units_sold`
  as a supporting figure.

---

## SQL Aggregation Rules

| Column            | Correct Aggregation                                              |
|--------------------|--------------------------------------------------------------------|
| revenue            | SUM                                                                |
| units_sold         | SUM                                                                |
| discount_amount    | SUM                                                                |
| customer_rating    | AVG (never SUM)                                                   |
| inventory_before / inventory_after | Never SUM across weeks — inventory is a snapshot, not additive. Use the value from a specific `week_start`, or AVG for a trend. |

- Group-by order for geography: `region` → `state` → `city`.
- When filtering by promotion, always also check `promotion_name` if the
  question refers to a specific campaign (multiple promotions can be
  active on overlapping dates/regions).

---

## Supported Analytical Questions

**Revenue Analysis**
- Highest/lowest revenue region, state, or category
- Weekly or monthly revenue trend
- Brand-to-brand revenue comparison

**Promotion Analysis**
- Promotion Lift for a given campaign/product/region
- Most effective promotion (highest lift)
- Discount impact on revenue and units sold

**Inventory Analysis**
- Inventory Reduction by store/product/week
- Products or stores running low on stock

**Customer Analysis**
- Highest-rated brands or products
- Category-level satisfaction trends
