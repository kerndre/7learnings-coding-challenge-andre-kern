# 7Learnings Data Scientist Coding Challenge  
*Author: André Kern*  

This repository contains my solution to the **7Learnings Data Scientist coding challenge**.  
The task is to predict whether it will snow *tomorrow* for selected weather stations,  
with the final evaluation focusing on the day corresponding to **today − 20 years**.

---

## 📌 Project Overview

- **Data Source:** NOAA GSOD (Global Surface Summary of the Day) via Google BigQuery  
- **Stations:** 10 stations (IDs 725300–725330)  
- **Period:** 2000–2005  
- **Target:** `snow_tomorrow` (derived with `LEAD(snow)` per station)  

---

## 🔧 Workflow

1. **Data Extraction (BigQuery)**  
   - Queried daily weather records for 2000–2005.  
   - Constructed canonical date column (`YYYY-MM-DD`).  
   - Created `snow_tomorrow` label (whether it snows the next day).  

2. **Cleaning & Preprocessing**  
   - Set missing `snow_depth` values to **0** (to keep sequence continuity).  
   - Dropped columns with excessive missingness (`min_temperature`, `mean_station_pressure`, `max_gust_wind_speed`).  
   - Imputed small gaps in continuous features (interpolation / median).  
   - Filled `total_precipitation` NaNs with **0** (interpreted as no rainfall/snowfall).  

3. **Feature Engineering**  
   - Cyclical encoding of **month** (`sin`, `cos`) for seasonality.  
   - One-hot encoding for station IDs (10 dummies).  
   - Added `daily_change_snow_depth` (snow depth difference vs. previous day).  

4. **Splitting Strategy**  
   - Train: **2000–2003**  
   - Validation: **2004**  
   - Test: **exactly one day, today − 20 years** (e.g. 2005-08-29).  
   - Ensures chronological integrity and prevents leakage.  

5. **Baselines**  
   - Always predict 0 → ~93% accuracy, Recall = 0.  
   - Snow today = snow tomorrow → Recall ~19%, Precision ~20%.  

6. **Models**  
   - **Logistic Regression** (`class_weight="balanced"`)  
   - Tuned probability threshold for **F1-score** on validation set.  
   - Evaluated with Accuracy, Precision, Recall, F1, AUC.  

---

## 📊 Results

### Validation (2004)
- **Logistic Regression:** Recall ~72%, Precision ~10%, AUC ~0.62.  
- Strong recall gain compared to baselines, but many false positives.  

### Final Test Day (2005-08-29)
- Out of 10 stations, only **station 725330** had snowfall.  
- The model **correctly identified this event** (Recall = 100%).  
- False positives for 4 other stations.  

**Test-day metrics:**  
- Accuracy = 60%  
- Precision = 20%  
- Recall = 100%  
- F1 = 0.33  

---

## ⚠️ Limitations & Next Steps

- Distinguish *“true zero snow depth”* vs. *“not measured”* with an indicator column.  
- Add rolling/lagged weather features (e.g., precipitation in last 3 days).  
- Test stronger models (Random Forest, Gradient Boosted Trees).  
- Evaluate multiple “today − 20 years” dates (sliding window) for robustness.  

---

## 🛠️ Setup & Run

Clone the repository and install dependencies:

```bash
git clone https://github.com/kerndre/7learnings-coding-challenge-andre-kern.git
cd 7learnings-coding-challenge-andre-kern
pip install -r requirements.txt
````

Run the notebook in **Google Colab** (recommended) or locally with BigQuery access.

* Authenticate with GCP:

  ```bash
  gcloud auth application-default login
  ```
* Execute all cells in `Kopie von Coding Challenge.ipynb`.

---

## 🙏 Acknowledgments

* Data source: [NOAA GSOD](https://console.cloud.google.com/marketplace/details/noaa-public/gsod) via Google BigQuery
* Challenge provided by **7Learnings**
