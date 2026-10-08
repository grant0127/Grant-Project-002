# Supermarket Performance Analysis (Power BI + DAX)

An end-to-end analytics project built on a **synthetic supermarket dataset**: 1,000 stores across all 50 U.S. states, reported yearly from 2020 to 2026. The Power BI report answers three business questions asked by different levels of stakeholders, using DAX measures built on a single flat table and a small disconnected dimension table.

> **Note:** All data is synthetic and was generated for this project. Trends in the report are artifacts of the generator, not real-world findings.

---

## Table of Contents

- [Business Questions](#business-questions)
- [Dataset](#dataset)
- [Data Model](#data-model)
- [DAX Measures](#dax-measures)
- [Report Pages](#report-pages)
- [Key Definitions](#key-definitions)
- [Repository Structure](#repository-structure)
- [How to Reproduce](#how-to-reproduce)
- [Tools Used](#tools-used)
- [Possible Next Steps](#possible-next-steps)

---

## Business Questions

The report is organized around one question per stakeholder level:

| Stakeholder | Question |
|---|---|
| **Executive (CFO / CEO)** | Is our profitability improving, and which states are driving it? |
| **Regional / operations manager** | Which states are underperforming their peers? |
| **Store Owner** | Which department is driving the most profit, and which is causes the most loses? |

---

## Dataset

**Grain:** one row per store per year (1,000 stores x 7 years = **7,000 rows**).

| Column group | Description |
|---|---|
| `Store_ID`, `State`, `Year` | Store identifier (unique 5-digit integers), U.S. state (weighted roughly by population), and reporting year (2020-2026) |
| `Total_Revenue`, `Total_Expenses` | Annual totals per store. Revenue is between $1M and $50M, expenses between $1M and $45M |
| `<Department>_Revenue` | Revenue for each of 10 departments |
| `<Department>_Expenses` | Expenses for each of 10 departments |
| `*_Profit_Ratio` | Revenue / Expenses for the store and for each department (added after generation) |

**Departments:** Produce, Meat & Seafood, Dairy & Eggs, Bakery, Deli & Prepared Foods, Frozen Foods, Dry Grocery, Beverages, Health & Beauty, Household & General Merchandise.

**Constraints enforced and validated:**

- `Store_ID` + `Year` is unique, and every store has exactly 7 years of data
- Department revenue sums **exactly** to `Total_Revenue` for every store-year
- Department expenses sum **exactly** to `Total_Expenses` for every store-year
- Expenses are below revenue in every row (margins between roughly 1.5% and 10%, averaging about 4%)

**Realism features:**

- Each store keeps its own size, cost ratio, and department mix across all years (a meat-heavy store stays meat-heavy), with small year-to-year noise
- Revenue drifts upward over time to mimic inflation
- Thin margins, like a real grocery chain

The data is seeded (`seed = 42`), so it is fully reproducible with `generate_data.py`.

---

## Data Model

The model is deliberately simple:

- **`supermarket_flat`**: the single fact table (wide format, one column per department metric)
- **`Departments`**: a small **disconnected** table with the 10 department names and a sort order

There are no relationships. Because department revenue and expenses live in separate columns, the department measures use a `SWITCH` on the selected value of `Departments[Department]` to pick the correct column. This lets one set of measures work for any department, instead of writing ten copies of each.

```dax
Departments =
SELECTCOLUMNS (
    {
        ( "Produce", 1 ), ( "Meat & Seafood", 2 ), ( "Dairy & Eggs", 3 ),
        ( "Bakery", 4 ), ( "Deli & Prepared Foods", 5 ), ( "Frozen Foods", 6 ),
        ( "Dry Grocery", 7 ), ( "Beverages", 8 ), ( "Health & Beauty", 9 ),
        ( "Household & General Merchandise", 10 )
    },
    "Department", [Value1],
    "Sort", [Value2]
)
```

`Departments[Department]` is sorted by the `Sort` column.

---

## DAX Measures

### Base measures (used by every page)

| Measure | Purpose |
|---|---|
| `Total Revenue` | Sum of store revenue |
| `Total Expenses` | Sum of store expenses |
| `Profit` | Revenue minus expenses |
| `Margin %` | Profit divided by revenue |
| `Revenue to Expense Ratio` | Revenue divided by expenses (matches the `*_Profit_Ratio` columns) |

### Q1: Profitability trend by state

| Measure | Purpose |
|---|---|
| `Revenue PY`, `Expenses PY` | Prior-year revenue and expenses |
| `Revenue YoY %`, `Expense YoY %` | Year-over-year growth, to see whether revenue is outpacing costs |
| `Margin Change (pp)` | Margin change in percentage points, first year to last year |
| `Profit Share of Total %` | Each state's share of total chain profit |

### Q2: Underperforming stores

| Measure | Purpose |
|---|---|
| `State Avg Margin %` | Average margin of the store's state (the peer benchmark) |
| `Margin Gap vs State (pp)` | Store margin minus its state average |
| `Store Rank` | Rank of each store by margin among the stores in view |
| `Underperformer Flag` | Flags stores with margin below **4.5%** |
| `Dept Cost Ratio Gap vs Chain (pp)` | Used on the drill-through page to find which department drags a store down |

### Q3: Department analysis

| Measure | Purpose |
|---|---|
| `Dept Revenue`, `Dept Expenses` | Revenue and expenses of the selected department |
| `Dept Cost Ratio` | Department expenses divided by department revenue |
| `Dept Revenue Share %` | Department revenue as a share of total revenue |
| `Dept Share Change (pp)` | Change in share since the first year |
| `Dept Margin %` | Department profit divided by department revenue |
| `Dept Revenue to Expense Ratio` | Department revenue divided by expenses |
| `Chain Margin %` | Chain-wide margin, for comparison against a selected department |

Example of the department pattern:

```dax
Dept Revenue =
VAR d = SELECTEDVALUE ( Departments[Department] )
RETURN
    SWITCH (
        d,
        "Produce", SUM ( supermarket_flat[Produce_Revenue] ),
        "Meat & Seafood", SUM ( supermarket_flat[Meat_Seafood_Revenue] ),
        -- ... one branch per department ...
        [Total Revenue]
    )
```

---

## Report Pages

1. **Executive overview:** margin and YoY growth by year (line chart), margin change and profit share by state (map / bar chart), year slicer
2. **Store performance:** table of stores with margin, gap vs state average, rank, and underperformer flag; state and year slicers; drill-through page showing department cost-ratio gaps for a selected store
3. **Department analysis:** single-select department slicer, line chart of department margin (optionally vs chain margin) by year, department comparison by state

*(Add screenshots here: `docs/page1.png`, `docs/page2.png`, `docs/page3.png`.)*

---

## Key Definitions

| Term | Definition |
|---|---|
| **Margin %** | (Revenue - Expenses) / Revenue |
| **Cost ratio** | Expenses / Revenue. **Above 1 means the unit loses money.** Lower is better |
| **Revenue to Expense Ratio** | Revenue / Expenses (the `*_Profit_Ratio` columns). **Above 1 means profitable.** Higher is better |
| **Revenue share** | Department revenue / total revenue. Can never exceed 100% for a single department |
| **pp** | Percentage points (a change from 4.2% to 4.6% is +0.4 pp) |

Ratios are always recomputed from summed revenue and expenses. Averaging the pre-computed ratio columns would give incorrect results.

---

## Sample Finding

Chain-wide margin rose from about **4.2% in 2020 to about 4.6% in 2026**, with a dip in 2023. Because the data is synthetic, this reflects the generator's built-in growth and noise rather than a real market trend.

---

## Repository Structure

```
.
├── README.md
├── generate_data.py                      # Synthetic data generator (seeded)
├── data/
│   ├── supermarket_Original_Data_V2.xlsx # Flat table used by Power BI (with profit ratio columns)
│   ├── supermarket_flat.csv              # Same data, CSV
│   ├── stores.csv                        # Store dimension (Store_ID, State)
│   ├── store_financials.csv              # Store-year totals
│   ├── department_revenue.csv            # Long format: store, year, department, revenue
│   └── department_expenses.csv           # Long format: store, year, department, expenses
├── powerbi/
│   └── supermarket_report.pbix           # Power BI report
└── docs/
    └── (screenshots)
```

Adjust the paths above to match your actual layout.

---

## How to Reproduce

1. **Generate the data (optional):**
   ```bash
   pip install numpy pandas
   python generate_data.py
   ```
   This writes `stores.csv`, `store_financials.csv`, `department_revenue.csv`, `department_expenses.csv`, and `supermarket_flat.csv`, and runs validation checks on every constraint.
2. **Open Power BI Desktop** and load the flat table (`supermarket_Original_Data_V2.xlsx` or `supermarket_flat.csv`). Name the table `supermarket_flat`, or update the table name in the DAX.
3. **Create the `Departments` table** (Modeling > New table) using the code above. Sort `Department` by `Sort`, and do not create any relationship for it.
4. **Add the measures** from the sections above, one at a time (Modeling > New measure).
5. **Format** percentage measures as Percentage and set `Year` and `Store_ID` to "Don't summarize".
6. **Build the visuals** described under [Report Pages](#report-pages).

---

## Tools Used

- **Python** (NumPy, pandas) for synthetic data generation and validation
- **Excel / CSV** for the source data
- **Power BI Desktop** for modeling and visualization
- **DAX** for measures

---

## Possible Next Steps

- Add a store-size tier (small / medium / large) so stores are compared against size-matched peers, not just state peers
- Add a parameter so users can change the underperformer threshold interactively
- Unpivot the department columns in Power Query to remove the `SWITCH` logic and simplify the measures
- Add regional groupings (Northeast, South, Midwest, West) above state level
- Load the data into BigQuery or another warehouse and connect Power BI to it

---

## Author

**[Grant Parker]** | [LinkedIn](https://www.linkedin.com/in/grant-parker-a119632b3/?isSelfProfile=true)
