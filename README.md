# RandomForestGenerator

Single-page, in-browser Random Forest trainer for CSV datasets. Everything runs client-side: upload a CSV, pick a target, train, inspect results/diagnostics, then export a portable artifact.

## Quick Start
- Open `RFG.html` in a modern browser.
- If your browser blocks ES module imports when opening via `file://`, run a tiny local server:
```sh
python3 -m http.server  # or: python -m http.server
```
- Navigate to `http://localhost:8000/RFG.html`.
- For a smoke test:
  - Click “Use Titanic sample” (embedded in `RFG.html`), or
  - Upload `titanic.csv` and set target to `Survived` (classification).

## Dependencies (CDN / esm.sh)
This app loads the following in the browser at runtime:
- PapaParse `5.4.1` (CSV parsing)
- Chart.js `4.4.1` (charts)
- JSZip `3.10.1` (zip export)
- `ml-random-forest` `2.1.0` (model) via `esm.sh` ESM import
- `ml-cart` `^2.1.1`, `ml-matrix` `^6.8.2`, `random-js` `^2.1.0` via `esm.sh` ESM imports

For offline/airgapped use, self-host/vendor these dependencies and update the script/import URLs in `RFG.html`.

## Workflow (in the UI)
- **Upload**: drop a CSV anywhere on the page (or browse). Preview the first rows; click a column header to quickly set it as the target.
- **Target**: confirm task type (regression/classification), split strategy, and typing/exclusions in “Review columns”.
- **Train**: configure RF settings (trees, `maxFeatures`, depth, min samples, seed) and start training (progress + Stop supported).
- **Results**: metrics + charts, feature importance, shift detector, pruning helpers, and a predictions explorer (download predictions CSV).
- **Export**: download `rf_artifact.zip` (model + preprocessing + sample loader + predictions).

## Features (What the app actually does)
- **Column typing + review**: infers `continuous` / `categorical` / `date`, flags “ID-like” columns, and lets you exclude columns or override inferred types.
  - `max_card` controls when integer-ish numeric columns are treated as categorical.
- **Splits**:
  - **Random** train/test split (seeded).
  - **Time-based** split (shown only if date columns exist): pick a date column and either supply a cutoff date or fall back to test-fraction splitting.
  - `maxRows` subsamples for speed; for time-based splits it keeps the last N rows in file order.
- **Targets**:
  - Regression supports optional `log1p` training transform (predictions inverted back with `expm1`).
  - Classification supports advanced multi-class helpers:
    - Keep top N classes (map the rest → `RARE`).
    - Derive a new boolean target column from a small “pythonic” expression language (evaluated against the current target cell value only).
- **Results + diagnostics**:
  - Regression: RMSE/MAE/R²/RMSLE + predicted-vs-true scatter.
  - Classification: accuracy/macro-F1 + confusion matrix (shows top classes for high-cardinality targets).
  - Feature importance (aggregate-by-column or show derived features like date parts and missing indicators).
  - OOB (out-of-bag) scoring when available (coverage shown).
  - Shift detector (“is_valid” model) to distinguish train vs test; shows top shift drivers.
  - Uncertainty via per-tree disagreement (fastai-style), plus error vs uncertainty scatter.
- **Pruning loop**:
  - “Prune low-importance → retrain”, “Prune top shift driver → retrain”, and “Undo prune” for quick iteration.
- **Export**:
  - `rf_artifact.json`: preprocessing pipeline + serialized `ml-random-forest` model (`toJSON()`).
  - `use_model.js`: sample code for loading the artifact and running predictions (browser ESM + Node).
  - `test_predictions.csv`: the test split with predictions + uncertainty columns.
  - `README.md`: notes and compatibility info for the artifact.

## Derived boolean target (pythonic expression)
Creates a new binary target column from a small expression language evaluated per-row on the current target cell value.

- **Value variable**: use the identifier shown in the UI (derived from the selected target column name; if the column name isn’t a valid identifier, it’s sanitized). `o` is also accepted.
- **Value type/coercion**: the target cell is converted to a trimmed string before evaluation.
- **Numeric vs string comparisons**: for `<`, `<=`, `>`, `>=` the app compares numerically only if both sides parse as numbers; otherwise it compares as strings.
- **Supported syntax**: `and`, `or`, `not`, parentheses, `len(x)`, comparisons (`==`, `!=`, `<`, `<=`, `>`, `>=`), and membership (`x in ( ... )`, `x not in ( ... )`) over literal lists (`number`, quoted `string`, `true`, `false`, `none`).
- **Not supported**: regex, arithmetic, or referencing other columns/row fields.

## Preprocessing (fit on train split only)
- **Continuous columns**: median imputation + a missing-indicator feature (`_na`).
- **Categorical columns**: label encoding; missing/unknown categories map to an “unknown” ID (0).
- **Date columns**: expanded into date parts (year/month/week/day/etc.) + a missing-indicator (`_na`).

## Predictions CSV columns
The “Download predictions CSV” button (and the export zip) includes your test split with extra columns:
- Always: `y_true`, `y_pred`, `err`, `abs_err`, `uncertainty`
- Regression: `tree_std`
- Classification: `vote_frac_mode`, `vote_margin`, `vote_entropy`

## Files
- `RFG.html`: UI, styling, and JavaScript logic (single-file app).
- `titanic.csv`: sample dataset for classification demos.

## Notes
- Your CSV data stays in the browser tab; the page still executes CDN-hosted scripts unless you self-host.
- Very large datasets may hit browser memory/CPU limits; reduce `maxRows` for faster iteration.
- In classification, some test rows can be dropped if their label never appears in the training split (train-only label encoding).

## License
MIT — see `LICENSE`.
