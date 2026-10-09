# Supermarket Performance Dashboard (Power BI + DAX)

A three-page Power BI report built on a **synthetic supermarket dataset**: 1,000 stores across all 50 U.S. states, reported yearly from 2020 to 2026. Each page answers one business question using DAX measures on a single flat table plus a small disconnected `Departments` table.

> **Note:** All data is synthetic and was generated for this project. The trends and rankings below describe the generated data, not real-world supermarkets.

---

## Table of Contents

- [Business Questions](#business-questions)
- [Dashboard](#dashboard)
  - [Page 1: Profitability trend and top states](#page-1-profitability-trend-and-top-states)
  - [Page 2: Underperforming states](#page-2-underperforming-states)
  - [Page 3: Department profit and losses](#page-3-department-profit-and-losses)
- [Dataset](#dataset)
- [Data Model](#data-model)
- [DAX Measures](#dax-measures)
- [Key Definitions](#key-definitions)
- [Repository Structure](#repository-structure)
- [How to Reproduce](#how-to-reproduce)
- [Tools Used](#tools-used)
- [Possible Next Steps](#possible-next-steps)

---

## Business Questions

| # | Question | Audience |
|---|---|---|
| 1 | **Is our profitability improving, and which states are driving it?** | Executives |
| 2 | **Which states are underperforming their peers?** | Regional / operations leaders |
| 3 | **Which department is driving the most profit, and which causes the most losses?** | Individual store owners |

---

## Dashboard

### Page 1: Profitability trend and top states

*Is our profitability improving, and which states are driving it?*

![Page 1: executive overview](docs/page1_executive_overview.png)

**What's on the page:** profit by year, margin % by year, a state slicer, and a state table (profit and margin %) with data bars on profit.

**Findings**

- **Profit is growing.** Chain-wide profit rose from **$449M in 2020 to $595M in 2026**, for a total of about **$3.67B** over the period at a **4.44%** overall margin.
- **Margin is improving, but not in a straight line.** It climbed from 4.29% (2020) to 4.58% (2026), with a dip to 4.32% in 2022 and a smaller one to 4.52% in 2025.
- **Three states drive about 30% of all profit.** Texas (about $433M), California (about $382M) and Florida (about $270M) together contribute roughly 29.5% of total profit.
- **Size and efficiency differ.** Texas earns the highest margin of the large states (4.71%), while New York, a large state, runs a thin 3.85%. Tennessee stands out with a 6.40% margin across 20 stores.

### Page 2: Underperforming states

*Which states are underperforming their peers?*

![Page 2: state performance](docs/page2_state_performance.png)

**What's on the page:** a table of states with their department margin, an **Underperformer Flag** (margin below 4.5%), and a count of store-years. A **Department** slicer and a **Year** slicer let you check performance for one department or one year. The screenshot shows Household & General Merchandise selected.

**Findings**

- **The flag is common.** 34 of the 50 states have an overall margin below the 4.5% threshold (the chain average is 4.44%), so the flag works best sorted from the worst states up, not as a simple yes/no.
- **Weakest margins:** Alaska (2.94%), Iowa (3.28%), Minnesota (3.37%), Utah (3.43%) and New Hampshire (3.57%). Several of these states have few stores (Alaska has a single store), so their margins are volatile.
- **The most important underperformer is New York.** At 3.85% across 55 stores it is far below the chain average and large enough to move the total.
- **Strongest margins:** North Dakota (7.17%), Tennessee (6.40%), South Dakota (5.63%), Arizona (5.41%) and Nebraska (5.28%).
- **Department changes the picture.** With Household & General Merchandise selected, Tennessee (7.08%), North Dakota (6.70%) and South Dakota (6.39%) lead, and the chain-wide margin for that department is 4.51%.

### Page 3: Department profit and losses

*Which department is driving the most profit, and which causes the most losses?*

![Page 3: department profit](docs/page3_department_profit.png)

**What's on the page:** four slicers (**State**, **Store_ID**, **Department**, **Year**) so a store owner can pick their own store, a profit-by-year chart, and a table of department profit and each department's **share of total profit**. With no store selected, the page shows the whole chain, as in the screenshot.

**Findings: most profit**

| Rank | Department | Profit | Share of profit |
|---|---|---|---|
| 1 | Dry Grocery | $807.1M | 21.98% |
| 2 | Meat & Seafood | $581.8M | 15.85% |
| 3 | Produce | $507.5M | 13.82% |
| ... | ... | ... | ... |
| 9 | Health & Beauty | $181.5M | 4.94% |
| 10 | Bakery | $150.4M | 4.10% |

- **Dry Grocery drives the most profit**, about 22% of the total, and is the top department in roughly 6 out of 10 stores.
- **Profit follows size, not efficiency.** Each department's share of profit is almost identical to its share of revenue, and department margins all sit between about 4.4% and 4.5%. The biggest departments earn the most profit because they sell the most.

**Findings: losses**

At the chain level, **every department is profitable**. Losses show up only at the level of an individual store in an individual year, where a department's expenses exceed its revenue. Across the dataset that happens in about **7,500 of 70,000 department-store-years (10.7%)**, with total losses of about **$90M**.

| View | Department | Detail |
|---|---|---|
| Most frequent losses | **Bakery** | Loses money in 12.4% of store-years, and has the lowest total profit |
| Largest dollar losses | **Dry Grocery** | About $15.8M, but only because it is the biggest department (losses are about 2% of its profit, and it loses money least often, 8.1% of store-years) |
| Largest relative drag | **Bakery** | About $4.5M in losses against $150M in profit (about 3%) |

Bakery is the department most worth a closer look: it is the smallest profit contributor and the one most likely to lose money at a given store.

---

## Dataset

**Grain:** one row per store per year (1,000 stores x 7 years = **7,000 rows**).

| Column group | Description |
|---|---|
| `Store_ID`, `State`, `Year` | Store identifier (unique 5-digit integers), U.S. state (weighted roughly by population), reporting year (2020-2026) |
| `Total_Revenue`, `Total_Expenses` | Annual totals per store. Revenue between $1M and $50M, expenses between $1M and $45M |
| `<Department>_Revenue` | Revenue for each of 10 departments |
| `<Department>_Expenses` | Expenses for each of 10 departments |
| `*_Profit_Ratio` | Revenue / Expenses for the store and each department (added after generation) |

**Departments:** Produce, Meat & Seafood, Dairy & Eggs, Bakery, Deli & Prepared Foods, Frozen Foods, Dry Grocery, Beverages, Health & Beauty, Household & General Merchandise.

**Constraints enforced and validated**

- `Store_ID` + `Year` is unique, and every store has exactly 7 years of data
- Department revenue sums **exactly** to `Total_Revenue` for every store-year
- Department expenses sum **exactly** to `Total_Expenses` for every store-year
- Store-level expenses are below revenue in every row (margins roughly 1.5% to 10%)

**Realism features**

- Each store keeps its own size, cost ratio and department mix across all years, with small year-to-year noise
- Revenue drifts upward over time to mimic inflation
- Thin margins, like a real grocery chain, with some individual departments losing money in some store-years

The data is seeded (`seed = 42`), so it is fully reproducible with `generate_data.py`.

---

## Data Model

- **`supermarket_flat`**: the single fact table (wide format, one column per department metric)
- **`Departments`**: a small **disconnected** table with the 10 department names and a sort order

There are no relationships. Department revenue and expenses live in separate columns, so the department measures use a `SWITCH` on the selected value of `Departments[Department]` to pick the right column. One set of measures then works for any department, and a Department slicer can drive them.

**Important behavior:** only the `Dept` measures (`Dept Revenue`, `Dept Profit`, `Dept Margin %` and so on) react to the Department slicer. The store-level measures (`Total Revenue`, `Profit`, `Margin %`) always show the whole store. When no department is selected, the `Dept` measures fall back to totals.

---

## DAX Measures

The full code for every measure is in [`dax_measures.dax`](dax_measures.dax). Summary by question:

### Base measures

| Measure | Purpose |
|---|---|
| `Total Revenue`, `Total Expenses` | Sums of store revenue and expenses |
| `Profit` | Revenue minus expenses |
| `Margin %` | Profit divided by revenue |
| `Store Count` | Distinct stores (`DISTINCTCOUNT`) |

### Q1: Profitability trend

| Measure | Purpose |
|---|---|
| `Revenue PY`, `Expenses PY` | Prior-year values |
| `Revenue YoY %`, `Expense YoY %` | Year-over-year growth |
| `Margin Change (pp)` | Margin change in percentage points, first year to last |
| `Profit Share of Total %` | Each state's share of total profit |

### Q2: Underperforming states

| Measure | Purpose |
|---|---|
| `Dept Revenue`, `Dept Expenses` | Revenue and expenses for the selected department |
| `Dept Margin %` | Selected department's margin (falls back to overall margin) |
| `Underperformer Flag` | "Underperforming" when margin is below **4.5%** |

### Q3: Department profit and losses

| Measure | Purpose |
|---|---|
| `Dept Profit` | Department revenue minus expenses |
| `Dept Profit Share %` | Department's share of the store's profit |
| `Dept Profit Rank` | Rank of each department by profit |
| `Top Profit Department`, `Top Dept Profit Share %` | Name and share of the biggest profit driver |
| `Profit Share vs Revenue Share (pp)` | Whether a department earns more or less profit than its size suggests |
| `Dept Loss Amount` | Sum of negative department profit across store-years |
| `Dept Loss Count` | Number of department-store-years that lost money |
| `Largest Loss Department` | Department with the largest dollar losses |

Example of the department pattern:

```dax
Dept Profit = [Dept Revenue] - [Dept Expenses]

Dept Loss Amount =
VAR d = SELECTEDVALUE ( Departments[Department] )
RETURN
    SWITCH (
        d,
        "Produce", SUMX ( supermarket_flat, MIN ( 0, supermarket_flat[Produce_Revenue] - supermarket_flat[Produce_Expenses] ) ),
        -- ... one branch per department ...
        BLANK ()
    )
```

---

## Key Definitions

| Term | Definition |
|---|---|
| **Margin %** | (Revenue - Expenses) / Revenue |
| **Cost ratio** | Expenses / Revenue. Above 1 means the unit loses money |
| **Revenue to Expense Ratio** | Revenue / Expenses (the `*_Profit_Ratio` columns). Above 1 means profitable |
| **Profit share** | Department profit / total profit |
| **Loss** | A department-store-year where department expenses exceed department revenue |
| **pp** | Percentage points (a change from 4.2% to 4.6% is +0.4 pp) |

Ratios are always recomputed from summed revenue and expenses. Averaging the pre-computed ratio columns gives incorrect results.

---

## Repository Structure

```
.
├── README.md
├── dax_measures.dax                       # Every DAX measure, in creation order
├── generate_data.py                       # Synthetic data generator (seeded)
├── data/
│   ├── supermarket_Original_Data_V2.xlsx  # Flat table used by Power BI
│   ├── supermarket_flat.csv               # Same data, CSV
│   ├── stores.csv                         # Store dimension (Store_ID, State)
│   ├── store_financials.csv               # Store-year totals
│   ├── department_revenue.csv             # Long format: store, year, department, revenue
│   └── department_expenses.csv            # Long format: store, year, department, expenses
├── powerbi/
│   └── supermarket_report.pbix            # Power BI report
└── docs/
    ├── page1_executive_overview.png
    ├── page2_state_performance.png
    └── page3_department_profit.png
```

Adjust the paths to match your actual layout.

---

## How to Reproduce

1. **Generate the data (optional):**
   ```bash
   pip install numpy pandas
   python generate_data.py
   ```
   This writes the CSV files and runs validation checks on every constraint.
2. **Open Power BI Desktop** and load the flat table. Name it `supermarket_flat`, or update the table name in the DAX.
3. **Create the `Departments` table** (Modeling > New table) using the first block in `dax_measures.dax`. Sort `Department` by `Sort`, and do not create any relationship for it.
4. **Add the measures** from `dax_measures.dax`, one at a time (Modeling > New measure), in the order listed.
5. **Format** percentage measures as Percentage, dollar measures with a thousands separator, and set `Year` and `Store_ID` to "Don't summarize".
6. **Build the three pages** shown in the [Dashboard](#dashboard) section. For the department slicer on Page 3, use **Edit interactions** to keep the department chart showing all ten departments.

---

## Tools Used

- **Python** (NumPy, pandas) for synthetic data generation and validation
- **Excel / CSV** for the source data
- **Power BI Desktop** for modeling and visualization
- **DAX** for measures

---

## Possible Next Steps

- Add store-size tiers (small / medium / large) to compare stores against size-matched peers
- Compare each department's expense growth to its revenue growth (CAGR) to catch cost pressure early
- Make the underperformer threshold adjustable with a what-if parameter
- Add regional groupings (Northeast, South, Midwest, West) above state level
- Unpivot the department columns in Power Query to remove the `SWITCH` logic
- Load the data into BigQuery and connect Power BI to the warehouse

---

## Author

**[Grant Parker]** | [LinkedIn](https://www.linkedin.com/in/grant-parker-a119632b3/?isSelfProfile=true) | 
