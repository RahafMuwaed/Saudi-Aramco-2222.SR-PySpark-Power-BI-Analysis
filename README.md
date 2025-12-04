# Saudi Aramco (2222.SR) – PySpark & Power BI Analysis

This project analyzes the historical stock performance of **Saudi Aramco (2222.SR)**
using **PySpark** for data preparation and **Power BI** for interactive visualization.

> **Note (ملاحظة):**  
> All results are based on the dataset available at the time of analysis  
> (from **2019-12-12** to **2025-12-02**). If you refresh the data in the future,
> the numbers in the dashboard will update accordingly.

---

## 1. Project Structure

```text
saudi-aramco-2222-analysis/
│
├─ notebooks/
│   └─ PySpark.ipynb
│
├─ data/
│   ├─ raw/
│   │   └─ 2222.SR.csv
│   └─ processed/
│       ├─ daily_data.csv
│       ├─ monthly_stats.csv
│       ├─ monthly_volume.csv
│       ├─ yearly_avg.csv
│       ├─ yearly_returns.csv
│       ├─ year_max_drawdown.csv
│       ├─ best_day.csv
│       ├─ worst_day.csv
│       └─ overall_metrics.csv
│
└─ powerbi/
    ├─ AramcoDashboard.png
    └─ AramcoDashboard.pdf   # optional
    # AramcoDashboard.pbix   # optional, if exported from Power BI Desktop
