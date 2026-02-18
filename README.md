# Retail Revenue Audit & Profitability Analysis

##  Project Overview
This project is a data-driven audit of a retail organization’s performance, focusing on identifying operational inefficiencies across **1,850+ SKUs**. By moving from high-level strategic overviews to granular product-level audits, this analysis provides actionable insights into inventory rationalization and margin recovery.

---

##  Business Questions Answered
* **Revenue Stability:** How dependent is our revenue on "Hero" products? (Result: Low; top 5 products represent < 9% of revenue).
* **Category Performance:** Which categories are generating high volume but failing to deliver profit? (Result: **Furniture** identified as a high-risk category).
* **Operational Complexity:** Are we managing too many underperforming products? (Result: Visualized the "Long Tail" of low-value SKUs).
* **SKU Rationalization:** Which specific products should be considered for removal ("The Kill List")?

---

## Tech Stack
* **SQL:** Data extraction, cleaning, and multi-table joins to prepare the 1,850+ SKU dataset.
* **Power BI:** Advanced data modeling and interactive dashboard design.
* **DAX:** Developed custom measures including:
    * **Profit Margin %**
    * **Total Revenue** & **Total Profit**
    * **Conditional Logic** for performance thresholds.

---

##  Dashboard Deep-Dive

### 1. Strategic Revenue Audit (Page 1)
* **Focus:** Executive-level view of revenue distribution and category health.
* **Key Visuals:** * **Revenue TreeMap:** Visualizes the "fragmentation" of the product catalog.
    * **Profit vs. Revenue Trend:** Highlights the margin dip in the Furniture segment.

### 2. Product Audit Detail (Page 2)
* **Focus:** Tactical "Control Room" for inventory decisions.
* **Key Visuals:**
    * **Profitability Matrix (Scatter Plot):** Maps volume vs. efficiency to identify "Hero" products and "Margin Leakers."
    * **Audit Matrix:** A detailed SKU-level list with "Stoplight" conditional formatting (Red for negative margins, Green for high performance).

---

##  SQL Implementation
The data was staged using SQL to ensure a clean "Single Source of Truth." These queries were used to identify both high-risk "leakers" and optimization opportunities.

### 1. Identifying Optimization Candidates
This query identifies the "Stable Middle"—products with margins between 5% and 20% that represent the best opportunity for margin growth through discount reduction.

```sql
SELECT 
    Product_Name,
    SUM(Sales) AS Total_Revenue,
    SUM(Profit) AS Total_Profit,
    SUM(Profit)/SUM(Sales)*100 AS Profit_margin_percent,
    AVG(Discount)*100 AS Avg_Discount_percent
FROM superstore.sales_dataset
GROUP BY Product_Name
HAVING (SUM(Profit)/SUM(Sales)*100) BETWEEN 5 AND 20
ORDER BY Total_Revenue DESC;
```
### 2: Profitability Optimization (The "Stable Middle")
** Identifies products with margins between 5% and 20% for margin growth.**

```sql
SELECT 
    Product_Name,
    Sum(Sales) AS Total_Revenue,
    Sum(Profit) AS Total_Profit,
    SUM(Profit)/SUM(Sales)*100 AS Profit_margin_percent,
    AVG(Discount)*100 AS Avg_Discount_percent
FROM superstore.sales_dataset
GROUP BY Product_Name
HAVING (SUM(Profit)/SUM(Sales)*100) BETWEEN 5 AND 20;
```
### 3: Margin Leakage "Kill List"
**Identifies the top 10 products with the lowest/negative margins.**

```sql
SELECT TOP 10
    Product_Name,
    SUM(Sales) AS Total_Sales,
    (SUM(Profit) / SUM(Sales)) AS Margin_Rate
FROM Retail_Sales_Data
GROUP BY Product_Name
ORDER BY Margin_Rate ASC;
```


### 4: Product Revenue Ranking
**Uses window functions to assign a market position to each product based on total sales volume**.

```sql
SELECT 
    Product_Name,
    SUM(Sales) AS Total_Revenue,
    RANK() OVER (ORDER BY SUM(Sales) DESC) AS Revenue_Rank
FROM superstore.sales_dataset
GROUP BY Product_Name;
```
### 5: Pareto Analysis (Cumulative Revenue %)
**Calculates cumulative contribution to determine if the business is over-reliant on a few items or spread too thin**

```sql
SELECT 
    Product_Name,
    SUM(Sales) AS Total_Revenue,
    SUM(SUM(Sales)) OVER (ORDER BY SUM(Sales) DESC) 
        / SUM(SUM(Sales)) OVER () AS Cumulative_Revenue_Pct
FROM superstore.sales_dataset
GROUP BY Product_Name
ORDER BY Total_Revenue DESC;
```

**Strategic Recommendations**
**Inventory Rationalization:** Immediately review the bottom 10% of products identified in the Margin Leakage Audit (SQL 3). These items are "margin leakers" that should be phased out or re-priced to stop subsidizing losses.

**Furniture Category Deep-Dive:** Conduct a secondary supply chain audit of the Furniture category. High sales volume currently masks critically low margins compared to the Technology and Office Supplies categories.

**Discount Optimization:** Targeted reduction of discounts for products in the "Stable Middle" (SQL 2). Reducing the average discount by just 2–5% on these items could move them into the "High-Performance" tier without significantly impacting sales volume.

**Operational Streamlining**: Address the Product Fragmentation issue. Since the top 5 products contribute less than 9% of revenue, the business is managing excessive overhead. Reducing the total SKU count from 1,850+ will lower warehouse costs and management complexity.

***

