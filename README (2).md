# Predictive Paradox — Power Demand Forecasting

> **IITG.ai Recruitment Task** | IIT Guwahati AI Club  
> *"Unravel the Predictive Paradox."*

A classical machine learning pipeline that forecasts Bangladesh's national grid electricity demand one hour ahead. Built using LightGBM with engineered temporal features — no deep learning, no autoregressive packages.

---

## Result

| Metric | Value |
|--------|-------|
| **Test MAPE** | ~2.04% |
| Model | LightGBM Regressor |
| Split strategy | Chronological 80/20 (no shuffle) |
| Features engineered | 18 |

---

## Repository Structure

```
predictive-paradox/
│
├── Predictive_Paradox_Pipeline.ipynb   # Main notebook — full end-to-end pipeline
├── Predictive_Paradox_Report.pdf       # Technical report (this document)
├── README.md                           # This file
│
└── data/                               # Place your data files here
    ├── PGCB_date_power_demand.xlsx
    ├── weather_data(Sheet1).csv
    └── economic_full_1.csv
```

---

## Datasets

| File | Description |
|------|-------------|
| `PGCB_date_power_demand.xlsx` | Hourly demand & generation data. Target variable source. Contains real-world anomalies. |
| `weather_data(Sheet1).csv` | Hourly temperature, humidity, precipitation. Has 3 metadata header rows. |
| `economic_full_1.csv` | Annual World Bank macroeconomic indicators. 1500+ indicators in wide format. |

---

## Pipeline Overview

```
Raw Data
   │
   ├── PGCB .xlsx  ──► try openpyxl / fallback xlrd ──► parse datetime ──► sort
   ├── Weather .csv ──► skiprows=3 ──► parse 'time' column
   └── Economic .csv ──► filter GDP|Population|Industry ──► melt ──► pivot by year
          │
          ▼
    Data Cleaning
      • pd.to_numeric(errors='coerce') on demand_mw
      • Linear interpolation for missing hours
      • Rolling 168h IQR anomaly detection → replace spikes with rolling median
          │
          ▼
    Merge
      • Power + Weather on datetime index (left join)
      • Add year column → merge economic data by year
      • ffill + bfill residual NaNs
          │
          ▼
    Feature Engineering
      • Calendar: hour, dayofweek, month, is_weekend
      • Lags: lag_1, lag_2, lag_24, lag_48
      • Rolling: rolling_24h_mean (on shifted series — no leakage)
      • Target: demand_mw.shift(-1) → target_demand_mw
      • Drop NaN rows (first 48 rows + last row)
          │
          ▼
    Train / Test Split
      • Sorted chronologically
      • First 80% → training set
      • Last 20%  → hold-out test set
      • shuffle = False (strictly enforced)
          │
          ▼
    LightGBM Regressor
      • n_estimators=200, learning_rate=0.05, random_state=42
      • Trained only on training set
          │
          ▼
    Evaluation
      • MAPE on test set ≈ 2.04%
      • Feature importance plot (gain-based)
```

---

## Key Design Decisions

### Why Rolling IQR for anomaly detection?
Power demand has strong daily and seasonal cycles. A global Z-score would flag normal peak hours as outliers because the global mean is pulled by seasonal variation. The 168-hour (one week) rolling window computes local bounds that adapt to the current demand regime. The rolling median is used as the replacement value because it is resistant to the very spikes being corrected.

### Why lag_24 and lag_48?
These are the most powerful temporal features after lag_1. Demand at the same hour yesterday (lag_24) captures the full daily pattern implicitly — a Monday 8pm reading compared to the previous Monday 8pm is more informative than any explicit encoding of "hour" alone. Lag_48 adds a second reference point for multi-day trends.

### Why no future leakage in rolling_24h_mean?
The rolling mean is computed on `demand_mw.shift(1)`, not on raw `demand_mw`. Without the shift, the rolling window at time t would include `demand_mw[t]` — the value we are trying to predict. Shifting ensures the window ends at t-1.

### Why annual economic data at hourly granularity?
GDP and industrial output capture the long-term upward drift in Bangladesh's electricity consumption. Without these, the model sees only cyclical patterns and may systematically underestimate demand in later years where the baseline has grown. Each hourly row receives the indicator value for its calendar year.

### Why chronological split and not k-fold?
Standard k-fold cross-validation randomly shuffles the dataset, scattering future observations into the training set. Since lag features depend on past values, a future row in training would effectively "know" about a past row in test — severe data leakage. Chronological splitting is the only valid evaluation strategy for time series.

---

## Running the Notebook

### Option 1 — Google Colab (recommended)

1. Open [colab.research.google.com](https://colab.research.google.com)
2. File → Upload notebook → select `Predictive_Paradox_Pipeline.ipynb`
3. In the left sidebar, click the folder icon → upload all three data files
4. Runtime → Run all

### Option 2 — Local

```bash
pip install lightgbm openpyxl xlrd pandas numpy matplotlib scikit-learn jupyter

jupyter notebook Predictive_Paradox_Pipeline.ipynb
```

Place the three data files in the same directory as the notebook before running.

---

## Dependencies

```
pandas
numpy
matplotlib
scikit-learn
lightgbm
openpyxl
xlrd
```

---

## Constraints Respected

- No deep learning architectures (no LSTMs, Transformers)
- No autoregressive packages (no ARIMA, no Prophet)
- Zero data leakage: all lag and rolling features use only past values
- Evaluation strictly on chronological hold-out test set
- MAPE used as primary evaluation metric

---

*IITG.ai Recruitment Task | Predictive Paradox | IIT Guwahati*
