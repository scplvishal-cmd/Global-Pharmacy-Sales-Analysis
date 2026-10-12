# Pharmacy Sales 2020-2025: Power BI Build Guide

A .pbix cannot be written without Power BI Desktop, so this guide plus `Pharmacy_Sales_PowerQuery.pq` and `Pharmacy_Sales_Measures.dax` reproduce the file exactly. `req.docx` never arrived, so the scope is a standard sales-performance report on the CSV; see the Word document for the assumptions.

## 1. Build steps

1. **Load data.** Open Power BI Desktop, then Home > Transform data. Create the `FilePath` parameter and each query from `Pharmacy_Sales_PowerQuery.pq`, in the order listed. Switch Enable load OFF for `FilePath`, `Raw` and `Sales_Clean`. Close & Apply.
2. **Create `Dim_Date`.** Modeling > New table, paste the DAX from the `.dax` file. Table tools > Mark as date table > `Date`. Sort `Month Name` by `Month Number`.
3. **Set sort orders.** `Dim_AgeGroup[Age Group]` by `Age Group Sort`; `Fact_Sales[Expiry Band]` by `Expiry Band Sort`. Hide the sort columns.
4. **Relationships** (Model view, all many-to-one, single direction, filter flows dimension to fact):

| From (Fact_Sales) | To |
|---|---|
| Date | Dim_Date[Date] |
| Country | Dim_Geography[Country] |
| Medicine | Dim_Medicine[Medicine] |
| Age Group | Dim_AgeGroup[Age Group] |
| Covid Flag | Dim_Period[Covid Flag] |

   `DQ_Log` stays unrelated.
5. **Measures.** Add every measure from the `.dax` file to `Fact_Sales`, then set the Display folders named in its section headers. Hide raw numeric columns (`Units Sold`, `Unit Price`, `Revenue`, `Stock Level`, `Days to Expiry`) so report authors use measures.
6. **Formats.** Total Revenue: `#,0.0,," M"`. Units: `#,0.0,," M"`. Avg Unit Price: `#,0.00`. Percent measures: `0.0%`. Median Stock Cover: `0.0`. Revenue has no currency symbol because the source states none.
7. **Page setup.** 16:9 canvas (1280 x 720). Name the pages exactly as below. Sync slicers S1 to S5 across pages 1 to 6 (View > Sync slicers), except where a page note says otherwise.

## 2. Pages and visuals

Visual IDs (P1-V1 etc.) are the ones the insights document cites.

### Page 1: Executive Dashboard (primary)
**Slicers (left or top strip):** S1 `Dim_Date[Year]` (tile), S2 `Dim_Geography[Region]`, S3 `Dim_Medicine[Category]`, S4 `Dim_AgeGroup[Age Group]`, S5 `Dim_Period[Period]`.

**KPI row (8 cards, top):**

| ID | Measure |
|---|---|
| P1-K1 | Total Revenue |
| P1-K2 | Total Units |
| P1-K3 | Avg Unit Price |
| P1-K4 | Records |
| P1-K5 | YoY Revenue % (latest year in view) |
| P1-K6 | COVID Demand Uplift % |
| P1-K7 | Low Cover % |
| P1-K8 | Near-Expiry % |

**Charts:**
- P1-V1: Line. X = `Dim_Date[Month Start]`, Y = Total Revenue.
- P1-V2: Clustered column. X = `Dim_Date[Year]`, Y = Total Revenue; tooltip = Revenue YoY %.
- P1-V3: Bar. Y = `Dim_Geography[Region]`, X = Total Revenue (sort descending).
- P1-V4: Clustered column. X = `Dim_Date[Month Name]`, Y = Avg Units per Record, Legend = `Dim_Period[Period]`. Clear S5 on this page so both periods show.

### Page 2: Time & Seasonality
- P2-V1: Line, `Month Start` x Total Revenue.
- P2-V2: Line, X = `Month Name`, Legend = `Year`, Y = Avg Units per Record.
- P2-V3: Matrix. Rows = Year, Columns = Month Name, Values = Total Revenue; heat-map conditional formatting.
- P2-V4: Table. Year, Total Revenue, Revenue YoY %, Total Units, Avg Units per Record, Records.
- P2-V5: Column. X = `Month Name`, Y = Seasonality Index. Page-level filter: Period = Post-COVID.

### Page 3: Geography
- P3-V1: Bar, Region x Total Revenue.
- P3-V2: Bar, Country x Total Revenue (descending).
- P3-V3: Bar, Country x Revenue per Record (axis starts at 0).
- P3-V4: Table. Region, Country, Records, Total Revenue, Revenue Share %, Revenue per Record, Avg Units per Record.
- Add a text box: "Country totals follow record counts; compare on Revenue per Record."

### Page 4: Product & Demographics
- P4-V1: Bar, Category x Total Revenue.
- P4-V2: Bar, Medicine x Total Revenue, Legend = Category.
- P4-V3: Column, Age Group (sorted by `Age Group Sort`) x Total Revenue.
- P4-V4: Matrix. Rows = Medicine, Columns = Age Group, Values = Avg Units per Record.
- P4-V5: Matrix. Rows = Category, Columns = Year, Values = Total Revenue.

### Page 5: COVID Impact (do not use S5 here; measures set the period themselves)
- P5-K1 to K6: cards for Avg Units per Record - COVID, Avg Units per Record - Post, COVID Demand Uplift %, Total Revenue - COVID, Total Revenue - Post, High-Volume Records.
- P5-V2: Clustered column. X = Category, Y = Avg Units per Record - COVID and - Post.
- P5-V3: Line. X = `Month Start`, Y = Avg Units per Record (shows the step at Jan 2022).
- P5-V4: Clustered column. X = Region, Y = the same two measures.
- P5-V5: Column. X = Year, Y = High-Volume Records; line = High-Volume Unit Share %.

### Page 6: Inventory & Expiry
- P6-K1 to K7: cards for Avg Stock Level, Median Stock Cover, Stock-out Records, Oversold Records, Low Cover %, Near-Expiry %, Near-Expiry Revenue.
- P6-V1: Line and column. X = Month Name, columns = Avg Units per Record, line = Avg Stock Level (secondary axis).
- P6-V2: Column. X = Month Name, Y = Oversold Records, Legend = Period.
- P6-V3: Column. X = Expiry Band, Y = Records (and Total Revenue as tooltip).
- P6-V4: Table. Category, Medicine, Avg Stock Level, Median Stock Cover, Stock-out Records, Oversold Records, Low Cover %, Near-Expiry %.

### Page 7: Data Quality & Reconciliation
- P7-V1: Table of `DQ_Log` (Check, Rows).
- P7-V2: Table. Year, Records, Total Units, Total Revenue; compare with section 3.
- Add a text box listing limitations (currency unstated, record-coverage differences, stock-to-sales relationship, price uniformity).

## 3. Reconciliation checks (clean data, after removing duplicates)

All pages must agree with these figures when no slicer is applied.

| Check | Expected |
|---|---|
| Rows in CSV / duplicates removed / loaded | 177,990 / 2,630 / 175,360 |
| Total Units | 78,534,584 |
| Total Revenue | 4,127,654,676 |
| Avg Unit Price (weighted) | 52.56 |
| Records by year 2020 to 2025 | 29,280 / 29,200 / 29,200 / 29,200 / 29,280 / 29,200 |
| Units by year 2020 to 2025 | 17,531,372 / 17,427,651 / 10,848,671 / 10,913,010 / 10,934,638 / 10,879,242 |
| Revenue by year (M) | 922.76 / 919.46 / 566.98 / 575.20 / 573.96 / 569.30 |
| COVID vs Post avg units per record | 597.8 / 372.8 (uplift 60.3%) |
| Stock-out / Oversold / Low Cover records | 128 / 967 / 9,263 |
| Near-Expiry records | 15,834 (9.0%) |
| High-Volume records | 661 |

The revenue sum on Page 1 K1, P2-V4, P3-V4 and P4-V5 must equal 4,127.65 M. If a number is off, check the relationships and that `Fact_Sales` has 175,360 rows.

## 4. Formatting suggestions
Use one accent colour for the COVID period and a neutral for post-COVID, applied consistently on P1-V4, P5-V2 to V4 and P6-V2. Add alt text to each visual. Set the page title as a text box with the page name and the active period.
