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

* Source: [Kaggle UK Online and Retail Sales Dataset](https://www.kaggle.com/datasets/gabrielramos87/an-online-shop-business)
* Size: 5,36,350 rows and 8 columns

### Features:

| Feature Name | Description |
| :--- | :--- |
| TransactionNo | Six-digit unique transaction number. C indicates a cancellation. |
| Date | Date of the transaction. |
| ProductNo | Unique identifier for a Product. |
| Product | Product Name. |
| Price | Unit price in Pounds (£). |
| Quantity | Quantity of product per transaction. |
| CustomerNo | Unique customer Identifier. |
| Country | Country where the Customer Resides.

> **Note on Cancellations:** A small percentage of cancellations is present, often due to out-of-stock conditions where customers prefer to cancel the entire order rather than receive a partial delivery.

---

## Data Cleaning & Preparation:

### Python/Power Query Implementation:

A crucial step in the data preparation involved handling missing customer IDs. This was addressed using a custom **Python script integrated with Power Query**:

* Goal: Impute missing CustomerNo values specifically within the **'United Kingdom'** (UK is the only country with nulls).
* Method: The null CustomerNo values were imputed by randomly assigning them a CustomerNo from the **Top 15 most frequent customers in the UK** (By Transaction Counts), ensuring the integrity of the transaction data was preserved.
* Justification: This method is preferred to the traditional mode imputation to avoid creating bias in the data set. This solution maintains some degree of randomness by selecting 1 customer randomly from the top 15 customers using simple random sampling.

```python code:
# Custom Python code for imputing null CustomerNo (integrated via Power Query)
# 'dataset' holds the input data for this script
dataset.drop_duplicates(inplace = True, ignore_index = True)
# Finding out all the transaction nos will null customers
null_transactions = dataset[dataset.isna().any(axis = 1)]['TransactionNo'].unique()
# UK is the only country with null customer values
subset = dataset[(dataset['Country'].str.lower() == "united kingdom") & (dataset['CustomerNo'].notna())]
top_customers = subset.groupby('CustomerNo')['TransactionNo'].nunique().nlargest(15).index
trans_cust_mapping = pd.Series(np.random.choice(top_customers, size = len(null_transactions)), index = null_transactions).to_dict()
dataset.loc[dataset['CustomerNo'].isna(), 'CustomerNo'] = dataset.loc[dataset['CustomerNo'].isna(), 'TransactionNo'].map(trans_cust_mapping)
````
-----

## Data Modeling:

The original flat sales table was transformed into a **Star Schema** model to optimize performance and analysis within Power BI:

  * **Fact Table:** The main Sales table.
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

### 2\. RFM (Recency, Frequency, Monetary) Segmentation:

Customer segmentation was performed using a **Python script integrated into Power Query** for custom scoring based on quantiles:

  * **Scoring Logic:** Recency, Frequency, and Monetary scores are assigned based on custom quantile thresholds (20th, 40th, 60th, 80th percentiles).
      * **Recency:** Scored **inversely** (lower days since last purchase = higher score).
      * **Frequency & Monetary:** Scored **directly** (higher value = higher score).
  * **Segmentation:** A combined **Frequency\_Monetary\_Score** and the **Recency Score** are used to assign customers to one of the following segments:
      * Champions, Loyal Customers, New Comers, Premium Customers at Risk, Moderate Value Customers, Low Engagement Customers, Dormant Customers, and Lost Customers.
   
 Note: This scoring logic gives higher weights to **Recency Score** over **Monetary and Frequency Scores**.

<!-- end list -->

```python code:
# Custom Python code for RFM Segmentation (integrated via Power Query)

# 'dataset' holds the input data for this script

dataset['Revenue'] = dataset['Quantity'] * dataset['Price']
dataset['Date'] = pd.to_datetime(dataset['Date'])
df = dataset.groupby(['CustomerNo', 'Country'])[['TransactionNo', 'Revenue', 'Date']].agg({'TransactionNo': 'nunique', 
'Revenue': 'sum', 'Date': 'max'}).reset_index().rename(columns = {'TransactionNo': 'Total_Purchases'})
df['Days_Since_Last_Purchase'] = (dataset['Date'].max() - df['Date']).dt.days
df.drop(['Date'], axis = 1, inplace = True)
q1, q2, q3, q4 = 0.2, 0.4, 0.6, 0.8

# Frequency Scores
freq_cond = [
    df.Total_Purchases.rank(method = 'dense', pct = True) <= q1,
    df.Total_Purchases.rank(method = 'dense', pct = True) <= q2,
    df.Total_Purchases.rank(method = 'dense', pct = True) <= q3,
    df.Total_Purchases.rank(method = 'dense', pct = True) <= q4
    ]
freq_choice = [5, 4, 3, 2]
df['Frequency Score'] = np.select(freq_cond, freq_choice, 1)


# Monetary Scores
rev_cond = [
    df.Revenue.rank(method = 'dense', pct = True) <= q1,
    df.Revenue.rank(method = 'dense', pct = True) <= q2,
    df.Revenue.rank(method = 'dense', pct = True) <= q3,
    df.Revenue.rank(method = 'dense', pct = True) <= q4
    ]
rev_choice = [5, 4, 3, 2]
df['Monetary Score'] = np.select(rev_cond, rev_choice, 1)

# Recency Scores
rec_cond = [
    df.Days_Since_Last_Purchase.rank(method = 'dense', pct = True) <= q1,
    df.Days_Since_Last_Purchase.rank(method = 'dense', pct = True) <= q2,
    df.Days_Since_Last_Purchase.rank(method = 'dense', pct = True) <= q3,
    df.Days_Since_Last_Purchase.rank(method = 'dense', pct = True) <= q4
    ]
rec_choice = [1, 2, 3, 4]
df['Recency Score'] = np.select(rec_cond, rec_choice, 5)


# FM Score Combined

df['Frequency_Monetary_Score'] = df['Frequency Score'] + df['Monetary Score']


# Customer Segmentation

seg_cond = [
(df['Recency Score'].between(1, 2)) & (df['Frequency_Monetary_Score'].between(2, 4)),
(df['Recency Score'].between(1, 2)) & (df['Frequency_Monetary_Score'].between(5, 7)),
(df['Recency Score'].between(1, 2)) & (df['Frequency_Monetary_Score'].between(8, 10)),
(df['Recency Score'].between(3, 4)) & (df['Frequency_Monetary_Score'].between(2, 4)),
(df['Recency Score'].between(3, 4)) & (df['Frequency_Monetary_Score'].between(5, 7)),
(df['Recency Score'].between(3, 4)) & (df['Frequency_Monetary_Score'].between(8, 10)),
(df['Recency Score'] == 5) & (df['Frequency_Monetary_Score'].between(2, 5))
]

seg_choice = ["Champions", "Loyal Customers", "New Comers", "Premium Customers at Risk", "Moderate Value Customers", "Low Engagement Customers", "Dormant Customers"]


df['Customer Segment'] = np.select(seg_cond, seg_choice, "Lost Customers")

```


---
---

## Key Visuals:

<h3 align="center"><strong>Data Model</strong></h3>
<br>
<p align="center">
  <img src="Key Visuals/Data Model.jpg" alt="Data Model">
</p>
<hr>


<h3 align="center"><strong>Report Page 1: Summary and KPIs</strong></h3>
<br>
<p align="center">
  <img src="Key Visuals/Page 1.jpg" alt="Summary and KPIs">
</p>
<hr>

<h3 align="center"><strong>Report Page 2: Customer Analysis</strong></h3>
<br>
<p align="center">
  <img src="Key Visuals/Page 2.jpg" alt="Customer Analysis">
</p>
<hr>

<h3 align="center"><strong>Report Page 3: Operations</strong></h3>
<br>
<p align="center">
  <img src="Key Visuals/Page 3.jpg" alt="Operations">
</p>
<hr>

<h3 align="center"><strong>Report Page 4: RFM Details</strong></h3>
<br>
<p align="center">
  <img src="Key Visuals/Page 4.jpg" alt="RFM Details">
</p>
<hr>

<h3 align="center"><strong>Report Page 5: Moving Avg</strong></h3>
<br>
<p align="center">
  <img src="Key Visuals/Page 5.jpg" alt="Moving Avg">
</p>
<hr>

---







