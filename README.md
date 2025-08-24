# 7Learnings Data Science Coding Challenge

This repository contains my solution for the **7Learnings Data Scientist coding challenge**.  
The goal of the challenge is to forecast whether it will snow *tomorrow* for a given set of weather stations,  
with the final evaluation focusing on the test day corresponding to **today − 20 years**.

---

## 📌 Project Structure

```

├── notebook.ipynb          # Main Jupyter/Colab notebook with full workflow
├── coding\_challenge.csv    # Provided backup dataset (not used in Part 1)
├── requirements.txt        # Dependencies
└── README.md               # This file

````

---

## 🚀 Workflow

1. **Data Access (BigQuery)**  
   - Queried NOAA GSOD data from `bigquery-public-data.samples.gsod`.  
   - Restricted to stations **725300–725330** and years **2000–2005**.  
   - Constructed canonical date column (`YYYY-MM-DD`).

2. **Data Cleaning**  
   - Replaced missing snow depth with `0` to preserve time-series continuity.  
   - Dropped columns with excessive missingness or little predictive value.  
   - Imputed small gaps in continuous features (interpolation/median).  
   - Added `snow_tomorrow` label with a window function (`LEAD`).

3. **Feature Engineering**  
   - **Cyclical month encoding**: `month_sin`, `month_cos`.  
   - **Station one-hot encoding** to capture station-specific effects.  
   - **Daily snow-depth change** feature.  

4. **Time-Aware Split**  
   - Train: **2000–2003**  
   - Validation: **2004**  
   - Test: **exactly today − 20 years (2005-08-29)**  

5. **Baselines**  
   - Always predict "no snow" → high accuracy, 0 recall.  
   - "Snow today = snow tomorrow" → slightly better recall, low precision.  

6. **Modeling**  
   - **Logistic Regression** with class balancing + probability threshold tuning.  
   - **Random Forest** as a non-linear benchmark.  
   - Optimized decision threshold for F1 on validation data.  

7. **Results**  
   - Logistic Regression achieved **~72% recall** on validation, capturing most snow days.  
   - On the final test day (2005-08-29), the model correctly predicted the only true snowfall event,  
     achieving **100% recall**, though at the cost of false positives.  

---

## 📊 Final Test-Day Results (2005-08-29)

| Station | True Snow Tomorrow | Predicted Probability | Prediction |
|---------|-------------------|-----------------------|------------|
| 725330  | 1                 | 0.86                  | 1 ✅ |
| others  | 0                 | 0.00–0.68             | some false positives |

- **Accuracy:** 60%  
- **Precision:** 20%  
- **Recall:** 100%  
- **F1:** 0.33  

---

## ⚠️ Limitations & Next Steps

- Distinguishing *“no snow depth reported”* vs. *“true zero”* could improve labeling.  
- Rolling/lagged weather features (e.g., precipitation in last 3 days) could improve signal.  
- Gradient Boosted Trees (XGBoost/LightGBM) could be tested for stronger performance.  
- Evaluate not just one test day but multiple “today − 20 years” days for robustness.

---

## 🛠️ Setup & Run

Clone the repository and install dependencies:

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
pip install -r requirements.txt
````

Run the notebook in **Google Colab** or locally (with BigQuery access):

* **Google Cloud setup**:

  * Create a project in GCP.
  * Run `gcloud auth application-default login` to authenticate.
* Open `notebook.ipynb` and execute all cells.

---

## 🙏 Acknowledgments

* Data source: [NOAA GSOD](https://console.cloud.google.com/marketplace/details/noaa-public/gsod) via Google BigQuery.
* Challenge provided by **7Learnings**.
