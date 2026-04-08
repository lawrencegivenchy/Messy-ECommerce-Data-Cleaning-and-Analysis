
# 🛒 Messy E-Commerce Sales Performance Analysis

## 📊 Project Overview

This project focuses on transforming a messy e-commerce sales dataset into structured, actionable business insights. The raw dataset contained inconsistencies such as currency formatting issues, missing values, leading whitespace in column names, and inconsistent date formats — all of which were resolved through data cleaning in Excel.

Using Databricks SQL and Power BI, the analysis explores key areas including revenue performance, customer behaviour, payment trends, and operational efficiency. The goal was not just to analyse the data, but to simulate how a data analyst would extract insights to support real business decision-making.

**Tools Used:** Microsoft Excel | Databricks SQL | Power BI

---

## 📁 Dataset

- **Source:** Synthetic e-commerce sales dataset
- **File:** `Clean_Messy_E-Com_Sales_Data.csv`
- **Records:** ~100 orders
- **Date Range:** 2023 – 2025

### Key Columns

| Column | Description |
|---|---|
| Customer_ID | Unique identifier for each customer |
| Customer_Name | Name of the customer |
| Order_ID | Unique identifier for each order |
| Order_Date | Date the order was placed |
| Product | Name of the product ordered |
| Category | Product category (Books, Home, Clothing, Sports, Electronics) |
| Quantity | Number of units ordered |
| Price | Unit price of the product |
| Payment_Method | Payment method used (Cash on Delivery, PayPal, Bank Transfer, Credit Card) |
| Status | Current order status (Delivered, Shipped, Processing, Cancelled) |
| Total | Grand total (Quantity × Price) |

---

## 🧹 Data Cleaning (Excel)

The raw dataset contained several quality issues that were resolved before analysis:

- Dollar signs (`$`) and commas in the `Price` and `Total` columns
- Leading whitespace in column names (e.g., ` Category`, ` Customer_Name`)
- Missing values in `Price` and `Total` for some records
- Inconsistent date formatting in `Order_Date`

These issues were resolved to ensure accurate querying and downstream analysis.

---

## 🧠 Data Modelling

The dataset was reviewed to identify three logical entities within the flat file:

- **Customer** — `Customer_ID`, `Customer_Name`
- **Order** — `Order_ID`, `Customer_ID`, `Order_Date`, `Quantity`, `Price`, `Payment_Method`, `Status`, `Total`
- **Product** — `Product`, `Category`, `Price`

A Customer can place many Orders. Each Order is associated with one Product. This structure reflects how the data would be organised in a normalised relational database.

An Entity Relationship Diagram was created to document this model — see `MessyEComERD.pdf`.

---

## ❓ Business Questions

The analysis was structured around the following business questions:

1. How does revenue perform over time?
2. Which product categories drive the most revenue?
3. What are the top-performing products by revenue?
4. Who are the top customers by total spend?
5. What is the order cancellation rate?
6. Which payment methods are most frequently used?
7. Which payment methods generate the highest average order value?

---

## 💻 SQL Analysis (Databricks)

All queries were run in Databricks SQL against the cleaned CSV file using `read_files()`.

### 1. Total Revenue by Month

```sql
SELECT 
    DATE_FORMAT(DATE_TRUNC('month', Order_Date), 'yyyy-MM') AS month,
    ROUND(SUM(CAST(REPLACE(REPLACE(Total, '$', ''), ',', '') AS DOUBLE)), 2) AS total_revenue
FROM read_files('/Volumes/workspace/default/messy_e-com_sales_data/Clean_Messy_E-Com_Sales_Data.csv')
WHERE Total IS NOT NULL
GROUP BY month
ORDER BY total_revenue DESC;
```

### 2. Revenue by Product Category

```sql
SELECT ` Category` AS Category, 
       ROUND(SUM(CAST(REPLACE(REPLACE(Total, '$', ''), ',', '') AS DOUBLE)), 2) AS Total
FROM read_files('/Volumes/workspace/default/messy_e-com_sales_data/Clean_Messy_E-Com_Sales_Data.csv')
GROUP BY ` Category`
ORDER BY Total DESC;
```

### 3. Top 10 Best-Selling Products by Revenue

```sql
SELECT Product, 
       ROUND(SUM(CAST(REPLACE(REPLACE(Total, '$', ''), ',', '') AS DOUBLE)), 2) AS Total
FROM read_files('/Volumes/workspace/default/messy_e-com_sales_data/Clean_Messy_E-Com_Sales_Data.csv')
GROUP BY Product
ORDER BY Total DESC
LIMIT 10;
```

### 4. Top 10 Customers by Total Spend

```sql
SELECT ` Customer_Name` AS Customer_Name, 
       ROUND(SUM(CAST(REPLACE(REPLACE(Total, '$', ''), ',', '') AS DOUBLE)), 2) AS Total
FROM read_files('/Volumes/workspace/default/messy_e-com_sales_data/Clean_Messy_E-Com_Sales_Data.csv')
GROUP BY ` Customer_Name`
ORDER BY Total DESC
LIMIT 10;
```

### 5. Order Cancellation Rate

```sql
SELECT 
    COUNT(*) AS Total,
    COUNT(*) FILTER(WHERE Status = 'Cancelled') AS Cancelled,
    ROUND(COUNT(*) FILTER(WHERE Status = 'Cancelled') * 100 / COUNT(*), 2) AS Percentage
FROM read_files('/Volumes/workspace/default/messy_e-com_sales_data/Clean_Messy_E-Com_Sales_Data.csv');
```

### 6. Most Used Payment Methods

```sql
SELECT Payment_Method, COUNT(*) AS Total
FROM read_files('/Volumes/workspace/default/messy_e-com_sales_data/Clean_Messy_E-Com_Sales_Data.csv')
GROUP BY Payment_Method
ORDER BY Total DESC;
```

### 7. Average Order Value by Payment Method

```sql
SELECT Payment_Method, 
       ROUND(AVG(CAST(REPLACE(REPLACE(Total, '$', ''), ',', '') AS DOUBLE)), 2) AS Avg_Revenue
FROM read_files('/Volumes/workspace/default/messy_e-com_sales_data/Clean_Messy_E-Com_Sales_Data.csv')
GROUP BY Payment_Method
ORDER BY Avg_Revenue DESC;
```

---

## 📈 Key Insights

- **Books** was the highest revenue-generating category, followed by Home and Clothing
- **Comics, Shoes, and Lamp** were the top 3 best-selling products by revenue, contributing disproportionately to total sales
- The cancellation rate was approximately **15.46%**, suggesting potential inefficiencies in order fulfilment or customer experience
- **Cash on Delivery** was the most frequently used payment method, which may indicate limited trust in digital payment systems among this customer base
- Revenue showed notable peaks in early 2025, suggesting possible seasonal buying patterns

---

## 💡 Business Impact

- Identifies high-performing categories to support better inventory planning and marketing decisions
- Highlights operational inefficiencies through cancellation rate analysis
- Provides insight into customer payment preferences and potential risks around digital payment adoption
- Supports data-driven decisions around pricing, promotions, and customer targeting

---

## 📊 Power BI Dashboard

The dashboard was built on top of seven pre-aggregated SQL query results, each loaded as a separate table in Power BI. No relationships were created between tables, as each table represents a standalone analytical output.

### Visuals Included

| Visual | Chart Type |
|---|---|
| Revenue by Category | Bar Chart |
| Revenue by Month | Line Chart |
| Top 10 Products by Revenue | Horizontal Bar Chart |
| Top 10 Customers by Spend | Horizontal Bar Chart |
| Order Cancellation Rate | Gauge |
| Revenue by Payment Method | Donut Chart |
| Average Order Value by Payment Method | Bar Chart |

---

## 📂 Project Structure

[Messy-ECom-Sales dataset](Messy-ECom-Sales.xlsx)
[Cleaned E-Com sales dataset](Clean_Messy_E-Com_Sales_Data.xlsx)
[E-Com Sales ERD](MessyEComERD.pdf)
[E-Com Power BI Dashboard](E-ComDashboard.pbix)
[E-Com Dashboard Image](DashboardImage.pdf)

---

## 👤 Author

**Lawrence T Makhafola**  
Aspiring Data Analyst | South Africa  
[GitHub](#) | [LinkedIn](#)
