# RandomForestGenerator

Single-page, in-browser Random Forest trainer for CSV datasets. Everything runs client-side: upload a CSV, pick a target, train a Random Forest, inspect results/diagnostics, then export a reusable artifact.

## Quick Start
- Open `RFG.html` in a modern browser.
- If the browser blocks ES module imports when using `file://`, run a tiny local server:
```sh
python -m http.server
```
- Navigate to `http://localhost:8000/RFG.html`.
- Load the included sample dataset `titanic.csv` to smoke-test the flow.

## Features
- CSV parsing + basic inspection (types, missingness, uniqueness hints).
- Classification or regression with random or time-based split.
- Automatic preprocessing for mixed tabular data (continuous/categorical/date).
- Training progress UI with early stop support.
- Results: metrics + charts, feature importance, shift diagnostics, and uncertainty.
- Export: a zip containing a serialized model + preprocessing pipeline + example loader + predictions CSV.

## Workflow (in the UI)
- **Upload**: drop in a CSV; preview rows and inferred column metadata.
- **Target**: choose the target column; optionally set task mode and target transform.
- **Train**: configure RF settings (trees, maxFeatures, seed, replacement) and split strategy.
- **Results**: inspect metrics, feature importance, shift detector, and prediction diagnostics.
- **Export**: download `rf_artifact.zip` for reuse/sharing.

## Modeling details
### Task types
- **Regression**: target is numeric (optionally with `log1p` transform).
- **Classification**: target is treated as discrete labels and encoded to integer class IDs.

### Split strategies
- **Random split**: standard train/test split using a fixed seed for reproducibility.
- **Time-based split**: uses a chosen date column to split chronologically (useful for leakage/shift checks).

### Preprocessing (fit on train split only)
- **Continuous columns**: median imputation + a missing-indicator feature (`_na`).
- **Categorical columns**: label encoding; missing/unknown categories map to ID `0`.
- **Date columns**: expanded into date parts (year/month/week/day/etc.) + a missing-indicator (`_na`).

## Diagnostics
### OOB (out-of-bag) metrics
When available, the app computes OOB predictions and reports OOB metrics (regression: RMSE/RMSLE; classification: accuracy/macro-F1). OOB vs test gaps can be a quick generalization sanity check.

### Shift detector (“is_valid” model)
Trains a classifier to distinguish train vs test rows (balanced 50/50). Strong performance suggests distribution shift between the splits; the UI shows feature importance to help identify the drivers.

### Uncertainty (fastai-style tree disagreement)
Uses per-tree predictions via `ml-random-forest`’s `predictionValues(toPredict)`:
- **Regression**: per-row standard deviation across tree predictions (`tree_std`). Higher = trees disagree more.
- **Classification**: agreement-based measures, including `vote_frac_mode` (fraction of trees voting for the modal class), plus `vote_margin` and `vote_entropy`. The primary uncertainty signal shown is `uncertainty = 1 - vote_frac_mode`.

In Results → Predictions explorer:
- **Worst errors**: top-k rows by absolute error.
- **Highest uncertainty**: top-k rows by tree disagreement.
- **Scatter**: `abs(error)` vs `uncertainty` (a useful leakage/shift sniff-test: highly uncertain + highly wrong rows often indicate problematic segments).

## Predictions CSV
The “Download predictions CSV” button (and the export zip) includes your **test split** with extra columns:
- Always: `y_true`, `y_pred`, `err`, `abs_err`, `uncertainty`
- Regression: `tree_std`
- Classification: `vote_frac_mode`, `vote_margin`, `vote_entropy`

## Export artifact
The exported `rf_artifact.zip` contains:
- `rf_artifact.json`: preprocessing pipeline + serialized `ml-random-forest` model (`toJSON()`).
- `use_model.js`: example code for loading the artifact and running predictions.
- `test_predictions.csv`: the test split with predictions + uncertainty columns.
- `README.md`: notes and compatibility info for the artifact.

## Files
- `RFG.html`: UI, styling, and JavaScript logic (single-file app).
- `titanic.csv`: sample dataset for demos.

## Notes
- Dependencies (PapaParse, Chart.js, JSZip, `ml-random-forest`, etc.) are loaded from CDNs at runtime. A network connection is required unless you vendor those scripts locally.
- All processing happens in your browser. Your data is not uploaded by this app, but CDN-hosted scripts are executed; for sensitive environments, vendor dependencies locally.
- Very large datasets may hit browser memory/CPU limits; reduce max rows or increase test fraction for faster iteration.
