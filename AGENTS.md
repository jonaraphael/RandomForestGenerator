# Repository Guidelines

## Project Structure & Module Organization
This repository is a single-page web app with everything in one HTML file.
- `RFG.html`: the full UI, styles, and JavaScript logic (ES module).
- `titanic.csv`: sample dataset for quick demos and smoke checks.
- `README.md`: short overview; update if usage changes.

## Build, Test, and Development Commands
There is no build system or package manager. Open the file directly, or run a tiny static server if the browser blocks ES module imports.
```sh
python -m http.server
```
Then open `http://localhost:8000/RFG.html`. The app loads dependencies from CDNs (PapaParse, Chart.js, JSZip, ml-random-forest), so a network connection is required unless you vendor those scripts.

## Coding Style & Naming Conventions
- Indentation: 2 spaces in HTML/CSS/JS.
- JavaScript: use semicolons, double quotes for strings, and trailing commas in multi-line objects/arrays.
- Naming: `camelCase` for functions and variables, `UPPER_SNAKE_CASE` for constants (e.g., `STEPS`), and descriptive, lower-case class names in HTML.
- Keep UI state centralized in the `APP` object; add helpers near the existing helper sections to stay organized.

## Testing Guidelines
There are no automated tests. Validate changes manually:
- Load `RFG.html` in a browser.
- Import `titanic.csv`, run training, and confirm charts and exports render without console errors.

## Commit & Pull Request Guidelines
Git history uses short, imperative, sentence-case messages (e.g., "Add README..."). Follow that style unless a new convention is introduced.
For pull requests, include:
- A concise description of behavior changes.
- UI screenshots or a short clip for visual updates.
- Any CDN dependency changes with version numbers.

## Security & Configuration Tips
All processing is client-side, but datasets and models still load external scripts from CDNs. For offline or sensitive environments, consider bundling those dependencies locally.
