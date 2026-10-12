# Product Requirements Document (PRD)
## Executive Sales Performance & Profitability Dashboard

- **Document Version:** 1.0.0
- **Status:** Approved / Implemented
- **Owner / BI Analyst:** Mustapha Outgougua
- **Target Platform:** Microsoft Power BI Desktop & Service

---

## 1. Executive Summary & Problem Statement

### 1.1 Business Context
Retail and commercial enterprises generate high-volume transaction data across diverse geographies, product hierarchies, and customer segments. Without centralized business intelligence, leadership faces:
- Inability to quickly pinpoint underperforming regions, product categories, or customer segments.
- Delayed visibility into margin dilution (high gross sales obscured by high return rates or steep discounts).
- Reliance on disparate, static Excel reports with no uniform metric definitions or interactive drill-through.

### 1.2 Proposed Solution
Develop an enterprise-grade Microsoft Power BI reporting suite backed by a clean Star Schema data model. The dashboard delivers interactive KPI tracking across Revenue, Profitability, Unit Volume, and Operational Performance, enabling regional sales directors and operational managers to transition from descriptive reporting to prescriptive decision-making.

---

## 2. Stakeholders & User Personas

| Persona | Role | Primary Objective / Pain Point |
|---|---|---|
| **VP of Sales / Executive Leadership** | Strategic Oversight | Monitor top-line growth (YTD, YoY), gross margins, and high-level KPI trends at a glance. |
| **Regional Sales Managers** | Tactical Management | Identify branch-level performance, regional quotas, and geographic variations in demand. |
| **Product & Merchandising Leads** | Portfolio Optimization | Evaluate product category profitability, inventory velocity, and margin drag. |
| **Data / BI Engineer** | Model & Pipeline Owner | Maintain a single source of truth, optimized DAX measures, and reliable refresh pipelines. |

---

## 3. Goals & Success Metrics (OKRs)

### 3.1 Business Goals
- Reduce monthly executive sales reporting delivery cycle from **5 business days to real-time**.
- Enable self-service root-cause discovery across region, product, and channel dimensions within **3 clicks**.
- Provide early warning on margin erosion by tracking Net Margin % against budget targets.

### 3.2 Technical Performance Targets
- **Model Efficiency:** Star Schema design; fact table query response time under **1.5 seconds**.
- **Data Quality:** 100% automated handling of nulls, data type inconsistencies, and referential integrity checks via Power Query / ETL.
- **DAX Best Practice:** Elimination of row-by-row iteration anti-patterns (`EARLIER`, unbounded `FILTER` loops) in favor of optimized filter context calculation (`CALCULATE`, `KEEPFILTERS`).

---

## 4. Key Performance Indicators (KPIs) & Business Logic

| Metric Name | Technical DAX Definition / Concept | Business Purpose | Target / Benchmark |
|---|---|---|---|
| **Total Revenue** | `SUM(Sales[SalesAmount])` | Core top-line cash generation | Prior Year +10% |
| **Total Profit** | `SUM(Sales[SalesAmount]) - SUM(Sales[TotalCost])` | Bottom-line gross financial outcome | Margin > 25% |
| **Profit Margin %** | `DIVIDE([Total Profit], [Total Revenue], 0)` | Operational unit economics efficiency | Consistent > 20% |
| **Total Orders** | `DISTINCTCOUNT(Sales[OrderNumber])` | Sales volume & transaction activity velocity | Period-over-Period growth |
| **Average Order Value (AOV)** | `DIVIDE([Total Revenue], [Total Orders], 0)` | Customer basket size and pricing health | Increasing trend |
| **Units Sold** | `SUM(Sales[OrderQuantity])` | Fulfillment and inventory throughput | Volume capacity alignment |

---

## 5. Functional & Dashboard Requirements

### 5.1 Architecture & Data Modeling
- **Schema Architecture:** Pure Star Schema.
  - **Fact Table:** `Fact_Sales` (grain: individual line item per invoice).
  - **Dimension Tables:** `Dim_Date`, `Dim_Product`, `Dim_Customer`, `Dim_Geography`.
- **Date Dimension:** Fully qualified continuous calendar table marked as official Date Table to ensure accurate Time Intelligence (`TOTALYTD`, `SAMEPERIODLASTYEAR`, `DATEADD`).

### 5.2 User Interface & Visual Layout
- **Header / Navigation:** Executive global slice bar (Date Range, Region, Customer Segment, Product Category).
- **KPI Summary Ribbon:** Top row KPI cards displaying Revenue, Profit, Profit Margin, Orders, and AOV with period-over-period delta indicators.
- **Trend Visualizations:** Monthly/Quarterly revenue and profit trajectory (Line & Clustered Column Chart).
- **Breakdown Analytics:**
  - Bar/Decomposition charts for Top Products and Underperforming Categories.
  - Map / Geo-visual showing revenue density and regional margins.
- **Detail Drill-Through:** Detailed grid view enabling granular inspection of invoice lines, discounts, and customer purchase histories.

---

## 6. Assumptions, Constraints & Future Scope

### 6.1 Constraints
- Source data delivered via static transaction extracts (`Sales Data.xlsx`); requires strict schema enforcement in Power Query.
- Row-Level Security (RLS) defined for regional roles if distributed via Power BI Service.

### 6.2 Future Enhancements (Phase II)
- Implement predictive sales forecasting using built-in exponential smoothing models.
- Introduce Customer Lifetime Value (CLV) and automated RFM customer tier segmentation.
- Integrate automated email alerts for negative margin transactions via Power Automate.
