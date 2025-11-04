# Retail Sales Analysis:

## Project Overview:

This project presents a **comprehensive Power BI dashboard** built to analyze a UK online and retail sales dataset. The analysis provides key performance indicators (KPIs) and actionable insights across revenue, customer behavior (including **RFM segmentation**), product performance, and sales trends to drive strategic business decisions.

---

## Business Questions & Key Insights:

The dashboard is designed to answer critical business questions and visualize the following:

* **Total Revenue (Sales):** The overall gross sales generated.
* **Country-Wise Total Revenue:** Breakdown of revenue contribution by country.
* **Top 5 Products:** Identification of the top-performing products based on **Quantity Sold**.
* **Unique Customers:** The total count of distinct customers who made a purchase.
* **Monthly Sales Trend & MoM Growth:** Analysis of monthly revenue movement and the calculation of Month-over-Month (MoM) growth percentage.
* **Cancelled Orders Analysis:** Total revenue and the percentage of total transactions represented by 'Cancelled' orders (indicated by 'C' in `TransactionNo`).
* **Busiest Day of the Week:** Determination of the day with the highest **volume of unique transactions**.
* **RFM Segmentation:** Classification of customers based on Recency, Frequency, and Monetary scores, highlighting the **Total Revenue contributed by the 'Champions' segment** (highest RFM scores).
* **Month Over Month Customer Retention Rate:** Tracking customer loyalty over time.

---

## Data Source:

The analysis is based on the **UK Online and Retail Sales Dataset**, which comprises **536,350 rows** and **8 columns**.

### Features:

| Column Name | Type | Description |
| :--- | :--- | :--- |
| `TransactionNo` | Categorical | Six-digit unique transaction number. **'C' indicates a cancellation.** |
| `Date` | Numeric | Date of the transaction. |
| `ProductNo` | Categorical | Unique identifier for a product. |
| `Product` | Categorical | Product/item name. |
| `Price` | Numeric | Unit price in **Pound Sterling (£)**. |
| `Quantity` | Numeric | Quantity of product per transaction. **Negative values indicate cancellations.** |
| `CustomerNo` | Categorical | Unique customer identifier. |
| `Country` | Categorical | Country where the customer resides.

> **Note on Cancellations:** A small percentage of cancellations is present, often due to out-of-stock conditions where customers prefer to cancel the entire order rather than receive a partial delivery.

---

## Data Cleaning & Preparation:

### Python/Power Query Implementation:

A crucial step in the data preparation involved handling missing customer IDs. This was addressed using a custom **Python script integrated with Power Query**:

* **Goal:** Impute missing `CustomerNo` values specifically within the **'United Kingdom'** (the only country with nulls).
* **Method:** The null `CustomerNo` values were imputed by randomly assigning them a `CustomerNo` from the **top 15 most frequent customers in the UK** (by transaction count), ensuring the integrity of the transaction data was preserved.

```python code:
# Custom Python code for imputing null CustomerNo (integrated via Power Query)
subset = dataset[(dataset['Country'].str.lower() == "united kingdom") & (dataset['CustomerNo'].notna())]
top_customers = subset.groupby('CustomerNo')['TransactionNo'].nunique().nlargest(15).index
# ... mapping and assignment logic followed (as provided in the input)
````

-----

## Data Modeling:

The original flat sales table was transformed into a **Star Schema** model to optimize performance and analysis within Power BI:

  * **Fact Table:** The main **Sales** table (8 columns).
  * **Dimension Tables:** **Products** and **Customer** master tables were derived from the Sales data.
  * **Calendar Table:** A custom **DAX-generated Calendar Table** was created for robust time-intelligence calculations (e.g., MoM growth, moving averages).
  * **Relationships:** **1-to-Many relationships** were established from the master tables to the Sales fact table.

-----

## Key Features of the Dashboard:

### 1\. Advanced Drill-Through Functionality:

The dashboard employs drill-throughs to facilitate deeper analysis:

  * **KPIs to Moving Average Analysis:**
      * **Source:** Summary and KPIs Page (Total Revenue, MoM Revenue %).
      * **Target:** **Moving Avg Analysis** Page, which dynamically calculates the **N-day Moving Average for Revenue and Quantity Sold**. The 'N' value is adjustable via a slicer (1 to 10) for a specific month.
  * **Customer Segmentation to RFM Details:**
      * **Source:** Customer Analysis Page (Matrix of Customer Segment and Count).
      * **Target:** **RFM Details** Page, showing the individual customer records, scores, and RFM attributes for the selected segment.

### 2\. RFM (Recency, Frequency, Monetary) Segmentation

Customer segmentation was performed using a **Python script integrated into Power Query** for custom scoring based on quantiles:

  * **Scoring Logic:** Recency, Frequency, and Monetary scores are assigned based on custom quantile thresholds (20th, 40th, 60th, 80th percentiles).
      * **Recency:** Scored **inversely** (lower days since last purchase = higher score).
      * **Frequency & Monetary:** Scored **directly** (higher value = higher score).
  * **Segmentation:** A combined **Frequency\_Monetary\_Score** and the **Recency Score** are used to assign customers to one of the following segments:
      * Champions, Loyal Customers, New Comers, Premium Customers at Risk, Moderate Value Customers, Low Engagement Customers, Dormant Customers, and Lost Customers.

<!-- end list -->

```python code:
# Custom Python code for RFM Segmentation (integrated via Power Query)
# ... calculates Revenue, Days_Since_Last_Purchase
# ... applies rank-based scoring using custom quantiles (0.2, 0.4, 0.6, 0.8)
# ... assigns final customer segments based on combined scores
```

