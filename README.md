# DSA5205 Project 2 — Equity Strategy Research

Team 34: Wan Gehong, Wang Tianyi, Wu Jiarui, Yan Tianao.

The main entry point is [Equity Strategy Research](notebooks/Equity_Strategy_Research.ipynb). It builds the research dataset from authorized inputs, screens signals, trains and validates models, and evaluates portfolios net of trading costs. See [项目最新进度.md](项目最新进度.md) for the changes since the previous team version and the remaining deliverables.

## Setup and execution

From the repository root, create a Python 3.13 environment and install the project:

```bash
python3.13 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
.venv/bin/python -m pip install -e . --no-deps
.venv/bin/python -m ipykernel install --user --name team34-project2 --display-name "Python (Team 34 Project 2)"
```

On macOS, XGBoost may require the OpenMP runtime (`brew install libomp`).

Obtain the team's authorized research snapshot and place its contents under `data/`. The notebook expects `data/panels/`, `data/metadata/` and `data/checksums.sha256`. This is a supplied market and accounting panel snapshot with a cached SPY benchmark, and is excluded from Git. The notebook verifies input hashes, audits coverage and financial availability, and writes a separate cleaned dataset with rebuilt labels.

Open the notebook with the **Python (Team 34 Project 2)** kernel and choose **Restart Kernel → Run All**. A terminal run from the repository root is also supported:

```bash
.venv/bin/jupyter nbconvert --to notebook --execute \
  --ExecutePreprocessor.kernel_name=team34-project2 \
  --ExecutePreprocessor.timeout=-1 \
  --output=Equity_Strategy_Research.executed \
  --output-dir=runs \
  notebooks/Equity_Strategy_Research.ipynb
```

Allow time and disk space for data preparation, 396 model fits, 38 portfolio ledgers and the daily factor model. A fresh execution builds a new timestamped experiment under `runs/` from the supplied snapshot.

**Release status.** Saved notebook outputs combine the main run, verified checkpoint recovery and later targeted reruns for the weighting comparison, OOS diagnostics and holdings-based attribution. The current notebook still needs one uninterrupted top-to-bottom rerun to verify the complete workflow. The report under `reports/` reflects the previous study, and the video for the current notebook is pending.

## Research design

| Stage | Dates and rule |
|---|---|
| WQ direction and screening | Calibrate directions in 2015–2017. Screen 36 WQ candidates in 2018–2019 using RankIC, coverage, annual stability and correlation. |
| Hyperparameter selection | Use expanding-window CV with annual scoring folds in 2020–2022. Compare fundamental, fundamental plus WQ, and engineered features for Ridge, Lasso and XGBoost. |
| Portfolio selection | In 2023, compare four scores under three weightings: equal, square-root capitalization and capitalization. Select the highest net Sharpe among the 12 candidates with finite Sharpe and mean cash at most 5%. |
| Chronological evaluation | Evaluate the frozen choices in 2024–2025. ML models refit every half-year on the preceding five calendar years using only labels that matured before the forecast boundary. |

Signals formed at the close are executed at the next close. The supervised target is the adjusted return from T+1 close to T+11 close. Portfolio tests use a Top100 target, rank-150 retention buffer, ten-session rebalancing, 2% formation weight cap and a 10 bps charge per traded dollar. SPY and matched full-pool portfolios are benchmarks. Accounting vintages, universe provenance and unresolved terminal payouts constrain interpretation.

## Results and diagnostics

The 2023 validation selects **six-factor direct scoring with capitalization weights**. Its 2024–2025 net CAGR is **24.92%**, net Sharpe **1.407**, and maximum drawdown **−20.76%**. On the same dates, SPY records **22.19%**, **1.307**, and **−18.76%**, respectively.

- [Portfolio summary](results/portfolio_summary.csv) and [selection record](results/frozen_family.json) document the candidate grid and decision.
- [Period metrics](results/period_metrics.csv) and [prediction OOS R²](results/period_oos_r2.csv) support the four-panel comparison. The six-factor score is mapped to return units with pooled OLS using mature labels from the same past five-year windows. This diagnostic leaves the portfolio's direct ranking rule intact.
- The notebook includes the selection, weighting and universe decomposition, plus holdings-based Barra-style attribution for the selected portfolio. [Factor exposures and contributions](results/barra_factors.csv) and [residual alpha](results/barra_alpha.csv) use the available market, SIC industry and nine style factors. Growth is unavailable. Residual alpha is conditional on this research risk model.
- [Portfolio audit](results/portfolio_audit.json) records ledger reconciliation. Full daily artifacts are written under `runs/`.

Run regression tests from the repository root:

```bash
.venv/bin/python -m pytest tests -q
```

Source is under `src/`, tests under `tests/`, and compact result tables under `results/`. The local `data/`, `backups/` and `runs/` directories are excluded from the repository.
