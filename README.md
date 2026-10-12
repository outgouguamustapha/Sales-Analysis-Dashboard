# Sales Performance & Profitability Analytics Dashboard

[![Live Dashboard](https://img.shields.io/badge/Power_BI-Live_Dashboard-F2C811?style=flat&logo=powerbi&logoColor=black)](https://app.powerbi.com/view?r=eyJrIjoiMjBmYTFjYWItODI3Yy00YWM2LTljNjItYmNjNjliNzA5MGY4IiwidCI6Ijg0ZDI3MGQyLTBiYzUtNGQ1NS1iZjBhLTI3NGYxYTU3NmNiZiJ9)
[![Documentation](https://img.shields.io/badge/Documentation-PRD_Included-orange.svg)](docs/PRD.md)
[![Data Model](https://img.shields.io/badge/Model-Star_Schema-green.svg)](#data-modeling--schema)
[![DAX](https://img.shields.io/badge/DAX-Calculated_Measures-blue.svg)](#key-dax-measures)

An end-to-end Power BI analytics solution engineered to analyze, monitor, and optimize revenue streams, order volumes, and gross margins across commercial sales channels.

---

## Interactive Live Report
Access the fully functional, interactive Power BI report directly in your browser:

<a href="https://app.powerbi.com/view?r=eyJrIjoiMjBmYTFjYWItODI3Yy00YWM2LTljNjItYmNjNjliNzA5MGY4IiwidCI6Ijg0ZDI3MGQyLTBiYzUtNGQ1NS1iZjBhLTI3NGYxYTU3NmNiZiJ9" target="_blank" rel="noopener noreferrer">Launch Interactive Sales Dashboard</a>

---

## Table of Contents
1. [About This Project](#1-about-this-project)
2. [Product Requirements Document (PRD)](#2-product-requirements-document-prd)
3. [Dashboard Architecture & Visual Interface](#3-dashboard-architecture--visual-interface)
4. [Data Modeling & Schema](#4-data-modeling--schema)
5. [Key DAX Measures](#5-key-dax-measures)
6. [Business Insights & Strategic Recommendations](#6-business-insights--strategic-recommendations)
7. [Repository Structure](#7-repository-structure)
8. [Author & Contact](#8-author--contact)

---

## 1. About This Project

Commercial organizations commonly face reporting silos, fragmented transactional spreadsheets, and delayed visibility into unit profitability. When leadership evaluates top-line gross revenue without contextual cost and margin data, unprofitable volume growth often masks underlying operational inefficiencies.

This project delivers a centralized business intelligence layer designed to solve that problem. Built with **Microsoft Power BI**, it translates raw transactional line items into an executive-level performance engine that surfaces the "why" behind commercial variance across time, geography, customer tiers, and product hierarchies.

### Core Deliverables
* **Executive Summary View:** High-level scorecard tracking overall revenue health, net margin trends, and order volumes.
* **Geographical & Category Drill-Downs:** Detailed visual slices enabling regional and category managers to isolate margin drag.
* **Governed Semantic Layer:** A clean Star Schema backed by explicit, performant DAX business logic.

---

## 2. Product Requirements Document (PRD)

To ensure technical development aligned with concrete business objectives rather than arbitrary visual design, the dashboard was built against a structured Product Requirements Document.

* **Full Specification:** [docs/PRD.md](docs/PRD.md)

### Key Requirements Satisfied
* **Automated Data Processing:** Standardized ingestion and transformation via Power Query to clean historical transactional records.
* **Standardized Metric Definitions:** Unified calculations for Revenue, Gross Margin, Average Order Value (AOV), and Quantity Sold.
* **Actionable Analytical Paths:** Navigation structured to allow users to move from global KPIs down to regional and product-line root causes within three clicks.

---

## 3. Dashboard Architecture & Visual Interface

### Executive Performance Overview
The main interface consolidates critical metrics, tracking commercial trajectory alongside categorical contribution:

![Sales Dashboard Overview](Image/Overview2.png)

*A complete, high-resolution exported artifact is available at [`Image/Sales Dashboard.pdf`](Image/Sales%20Dashboard.pdf).*

---

## 4. Data Modeling & Schema

The data model is structured in a strict **Star Schema** to ensure fast analytical processing, maintain referential integrity, and eliminate ambiguous filter propagation:
```text
                         +-------------------+
                         |     Dim_Date      |
                         +-------------------+
                                   | 1
                                   | 
                                   | *
+-------------------+     +-------------------+     +-------------------+
|   Dim_Product      |----|    Fact_Sales      |----|   Dim_Customer    |
+-------------------+ 1   +-------------------+   1 +-------------------+
         | *
         |
         | 1
+-------------------+
|   Dim_Geography   |
+-------------------+
```
* **`Fact_Sales` (Fact Table):** Captures atomic transaction lines, including quantities, unit selling prices, standard costs, and discount values.
* **`Dim_Product`:** Hierarchical categorization covering categories, sub-categories, and SKU identifiers.
* **`Dim_Geography`:** Regional, country, and city mappings for geographic slicing.
* **`Dim_Customer`:** Account metadata and segment groupings.
* **`Dim_Date`:** A dedicated, continuous calendar dimension marked as the official Date Table to guarantee accurate time intelligence calculations.

---

## 5. Key DAX Measures

The semantic layer is powered by modular DAX measures organized by business domains (Financial Core, Time Intelligence, Target Pacing, and Agent Productivity):

### 1. Core Financials & Margin
```
// Total Top-Line Revenue
Total Revenue = SUM(Fact_Sales[Sales])

// Total Bottom-Line Profit
Total Profit = SUM(Fact_Sales[Profit])

// Month-to-Date Profit Margin
Profit Margin MTD = DIVIDE([Total Profit], [Total Revenue])
```
### 2. Time Intelligence & MoM Pacing
```
// Revenue Month-to-Date
Revenue MTD = 
CALCULATE(
    SUM(Fact_Sales[Sales]),
    DATESMTD('DimDate'[Date])
)

// Revenue Month-over-Month % Change
Revenue MoM % Change = 
VAR Curr = [Current Month Sales]
VAR Prev = [Previous Month Sales]
RETURN
IF(
    NOT(ISBLANK(Curr)) && NOT(ISBLANK(Prev)),
    DIVIDE(Curr - Prev, Prev),
    0
)
```
### 3. Target & Variance Tracking
```
// Target Achievement Percentage Against Budget ($100k Benchmark)
Target Achievement % = 
VAR _Actual = [Revenue MTD]
VAR _Target = [Revenue Target] -- $100,000
RETURN
IF(
    ISBLANK(_Actual),
    BLANK(),
    DIVIDE(_Actual, _Target)
)

// Target Performance Classification
Status = 
VAR _TargetAchievement = [Target Achievement %]
RETURN
SWITCH(
    TRUE(),
    ISBLANK(_TargetAchievement), "-",
    _TargetAchievement >= 1, "Exceeding Target",
    _TargetAchievement >= 0.95, "Near Target",
    _TargetAchievement < 0.95, "Missing Target"
)
```
### 3. Target & Variance Tracking
```
// Proportion of Sales Force Meeting Target
Agents Meeting Target % = 
VAR AgentsList = VALUES(Dim_Agent[Sales Agent Key])
VAR MetTargetCount = 
    SUMX(
        AgentsList,
        VAR _agent = Dim_Agent[Sales Agent Key]
        VAR _agentRevenue = 
            CALCULATE(
                [Revenue MTD],
                FILTER(ALL(Dim_Agent), Dim_Agent[Sales Agent Key] = _agent)
            )
        RETURN
            IF(_agentRevenue >= [Revenue Target], 1, 0)
    )
VAR TotalAgents = COUNTROWS(AgentsList)
RETURN
IF(TotalAgents = 0, 0, DIVIDE(MetTargetCount, TotalAgents, 0))

// Average Sales Volume per Active Rep MTD
Avg Sales per Agent MTD = 
DIVIDE(
    [Revenue MTD],
    DISTINCTCOUNT(Dim_Agent[Sales Agent Key])
)
```



## 6. Business Insights & Strategic Recommendations
An analytical audit of the resulting data patterns reveals several structural levers for commercial management:

### 1. Address Top-Line Volume vs. Margin Dilution
* **Finding:** While specific high-volume product categories account for substantial gross revenue, their net profit margin falls below the 28% operational benchmark, driven by heavy baseline discounting and promotional price erosion.

* **Actionable Recommendation:** Institute a dynamic discounting cap for tier-one accounts and pivot sales incentives from pure volume to gross margin contribution (Profit Margin MTD). Shifting volume toward higher-margin complementary SKUs can recover an estimated 200–350 basis points of gross margin.

### 2. Capitalize on Sales Force Distribution & Quota Balance
* **Finding:** A significant variance exists in agent-level quota achievement: a concentrated subset of agents consistently exceeds the $100,000 revenue target, while aggregate Agents Meeting Target % indicates underperformance in the lower quartile.

* **Actionable Recommendation:** Actionable Recommendation: Benchmark the sales playbooks of top-performing reps and rebalance lead distribution algorithms. Pair underperforming tiers with structured enablement to lift average rep sales volume (Avg Sales per Agent MTD).

### 3. Normalize Period-End Revenue Spikes
* **Finding:** Significant surges in transaction volume materialize during the final 10 days of each financial quarter, frequently followed by higher return rates and administrative processing friction in the subsequent month.

* **Actionable Recommendation:** Transition operational and demand forecasting to 30-day moving rolling averages rather than static end-of-quarter quotas. Rebalancing pipeline targets across monthly milestones will ease supply chain stress and stabilize fulfillment overhead.


## 7. Repository Structure
```
├── Data/
│   └── Sales Data.xlsx             # Source transactional data extract
├── docs/
│   └── PRD.md                      # Complete Product Requirements Document
├── Image/
│   ├── Overview2.png               # High-resolution dashboard screenshot
│   └── Sales Dashboard.pdf         # Exported pdf report
└── README.md                       # Main repository overview and documentation
```

---

## 8. Author & Contact

**OUTGOUGUA MUSTAPHA**

Data Analyst | Power BI Developer | Business Intelligence Specialist

- 💼 [LinkedIn](https://www.linkedin.com/in/mustapha-outgougua/)
- 🐙 [GitHub](https://github.com/outgouguamustapha)
- 📧 Open to collaboration and opportunities

---


<div align="center">

**If you found this project insightful, please consider giving it a ⭐!**

Made with ❤️ and Power BI

</div>
