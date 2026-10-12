# DA-05 · COVID-19 India Data Exploration

**Category:** Data Analytics · **Tools:** Power BI Desktop, Power Query, DAX · **Author:** Amal Lalu K K

![Dashboard preview](preview/dashboard_preview.png)

## 1. Business problem

State health teams want one place to compare how the three COVID-19 waves hit each state: case loads, deaths, testing and how quickly people were vaccinated.

**Questions this dashboard answers**

- How big was each wave and when did it peak?
- Which states had the highest cases per million?
- How did the case fatality rate change between waves?
- How far has each state got with full vaccination?

> **About the data:** **Simulated data** shaped like India's three waves (Sep 2020, May 2021, Jan 2022) for practising the techniques. Do not quote these numbers as real. For the real figures, use the Ministry of Health (MoHFW) / covid19india.org archive data and keep the same column layout.

## 2. What's in this folder

| Path | What it is |
|---|---|
| `data/dailycases.csv` | Fact table: one row per state per day (20,844 rows) |
| `data/vaccinations.csv` | One row per state per day from 16 Jan 2021 (start of the vaccine drive) (14,300 rows) |
| `data/states.csv` | One row per state (20 largest states) (20 rows) |
| `powerquery/*.m` | Power Query code for each table, ready to paste into the Advanced Editor |
| `dax/measures.dax` | Every DAX measure used in the report |
| `dax/date_table.dax` | The calendar (Date) table |
| `theme/AmalAnalytics_theme.json` | Report colour theme (colourblind-safe palette) |
| `preview/dashboard_preview.png` | What the finished dashboard looks like |
| `STEPS.html` | This guide, formatted to open in any browser |

## 3. Data dictionary

### DailyCases — `data/dailycases.csv`

Fact table: one row per state per day

| Column | Type | Description |
|---|---|---|
| `Date` | Date |  |
| `State` | Text |  |
| `NewCases` | Whole number | New confirmed cases reported that day |
| `NewRecovered` | Whole number | Recoveries reported that day |
| `NewDeaths` | Whole number | Deaths reported that day |
| `Tests` | Whole number | Tests carried out that day |

*Raw file contains deliberate data-quality issues to clean in Power Query: 20,844 raw rows → 20,638 clean rows.*

### Vaccinations — `data/vaccinations.csv`

One row per state per day from 16 Jan 2021 (start of the vaccine drive)

| Column | Type | Description |
|---|---|---|
| `Date` | Date |  |
| `State` | Text |  |
| `Dose1` | Whole number | First doses given that day |
| `Dose2` | Whole number | Second doses given that day |

### States — `data/states.csv`

One row per state (20 largest states)

| Column | Type | Description |
|---|---|---|
| `State` | Text |  |
| `Region` | Text |  |
| `Population` | Whole number | Approximate 2021 population |

## 4. Build it step by step

### Step 1 · Set up a new report

1. Copy this whole folder somewhere permanent, e.g. `C:\PowerBI\DA-05_COVID_19_India_Data_Exploration\`.
2. Open **Power BI Desktop** → **Blank report**.
3. **File → Options and settings → Options → Current file → Data Load** → untick **Auto date/time** (we use our own Date table). Also untick **Autodetect new relationships after data is loaded** under *Relationships*; you will create them yourself in a later step.
4. **View → Themes → Browse for themes** → pick `theme/AmalAnalytics_theme.json`.
5. **File → Save as** → `DA-05_COVID_19_India_Data_Exploration.pbix` in this folder. Save often (Ctrl+S).

### Step 2 · Create the folder path parameter

All queries read files relative to one parameter, so the report works on any PC.

1. **Home → Transform data** to open the Power Query Editor.
2. **Home → Manage Parameters → New Parameter**.
3. Name: `FolderPath` · Type: **Text** · Current Value: the full path of this folder **ending with a backslash**, e.g. `C:\PowerBI\DA-05_COVID_19_India_Data_Exploration\`.
4. Click **OK**.

### Step 3 · Load and clean the data in Power Query

Repeat for each table below: **Home → New Source → Blank Query** → **Home → Advanced Editor** → delete what's there → paste the code → **Done** → in *Query Settings* on the right, rename the query to the name shown.

#### Query `DailyCases`  (file `powerquery/DailyCases.m`)

What it does:

- reads `data\dailycases.csv` and promotes the first row to headers
- sets the data type of every column (using the en-US locale so dates and decimals parse the same on any PC)
- trims stray spaces in `State`
- fixes inconsistent capitalisation (e.g. `north`, `NORTH` → `North`) in `State`
- removes incomplete rows where `NewCases` is blank
- removes exact duplicate rows

```powerquery
let
    Source = Csv.Document(File.Contents(FolderPath & "data\dailycases.csv"), [Delimiter=",", Encoding=65001, QuoteStyle=QuoteStyle.Csv]),
    #"Promoted Headers" = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers", {{"Date", type date}, {"State", type text}, {"NewCases", Int64.Type}, {"NewRecovered", Int64.Type}, {"NewDeaths", Int64.Type}, {"Tests", Int64.Type}}, "en-US"),
    #"Trimmed Text" = Table.TransformColumns(#"Changed Type", {{"State", Text.Trim, type text}}),
    #"Capitalized Each Word" = Table.TransformColumns(#"Trimmed Text", {{"State", Text.Proper, type text}}),
    #"Removed Blank NewCases" = Table.SelectRows(#"Capitalized Each Word", each [NewCases] <> null),
    #"Removed Duplicates" = Table.Distinct(#"Removed Blank NewCases")
in
    #"Removed Duplicates"
```

#### Query `Vaccinations`  (file `powerquery/Vaccinations.m`)

What it does:

- reads `data\vaccinations.csv` and promotes the first row to headers
- sets the data type of every column (using the en-US locale so dates and decimals parse the same on any PC)

```powerquery
let
    Source = Csv.Document(File.Contents(FolderPath & "data\vaccinations.csv"), [Delimiter=",", Encoding=65001, QuoteStyle=QuoteStyle.Csv]),
    #"Promoted Headers" = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers", {{"Date", type date}, {"State", type text}, {"Dose1", Int64.Type}, {"Dose2", Int64.Type}}, "en-US")
in
    #"Changed Type"
```

#### Query `States`  (file `powerquery/States.m`)

What it does:

- reads `data\states.csv` and promotes the first row to headers
- sets the data type of every column (using the en-US locale so dates and decimals parse the same on any PC)

```powerquery
let
    Source = Csv.Document(File.Contents(FolderPath & "data\states.csv"), [Delimiter=",", Encoding=65001, QuoteStyle=QuoteStyle.Csv]),
    #"Promoted Headers" = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),
    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers", {{"State", type text}, {"Region", type text}, {"Population", Int64.Type}}, "en-US")
in
    #"Changed Type"
```

When all queries show data, click **Home → Close & Apply**.

### Step 4 · Create the Date table

1. **Modeling → New table** and paste:

```dax
Date =
ADDCOLUMNS (
    CALENDAR ( DATE ( 2020, 3, 1 ), DATE ( 2022, 12, 31 ) ),
    "Year", YEAR ( [Date] ),
    "Quarter", "Q" & QUARTER ( [Date] ),
    "Month Num", MONTH ( [Date] ),
    "Month", FORMAT ( [Date], "MMM" ),
    "Year-Month", FORMAT ( [Date], "YYYY-MM" ),
    "Month Start", DATE ( YEAR ( [Date] ), MONTH ( [Date] ), 1 ),
    "Weekday Num", WEEKDAY ( [Date], 2 ),
    "Weekday", FORMAT ( [Date], "ddd" )
)
```

2. With the table selected: **Table tools → Mark as date table** → choose the `Date` column.
3. Select the `Month` column → **Column tools → Sort by column → Month Num**. Do the same for `Weekday` → **Weekday Num**.

### Step 5 · Create the relationships (Model view)

Open **Model view** (left sidebar). For each row, drag the first column onto the second, or use **Home → Manage relationships → New**. Delete any relationship Power BI created that is not in this list.

| From (many side) | To (one side) | Cardinality | Cross-filter |
|---|---|---|---|
| `DailyCases[State]` | `States[State]` | Many-to-one (*:1) | Single |
| `Vaccinations[State]` | `States[State]` | Many-to-one (*:1) | Single |
| `DailyCases[Date]` | `Date[Date]` | Many-to-one (*:1) | Single |
| `Vaccinations[Date]` | `Date[Date]` | Many-to-one (*:1) | Single |

Result: a star schema with the fact table(s) in the middle and lookup tables around them.

### Step 6 · Create the measures

1. **Home → Enter data** → name the table `_Measures` → **Load**. (A dedicated table keeps all measures in one place.)
2. Select `_Measures` in the Data pane → **Home → New measure**, paste one measure, press Enter. Repeat for each.
3. After each one, set the format in **Measure tools → Format** as listed.
4. Once you have measures, delete the empty `Column1` from `_Measures`; the table moves to the top of the Data pane.

| Measure | Format | What it means |
|---|---|---|
| `Total Cases` | Whole number, thousands separator | Confirmed cases |
| `Total Deaths` | Whole number, thousands separator | Deaths |
| `Total Recovered` | Whole number, thousands separator | Recoveries |
| `Total Tests` | Whole number, thousands separator | Tests done |
| `Case Fatality Rate` | Percentage, 2 decimals | Deaths ÷ cases |
| `Test Positivity Rate` | Percentage, 1 decimal | Cases ÷ tests |
| `Cases per Million` | Whole number, thousands separator | Cases per million people |
| `Cases 7-Day Avg` | Whole number, thousands separator | Rolling 7-day average of daily cases (use on a chart by Date) |
| `Doses Administered` | Whole number, thousands separator | First + second doses |
| `Fully Vaccinated %` | Percentage, 1 decimal | Second doses ÷ population |

```dax
Total Cases =
    SUM ( DailyCases[NewCases] )

Total Deaths =
    SUM ( DailyCases[NewDeaths] )

Total Recovered =
    SUM ( DailyCases[NewRecovered] )

Total Tests =
    SUM ( DailyCases[Tests] )

Case Fatality Rate =
    DIVIDE ( [Total Deaths], [Total Cases] )

Test Positivity Rate =
    DIVIDE ( [Total Cases], [Total Tests] )

Cases per Million =
    DIVIDE ( [Total Cases], SUM ( States[Population] ) ) * 1000000

Cases 7-Day Avg =
    AVERAGEX (
        DATESINPERIOD ( 'Date'[Date], MAX ( 'Date'[Date] ), -7, DAY ),
        [Total Cases]
    )

Doses Administered =
    SUM ( Vaccinations[Dose1] ) + SUM ( Vaccinations[Dose2] )

Fully Vaccinated % =
    DIVIDE ( SUM ( Vaccinations[Dose2] ), SUM ( States[Population] ) )
```

### Step 7 · Build the report pages

Set the page size first: **View → Page view → Fit to page**, and in **Format page → Canvas settings** use 16:9. Add a text box at the top of each page with the page title.

#### Page: National Picture

| # | Visual | Fields to drag in | Settings |
|---|---|---|---|
| 1 | Card (x5) | Total Cases · Total Deaths · Case Fatality Rate · Test Positivity Rate · Fully Vaccinated % |  |
| 2 | Line chart | X-axis: Date[Date] · Y-axis: Cases 7-Day Avg | Shows the three waves clearly; turn off the X-axis concatenation |
| 3 | Clustered column chart | X-axis: Date[Year] · Y-axis: Case Fatality Rate | Deaths per case dropped sharply in the Omicron wave |
| 4 | Slicers | Date[Date] (Between style) · States[Region] |  |

#### Page: State Comparison

| # | Visual | Fields to drag in | Settings |
|---|---|---|---|
| 1 | Filled map or Shape map | Location: States[State] · Color saturation: Cases per Million | If the map is disabled by your admin, use a bar chart instead |
| 2 | Clustered bar chart | Y-axis: States[State] · X-axis: Cases per Million | Top N filter = 10 |
| 3 | Table | States[State], Total Cases, Total Deaths, Case Fatality Rate, Cases per Million, Fully Vaccinated % | Data bars on Fully Vaccinated % |

#### Page: Vaccination

| # | Visual | Fields to drag in | Settings |
|---|---|---|---|
| 1 | Line chart | X-axis: Date[Year-Month] · Y-axis: Doses Administered |  |
| 2 | Clustered bar chart | Y-axis: States[State] · X-axis: Fully Vaccinated % | Add a constant line at 70% |

### Step 8 · Check your numbers

With **no slicers or filters applied**, your cards should show these values. If one is different, re-check that measure and the cleaning steps.

| Measure | Expected value |
|---|---|
| `Total Cases` | 39,364,934 |
| `Total Deaths` | 473,026 |
| `Total Recovered` | 38,890,379 |
| `Total Tests` | 500,766,601 |
| `Case Fatality Rate` | 1.20% |
| `Test Positivity Rate` | 7.9% |
| `Cases per Million` | 29,629 |
| `Cases 7-Day Avg` | 683 (last 7 days of data, 25–31 Dec 2022) |
| `Doses Administered` | 2,047,071,737 |
| `Fully Vaccinated %` | 70.5% |

### Step 9 · Save and share

1. **File → Save** — your finished `DA-05_COVID_19_India_Data_Exploration.pbix`.
2. **File → Export → Export to PDF** for a static copy, or take a screenshot of each page for your portfolio and GitHub.
3. Optional: **Home → Publish** to Power BI Service (needs a work or school account) to get a shareable link.

## 5. Key findings from this data

- **Cases per million, top 10 states:** highest is Kerala (125,472), lowest is Uttarakhand (29,850).
- **Case fatality rate by year:** highest is 2020 (1.76%), lowest is 2022 (0.21%).
- **Weekly new cases, all 20 states (simulated):** peaks at 2021-05 (2,525,105); latest value 5,843.

## 6. Skills shown

- DAX rolling averages with DATESINPERIOD
- Two fact tables sharing State and Date dimensions
- Ratio measures (CFR, positivity, per-million)
- Maps and time-series storytelling
