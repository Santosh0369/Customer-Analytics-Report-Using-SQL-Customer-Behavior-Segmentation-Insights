# 📊 Customer Analytics Report Using SQL (Customer Behaviour & Segmentation Insights)

---

## 1️⃣ Executive Summary

This project focuses on building a **customer-centric analytics report** using SQL to understand customer behaviour, spending patterns, and engagement levels. By analyzing transactional data, the project identifies high-value customers, tracks purchasing activity, and segments customers into meaningful groups.

The insights generated help businesses improve customer retention, optimize marketing strategies, and increase overall revenue.

---

## 2️⃣ Business Problem

Businesses often struggle to:

* Identify their most valuable customers
* Understand customer spending behaviour
* Detect low-engagement or inactive customers
* Segment customers for targeted marketing

Without proper analysis, companies risk:

* Losing high-value customers
* Inefficient marketing campaigns
* Poor customer retention

---

# 👥 Customer Analytics Report (SQL)

## 📌 Overview

This project builds a **customer-level analytics report** using SQL by creating a view:
`gold.report_customers`.

The report consolidates transactional and customer data to generate **actionable insights**, enabling better understanding of customer behavior, segmentation, and value.

---

## 🎯 Objectives

* Combine customer and sales data into a single analytical layer
* Segment customers based on **value and engagement**
* Calculate key **business KPIs**
* Enable downstream reporting and dashboarding

---

## 🏗️ Data Model

This report is built using:

* `gold.fact_sales` → transactional data
* `gold.dim_customers` → customer details

---

## ⚙️ Key Features

### 1️⃣ Customer Profile Enrichment

* Full customer name
* Age calculation
* Unique customer identifiers

### 2️⃣ Customer Segmentation

#### 🔹 Age Groups

* Under 20
* 20–29
* 30–39
* 40–49
* 50 and above

#### 🔹 Customer Segments

* **VIP** → High value + long relationship
* **Regular** → Moderate value
* **New** → Recent customers

---

### 3️⃣ Aggregated Metrics

* 🧾 Total Orders
* 💰 Total Sales
* 📦 Total Quantity Purchased
* 🛍️ Total Unique Products
* 📅 Last Order Date
* ⏳ Customer Lifespan (months)

---

### 4️⃣ Key Performance Indicators (KPIs)

* 📊 **Recency** → Months since last purchase
* 💵 **Average Order Value (AOV)**
* 📈 **Average Monthly Spend**

---

## 🧠 SQL Logic Breakdown

### 🔹 Step 1: Base Query

* Joins fact and dimension tables
* Extracts core transactional + customer attributes
* Filters valid orders

---

### 🔹 Step 2: Customer Aggregation

* Groups data at customer level
* Computes totals and lifespan
* Identifies last purchase date

---

### 🔹 Step 3: Final Transformation

* Creates:

  * Age groups
  * Customer segments
* Calculates KPIs:

  * Recency
  * AOV
  * Monthly spend

---

## 🧾 SQL Implementation

```sql
-- Create View: gold.report_customers

IF OBJECT_ID('gold.report_customers', 'V') IS NOT NULL
    DROP VIEW gold.report_customers;
GO

CREATE VIEW gold.report_customers AS

WITH base_query AS (
    SELECT
        f.order_number,
        f.product_key,
        f.order_date,
        f.sales_amount,
        f.quantity,
        c.customer_key,
        c.customer_number,
        CONCAT(c.first_name, ' ', c.last_name) AS customer_name,
        DATEDIFF(year, c.birthdate, GETDATE()) AS age
    FROM gold.fact_sales f
    LEFT JOIN gold.dim_customers c
        ON c.customer_key = f.customer_key
    WHERE order_date IS NOT NULL
),

customer_aggregation AS (
    SELECT 
        customer_key,
        customer_number,
        customer_name,
        age,
        COUNT(DISTINCT order_number) AS total_orders,
        SUM(sales_amount) AS total_sales,
        SUM(quantity) AS total_quantity,
        COUNT(DISTINCT product_key) AS total_products,
        MAX(order_date) AS last_order_date,
        DATEDIFF(month, MIN(order_date), MAX(order_date)) AS lifespan
    FROM base_query
    GROUP BY 
        customer_key,
        customer_number,
        customer_name,
        age
)

SELECT
    customer_key,
    customer_number,
    customer_name,
    age,

    CASE 
        WHEN age < 20 THEN 'Under 20'
        WHEN age BETWEEN 20 AND 29 THEN '20-29'
        WHEN age BETWEEN 30 AND 39 THEN '30-39'
        WHEN age BETWEEN 40 AND 49 THEN '40-49'
        ELSE '50 and above'
    END AS age_group,

    CASE 
        WHEN lifespan >= 12 AND total_sales > 5000 THEN 'VIP'
        WHEN lifespan >= 12 AND total_sales <= 5000 THEN 'Regular'
        ELSE 'New'
    END AS customer_segment,

    last_order_date,
    DATEDIFF(month, last_order_date, GETDATE()) AS recency,
    total_orders,
    total_sales,
    total_quantity,
    total_products,
    lifespan,

    -- Average Order Value
    CASE 
        WHEN total_orders = 0 THEN 0
        ELSE total_sales / total_orders
    END AS avg_order_value,

    -- Average Monthly Spend
    CASE 
        WHEN lifespan = 0 THEN total_sales
        ELSE total_sales / lifespan
    END AS avg_monthly_spend

FROM customer_aggregation;
```

---

## 🚀 Use Cases

* 📊 Power BI / Tableau dashboards
* 🎯 Customer segmentation analysis
* 📈 Marketing campaign targeting
* 💡 Customer lifetime value analysis

---

## 🧩 Future Improvements

* Add **RFM segmentation (Recency, Frequency, Monetary)**
* Include **customer churn prediction features**
* Integrate with **real-time data pipelines**
* Add **geographical insights**

---

## 📌 Key Takeaways

* Transforms raw sales data into **business-ready insights**
* Demonstrates strong **SQL aggregation + analytical thinking**
* Suitable for **data analyst portfolio projects**


