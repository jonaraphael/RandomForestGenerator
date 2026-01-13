# RandomForestGenerator

Single-page, in-browser Random Forest trainer for CSV datasets. The app runs fully client-side and guides users through upload, target selection, training, results, and export.

## Quick Start
- Open `RFG.html` directly in a modern browser, or run a tiny local server if ES module imports are blocked:
```sh
python -m http.server
```
- Navigate to `http://localhost:8000/RFG.html`.
- Load the included sample dataset `titanic.csv` to smoke-test the flow.

## Features
- CSV parsing, basic data inspection, and target selection.
- Classification or regression with automatic train/test split.
- Charts and export artifacts (model + metrics) for analysis.

## Files
- `RFG.html`: UI, styling, and JavaScript logic (single-file app).
- `titanic.csv`: sample dataset for demos.

## Notes
Dependencies (PapaParse, Chart.js, JSZip, ml-random-forest) are loaded from CDNs at runtime. A network connection is required unless you vendor those scripts locally.
