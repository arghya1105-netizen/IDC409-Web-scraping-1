# Reproducibility and review notes

## What is reproducible from this repository?

- The stock collection notebook can request fresh market data through `yfinance`.
- The headline collection script can request fresh ticker news through the Massive/Polygon API when a valid API key and suitable account access are available.
- The stock-only exploratory/model notebook can be run using the committed market CSV.
- The news-aware modelling notebook can build its merged feature table and train its Random Forest from the committed market and daily sentiment CSVs.
- The supplied charts and sentiment tables allow inspection of the analysis results without rerunning every collection/scoring step.

## What is not fully reproducible yet?

The uploaded files do not include the original complete FinBERT and VADER inference/scoring code. The sentiment output files are present, but the exact model loading, preprocessing, tokenisation, batching, scoring, and aggregation workflow cannot be reconstructed confidently from output files alone. Add that pipeline (with model names/versions and preprocessing details) before claiming the sentiment scores can be regenerated end to end.

The archive also does not include a generated `final_dataset.csv`, the local `market.db`, or code for the planned LightGBM and quantile-regression experiments.

## Time alignment / possible leakage

The news-aware notebook maps calendar-day sentiment to the next available trading date using a forward as-of merge. This is suitable for shifting weekend and holiday news forward, but it does not distinguish pre-market, regular-session, and after-market-close publications on ordinary weekdays. As a result, a day's sentiment aggregate can include information published after that day's close while being paired with that same day's market row.

Before treating the test metrics as evidence of forecasting ability, use `published_utc` to assign each headline to the first trading session whose information cutoff occurs after publication. Define the cutoff (for example, before the next session opens), regenerate daily sentiment features, and rerun the chronological evaluation.

## Recommended validation improvements

1. Use expanding-window or rolling walk-forward validation rather than relying on a single 80/20 chronological split.
2. Compare the model with a persistence baseline and a simple linear model on identical dates.
3. Report both regression and direction metrics, including class balance and confusion matrix.
4. Keep every transformation that learns from data inside the training window; avoid fitting preprocessing on the full timeline.
5. Record package versions, data retrieval dates, API parameters, model checkpoints, and random seeds.
6. Check for target leakage and ensure all feature values would have been available at the prediction timestamp.
