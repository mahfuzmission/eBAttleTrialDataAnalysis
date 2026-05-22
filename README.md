# eBATTLE Obesity Study — Reproducible Analysis Pipeline

This repository contains the full data analysis pipeline for the eBATTLE obesity intervention study. The code processes Garmin wearable data from two pilot cohorts, engineers cumulative behavioural features, trains and evaluates machine learning classifiers for physical activity target prediction, and generates all thesis figures.

---

## Project Structure

```
finalCode/
├── Data/
│   ├── pilot_utrekk1_271125/   # Phase 1 cohort (LCD, 14-day monitoring)
│   └── pilot_utrekk2_271125/   # Phase 2 cohort (eBT, 28-day monitoring)
├── results_v3/                  # Pipeline outputs (auto-generated)
├── eBATTLE_Feature.ipynb        # Step 1 — Feature engineering
├── eBATTLE_Analysis.ipynb       # Step 2 — Model training and evaluation
├── generateFigures.ipynb        # Step 3 — Thesis figure generation
└── README.md
```

---

## Prerequisites

### Python Environment

The pipeline was developed and tested on **Python 3.8.20**. A dedicated virtual environment or conda environment is recommended.

**Required packages:**

```
numpy
pandas
matplotlib
seaborn
scipy
scikit-learn
xgboost
shap
pyarrow          # for Parquet I/O
jupyter
```

Install all dependencies at once:

```bash
pip install numpy pandas matplotlib seaborn scipy scikit-learn xgboost shap pyarrow jupyter
```

Or with conda:

```bash
conda create -n ebattle python=3.8
conda activate ebattle
conda install numpy pandas matplotlib seaborn scipy scikit-learn pyarrow jupyter
pip install xgboost shap
```

### Jupyter Kernel

Register the environment as a Jupyter kernel before running the notebooks:

```bash
pip install ipykernel
python -m ipykernel install --user --name ebattle --display-name "eBATTLE"
```

When opening each notebook, select the **eBATTLE** kernel from the kernel menu.

---

## Data Requirements

Each cohort folder under `Data/` must contain the Garmin Connect export files. The pipeline reads the following file for each participant:

| File | Contents |
|------|----------|
| `daily-health-log.csv` | Primary source: daily HR, steps, stress, sleep, HRV |
| `dailies.csv` | Supplementary daily summary |
| `body-composition.csv` | Weight and BMI measurements |
| `activities.csv` | Logged exercise sessions |

All other Garmin export files in the folder are ignored.

---

## Execution Order

The notebooks must be run in the following sequence. Each notebook depends on the outputs of the previous one.

> **No path configuration required.** All three notebooks use `Path().resolve()` to detect the `finalCode/` directory automatically at runtime. This works as long as Jupyter is launched from inside the `finalCode/` folder (see [Starting Jupyter](#starting-jupyter) below).

### Step 1 — Feature Engineering

**Notebook:** `eBATTLE_Feature.ipynb`

Loads raw Garmin CSVs from both cohorts, constructs the daily panel, computes rolling and cumulative physiological features, and writes the feature matrix to `results_v3/`.

1. Open `eBATTLE_Feature.ipynb` in Jupyter.
2. Confirm the kernel is set to **eBATTLE** (or your Python 3.8 environment).
3. Run all cells: **Kernel → Restart & Run All**.
4. Expected outputs in `results_v3/`:
   - `daily_panel_v3_<date>.parquet`
   - `feature_dataframe_v3_<date>.parquet`
   - `cumul_features_cache.parquet`
   - `data_quality_report.csv`
   - `weight_validation_merged.csv`

Estimated runtime: ~2–5 minutes depending on hardware.

---

### Step 2 — Model Training and Evaluation

**Notebook:** `eBATTLE_Analysis.ipynb`

Reads the feature matrix produced in Step 1, trains Logistic Regression, Random Forest, Gradient Boosting, and XGBoost classifiers using stratified group cross-validation, evaluates calibration and discrimination, computes SHAP feature importance, and writes all results to `results_v3/`.

1. Open `eBATTLE_Analysis.ipynb` in Jupyter.
2. Confirm the kernel is set to **eBATTLE**.
3. Run all cells: **Kernel → Restart & Run All**.
4. Expected outputs in `results_v3/`:
   - `feature_importance.csv`
   - `feature_importance.png`
   - `feature_type_distribution.png`
   - `target_comparison.png`
   - `cumulative_results.csv`

Estimated runtime: ~10–20 minutes (cross-validation with hyperparameter search).

---

### Step 3 — Figure Generation

**Notebook:** `generateFigures.ipynb`

Reads the parquet panel and feature importance CSV from `results_v3/` and renders all publication-quality figures at 300 DPI. The most recent `daily_panel_v3_*.parquet` file is picked up automatically.

1. Open `generateFigures.ipynb` in Jupyter.
2. Confirm the kernel is set to **eBATTLE**.
3. Run all cells: **Kernel → Restart & Run All**.
4. Figures are saved to `figures/` inside `finalCode/` at 300 DPI.

---

### Starting Jupyter

All paths in the notebooks are resolved relative to the working directory of the Jupyter kernel. Launch Jupyter from inside `finalCode/` to ensure paths resolve correctly:

```bash
cd /path/to/finalCode
jupyter notebook
# or
jupyter lab
```

If you open the notebooks from a different directory, `Data/` and `results_v3/` will not be found and the notebooks will fail at the data-loading step.

---

## Reproducibility Notes

- **Random seed**: All stochastic components (cross-validation splits, hyperparameter search, XGBoost) use `random_state=42`.
- **Patient-level splitting**: `StratifiedGroupKFold` is used throughout to ensure no participant appears in both training and test folds within the same split.
- **Cached features**: `cumul_features_cache.parquet` is written on first run of Step 1. Subsequent runs load from cache automatically. Delete this file to force recomputation from raw CSVs.
- **Date-stamped outputs**: Feature matrix files include a date suffix (e.g., `feature_dataframe_v3_20260522.parquet`). Step 2 reads the most recent file matching the pattern automatically.

---

## Targets

| Target | Definition | WHO Guideline |
|--------|-----------|---------------|
| `mvpa_420` | ≥ 420 min/week moderate-to-vigorous PA | Paediatric (5–17 years) |
| `mvpa_150` | ≥ 150 min/week moderate-to-vigorous PA | Adult (18+ years) |

---

## Contact

For questions regarding the data pipeline or analysis methodology, refer to the thesis manuscript or contact the author directly.
