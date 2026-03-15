# 📊 MAXAB RETAIL PERFORMANCE & CHURN ANALYTICS DASHBOARD

<div align="center">

![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-0078D4?style=for-the-badge&logo=microsoft&logoColor=white)
![SQL Server](https://img.shields.io/badge/SQL_Server-CC292B?style=for-the-badge&logo=microsoft-sql-server&logoColor=white)
![Star Schema](https://img.shields.io/badge/Data_Model-Star_Schema-4CAF50?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-blue?style=for-the-badge)

**Real-time insights into retailer behavior, operational performance, and customer lifecycle across the Maxab distribution network in Egypt.**

</div>

---

## 📌 Project Snapshot

- **Version:** 1.0  
- **Last Updated:** February 2026  
- **Dashboard Pages:** 6 (Home + 5 Analysis Pages)  
- **Total Measures:** 75+ DAX Measures  
- **Data Model:** Star Schema  
- **Refresh Frequency:** Daily at 6:00 AM  

---

## 📋 TABLE OF CONTENTS

1. [Executive Summary](#executive-summary)
2. [Project Overview](#project-overview)
3. [Data Model Architecture](#data-model-architecture)
4. [Dashboard Pages](#dashboard-pages)
5. [DAX Measures Library](#dax-measures-library)
6. [Implementation Guide](#implementation-guide)
7. [Maintenance & Operations](#maintenance--operations)
8. [Troubleshooting](#troubleshooting)
9. [Appendices](#appendices)

---

## 1. EXECUTIVE SUMMARY

### Purpose
The Maxab Retail Performance & Churn Analytics Dashboard provides real-time insights into retailer behavior, operational performance, and customer lifecycle across the Maxab distribution network in Egypt.

### Key Capabilities
- **Executive Overview:** YTD performance tracking with MoM comparisons
- **Delivery Analytics:** SLA monitoring with dynamic thresholds
- **Churn Analysis:** Cohort retention tracking and activation metrics
- **Drop Investigation:** Week-over-week sales decomposition (May 2025 focus)
- **Commercial Insights:** Juice category NMV analysis and brand mix

### Target Audience
- C-suite executives (high-level KPIs)
- Operations managers (delivery SLA, failure rates)
- Commercial teams (category performance, brand trends)
- Retention analysts (churn patterns, cohort behavior)

### Business Impact
- **Reduced Churn:** Identify at-risk segments before they leave
- **Improved SLA:** Real-time delivery performance monitoring
- **Revenue Protection:** Early detection of sales drops
- **Strategic Planning:** Data-driven decisions on category investments

---

## 2. PROJECT OVERVIEW

### 2.1 Business Context

**Client:** Maxab — B2B food & grocery distribution platform (Egypt)  
**Challenge:** High retailer churn (38.9%), delivery SLA misses, unexplained May 2025 sales drop  
**Solution:** Unified analytics dashboard combining performance, churn, and commercial metrics

### 2.2 Technical Stack

| Component | Technology |
|---|---|
| **BI Tool** | ![Power BI Desktop](https://img.shields.io/badge/Power_BI_Desktop-F2C811?style=flat-square&logo=powerbi&logoColor=black) (February 2026) |
| **Data Source** | ![SQL Server](https://img.shields.io/badge/SQL_Server-CC292B?style=flat-square&logo=microsoft-sql-server&logoColor=white) / ![CSV](https://img.shields.io/badge/CSV-0078D4?style=flat-square&logo=microsoftexcel&logoColor=white) |
| **Connectivity** | ⚡ DirectQuery + Import (hybrid) |
| **Refresh** | 🔄 Scheduled daily at 6:00 AM |
| **Compatibility** | 📱 Power BI Service, Mobile App |

### 2.3 Data Coverage

- **Date Range:** January 2024 — May 2024 (5 months)
- **Regions:** Cairo, Alexandria, Delta
- **Channels:** Online, Retail
- **Granularity:** Order line-level (SKU × Order × Date)
- **Row Count:** 500K+ order lines, 47K+ orders, 70K+ retailers

### 2.4 Dashboard Specifications

- **Canvas Size:** 1440×900 (16:10 aspect ratio, custom)
- **Theme:** Dark analytics theme (#0b0d0f background)
- **Pages:** 6 total (1 Home + 5 Analysis)
- **Visuals:** 45+ charts, cards, tables across all pages
- **Parameters:** 2 dynamic slicers (SLA threshold, Churn lookback window)

---

## 3. DATA MODEL ARCHITECTURE

### 3.1 Model Type: Star Schema

```
              Dim_Date
                 |
                 |
    Dim_Region --+-- Fact_Order -------- Dim_Product
                 |       |
    Dim_Channel -+       +-- Fact_OrderLines
                 |
              Dim_Brand
```

### 3.2 Fact Tables

#### Fact_Order (Primary Transaction Table)
**Grain:** One row per order  
**Rows:** 47,251

| Column | Type | Description |
|---|---|---|
| OrderId | Int | Primary key |
| RetailerID | Int | Foreign key to retailer dimension |
| Created | Date | Order creation timestamp |
| Delivered | Date | Delivery timestamp (nullable for failed orders) |
| Order Status | String | "Delivered" or "Failed" |
| Sales | Decimal | Gross order value (EGP) |
| NetSales | Decimal | Net order value after discounts |
| Discount Amount | Decimal | Total discounts applied |
| RegionID | Int | Foreign key to Dim_Region |
| ChannelID | Int | Foreign key to Dim_Channel |

**Relationships:**
- `Fact_Order[Created]` → `Dim_Date[Date]` (Many-to-One, Active)
- `Fact_Order[RegionID]` → `Dim_Region[RegionID]` (Many-to-One)
- `Fact_Order[ChannelID]` → `Dim_Channel[ChannelID]` (Many-to-One)

---

#### Fact_OrderLines (Line-Level Detail)
**Grain:** One row per SKU per order  
**Rows:** 500K+

| Column | Type | Description |
|---|---|---|
| OrderLineID | Int | Primary key |
| OrderId | Int | Foreign key to Fact_Order |
| SKU | String | Stock keeping unit code |
| OrderCreation | Date | Date order was created |
| Sales | Decimal | Line-level sales (EGP) |
| Quantity | Int | Units sold |

**Relationships:**
- `Fact_OrderLines[OrderId]` → `Fact_Order[OrderId]` (Many-to-One)
- `Fact_OrderLines[SKU]` → `Dim_Product[SKU]` (Many-to-One)
- `Fact_OrderLines[OrderCreation]` → `Dim_Date[Date]` (Many-to-One, Inactive — use Fact_Order[Created] instead)

---

#### Fact_JuiceNMV (Juice Category Sales)
**Grain:** One row per brand per date  
**Rows:** ~1,000

| Column | Type | Description |
|---|---|---|
| Date | Date | Transaction date |
| Brand | String | Juice brand name |
| Sales | Decimal | Net merchandise value (EGP) |

**Relationships:**
- `Fact_JuiceNMV[Date]` → `Dim_Date[Date]` (Many-to-One)
- `Fact_JuiceNMV[Brand]` → `Dim_Brand[Brand]` (Many-to-One)

---

#### Fact_MaxabNMV (Total Category Sales)
**Grain:** One row per date  
**Rows:** ~150

| Column | Type | Description |
|---|---|---|
| Date | Date | Transaction date |
| Total Sales | Decimal | Total NMV across all categories (EGP) |

**Relationships:**
- `Fact_MaxabNMV[Date]` → `Dim_Date[Date]` (Many-to-One)

---

### 3.3 Dimension Tables

#### Dim_Date (Calendar Table)
**Grain:** One row per day  
**Rows:** 365+ (full 2024-2025 calendar)

| Column | Type | Description |
|---|---|---|
| Date | Date | Primary key |
| Year | Int | 2024, 2025 |
| MonthNumber | Int | 1-12 |
| MonthName | String | "January", "February", etc. |
| YearMonth | String | "2024-01", "2024-02", etc. (sortable) |
| WeekOfMonth | Int | 1-5 (week number within month) |
| DayOfWeek | Int | 1=Monday, 7=Sunday |
| DayOfWeekName | String | "Monday", "Tuesday", etc. |
| IsWorkingDay | Boolean | TRUE for Mon-Fri, FALSE for weekends |
| Quarter | Int | 1-4 |
| QuarterName | String | "Q1 2024", "Q2 2024", etc. |

**Key Features:**
- Mark as Date Table: ✅ Enabled
- Working days calculated for SLA metrics
- YearMonth column enables month-level sorting

---

#### Dim_Region
**Grain:** One row per region  
**Rows:** 3

| Column | Type | Description |
|---|---|---|
| RegionID | Int | Primary key |
| Region | String | "Cairo", "Alexandria", "Delta" |
| Latitude | Decimal | Geographic coordinate |
| Longitude | Decimal | Geographic coordinate |

---

#### Dim_Channel
**Grain:** One row per sales channel  
**Rows:** 2

| Column | Type | Description |
|---|---|---|
| ChannelID | Int | Primary key |
| Channel | String | "Online", "Retail" |

---

#### Dim_Product
**Grain:** One row per SKU  
**Rows:** 500+

| Column | Type | Description |
|---|---|---|
| SKU | String | Primary key (e.g., "SKU-4821") |
| Category | String | Product category (Beverages, Snacks, etc.) |
| ProductName | String | Full product name |

---

#### Dim_Brand (Juice Brands)
**Grain:** One row per juice brand  
**Rows:** 8

| Column | Type | Description |
|---|---|---|
| Brand | String | Primary key ("Minute Maid", "Cappy", "Juhayna", etc.) |

---

### 3.4 Calculated Tables

#### Cohort_Matrix
**Purpose:** Scaffold for cohort retention analysis  
**Grain:** One row per cohort month × month offset combination  
**Rows:** 25 (5 cohort months × 5 offsets)

**Expression:**
```dax
Cohort_Matrix = 
CROSSJOIN(
    SELECTCOLUMNS(
        SUMMARIZE(
            ADDCOLUMNS(
                SUMMARIZE(ALL(Fact_Order), Fact_Order[RetailerID]),
                "FirstMonth", FORMAT(CALCULATE(MIN(Fact_Order[Created]), ALL(Dim_Date)), "YYYY-MM")
            ),
            [FirstMonth]
        ),
        "CohortMonth", [FirstMonth]
    ),
    SELECTCOLUMNS(
        GENERATESERIES(0, 4, 1),
        "MonthOffset", [Value]
    )
)
```

**Columns:**
- `CohortMonth` (String): "2024-01", "2024-02", etc.
- `MonthOffset` (Int): 0, 1, 2, 3, 4

**Usage:** Matrix visual rows=CohortMonth, columns=MonthOffset, values=[Cohort Retention %]

---

### 3.5 Parameters (Numeric Range Tables)

#### DeliveryThresholdDays
**Purpose:** Dynamic SLA threshold slicer  
**Values:** 1, 2, 3, 4, 5 working days  
**Default:** 2

#### ChurnLookbackMonths
**Purpose:** Churn definition window slicer  
**Values:** 1, 2, 3, 4, 5, 6 months  
**Default:** 3

---

### 3.6 Measure Table

**Name:** MeasureLibrary  
**Type:** Calculated table (no data, holds all measures)  
**Expression:** `MeasureLibrary = ROW("Placeholder", 1)`

**Purpose:** 
- Centralized measure storage
- Clean Fields pane organization
- No relationship to other tables (independent)

---

### 3.7 Relationship Summary

| From | To | Cardinality | Cross-Filter | Active |
|---|---|---|---|---|
| Fact_Order[Created] | Dim_Date[Date] | Many-to-One | Single | ✅ Yes |
| Fact_Order[RegionID] | Dim_Region[RegionID] | Many-to-One | Single | ✅ Yes |
| Fact_Order[ChannelID] | Dim_Channel[ChannelID] | Many-to-One | Single | ✅ Yes |
| Fact_OrderLines[OrderCreation] | Dim_Date[Date] | Many-to-One | Single | ❌ No (use Fact_Order[Created]) |
| Fact_OrderLines[SKU] | Dim_Product[SKU] | Many-to-One | Single | ✅ Yes |
| Fact_JuiceNMV[Date] | Dim_Date[Date] | Many-to-One | Single | ✅ Yes |
| Fact_JuiceNMV[Brand] | Dim_Brand[Brand] | Many-to-One | Single | ✅ Yes |
| Fact_MaxabNMV[Date] | Dim_Date[Date] | Many-to-One | Single | ✅ Yes |

**Total Relationships:** 8  
**Inactive Relationships:** 1 (Fact_OrderLines to Dim_Date — use USERELATIONSHIP if needed)

---

## 4. DASHBOARD PAGES

### 4.1 Page 00 — Home (Landing Page)

**Purpose:** Navigation hub and at-a-glance dashboard health check

**Layout:**
```
┌─────────────────────────────────────────────┐
│  MAXAB ANALYTICS              Last Refresh  │
├─────────────────────────────────────────────┤
│                                             │
│    RETAIL PERFORMANCE & CHURN ANALYTICS     │
│    Real-time insights into behavior         │
│                                             │
│  ┌──────┐ ┌──────┐ ┌──────┐                │
│  │ £4.2M│ │70.9K │ │38.9% │   ← Key Stats  │
│  └──────┘ └──────┘ └──────┘                │
│                                             │
│  🚨 CRITICAL ALERTS                         │
│  • Success Rate 84.8% (below target)        │
│  • Churn at 38.9% — Cairo worst             │
│                                             │
│  [01 Executive] [02 Delivery] [03 Churn]    │
│  [04 Drop]      [05 Commercial]             │
│                                             │
└─────────────────────────────────────────────┘
```

**Components:**

| Visual | Position | Field/Measure | Notes |
|---|---|---|---|
| Hero Title | X=300 Y=150 | "RETAIL PERFORMANCE & CHURN ANALYTICS" | Syne 36px Bold |
| Key Stat 1 | X=390 Y=290 | [Total Sales YTD] | Amber accent |
| Key Stat 2 | X=620 Y=290 | [Active Retailers] | Teal accent |
| Key Stat 3 | X=850 Y=290 | [Churn Rate %] | Red accent |
| Alert Banner | X=300 Y=410 | [Home Alert Text] | Dynamic, shows if issues |
| Nav Card 1 | X=300 Y=510 | "01 EXECUTIVE Overview" | Links to Page 1 |
| Nav Card 2 | X=574 Y=510 | "02 DELIVERY & SLA" | Links to Page 2 |
| Nav Card 3 | X=848 Y=510 | "03 CHURN Analysis" | Links to Page 3 |
| Nav Card 4 | X=1122 Y=510 | "04 DROP Analysis" | Links to Page 4 |
| Nav Card 5 | X=300 Y=674 | "05 COMMERCIAL Juice" | Links to Page 5 |
| Footer | X=900 Y=720 | [Last Refresh Info] + [Home Filter Context] | Right-aligned |

**Key Measures:**
- [Home Alert Text] — Dynamic alert generation
- [Nav Card P2 Stats] through [Nav Card P5 Stats] — Mini preview stats
- [Home Filter Context] — Shows current filter state

---

### 4.2 Page 01 — Executive Overview

**Purpose:** High-level YTD performance with MoM trend analysis

**Page-Level Filters:**
- `Dim_Date[YearMonth]` — Default: All (YTD view)
- `Dim_Region[Region]` — Default: All
- `Dim_Channel[Channel]` — Default: All

**Layout:**
```
┌─────────────────────────────────────────────────────┐
│ MAXAB Analytics          [Tabs: 01* 02 03 04 05]   │
├─────────────────────────────────────────────────────┤
│ [Filters]  RETAIL PERFORMANCE SUMMARY · YTD 2024    │
│            ─────────────────────────────────────    │
│  Period    ┌──────┐┌──────┐┌──────┐┌──────┐┌──────┐│
│  Region    │£4.2M ││47.2K ││70.9K ││£89.4 ││84.8% ││ KPIs
│  Channel   │Sales ││Orders││Retail││ AOV  ││Success│
│            └──────┘└──────┘└──────┘└──────┘└──────┘│
│            ┌───────────────────┐┌────────┐          │
│            │  Sales Trend      ││ Region │          │
│            │  Line Chart       ││ Bar    │          │
│            │                   ││ Chart  │          │
│            └───────────────────┘└────────┘          │
│            ┌───────────────────┐┌────────┐          │
│            │  Sales vs Success ││  Map   │          │
│            │  Combo Chart      ││ Visual │          │
│            │                   ││        │          │
│            └───────────────────┘└────────┘          │
└─────────────────────────────────────────────────────┘
```

**Visuals:**

| # | Visual Type | Position | Fields | Format Notes |
|---|---|---|---|---|
| 1 | New Card | X=216 Y=100 | [Total Sales YTD] | Amber top accent, MoM delta |
| 2 | New Card | X=443 Y=100 | [Total Orders] | Teal accent, MoM delta |
| 3 | New Card | X=670 Y=100 | [Total Retailers] | Blue accent, MoM delta |
| 4 | New Card | X=897 Y=100 | [AOV] | Green accent, MoM delta |
| 5 | New Card | X=1124 Y=100 | [Success Rate %] | Red accent, MoM pp delta |
| 6 | Line Chart | X=216 Y=208 | X: MonthName<br>Y: [Total Sales]<br>Y2: [AOV] (dashed) | Area fill #f5a623 15% opacity |
| 7 | Bar Chart | X=1048 Y=208 | Y: Region<br>X: [Total Sales] | Custom colors per region |
| 8 | Combo Chart | X=216 Y=446 | X: MonthName<br>Bars: [Total Orders]<br>Line: [Success Rate %] | Dual Y-axis |
| 9 | Map (or Regional Cards) | X=1048 Y=446 | Location: Region<br>Size: [Total Sales] | Azure Maps dark OR 3 regional cards |

**Key Measures (Display Folder: P1 — Executive Overview):**
- [Total Sales YTD], [Total Sales PM], [Total Sales MoM %]
- [Total Orders PM], [Total Orders MoM %]
- [Total Retailers PM], [Total Retailers MoM %]
- [AOV PM], [AOV MoM %]
- [Success Rate PM], [Success Rate MoM pp]
- [Net Sales], [Total Discount Amount]

**Conditional Formatting:**
- KPI delta labels: Font color via [CF Sales Color] (green for positive, red for negative)

---

### 4.3 Page 02 — Performance & Delivery

**Purpose:** Operational KPI monitoring with dynamic SLA threshold

**Page-Level Filters:**
- `Dim_Date[YearMonth]` — Default: All
- `Dim_Region[Region]` — Default: All
- `DeliveryThresholdDays` slicer — Default: 2 working days

**Layout:**
```
┌─────────────────────────────────────────────────────┐
│ MAXAB Analytics          [Tabs: 01 02* 03 04 05]   │
├─────────────────────────────────────────────────────┤
│ [Filters]  OPERATIONAL KPIs · SLA THRESHOLD: 2 DAYS │
│            ─────────────────────────────────────    │
│  Period    ┌──────┐┌──────┐┌──────┐┌──────┐        │
│  Region    │84.8% ││15.2% ││7,183 ││1.9d  ││ KPIs  │
│  SLA:  [2] │Success││Delay ││Delayed││Avg  │        │
│            └──────┘└──────┘└──────┘└──────┘        │
│            ┌───────────────────┐┌────────┐          │
│            │  Success vs Fail  ││ SLA    │          │
│            │  Stacked Bar      ││ Donut  │          │
│            │  by Region        ││ Chart  │          │
│            └───────────────────┘└────────┘          │
│            ┌─────────────────────────────┐          │
│            │  Delivery Heatmap           │          │
│            │  Day × Week Matrix          │          │
│            │  (Color = Avg Working Days) │          │
│            └─────────────────────────────┘          │
└─────────────────────────────────────────────────────┘
```

**Visuals:**

| # | Visual Type | Position | Fields | Format Notes |
|---|---|---|---|---|
| 1 | New Card | X=216 Y=100 | [Success Rate %] | Green/orange/red via [CF SLA Color] |
| 2 | New Card | X=519 Y=100 | [Delay Rate %] | Red accent |
| 3 | New Card | X=822 Y=100 | [Delayed Orders] | Amber accent |
| 4 | New Card | X=1125 Y=100 | [Delivery SLA (Working Days) Format] | Teal accent, "1.9d avg" |
| 5 | Stacked Bar | X=216 Y=208 | Y: Region<br>Values: [Orders Success] + [Orders Fail] | 100% stacked, green/red colors |
| 6 | Donut Chart | X=1048 Y=208 | Legend: "On Time" / "Late"<br>Values: [On Time Orders] + [Late Orders] | Amber/red, center label |
| 7 | Matrix | X=216 Y=466 | Rows: DayOfWeek<br>Cols: WeekOfMonth<br>Values: [Heatmap Avg Delivery] | Cell background CF via [CF Delivery Heatmap] |

**Parameter Slicer:**
- Visual: Numeric range slicer
- Field: `DeliveryThresholdDays[Delivery Threshold Days]`
- Position: Sidebar X=20 Y=360
- Style: Between slider, amber fill

**Key Measures (Display Folder: P2 — Performance & Delivery):**
- [Delay Rate %]
- [Delayed Orders] (alias for [Orders Fail])
- [On Time Orders], [Late Orders]
- [Delivery SLA (Working Days) Format]
- [Success Rate vs Target]
- [Heatmap Avg Delivery]

**Conditional Formatting:**
- Success Rate card: Background via [CF SLA Color]
- Heatmap cells: Background via [CF Delivery Heatmap] (6-tier gradient green→red)

---

### 4.4 Page 03 — Churn & Activation

**Purpose:** Retailer lifecycle analysis with cohort retention tracking

**Page-Level Filters:**
- `Dim_Date[YearMonth]` — Default: All (for activation metrics)
- `Dim_Region[Region]` — Default: All
- `ChurnLookbackMonths` slicer — Default: 3 months

**Layout:**
```
┌─────────────────────────────────────────────────────┐
│ MAXAB Analytics          [Tabs: 01 02 03* 04 05]   │
├─────────────────────────────────────────────────────┤
│ [Filters]  RETAILER LIFECYCLE · CHURN WINDOW: 3 MO  │
│            ─────────────────────────────────────    │
│  Period    ┌──────┐┌──────┐┌──────┐┌──────┐        │
│  Region    │12.8K ││38.9% ││2,340 ││+918  ││ KPIs  │
│  Churn:[3] │Active││Churn ││New   ││Net   │        │
│            └──────┘└──────┘└──────┘└──────┘        │
│            ┌───────────────────┐┌────────┐          │
│            │  Cohort Retention ││ Activ. │          │
│            │  Matrix           ││ Heatmap│          │
│            │  (CohortMonth×M+N)││        │          │
│            └───────────────────┘└────────┘          │
│            ┌─────────────────────────────┐          │
│            │  Churned Segment Table      │          │
│            │  Region|Channel|Churned|Rate│          │
│            └─────────────────────────────┘          │
└─────────────────────────────────────────────────────┘
```

**Visuals:**

| # | Visual Type | Position | Fields | Format Notes |
|---|---|---|---|---|
| 1 | New Card | X=216 Y=100 | [Active Retailers] | Teal accent |
| 2 | New Card | X=519 Y=100 | [Churn Rate %] | Red accent, CF via [CF Churn Color] |
| 3 | New Card | X=822 Y=100 | [Activation Retailers] | Green accent |
| 4 | New Card | X=1125 Y=100 | [Net Retailer Growth] | Purple accent, signed format |
| 5 | Matrix | X=216 Y=208 | Rows: Cohort_Matrix[CohortMonth]<br>Cols: Cohort_Matrix[MonthOffset]<br>Values: [Cohort Retention %] | Cell background CF via [CF Cohort Color] |
| 6 | Matrix | X=1048 Y=208 | Rows: Region<br>Cols: WeekOfMonth<br>Values: [Activation Retailers] | Gradient CF min=#111418 max=#f5a623 |
| 7 | Table | X=216 Y=506 | Cols: Region, Channel, Category, [Churned Retailers], [Churn Rate %] | Sort DESC by Churned, data bars on count |

**Parameter Slicer:**
- Visual: Numeric range slicer
- Field: `ChurnLookbackMonths[Churn Lookback Months]`
- Position: Sidebar X=20 Y=420
- Style: Between slider, purple fill

**Key Measures (Display Folder: P3 — Churn & Activation):**
- [Net Retailer Growth], [Retailer Retention Rate %], [Activation Rate %]
- [Cohort Retention %], [Cohort Size]

**Conditional Formatting:**
- Churn Rate card: Font color via [CF Churn Color]
- Cohort matrix cells: Background via [CF Cohort Color] (5-tier gradient)
- Table Churn Rate column: Font color via [CF Churn Color]

**Important Notes:**
- Cohort matrix will only show data for cohorts with enough elapsed time
- Jan 2024 cohort shows M+0 through M+4 (full retention curve)
- Feb-May cohorts may only show M+0 (100%) depending on current date

---

### 4.5 Page 04 — Drop Analysis

**Purpose:** Week-over-week sales decomposition for May 2025 investigation

**Page-Level Filters:**
- **HARDCODED:** `Dim_Date[YearMonth]` = "2024-05" (applies to all visuals)
- `Dim_Region[Region]` — Default: All
- `Dim_Product[Category]` — Default: All

**Layout:**
```
┌─────────────────────────────────────────────────────┐
│ MAXAB Analytics          [Tabs: 01 02 03 04* 05]   │
├─────────────────────────────────────────────────────┤
│ [Filters]  WEEKLY SALES DECOMPOSITION · MAY 2024    │
│            ─────────────────────────────────────    │
│  Period    ┌──────┐┌──────┐┌──────┐┌──────┐        │
│  Region    │£842K ││Wk 3  ││-22%  ││Juices││ KPIs  │
│  Category  │May   ││Lowest││Wk3vsWk2││Drop │        │
│            └──────┘└──────┘└──────┘└──────┘        │
│            ┌───────────────────┐┌────────┐          │
│            │  Weekly Sales     ││Category│          │
│            │  Line Chart       ││ Mix    │          │
│            │  (Wk1→Wk2→Wk3→Wk4)││ Change │          │
│            └───────────────────┘└────────┘          │
│            ┌─────────────────────────────┐          │
│            │  SKU Drop Table             │          │
│            │  SKU|Cat|Wk2|Wk3|Δ|Δ%       │          │
│            └─────────────────────────────┘          │
└─────────────────────────────────────────────────────┘
```

**Visuals:**

| # | Visual Type | Position | Fields | Format Notes |
|---|---|---|---|---|
| 1 | New Card | X=216 Y=100 | [May Wk1 Sales] + [May Wk2 Sales] + [May Wk3 Sales] + [May Wk4 Sales] (sum) | Amber accent, total May |
| 2 | New Card | X=519 Y=100 | [Lowest Week Label] | Red accent, shows "Week 3" |
| 3 | New Card | X=822 Y=100 | [May Wk3 vs Wk2 Delta %] | Red accent, "-22%" |
| 4 | New Card | X=1125 Y=100 | "Juices" or [SLA Threshold Label] | Purple accent |
| 5 | Line Chart | X=216 Y=208 | X: WeekOfMonth<br>Y: [Weekly Sales] | Analytics: Min line (red), Avg line (gray) |
| 6 | Bar Chart | X=1048 Y=208 | X: Category<br>Y: [May Wk2 Sales] + [May Wk3 Sales] | Paired bars, Top 6 filter |
| 7 | Table | X=216 Y=446 | Cols: SKU, Category, [SKU Sales Wk2], [SKU Sales Wk3], [SKU Drop Delta], [SKU Drop Delta %] | Sort ASC by Delta, Top 20, data bars |

**Key Measures (Display Folder: P4 — Drop Analysis):**
- [May Wk1 Sales] through [May Wk4 Sales]
- [May Wk3 vs Wk2 Delta], [May Wk3 vs Wk2 Delta %]
- [SKU Sales Wk2], [SKU Sales Wk3], [SKU Drop Delta], [SKU Drop Delta %]
- [Lowest Week Label]

**Conditional Formatting:**
- SKU Drop Delta column: Font color (red if negative, green if positive)
- SKU Drop Delta % column: Font color + data bars (red gradient)

**Important Note:**
- This page is **hardcoded to May 2024** (`Dim_Date[YearMonth] = "2024-05"`)
- Remove page-level filter if you want to analyze other months dynamically

---

### 4.6 Page 05 — Commercial / Juice

**Purpose:** Juice category NMV analysis and brand mix tracking

**Page-Level Filters:**
- `Dim_Date[YearMonth]` — Default: All (Jan-May 2024)
- No region/channel filters (category analysis is cross-region)

**Layout:**
```
┌─────────────────────────────────────────────────────┐
│ MAXAB Analytics          [Tabs: 01 02 03 04 05*]   │
├─────────────────────────────────────────────────────┤
│ [Filters]  NMV ANALYSIS · BRAND MIX · Jan–May 2024  │
│            ─────────────────────────────────────    │
│  Period    ┌──────────┐┌──────────┐┌──────────┐    │
│            │ £621K    ││ 14.8%    ││ 72.2%    ││KPIs│
│            │ Juice NMV││ Share    ││ Top 3    │    │
│            └──────────┘└──────────┘└──────────┘    │
│            ┌───────────────────┐┌────────┐          │
│            │  NMV Stacked      ││ Brand  │          │
│            │  Column Chart     ││Treemap │          │
│            │  (Other + Juice)  ││        │          │
│            └───────────────────┘└────────┘          │
│            ┌─────────────────────────────┐          │
│            │  Brand Small Multiples      │          │
│            │  (3×2 grid, 6 brands)       │          │
│            └─────────────────────────────┘          │
└─────────────────────────────────────────────────────┘
```

**Visuals:**

| # | Visual Type | Position | Fields | Format Notes |
|---|---|---|---|---|
| 1 | New Card | X=216 Y=100 | [Juice NMV] | Amber accent, MoM delta |
| 2 | New Card | X=597 Y=100 | [Juice Share %] | Teal accent, MoM pp delta |
| 3 | New Card | X=978 Y=100 | [Top 3 Brands NMV Share] | Purple accent, static |
| 4 | Stacked Column | X=216 Y=208 | X: MonthName<br>Y: [Other NMV] + [Juice NMV] | Data labels on Juice segment only |
| 5 | Treemap | X=1048 Y=208 | Group: Brand<br>Values: [Brand NMV] | Custom colors per brand |
| 6 | Small Multiples Line | X=216 Y=446 | X: MonthName<br>Y: [Brand NMV]<br>Small multiples: Brand | 3×2 grid layout |

**Key Measures (Display Folder: P5 — Commercial / Juice):**
- [Other NMV] (calculated as [Total NMV] - [Juice NMV])
- [Juice NMV PM], [Juice NMV MoM %]
- [Juice Share PM], [Juice Share MoM pp]
- [Top Brand Name], [Top 3 Brands NMV Share]
- [Brand NMV] (for treemap/small multiples)

**Color Palette:**
- Minute Maid: #f5a623 (amber)
- Cappy: #2dd4bf (teal)
- Juhayna: #60a5fa (blue)
- Beyti: #a78bfa (purple)
- Others: #2a3038 (gray)

---

## 5. DAX MEASURES LIBRARY

### 5.1 Measure Organization

**Total Measures:** 75+  
**Display Folders:** 12

| Folder Name | Count | Purpose |
|---|---|---|
| Performance KPIs | 7 | Core business metrics (Sales, Orders, AOV, Success Rate, SLA) |
| Success & Fail | 3 | Order status breakdown |
| Activation | 2 | New retailer tracking |
| Churn measures | 3 | Churn rate and churned retailer count |
| P1 — Executive Overview | 13 | YTD and MoM comparison measures |
| P2 — Performance & Delivery | 7 | Delivery SLA and on-time metrics |
| P3 — Churn & Activation | 5 | Cohort retention and net growth |
| P4 — Drop Analysis | 11 | Weekly sales and SKU-level drops |
| P5 — Commercial / Juice | 8 | Juice category and brand metrics |
| Tooltips | 3 | Multi-line tooltip text |
| Conditional Format Colors | 5 | Dynamic color hex values for CF |
| Page Subtitles | 5 | Dynamic page subtitle text |
| Parameters | 4 | Parameter slicer values and labels |
| Home Page | 6 | Landing page navigation stats |

---

### 5.2 Core Business Measures

#### Total Sales
```dax
Total Sales = SUM(Fact_Order[Sales])
```
**Description:** Gross revenue from all orders  
**Format:** £#,0 (currency with thousands separator)  
**Used in:** P1, P2, Home page

---

#### Total Orders
```dax
Total Orders = DISTINCTCOUNT(Fact_Order[OrderId])
```
**Description:** Count of unique orders  
**Format:** #,0 (integer)  
**Used in:** P1, P2

---

#### Total Retailers
```dax
Total Retailers = DISTINCTCOUNT(Fact_Order[RetailerID])
```
**Description:** Count of distinct retailers who placed orders  
**Format:** #,0 (integer)  
**Used in:** P1, P3, Home page

---

#### AOV (Average Order Value)
```dax
AOV = DIVIDE([Total Sales], [Total Orders])
```
**Description:** Average order value (Total Sales ÷ Total Orders)  
**Format:** #,0.0,, EGP (thousands with 1 decimal)  
**Used in:** P1

---

#### Success Rate %
```dax
Success Rate % = DIVIDE([Orders Success], [Total Orders], 0)
```
**Description:** Percentage of successfully delivered orders  
**Format:** 0.00% (percentage with 2 decimals)  
**Used in:** P1, P2, Home page

---

### 5.3 Time Intelligence Measures

#### Total Sales YTD
```dax
Total Sales YTD = 
CALCULATE([Total Sales], DATESYTD(Dim_Date[Date]))
```
**Description:** Year-to-date cumulative sales using DATESYTD  
**Format:** #,0.0,, EGP  
**Used in:** P1 KPI card

---

#### Total Sales PM (Prior Month)
```dax
Total Sales PM = 
CALCULATE([Total Sales], DATEADD(Dim_Date[Date], -1, MONTH))
```
**Description:** Sales from prior month for MoM comparison  
**Format:** #,0.0,, EGP  
**Used in:** [Total Sales MoM %] calculation

---

#### Total Sales MoM %
```dax
Total Sales MoM % = 
DIVIDE([Total Sales] - [Total Sales PM], [Total Sales PM], 0)
```
**Description:** Month-over-month percentage change in sales  
**Format:** +0.0%;-0.0%;0.0% (signed percentage)  
**Used in:** P1 KPI delta label

---

### 5.4 Delivery & SLA Measures

#### Delivery SLA (Working Days)
```dax
Delivery SLA (Working Days) = 
AVERAGEX(
    FILTER(Fact_Order, NOT ISBLANK(Fact_Order[Delivered])),
    CALCULATE(
        COUNTROWS(Dim_Date),
        DATESBETWEEN(Dim_Date[Date], Fact_Order[Created], Fact_Order[Delivered]),
        Dim_Date[IsWorkingDay] = 1
    )
)
```
**Description:** Average working-day delivery duration per order  
**Format:** 0 (integer days)  
**Used in:** P2 KPI card

---

#### On Time Orders
```dax
On Time Orders = 
VAR Threshold = [Delivery Threshold Days Value]
VAR DeliveredOrders =
    FILTER(Fact_Order, NOT ISBLANK(Fact_Order[Delivered]))
RETURN
COUNTROWS(
    FILTER(
        DeliveredOrders,
        CALCULATE(
            COUNTROWS(Dim_Date),
            DATESBETWEEN(Dim_Date[Date], Fact_Order[Created], Fact_Order[Delivered]),
            Dim_Date[IsWorkingDay] = 1
        ) <= Threshold
    )
)
```
**Description:** Count of orders delivered within dynamic SLA threshold  
**Format:** #,0 (integer)  
**Used in:** P2 donut chart

---

#### Heatmap Avg Delivery
```dax
Heatmap Avg Delivery = 
CALCULATE(
    AVERAGEX(
        FILTER(Fact_Order, NOT ISBLANK(Fact_Order[Delivered])),
        CALCULATE(
            COUNTROWS(Dim_Date),
            DATESBETWEEN(Dim_Date[Date], Fact_Order[Created], Fact_Order[Delivered]),
            Dim_Date[IsWorkingDay] = 1
        )
    )
)
```
**Description:** Average working-day delivery per DayOfWeek×WeekOfMonth cell  
**Format:** 0.0 (1 decimal)  
**Used in:** P2 heatmap matrix

---

### 5.5 Churn & Retention Measures

#### Churned Retailers
```dax
Churned Retailers = 
VAR LookbackMonths = [Churn Lookback Months Value]

// Use the LAST month in the current filter context
VAR MaxDateInContext = MAX(Dim_Date[Date])
VAR CurrentMonthStart = EOMONTH(MaxDateInContext, -1) + 1
VAR CurrentMonthEnd = EOMONTH(MaxDateInContext, 0)

// Prior N months window (BEFORE the current month)
VAR PriorWindowStart = EOMONTH(CurrentMonthStart, -LookbackMonths) + 1
VAR PriorWindowEnd = CurrentMonthStart - 1

// Retailers active in the PRIOR window
VAR RetailersPrior =
    CALCULATETABLE(
        VALUES(Fact_Order[RetailerID]),
        FILTER(
            ALL(Dim_Date),
            Dim_Date[Date] >= PriorWindowStart &&
            Dim_Date[Date] <= PriorWindowEnd
        )
    )

// Retailers active in the CURRENT month
VAR RetailersCurrent =
    CALCULATETABLE(
        VALUES(Fact_Order[RetailerID]),
        FILTER(
            ALL(Dim_Date),
            Dim_Date[Date] >= CurrentMonthStart &&
            Dim_Date[Date] <= CurrentMonthEnd
        )
    )

// Churned = those in prior window but NOT in current month
VAR Churned = EXCEPT(RetailersPrior, RetailersCurrent)

RETURN
COUNTROWS(Churned)
```
**Description:** Count of retailers active in prior N months but absent in latest month  
**Format:** #,0 (integer)  
**Used in:** P3 KPI card, churn table

---

#### Churn Rate %
```dax
Churn Rate % = 
DIVIDE([Churned Retailers], [Active Retailers], 0)
```
**Description:** Churned retailers as percentage of active base  
**Format:** 0.00% (percentage)  
**Used in:** P3 KPI card, Home page

---

#### Cohort Retention %
```dax
Cohort Retention % = 
VAR CohortMonth = SELECTEDVALUE('Cohort_Matrix'[CohortMonth])
VAR MonthOffset = SELECTEDVALUE('Cohort_Matrix'[MonthOffset])

// Error handling: return BLANK if no selection
VAR HasSelection = NOT ISBLANK(CohortMonth) && NOT ISBLANK(MonthOffset)

RETURN
IF(
    NOT HasSelection,
    BLANK(),
    
    // Main logic
    VAR CohortStart = DATEVALUE(CohortMonth & "-01")
    VAR CohortEnd   = EOMONTH(CohortStart, 0)
    
    VAR OffsetStart = EOMONTH(CohortStart, MonthOffset - 1) + 1
    VAR OffsetEnd   = EOMONTH(CohortStart, MonthOffset)
    
    -- Get retailers whose FIRST order is inside cohort month
    VAR RetailersInCohort =
        FILTER(
            VALUES(Fact_Order[RetailerID]),
            VAR FirstOrderDate =
                CALCULATE(
                    MIN(Fact_Order[Created]),
                    ALL(Fact_Order)
                )
            RETURN
                FirstOrderDate >= CohortStart &&
                FirstOrderDate <= CohortEnd
        )
    
    -- Retailers active in offset month
    VAR RetainedInOffset =
        CALCULATETABLE(
            VALUES(Fact_Order[RetailerID]),
            Fact_Order[Created] >= OffsetStart,
            Fact_Order[Created] <= OffsetEnd
        )
    
    -- Intersection
    VAR Retained =
        INTERSECT(
            RetailersInCohort,
            RetainedInOffset
        )
    
    RETURN
    IF(
        MonthOffset = 0,
        1,
        DIVIDE(
            COUNTROWS(Retained),
            COUNTROWS(RetailersInCohort)
        )
    )
)
```
**Description:** Cohort retention rate for Matrix visual (rows=CohortMonth, cols=MonthOffset)  
**Format:** 0% (percentage, no decimals)  
**Used in:** P3 cohort matrix

---

### 5.6 Conditional Formatting Measures

#### CF Sales Color
```dax
CF Sales Color = 
VAR Delta = [Total Sales MoM %]
RETURN
SWITCH(TRUE(),
    Delta > 0.05,  "#4ade80",  -- Green: >5% growth
    Delta > 0,     "#a3e635",  -- Lime: 0-5% growth
    Delta > -0.05, "#fb923c",  -- Orange: 0 to -5% decline
    "#f87171"                   -- Red: <-5% decline
)
```
**Description:** Returns hex color for sales delta KPI cards  
**Format:** Text (hex color code)  
**Used in:** P1 KPI card conditional formatting

---

#### CF Cohort Color
```dax
CF Cohort Color = 
VAR Rate = [Cohort Retention %]
RETURN
SWITCH(TRUE(),
    ISBLANK(Rate), "#111418",  -- Dark gray for blank cells
    Rate >= 0.75,  "#166534",  -- Dark green: ≥75% retention
    Rate >= 0.60,  "#4ade80",  -- Green: 60-75%
    Rate >= 0.45,  "#ca8a04",  -- Amber: 45-60%
    Rate >= 0.30,  "#f97316",  -- Orange: 30-45%
    "#dc2626"                   -- Red: <30%
)
```
**Description:** Returns hex color for cohort retention matrix cells  
**Format:** Text (hex color code)  
**Used in:** P3 cohort matrix background conditional formatting

---

### 5.7 Home Page Measures

#### Home Alert Text
```dax
Home Alert Text = 
VAR Alert1_SLA = IF([Success Rate %] < 0.86, "• Success Rate at " & FORMAT([Success Rate %], "0.0%") & " (below 86% target)" & UNICHAR(10), "")
VAR Alert2_Churn = IF([Churn Rate %] > 0.30, "• Churn at " & FORMAT([Churn Rate %], "0.0%") & " — Cairo Online worst segment" & UNICHAR(10), "")
VAR Alert3_Drop = IF([May Wk3 vs Wk2 Delta %] < -0.15, "• May Wk3 sales dropped " & FORMAT([May Wk3 vs Wk2 Delta %], "0%") & " vs Wk2", "")

VAR AllAlerts = Alert1_SLA & Alert2_Churn & Alert3_Drop

RETURN
IF(
    LEN(AllAlerts) > 0,
    AllAlerts,
    "✓ All metrics within target thresholds"
)
```
**Description:** Dynamic alert banner text — shows critical issues or "all good" message  
**Format:** Text (multi-line with UNICHAR(10) line breaks)  
**Used in:** Home page alert banner

---

#### Nav Card P2 Stats
```dax
Nav Card P2 Stats = 
"SLA: " & FORMAT([SLA Status], "0.0%") & " · Avg: " & FORMAT([Delivery SLA (Working Days)], "0.0") & "d"
```
**Description:** Mini preview stat for Page 2 navigation card  
**Format:** Text ("SLA: 84.8% · Avg: 1.9d")  
**Used in:** Home page navigation card 2

---

### 5.8 Parameter Measures

#### Delivery Threshold Days Value
```dax
Delivery Threshold Days Value = 
SELECTEDVALUE('DeliveryThresholdDays'[Delivery Threshold Days], 2)
```
**Description:** Returns selected SLA threshold from slicer, default 2 days  
**Format:** 0" days" (integer with "days" suffix)  
**Used in:** All SLA-related measures

---

#### Churn Lookback Months Value
```dax
Churn Lookback Months Value = 
SELECTEDVALUE('ChurnLookbackMonths'[Churn Lookback Months], 3)
```
**Description:** Returns selected churn lookback window from slicer, default 3 months  
**Format:** 0" months" (integer with "months" suffix)  
**Used in:** [Churned Retailers] measure

---

## 6. IMPLEMENTATION GUIDE

### 6.1 Prerequisites

**Software:**
- Power BI Desktop (February 2026 or later)
- LibreOffice or MS Office (for theme font installation)

**Fonts (install system-wide before opening Power BI):**
- DM Sans (body text)
- IBM Plex Mono (data labels, code)
- Syne (headings)

**Data Files:**
- Fact_Order.csv
- Fact_OrderLines.csv
- Fact_JuiceNMV.csv
- Fact_MaxabNMV.csv
- Dim_Date.csv
- Dim_Region.csv
- Dim_Channel.csv
- Dim_Product.csv
- Dim_Brand.csv

---

### 6.2 Implementation Steps

#### Step 1: Install Fonts (10 min)
1. Download fonts from Google Fonts:
   - DM Sans: https://fonts.google.com/specimen/DM+Sans
   - IBM Plex Mono: https://fonts.google.com/specimen/IBM+Plex+Mono
   - Syne: https://fonts.google.com/specimen/Syne
2. Install system-wide (Windows: Right-click → Install for all users)
3. Restart computer if fonts don't appear in Power BI

#### Step 2: Import Theme JSON (5 min)
1. Open Power BI Desktop
2. View → Themes → Browse for themes
3. Select `maxab-theme.json`
4. Click "Apply"
5. Verify colors: Background should be #0b0d0f

#### Step 3: Load Data (15 min)
1. Home → Get Data → Text/CSV
2. Load all 9 CSV files
3. Power Query Editor:
   - Set data types for all date columns
   - Set RegionID, ChannelID as whole numbers
   - Set Sales columns as Fixed Decimal
   - Apply changes
4. Close & Apply

#### Step 4: Create Relationships (10 min)
1. Model view → Drag relationships:
   - Fact_Order[Created] → Dim_Date[Date]
   - Fact_Order[RegionID] → Dim_Region[RegionID]
   - Fact_Order[ChannelID] → Dim_Channel[ChannelID]
   - Fact_OrderLines[SKU] → Dim_Product[SKU]
   - Fact_JuiceNMV[Date] → Dim_Date[Date]
   - Fact_JuiceNMV[Brand] → Dim_Brand[Brand]
   - Fact_MaxabNMV[Date] → Dim_Date[Date]
2. Set Fact_Order[Created] → Dim_Date relationship to Active
3. Set Fact_OrderLines[OrderCreation] → Dim_Date to Inactive

#### Step 5: Mark Date Table (2 min)
1. Right-click Dim_Date → Mark as date table
2. Select Dim_Date[Date] as date column
3. Verify: Table should show calendar icon

#### Step 6: Create Measure Library Table (5 min)
1. Modeling → New Table
2. Expression: `MeasureLibrary = ROW("Placeholder", 1)`
3. Hide the "Placeholder" column
4. All measures will be created in this table

#### Step 7: Create Parameters (5 min)
1. Modeling → New Parameter → Numeric range
2. DeliveryThresholdDays:
   - Min: 1, Max: 5, Increment: 1, Default: 2
   - Add to report as slicer: ✅ Yes
3. ChurnLookbackMonths:
   - Min: 1, Max: 6, Increment: 1, Default: 3
   - Add to report as slicer: ✅ Yes

#### Step 8: Create Cohort_Matrix Table (5 min)
1. Modeling → New Table
2. Paste the Cohort_Matrix expression (see Section 3.4)
3. Verify row count: Should be 25 rows (5 cohorts × 5 offsets)
4. Format MonthOffset: Custom format "M+"0

#### Step 9: Create DAX Measures — Batch Import (30 min)
Using the Power BI Desktop MCP server or manually:
1. Create all measures listed in Section 5
2. Organize into display folders as specified
3. Set format strings for each measure
4. Verify: Should have 75+ measures in MeasureLibrary table

**Tip:** Use the bulk create feature in the implementation guide to import all measures at once.

#### Step 10: Build Pages (120 min total)
Follow the visual specifications in Section 4 for each page:
- **Page 00 — Home:** 30 min
- **Page 01 — Executive:** 20 min
- **Page 02 — Delivery:** 20 min
- **Page 03 — Churn:** 25 min
- **Page 04 — Drop:** 20 min
- **Page 05 — Commercial:** 25 min

#### Step 11: Apply SVG Backgrounds (10 min)
1. For each page:
   - Format pane → Canvas background → Image
   - Browse → Select corresponding SVG file
   - Image fit: Normal (since canvas is 1440×900)
   - Transparency: 0%

#### Step 12: Configure Slicers (10 min)
1. Left sidebar panel (200px wide, all pages):
   - Period: Dim_Date[YearMonth] → Tile horizontal
   - Region: Dim_Region[Region] → Tile horizontal
   - Channel: Dim_Channel[Channel] → Tile horizontal
   - SLA Threshold: DeliveryThresholdDays → Between slider (P2 only)
   - Churn Lookback: ChurnLookbackMonths → Between slider (P3 only)
2. Set default selections: All, All, All, 2, 3

#### Step 13: Set Page Navigation (10 min)
1. Home page → Select each navigation card
2. Add button action:
   - Type: Page navigation
   - Destination: [Target page]
3. Test all 5 navigation buttons

#### Step 14: Create Bookmarks (5 min)
1. View → Bookmarks pane
2. Add bookmark: "Default View"
3. Capture: All pages, all slicers reset to default
4. Add "Reset" button on each page → Action → Bookmark → Default View

#### Step 15: Set Default Page (2 min)
1. Right-click "00 — Home" tab
2. Select "Set as default page"
3. Save file

#### Step 16: Publish to Service (5 min)
1. Home → Publish
2. Select workspace
3. Configure scheduled refresh: Daily at 6:00 AM
4. Set data gateway connection (if using SQL Server)

---

### 6.3 Quality Assurance Checklist

**Data Model:**
- [ ] All 9 tables loaded successfully
- [ ] 8 relationships created (7 active, 1 inactive)
- [ ] Dim_Date marked as date table
- [ ] Cohort_Matrix has 25 rows
- [ ] Parameters show in Fields pane

**Measures:**
- [ ] 75+ measures created in MeasureLibrary table
- [ ] All measures organized into display folders
- [ ] Format strings applied correctly
- [ ] No circular dependency errors

**Pages:**
- [ ] 6 pages created (Home + 5 analysis)
- [ ] All pages use 1440×900 canvas size
- [ ] SVG backgrounds applied to all pages
- [ ] All visuals positioned correctly

**Functionality:**
- [ ] All KPI cards show values (not blank)
- [ ] Slicers filter all visuals on page
- [ ] Parameters update dependent measures
- [ ] Navigation buttons work from Home page
- [ ] Conditional formatting colors show correctly
- [ ] Cohort matrix shows retention percentages

**Publishing:**
- [ ] Report publishes without errors
- [ ] Scheduled refresh configured
- [ ] Data gateway connected (if needed)
- [ ] Mobile layout created (optional)

---

## 7. MAINTENANCE & OPERATIONS

### 7.1 Daily Operations

**Automated:**
- Scheduled refresh: 6:00 AM daily
- Data pulls from source systems
- Measure recalculation

**Manual (5 min/day):**
- Check [Home Alert Text] for critical issues
- Verify [Last Refresh Date] is current
- Scan P2 Success Rate card for SLA misses

---

### 7.2 Weekly Review

**Performance Check (15 min):**
1. P1: Review MoM % changes — flag any >20% variance
2. P2: Check Success Rate trend — escalate if <80% for 3+ days
3. P3: Monitor Churn Rate — alert if >40%
4. P4: (May only) Review SKU drop table for stock issues
5. P5: Check Top 3 Brands share — flag if concentration >80%

**Data Quality (10 min):**
1. Verify no missing dates in Dim_Date
2. Check for duplicate OrderIDs in Fact_Order
3. Validate RegionID/ChannelID foreign keys
4. Confirm Fact_JuiceNMV brands match Dim_Brand

---

### 7.3 Monthly Maintenance

**Threshold Review (30 min):**
- Review SLA threshold (currently 2 days) — adjust if needed
- Review Churn lookback window (currently 3 months) — adjust if needed
- Update alert thresholds in [Home Alert Text] measure:
  - Success Rate target: Currently 86%
  - Churn Rate alert: Currently 30%
  - Drop alert: Currently -15%

**Model Optimization (60 min):**
1. Delete unused columns from fact tables
2. Check DAX measure performance (DAX Studio)
3. Archive old data (keep rolling 12 months)
4. Rebuild Cohort_Matrix if new cohorts added

**Documentation Updates:**
- Update this document if measures/visuals change
- Document any new calculated columns
- Update data dictionary if source schema changes

---

### 7.4 Quarterly Review

**Business Alignment (90 min):**
1. Meet with stakeholders to review KPIs
2. Confirm metrics still align with business goals
3. Identify new analysis requirements
4. Plan dashboard enhancements

**Performance Audit:**
1. Review query performance (Power BI Performance Analyzer)
2. Check report load times (target: <3 seconds)
3. Optimize slow visuals (simplify, aggregate, or use DirectQuery)
4. Review dataset size (target: <1 GB)

---

## 8. TROUBLESHOOTING

### 8.1 Common Issues

#### Issue: Cohort Retention % shows blank cells
**Symptoms:** Matrix visual shows "—" or blank for some cohort/offset combinations

**Root Cause:** Not enough time elapsed — e.g., Feb cohort can't show M+4 retention in March

**Solution:** This is expected behavior. Only cohorts with sufficient history will show retention curves.

**Workaround:** Add a note below the matrix: "Blank cells indicate insufficient data (cohort too recent)"

---

#### Issue: Churn Rate % is blank
**Symptoms:** P3 KPI card shows "—" instead of a percentage

**Root Cause:** 
1. No date filter selected (measure needs a "current month")
2. Parameter slicer not set (default 3 months)

**Solution:**
1. Verify `Dim_Date[YearMonth]` slicer has a selection
2. Check ChurnLookbackMonths parameter is set to 3
3. Verify [Churned Retailers] measure is not returning an error

**Verification Query:**
```dax
EVALUATE ROW(
    "Churned", [Churned Retailers],
    "Active", [Active Retailers],
    "Churn %", [Churn Rate %]
)
```
Should return numbers, not blanks.

---

#### Issue: SLA Status shows 0% when it should be 80%+
**Symptoms:** P2 Success Rate card shows 0.0% but most orders are delivered

**Root Cause:** 
1. `Fact_Order[Delivered]` column has wrong data type (Text instead of Date)
2. Working days not calculated correctly in Dim_Date

**Solution:**
1. Power Query → Check Fact_Order[Delivered] data type = Date/Time
2. Verify Dim_Date[IsWorkingDay] column:
   - Monday-Friday = 1 (TRUE)
   - Saturday-Sunday = 0 (FALSE)
3. Recalculate [Delivery SLA (Working Days)] measure

---

#### Issue: May Wk3 measures show blank on Page 4
**Symptoms:** [May Wk3 Sales] returns blank or error

**Root Cause:** 
1. Dim_Date[YearMonth] column format is wrong
2. WeekOfMonth column missing or miscalculated
3. Page-level filter not set to "2024-05"

**Solution:**
1. Verify Dim_Date[YearMonth] format is "YYYY-MM" (e.g., "2024-05")
2. Check WeekOfMonth column exists and has values 1-5
3. Verify Page 4 has page-level filter: Dim_Date[YearMonth] = "2024-05"

---

#### Issue: Navigation buttons don't work from Home page
**Symptoms:** Clicking nav cards doesn't navigate to target page

**Root Cause:** Button action not configured or page name mismatch

**Solution:**
1. Select button visual (not the shape behind it)
2. Format → Action → Type = Page navigation
3. Destination = [Exact page name, case-sensitive]
4. Verify page names match:
   - "01 — Executive Overview"
   - "02 — Performance & Delivery"
   - "03 — Churn & Activation"
   - "04 — Drop Analysis"
   - "05 — Commercial / Juice"

---

#### Issue: Conditional formatting colors not showing
**Symptoms:** Matrix cells are all the same color instead of green→red gradient

**Root Cause:** CF measure not applied, or returning wrong format

**Solution:**
1. Select visual → Format → Cell elements → Background color
2. Format style: Field value
3. What field should we base this on?: [CF Cohort Color] (or other CF measure)
4. Verify CF measure returns hex color strings like "#4ade80"

**Test Query:**
```dax
EVALUATE ROW("Color", [CF Cohort Color])
```
Should return "#4ade80" (or similar hex code), not a number.

---

#### Issue: Visuals load slowly (>5 seconds)
**Symptoms:** Page takes a long time to render or slicers are laggy

**Root Causes:**
1. Too many visuals on page (>15)
2. Complex DAX measures (nested CALCULATE, SUMMARIZE)
3. Large fact tables (>1M rows) without aggregations

**Solutions:**
1. Reduce visual count: Combine related charts into small multiples
2. Optimize DAX: Use variables, avoid row-by-row iteration
3. Add aggregation tables: Pre-calculate common metrics
4. Use DirectQuery for large datasets (if needed)

**Performance Profiling:**
1. View → Performance Analyzer → Start recording
2. Interact with page (change slicer, click visual)
3. Stop recording → Review top 5 slowest queries
4. Optimize those measures/visuals first

---

### 8.2 Data Refresh Errors

#### Error: "Couldn't refresh the dataset"
**Symptoms:** Scheduled refresh fails in Power BI Service

**Root Causes:**
1. Data source credentials expired
2. Source table schema changed
3. Data gateway offline

**Solutions:**
1. Power BI Service → Workspace → Dataset settings → Data source credentials → Re-enter
2. Check source system: Did column names/types change?
3. Check data gateway: Is it running? Is network accessible?

---

#### Error: "Circular dependency detected"
**Symptoms:** Model won't save, shows "circular dependency" error

**Root Cause:** Measure A references Measure B which references Measure A

**Solution:**
1. Review measure dependencies (use DAX Studio)
2. Identify the circular loop
3. Refactor measures to break the cycle (use variables, split into multiple measures)

---

### 8.3 Getting Help

**Internal Resources:**
- This documentation (master reference)
- DAX query library (Section 5)
- Implementation guide (Section 6)

**External Resources:**
- Power BI Community: https://community.powerbi.com
- DAX Patterns: https://www.daxpatterns.com
- SQLBI Blog: https://www.sqlbi.com/blog

**Support Contacts:**
- Data Team: data@maxab.com
- BI Administrator: bi-admin@maxab.com
- IT Support: helpdesk@maxab.com

---

## 9. APPENDICES

### Appendix A: Theme JSON

```json
{
  "name": "Maxab Dark Analytics",
  "dataColors": [
    "#f5a623", "#2dd4bf", "#60a5fa", "#4ade80", 
    "#f87171", "#a78bfa", "#fb923c", "#e2e8f0"
  ],
  "background": "#0b0d0f",
  "foreground": "#c8cdd4",
  "tableAccent": "#f5a623"
}
```

*Full theme JSON file: `maxab-theme.json`*

---

### Appendix B: Keyboard Shortcuts

| Action | Shortcut |
|---|---|
| Refresh data | Ctrl + R |
| Open Performance Analyzer | Alt + Shift + P |
| Toggle between pages | Ctrl + Page Up/Down |
| Select multiple visuals | Ctrl + Click |
| Duplicate visual | Ctrl + D |
| Align visuals | Format → Align → Distribute |
| Open Fields pane | Alt + F |
| Open Format pane | Alt + M |

---

### Appendix C: Change Log

| Version | Date | Changes | Author |
|---|---|---|---|
| 1.0 | Feb 2026 | Initial documentation | Analytics Team |

---

### Appendix D: Glossary

| Term | Definition |
|---|---|
| **AOV** | Average Order Value — Total Sales ÷ Total Orders |
| **Churn** | Retailers who were active in prior N months but not in current month |
| **Cohort** | Group of retailers who placed first order in the same month |
| **MoM** | Month-over-Month — Comparison to prior month |
| **NMV** | Net Merchandise Value — Total sales for a category |
| **pp** | Percentage points — Absolute change in percentage (e.g., 10% to 12% = +2pp) |
| **SLA** | Service Level Agreement — Target delivery time (default: 2 working days) |
| **Working Days** | Monday-Friday only (excludes weekends) |
| **YTD** | Year-To-Date — Cumulative from Jan 1 to current date |

---

### Appendix E: File Manifest

**Root Directory:**
```
📁 Maxab_Dashboard_Project/
├── 📄 Maxab_Dashboard.pbix                    (Main Power BI file)
├── 📄 MAXAB_Dashboard_Documentation.md         (This document)
├── 📄 maxab-theme.json                         (Theme file)
├── 📁 data/
│   ├── Fact_Order.csv
│   ├── Fact_OrderLines.csv
│   ├── Fact_JuiceNMV.csv
│   ├── Fact_MaxabNMV.csv
│   ├── Dim_Date.csv
│   ├── Dim_Region.csv
│   ├── Dim_Channel.csv
│   ├── Dim_Product.csv
│   └── Dim_Brand.csv
├── 📁 backgrounds/
│   ├── bg-p1-executive.svg
│   ├── bg-p2-delivery.svg
│   ├── bg-p3-churn.svg
│   ├── bg-p4-drop.svg
│   └── bg-p5-juice.svg
├── 📁 wireframes/
│   ├── Maxab_P1_Executive_Overview.pptx
│   ├── Maxab_P2_Performance_Delivery.pptx
│   ├── Maxab_P3_Churn_Activation.pptx
│   ├── Maxab_P4_Drop_Analysis.pptx
│   └── Maxab_P5_Commercial_Juice.pptx
└── 📁 fonts/
    ├── DMSans-Regular.ttf
    ├── IBMPlexMono-Regular.ttf
    └── Syne-Bold.ttf
```

**Total File Count:** 29 files  
**Total Project Size:** ~50 MB (excluding .pbix)

---

## END OF DOCUMENTATION

**Document Version:** 1.0  
**Last Updated:** February 2026  
**Next Review Date:** May 2026  
**Document Owner:** Analytics Team  
**Status:** Production

---

*For questions, updates, or feedback on this documentation, contact: data@maxab.com*
