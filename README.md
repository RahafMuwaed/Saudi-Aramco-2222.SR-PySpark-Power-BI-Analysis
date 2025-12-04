# Saudi Stock Analysis with PySpark & Power BI  
### 📈 Case Study: Aramco (Ticker: 2222.SR)

## 1. Project Overview

This project analyzes **Saudi Aramco stock (2222.SR)** using:

- **PySpark** for data preparation & analytics.
- **Power BI (web)** for building an interactive dashboard.

The goal is to give a **clean, professional** view of the stock performance over the last years, answering questions like:

- How did the stock price move over time?
- What are the **highest / lowest** prices in the whole period?
- What is the **average price** and **average trading volume**?
- How does price and volume change **by year**?

> **Dataset period:** from **2019-12-12** to **2025-12-02**  
> (This is the range in the `2222.SR.csv` file used in this project.)

---

## 2. Data

### 2.1 Source

- The main dataset is a CSV file downloaded from a **public financial data website** (e.g. Tadawul/Yahoo Finance).
- File name used in the notebook:  
  `data/raw/2222.SR.csv`

### 2.2 Main Columns (Raw Data)

Typical columns in the file:

- `Date` – Trading day (e.g. `2024-01-15`).
- `Open` – Price at market open.
- `High` – Highest price during the day.
- `Low` – Lowest price during the day.
- `Close` – Closing price of the day.
- `Adj Close` – Adjusted closing price (after dividends/splits if available).
- `Volume` – Number of traded shares in that day.

These raw columns are the base for all calculations in PySpark and Power BI.

---

## 3. Tools & Technologies

- **OS:** macOS  
- **Environment:** Anaconda / Jupyter Notebook  
- **Language:** Python 3 + **PySpark**
- **Visualization:** **Power BI Service (web)**

---

## 4. Project Structure 



```text
saudi-stocks-spark/
├── data/
│   └── raw/
│       └── 2222.SR.csv        
├── notebooks/
│   └── PySpark.ipynb          
└── README.md
powerbi/
└── 2222_Aramco_Report.pbix   


## 5. PySpark Analysis

All analysis is done in:  
`notebooks/PySpark.ipynb`

Below is a summary of the main steps and what each part does.

### 5.1 Loading the Data

We create a `SparkSession` and load the CSV file:

```python
df_prices = (
    spark.read
    .option("header", True)
    .option("inferSchema", True)
    .csv("data/raw/2222.SR.csv")
)

### 5.2 Cleaning & Selecting Columns

We keep only the relevant columns:
	•	Date
	•	Open, High, Low, Close
	•	Volume

We also ensure that:
	•	Date is converted to a proper date type (if needed).
	•	Numeric columns are correctly inferred as numbers (not strings).

### 5.3Daily Returns

We calculate daily percentage return based on the closing price:

[
\text{daily_return_pct} = \left( \frac{\text{Close}\text{today}}{\text{Close}\text{yesterday}} - 1 \right) \times 100
]

Implementation idea:
	•	Use a Window function ordered by Date to get prev_close.
	•	Create a new column: daily_return_pct.

Result: DataFrame df_ret_filtered with around 1,486 rows
(the first day has no previous close, so it’s excluded from returns).

This allows us to analyze:
	•	Best and worst daily performance.
	•	Volatility patterns.
