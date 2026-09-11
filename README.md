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
- **Why PostgreSQL?**
 PostgreSQL was used for:
- Data storage
- Data validation
- SQL analysis
- Aggregation
- Window functions
- Customer analysis
- RFM calculation
-------------------------------------------------------------------------------------------------------------------------------- 


















