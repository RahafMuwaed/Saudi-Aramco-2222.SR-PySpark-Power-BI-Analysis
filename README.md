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
All stock price data used in this project was downloaded from **Yahoo Finance**
using the *Historical Data* tab for each ticker.

Main site:
- `https://finance.yahoo.com/`

Example (Saudi Aramco – 2222.SR):
- `https://finance.yahoo.com/quote/2222.SR/history?p=2222.SR`

The CSV files in `data/raw/` (such as `2222.SR.csv`, `1120.SR.csv`, etc.)
are exported directly from Yahoo Finance without manual modification, except
for basic cleaning done later in PySpark.


> Note: The data is used for learning and academic purposes only.


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
├─ data/
│  └─ raw/
│     └─ 2222.SR.csv
├─ notebooks/
│  └─ PySpark.ipynb
└─ README.md
powerbi/
└─ 2222_Aramco_Report.pbix
```

---

## 5. PySpark Analysis

All analysis is done in:  
`notebooks/PySpark.ipynb`

Below is a summary of the main steps and what each part does.

### 5.1 Loading the Data

```python
df_prices = (
    spark.read
        .option("header", True)
        .option("inferSchema", True)
        .csv("data/raw/2222.SR.csv")
)
```

### 5.2 Cleaning & Selecting Columns

We keep only the relevant columns:
	•	Date
	•	Open, High, Low, Close
	•	Volume

We also ensure that:
	•	Date is converted to a proper date type (if needed).
	•	Numeric columns are correctly inferred as numbers (not strings).

### 5.3 Daily Returns

We calculate daily percentage return based on the closing price:
(Close_today / Close_yesterday - 1) * 100


Implementation idea:
	•	Use a Window function ordered by Date to get prev_close.
	•	Create a new column: daily_return_pct.
	

Result: DataFrame df_ret_filtered with around 1,486 rows
(the first day has no previous close, so it’s excluded from returns).

This allows us to analyze:
	•	Best and worst daily performance.
	•	Volatility patterns.
	

### 5.4 Monthly Statistics

We aggregate by Year-Month to get:
	•	Average close per month.
	•	Minimum & maximum close per month.
	•	Average daily return per month.
	•	Total monthly volume.

Result: DataFrame monthly_stats with 73 rows (each row = one month).

We also calculate monthly_volume separately if needed.


### 5.5 Yearly Averages

We aggregate by Year to get:
	•	yearly_avg_close – average closing price per year.
	•	yearly_avg_volume – average daily volume per year.

Result: DataFrame yearly_avg with 7 rows (one per year in the data).


### 5.6 Yearly Returns

We compute Year Return %:
{Year Return %} = \left( \frac{\text{Last Close in year}}{\text{First Close in year}} - 1 \right) \times 100


Steps:
	1.	For each year:
	•	Get first trading day close.
	•	Get last trading day close.
	2.	Apply the formula and create year_return_pct.

Result: DataFrame year_returns with 7 rows (Year + Year Return %).



### 5.7 Max Drawdown Per Year

Max Drawdown = the worst (largest) drop from a peak to a bottom during the year.

For each year:
	•	Track running maximum of Close.
	•	Compute drawdown:
	drawdown = (Close / running_max - 1) * 100
	

•Take the minimum drawdown (most negative) as max_drawdown_pct.

Result: DataFrame max_drawdown with 7 rows (Year + Max Drawdown %).

## 6. Power BI Dashboard

Since the device is a Mac, the dashboard is built using **Power BI Service (web)**:

1. Go to `https://app.powerbi.com`.
2. Use **My workspace** (or create a new workspace).
3. Upload the dataset `2222.SR.csv` as a new semantic model.
4. Click **Create report** and start building visuals.

---

### 6.1 Dataset in Power BI

The uploaded dataset contains (at minimum):

- `Date` 
- `Open`, `High`, `Low`, `Close` 
- `Adj Close` 
- `Volume` 

Additional calculated fields:

- **Year**: calculated column  
  `Year = YEAR([Date])`
- **Year Return %**: imported from PySpark output (or created as a measure/table  
  that holds the annual percentage return for the Close price).

---

### 6.2 Main KPIs (Cards)

We use **Card** visuals to show important summary numbers:

1. **Highest Close Price**  
   - Field: `Close`  
   - Aggregation: **Max**  
   - Meaning: highest recorded daily closing price over the whole period.

2. **Lowest Close Price**  
   - Field: `Close`  
   - Aggregation: **Min**  
   - Meaning: lowest recorded daily closing price over the whole period.

3. **Average Close Price**  
   - Field: `Close`  
   - Aggregation: **Average**  
   - Meaning: average closing price across all trading days in the dataset.

4. **Average Daily Volume**  
   - Field: `Volume`  
   - Aggregation: **Average**  
   - Meaning: average number of shares traded per day.

These four cards give a quick, high-level view of the stock behaviour.

---

### 6.3 Column Chart – Total Trading Volume by Year

- **Visual**: Clustered column chart  
- **Axis**: `Year`  
- **Values**: `Volume` with aggregation = **Sum**  

**What it shows**

- Total traded volume per year.  
- Highlights which years were the most active in terms of trading.

---

### 6.4 Donut Chart – Share of Trading Volume by Year

- **Visual**: Donut chart  
- **Legend**: `Year`  
- **Values**: `Volume` with aggregation = **Sum**

**What it shows**

- Percentage share of total trading volume for each year.  
- Quickly compares which years dominate the overall trading activity.

---

### 6.5 Column Chart – Yearly Return % for Close Price

- **Visual**: Clustered column chart  
- **Axis**: `Year`  
- **Values**: `Year Return %` 

**What it shows**

- For each year, the percentage return based on the Close price  
  (from first trading day of the year to the last).
    
- Positive bars indicate profitable years; negative bars indicate loss years.

---

### 6.6 Line Chart – Monthly Average Close Price

- **Visual**: Line chart  
- **Axis**: `Month` 
- **Values**: `Close` with aggregation = **Average**  
- **Interactivity**: filtered by the **Year slicer** 

**What it shows**

- For the selected year, how the average closing price moves from month to month.  
- Helps detect seasonal patterns or specific months with higher/lower prices.

---

### 6.7 Line Chart – Daily Average Close Price by Year

- **Visual**: Line chart  
- **Axis**: `Year`  
- **Values**: `Close` with aggregation = **Average**

**What it shows**

- Trend of the average daily closing price across years.  
- Summarises how the stock’s typical daily price evolved over time.

---

### 6.8 Slicers

To make the dashboard more interactive, we use:

1. **Year Slicer**
   - Field: `Year`
   - Type: list (radio buttons)
   - Effect: filters the monthly line chart and other visuals to a single year.

---

### Note

All results are based on the dataset available at the time of analysis  
(2019-12-12 to 2025-12-02).  
If the dataset is refreshed in the future (more rows are added), the numbers in
the dashboard will update automatically.

---

## 7. Extra Sample Datasets (Other Saudi Stocks)

For anyone who wants to reuse this project for **learning purposes**, I also included
additional CSV files for other Saudi stocks:

- `1120.SR.csv`
- `1180.SR.csv`
- `1211.SR.csv`
- `1810.SR.csv`
- `2010.SR.csv`
- `2050.SR.csv`
- `2222.SR.csv`  *(Aramco – main focus of this report)*
- `4003.SR.csv`
- `5110.SR.csv`
- `7010.SR.csv`

All these files are located in:

```text
data/raw/
```

and they share the same basic structure as 2222.SR.csv:
	•	Date
	•	Open, High, Low, Close
	•	Adj Close 
	•	Volume

You can repeat the same PySpark notebook and Power BI steps with any of these
tickers (for example, by changing the input file name) to build similar
dashboards for other companies.
