# ⚡ Predictive Paradox — Electricity Demand Forecasting

> **Can a computer predict how much electricity Bangladesh will need in the next hour?**
> This project answers that question — with 98% accuracy.

---

## 🎯 What This Project Does

Every hour, Bangladesh's power grid needs to know how much electricity to generate.
Too much = wasted money. Too little = blackouts.

This project builds a **machine learning model** that looks at the last few hours of electricity usage, the weather, and economic data — then predicts what demand will be **one hour from now**.

**Final Result: ~2% error on data the model had never seen before.**

---

## 📂 Data Used

| Data Source | What It Contains |
|---|---|
| 🔌 **Power Grid Records** | Hour-by-hour electricity demand across multiple years |
| 🌤️ **Weather Data** | Temperature, humidity, and rainfall for each hour |
| 📊 **Economic Indicators** | Bangladesh's GDP, population, and industrial output per year |

---

## 🔄 How It Works — In Plain English

### Step 1 — Clean the Data
The raw electricity data had **missing hours** and **sudden impossible spikes** (like demand randomly doubling for one hour). These were detected and fixed automatically before any analysis.

### Step 2 — Combine Everything
The three datasets were merged into one big table — every row is one hour, with columns for demand, weather, and economic context all in the same place.

### Step 3 — Give the Model a Sense of Time
The machine learning model doesn't automatically understand that 11pm and midnight are close together, or that demand patterns repeat every 24 hours. So this context was manually built in as extra columns:

- 🕐 **What time is it?** (encoded as a smooth circular number, not just "hour 23")
- 📅 **What day of the week is it?** (weekends have lower industrial demand)
- 🌡️ **What was the temperature 1–2 hours ago?**
- ⚡ **What was demand 1, 2, 24, and 48 hours ago?**
- 📈 **Is demand currently rising or falling?**

### Step 4 — Train Without Cheating
The model was trained **only on past data**. It never saw future hours during training — which is how it works in the real world. The last 20% of the timeline was locked away as a test.

### Step 5 — Make Predictions
A **LightGBM** model (a fast, powerful decision-tree algorithm) was trained on the historical data and then tested on the locked-away future data.

---

## 📈 Result

```
Test MAPE ≈ 2.04%
```

This means the model's predictions were **off by only 2% on average** — on data it had never seen.

For reference: professional electricity forecasting systems typically aim for 1–5% error.

---

## 🗂️ Files in This Repository

```
📁 predictive-paradox/
│
├── 📓 Predictive_Paradox_Nilesh.ipynb   ← The full analysis notebook
├── 📄 README.md                          ← This file
│
└── 📁 data/                              ← Put your data files here
    ├── PGCB_date_power_demand.xlsx
    ├── weather_data(Sheet1).csv
    └── economic_full_1.csv
```

---

## 🚀 How to Run It

1. Go to [colab.research.google.com](https://colab.research.google.com)
2. Upload `Predictive_Paradox_Nilesh.ipynb`
3. Upload the 3 data files using the folder icon on the left sidebar
4. Click **Runtime → Run All**

That's it. No setup required.

---

## 🛠️ Tools Used

| Tool | Purpose |
|---|---|
| `Python` | Programming language |
| `pandas` | Loading and cleaning data |
| `LightGBM` | The prediction model |
| `matplotlib` | Charts and visualisations |
| `scikit-learn` | Measuring prediction accuracy |

---

## ⚠️ Rules Followed

This project was built under strict constraints set by the IITG.ai recruitment task:
- ✅ Only classical ML allowed — no AI neural networks
- ✅ Model never trained on future data
- ✅ Accuracy measured using MAPE on an unseen test set

---

*Built for the IITG.ai Recruitment Task — Predictive Paradox | IIT Guwahati*
