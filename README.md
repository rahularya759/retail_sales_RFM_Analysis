### Sales Performance & Customer RFM Analysis
### 1. Project Overview:
---------------------------------------------------------------------------------------------------------------------------------
    This project is an end-to-end Sales Analytics and Customer Segmentation project designed to analyze sales performance, 
    customer purchasing behavior, product performance, and customer value.
    The project uses Python for data cleaning, PostgreSQL for data analysis and RFM calculations, and Power BI for interactive 
    visualization and business insights.
 **Tech stack**
- **Python / Pandas**:  Data Cleaning & Preprocessing.
- **PostgreSQL**:  Data Storage, SQL Analysis & RFM.
- **Power BI**:  Dashboard & Data Visualization.
- **Excel/CSV**: Dataset & intermediate files.
- **GitHub**:  Project Documentation & Version Control.
---------------------------------------------------------------------------------------------------------------------------------
### 2. Business Problem:

     A retail/e-commerce business generates large amounts of sales transaction data, but raw transactional data alone does not 
     provide clear answers to important business questions.
 **Actinable insight**
- Which products generate the highest revenue?
- Which countries/markets perform best?
- Which customers contribute the most revenue?
- How does sales performance change over time?
- Which customers are loyal and frequent buyers?
- Which customers are at risk of becoming inactive?
- Which customer segments should receive marketing attention?
- Which product categories generate the most revenue?     
---------------------------------------------------------------------------------------------------------------------------------
### 3.Project Objectives:
**Primary Objectives**
- 1.Clean and preprocess the raw sales dataset.
- 2.Store the cleaned dataset in PostgreSQL.
- 3.Perform exploratory and business analysis using SQL.
- 4.Calculate RFM metrics:
       Recency,
       Frequency,
       Monetary
- 5.Segment customers based on their purchasing behavior.
- 6.Identify high-value and at-risk customers.
- 7.Build an interactive Power BI dashboard.
- 8.Generate business recommendations from the analysis.
---------------------------------------------------------------------------------------------------------------------------------
### 4. Data Cleaning — Python:
The raw dataset was first cleaned using Python and Pandas.
**Cleaning operations performed**
- **1. Load the dataset**
```
import pandas as pd
df = pd.read_csv(
    "sales_data_sample.csv",
    encoding="cp1252")
```
- **2. Standardize column names**
```
df.columns = (
    df.columns
    .str.strip()
    .str.lower()
    .str.replace(" ", "_")
)  
```
- **3. Remove unnecessary spaces**
```
for col in df.select_dtypes(include="object").columns:
    df[col] = df[col].str.strip()
```
- **4.Remove duplicate records**
```
df = df.drop_duplicates()
```
- **5.Convert ORDERDATE**
```
df["orderdate"] = pd.to_datetime(
    df["orderdate"],
    errors="coerce"
)
```
- **6.Convert numerical columns**
```
numeric_columns = [
    "ordernumber",
    "quantityordered",
    "priceeach",
    "orderlinenumber",
    "sales",
    "qtr_id",
    "month_id",
    "year_id",
    "msrp"
]

for col in numeric_columns:
    df[col] = pd.to_numeric(
        df[col],
        errors="coerce"
    )
```
- **7.Check missing values**
```
df.isnull().sum()
```
- **8.Check duplicates**
```
df.duplicated().sum()
```
- **9. Export cleaned dataset**
```
df.to_csv(
    "sales_data_cleaned.csv",
    index=False
)
```
---------------------------------------------------------------------------------------------------------------------------------
### 5. PostgreSQL
The cleaned dataset was imported into PostgreSQL for further analysis.

 **PostgreSQL was used for**:
- Data storage
- Data validation
- SQL analysis
- Aggregation
- Window functions
- Customer analysis
- RFM calculation
-------------------------------------------------------------------------------------------------------------------------------- 
### 6. SQL Data Validation
- **creating table**
```
CREATE TABLE sales_data (
    ordernumber        INT,
    quantityordered    INT,
    priceeach          NUMERIC,
    orderlinenumber    INT,
    sales              NUMERIC(10,2),
    orderdate          DATE,
    status             VARCHAR(100),
    qtr_id             INT,
    month_id           INT,
    year_id            INT,
    productline        VARCHAR(100),
    msrp               NUMERIC(10,2),
    productcode        VARCHAR(100),
    customername       VARCHAR(150),
    phone              VARCHAR(100),
    addressline1       VARCHAR(200),
    addressline2       VARCHAR(200),
    city               VARCHAR(100),
    state              VARCHAR(100),
    postalcode         VARCHAR(100),
    country            VARCHAR(100),
    territory          VARCHAR(100),
    contactlastname    VARCHAR(100),
    contactfirstname   VARCHAR(100),
    dealsize           VARCHAR(100)
);
```
After importing the cleaned dataset, the following checks were performed.
- **Check total records**
```
SELECT COUNT(*)
FROM sales_data;
```
- **Check duplicate orders**
```
SELECT
    ordernumber,
    COUNT(*)
FROM sales_data
GROUP BY ordernumber
HAVING COUNT(*) > 1;
```
- **Check missing values**
```
 SELECT
    COUNT(*) FILTER (WHERE customername IS NULL) AS null_customer,
    COUNT(*) FILTER (WHERE orderdate IS NULL) AS null_orderdate,
    COUNT(*) FILTER (WHERE sales IS NULL) AS null_sales
FROM sales_data;
```
-----------------------------------------------------------------------------------------------------------------------------------------
### 7. Business Analysis Using SQL
Several business questions were answered using PostgreSQL
- **Total Revenue**
```
SELECT ROUND(SUM(sales), 2) AS total_revenue
FROM sales_data;
```
- **Revenue by Product Line**
```
SELECT
    productline,
    ROUND(SUM(sales), 2) AS revenue
FROM sales_data
GROUP BY productline
ORDER BY revenue DESC;
```
- **Revenue by Country**
```
SELECT
    country,
    ROUND(SUM(sales), 2) AS revenue
FROM sales_data
GROUP BY country
ORDER BY revenue DESC;
```
- **Monthly Sales Trend**
```
SELECT
    year_id,
    month_id,
    ROUND(SUM(sales), 2) AS monthly_sales
FROM sales_data
GROUP BY year_id, month_id
ORDER BY year_id, month_id;
```
- **Top Customers**
```
SELECT
    customername,
    ROUND(SUM(sales), 2) AS total_sales
FROM sales_data
GROUP BY customername
ORDER BY total_sales DESC
LIMIT 10;
```
-----------------------------------------------------------------------------------------------------------------------------------------
### 8. RFM Analysis
A major component of the project is RFM Customer Segmentation.

**What is RFM?**
**RFM = Recency + Frequency + Monetary**

- **Recency**
How recently did the customer purchase?
```
Lower Recency = Better
```
- **Frequency**
How often does the customer purchase?
```
Higher Frequency = Better
```
- **Monetary**
How much money has the customer spent?
```
Higher Monetary = Better
```
- **Check the date range**
```
SELECT
    MIN(orderdate) AS first_order_date,
    MAX(orderdate) AS last_order_date
FROM sales_data;
```
- **Create the customer-level RFM table**
```
CREATE TABLE customer_rfm AS
SELECT
    customername,
    (
        MAX(orderdate)::date
        - (SELECT MAX(orderdate)::date FROM sales_data)
    ) * -1 + 1 AS recency,
    COUNT(DISTINCT ordernumber) AS frequency,
    ROUND(SUM(sales), 2) AS monetary
FROM sales_data
WHERE status = 'Shipped'
GROUP BY customername;
```
- **Create RFM scores**
```
CREATE TABLE rfm_scores AS
SELECT
    customername,
    recency,
    frequency,
    monetary,
    NTILE(5) OVER (
        ORDER BY recency DESC
    ) AS r_score,

    NTILE(5) OVER (
        ORDER BY frequency
    ) AS f_score,

    NTILE(5) OVER (
        ORDER BY monetary
    ) AS m_score
FROM customer_rfm;
```
- **Create the combined RFM score**
```
CREATE TABLE rfm_final AS
SELECT
    customername,
    recency,
    frequency,
    monetary,

    r_score,
    f_score,
    m_score,

    CONCAT(
        r_score,
        f_score,
        m_score
    ) AS rfm_score,

    r_score + f_score + m_score AS rfm_total_score
FROM rfm_scores;
select * from rfm_final;
```
**Customer Segmentation**

Based on RFM scores, customers can be classified into segments such as:

**1.Champions**
High Recency + High Frequency + High Monetary
- **Action**: Reward with loyalty programs and exclusive offers.

**2.Loyal Customers**
Frequent purchasers with strong spending.
- **Action**: Encourage repeat purchases.
 
**3.Potential Loyalists**
Recent customers with moderate frequency.
- **Action**: Convert them into loyal customers.

**4.At Risk**
Previously valuable customers who haven't purchased recently.
- **Action**: Run reactivation campaigns.

**5.Lost Customers**
Low Recency, Frequency and Monetary value.
- **Action**: Consider targeted win-back campaigns.
  
- **Create customer segments**
```
CREATE TABLE customer_segments AS
SELECT
    *,
    CASE

        WHEN r_score >= 4
         AND f_score >= 4
         AND m_score >= 4
        THEN 'Champions'

        WHEN r_score >= 4
         AND f_score >= 3
        THEN 'Loyal Customers'

        WHEN r_score >= 4
         AND f_score <= 2
        THEN 'Recent Customers'

        WHEN r_score = 3
         AND f_score >= 3
        THEN 'Potential Loyalists'

        WHEN r_score <= 2
         AND f_score >= 4
         AND m_score >= 4
        THEN 'At Risk High Value'

        WHEN r_score <= 2
         AND f_score >= 3
        THEN 'At Risk'

        WHEN r_score <= 2
         AND f_score <= 2
         AND m_score >= 3
        THEN 'Hibernating High Value'

        WHEN r_score <= 2
         AND f_score <= 2
        THEN 'Lost Customers'

        ELSE 'Others'

    END AS customer_segment
FROM rfm_final;
```
- **customer segments**
```
SELECT
    customer_segment,
    COUNT(*) AS customer_count
FROM customer_segments
GROUP BY customer_segment
ORDER BY customer_count DESC;
```
- **Calculate revenue by segment**
```
SELECT
    customer_segment,
    COUNT(*) AS customers,
    ROUND(SUM(monetary), 2) AS total_revenue,
    ROUND(AVG(monetary), 2) AS avg_customer_value,
    ROUND(AVG(frequency), 2) AS avg_frequency,
    ROUND(AVG(recency), 2) AS avg_recency
FROM customer_segments
GROUP BY customer_segment
ORDER BY total_revenue DESC;
```
- **find the champions**
```
SELECT
    customername,
    recency,
    frequency,
    monetary,
    rfm_score,
    rfm_total_score
FROM customer_segments
WHERE customer_segment = 'Champions'
ORDER BY monetary DESC;
```
- **find At-risk high value customers**
```
SELECT
    customername,
    recency,
    frequency,
    monetary,
    rfm_score,
    rfm_total_score
FROM customer_segments
WHERE customer_segment = 'At Risk High Value'
ORDER BY monetary DESC;
```
- **find most loyal customer**
```
SELECT
    customername,
    frequency,
    monetary,
    recency
FROM customer_segments
ORDER BY frequency DESC
LIMIT 20;
```

- **final dataset for power bi**
```
SELECT
    customername,
    recency,
    frequency,
    monetary,
    r_score,
    f_score,
    m_score,
    rfm_score,
    rfm_total_score,
    customer_segment
FROM customer_segments
ORDER BY rfm_total_score DESC;
```
-----------------------------------------------------------------------------------------------------------------------------------------
### 9. Key Business Questions
Your project should explicitly answer these questions:
### Sales
- What is the total revenue?
- What is the monthly/quarterly sales trend?
- Which year generated the highest revenue?
- Which product line performs best?
 
### Customers
- Who are the top customers?
- Which customers generate the most revenue?
- How frequently do customers purchase?
- Which customers are at risk?

### Products
- Which product lines generate the most revenue?
- Which products have the highest sales?
- Which product categories have the highest quantity sold?
 
### Geography
- Which countries generate the highest revenue?
- Which markets have the strongest customer base?

### RFM
- Who are the Champions?
- Who are the Loyal Customers?
- Who are the At-Risk Customers?
- Which customers should be targeted for reactivation?
-----------------------------------------------------------------------------------------------------------------------------------------
### 10.Business Recommendations

Based on the analysis, recommendations can include:

- **Customer Retention**
```Focus on Champions and Loyal Customers through loyalty programs and exclusive offers.```

- **Customer Reactivation**
```Target At-Risk customers with personalized discounts and re-engagement campaigns.```

- **Product Strategy**
```Prioritize high-performing product lines while investigating weaker categories.```

- **Market Strategy**
```Focus marketing efforts on high-revenue countries and identify opportunities in lower-performing markets.```
-----------------------------------------------------------------------------------------------------------------------------------------
### 11.Project Workflow

Your GitHub project architecture can look like this:
```
Sales-RFM-Analysis/
│
├── data/
│   ├── sales_data_sample.csv
│   └── sales_data_cleaned.csv
│
├── python/
│   └── data_cleaning.py
│
├── sql/
│   └── rfm_analysis.sql
│
├── powerbi/
│   └── sales_rfm_dashboard.pbix
│
└── README.md
```
----------------------------------------------------------------------------------------------------------------------------------------
### 12. Skills Demonstrated

This project demonstrates:
- **1.Python**
- **2.Sql**
- **3.Power Bi**

-----------------------------------------------------------------------------------------------------------------------------------------
### 13.Final Project Summary

Sales Performance & Customer RFM Analysis is an end-to-end data analytics project that transforms raw transactional sales data into actionable business insights. Python and Pandas were used for data cleaning and preprocessing, PostgreSQL was used for data validation, SQL-based business analysis and RFM customer segmentation, and Power BI was used to develop interactive dashboards for sales, customer, product and market performance analysis.





