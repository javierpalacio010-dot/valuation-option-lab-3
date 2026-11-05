# Valuation-to-Option Pricing Lab v4

This is an integrated analytical build:

- `valuation_option_lab.ipynb` performs the analysis and validation.
- The notebook writes `docs/model_results.json`.
- `docs/index.html` reads the generated JSON and renders the dashboard.
- GitHub Actions executes the notebook before every deployment.
- Deployment fails if the notebook fails or does not produce dashboard data.

## Publish
Upload all contents to the repository root, including `.github/workflows/deploy-pages.yml`. In repository Settings > Pages, select GitHub Actions.

## Extended analysis (v3)
- `docs/analysis.html` is a supplementary dashboard built from the same `docs/model_results.json` (new results live under the `extended` key, so `docs/index.html` is unchanged).
- New in the notebook: distribution diagnostics, an explicit DCF build from `data/dcf_inputs.csv`, scenario-weight sensitivity, a KDE-simulated belief value with a bootstrap interval, a Black-Scholes benchmark, implied volatility and mispricing metrics, sensitivity tables, maturity-alignment and model-risk flags, a source-documentation audit, and dated input snapshots in `docs/snapshots/`.
- `data/dcf_inputs.csv` ships with template assumptions. Replace them, and fill `source_url` and `retrieval_date` in `data/source_documentation.csv`, with researched values.
- Protected infrastructure (workflow, `requirements.txt`, CRR function, validation assertions) is unchanged. `docs/index.html` received label, DCF-source and synthetic-data-banner updates in v4.

## v4 changes (applied from a GitHub Copilot review)
1. **Subjective values are labelled as such.** The five-state and KDE values are subjective expected payoffs discounted at the risk-free rate. The weights are not risk-neutral, so these are *not* fair option prices. Both dashboards and the notebook now say so, and `model_results.json` carries an `interpretation` note.
2. **American-aware implied volatility.** Implied vol is inverted from Black-Scholes-Merton for European options and from the American CRR lattice for American options (`implied_vol_model` records which). The BSM benchmark check is marked not applicable for American exercise.
3. **Auditable DCF.** `dcf_inputs.csv` is validated (required parameters, decimals, WACC > g). When the explicit build exists, its value is the DCF shown on the dashboards. Model-risk flags cover reconciliation with the reported `dcf_value` (5% tolerance), terminal consistency (implied terminal ROIC = g ÷ reinvestment rate, which should sit between WACC and 40%), and leftover template assumptions. The WACC × g sensitivity grid is shown on `analysis.html`.
4. **Strict input validation and a synthetic-data banner.** Every CSV is checked for required columns, numeric and finite values, YYYY-MM-DD dates and valid ranges before any calculation runs, with errors that give spreadsheet row numbers. Both dashboards show a warning banner while synthetic or template inputs remain.
5. **Robust KDE.** The kernel density is fitted on log prices, so every draw is positive without clipping. Zero-IQR and degenerate samples are handled. The Monte Carlo standard error is reported separately from the bootstrap sampling interval, which uses common random numbers. With few analyst targets, treat the KDE as a smoothed picture of analyst opinion, not a forecast or a market-implied distribution.

## Reproduce locally
```bash
python -m pip install -r requirements.txt
python -m jupyter nbconvert --to notebook --execute valuation_option_lab.ipynb --output executed.ipynb
cd docs && python -m http.server 8000   # open http://localhost:8000 and /analysis.html
```
