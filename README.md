# NVIDIA Financial News Sentiment & Stock Analysis

A data-science project exploring whether financial-news sentiment contains useful information for analysing NVIDIA (NVDA) stock movements. The workflow combines historical market data, finance-related headlines, precomputed FinBERT/VADER sentiment scores, exploratory analysis, and machine-learning models for next-day return analysis.

> **Research and educational project only.** This repository is not investment advice and does not provide a trading recommendation. Historical relationships and model scores do not guarantee future performance.

## Project overview

The project investigates the relationship between NVDA market behaviour and financial-news sentiment through three implemented modelling tracks:

1.  **Stock-only Random Forest:** uses historical NVDA price- and volume-derived features to estimate the next trading day's return. It serves as a market-data-only benchmark.
2.  **News-aware Random Forest:** combines market features with daily sentiment, sentiment lags/rolling averages, headline volume, and interaction features to estimate the next trading day's return.
3.  **Quantile Regression:** fits separate regression models for the 10th, 25th, 50th, 75th, and 90th percentiles of next-day returns. This produces a forecast distribution/interval rather than only a single point estimate.

A **LightGBM** folder is present in the project structure, but the latest uploaded project archive contained no LightGBM notebook or source file in that folder. Consequently, LightGBM is listed as a planned/incomplete modelling track and no method or performance result is attributed to it here.

The supplied sentiment outputs include FinBERT and VADER results, as well as a combined net-sentiment dataset. These are precomputed outputs; the complete original scoring pipeline was not present in the supplied archive.

## Repository structure

``` text
nvidia-financial-news-sentiment-analysis/
├── README.md
├── requirements.txt
├── .gitignore
├── .gitattributes
├── .github/
│   └── workflows/
│       └── validate.yml
├── Data collection/
│   ├── News headlines/
│   │   ├── nvidia_total_headlines.py
│   │   ├── NVIDIA_total_headlines_Massive.csv
│   │   └── NVIDIA_Finance_Massive_Headlines_Filtered.xlsx
│   └── Stocks and returns/
│       ├── 01_data_collection_NVDA_yfonly.ipynb
│       └── stocks_data_clean.csv
├── Sentiment Analysis/
│   ├── daily_sentiment_scores.csv
│   ├── FinBERT_VADER_Combined_Net_Sentiment.csv
│   ├── NVDA_VADER_Sentiment_Analysis.csv
│   ├── NVIDIA_FinBERT_Sentiment_Analysis.xlsx
│   └── analysis charts and summary tables
├── Stock Data Analysis (yfinance)/
│   ├── 02_eda_NVDA_onlystocksdata.ipynb
│   └── stock-only analysis charts
├── Models/
│   ├── Random Forest/
│   │   ├── 02_eda_NVDA_stocksandheadlines.ipynb
│   │   └── model evaluation charts
│   ├── Quantile regressor/
│   │   ├── Quantile_Regressor_Model.ipynb
│   │   ├── quantile_predictions.csv
│   │   ├── confusion_matrix.png
│   │   └── feature_importance_qr.png
│   └── LightGBM/
│       └── (source code not present in the latest supplied archive)
└── docs/
    ├── data_dictionary.md
    └── reproducibility_notes.md
```

Folder names above reflect the cleaned GitHub repository layout. If your existing GitHub repository uses slightly different capitalisation or spelling, keep the actual folder names shown in GitHub when opening notebooks.

## Workflow

### 1. Set up the environment

Use Python 3.10 or newer. From the repository root:

``` bash
python -m venv .venv
```

Activate the environment and install the dependencies:

``` bash
# Windows PowerShell
.venv\Scripts\Activate.ps1

# macOS / Linux
source .venv/bin/activate

python -m pip install --upgrade pip
pip install -r requirements.txt
```

The notebooks can be opened in Jupyter, VS Code, or Google Colab. Some notebooks use `google.colab.files.upload()` and `google.colab.files.download()`; those cells are Colab-specific and need to be skipped or adapted when running locally.

### 2. Collect market prices

Open:

`Data collection/Stocks and returns/01_data_collection_NVDA_yfonly.ipynb`

The notebook downloads adjusted closing-price series using `yfinance` for:

| Asset field | Yahoo Finance symbol |
|-------------|----------------------|
| `stock`     | `NVDA`               |
| `gold`      | `GC=F`               |
| `oil`       | `CL=F`               |
| `usdinr`    | `USDINR=X`           |

It calculates daily returns and exports `stocks_data_clean.csv`. The repository already contains a market-data snapshot, so recollection is optional when reproducing the analysis against the supplied data.

### 3. Collect and filter finance headlines

The collection script is:

`Data collection/News headlines/nvidia_total_headlines.py`

It queries the Massive/Polygon reference-news endpoint for ticker `NVDA` and filters headlines using configurable finance-related keywords. The script requires a valid API key and suitable provider access. Do not commit API keys or place credentials directly in source code. Check the provider's terms, usage limits, and data-redistribution conditions before collecting or sharing new data.

The repository also contains a historical headline snapshot and a filtered workbook.

### 4. Use the supplied sentiment outputs

The `Sentiment Analysis/` directory contains headline-level and daily-level sentiment results, combined net-sentiment values, summary tables, and plots.

The modelling notebooks use different sentiment inputs:

- **News-aware Random Forest:** `daily_sentiment_scores.csv`
- **Quantile Regression:** `FinBERT_VADER_Combined_Net_Sentiment.csv`

The complete original code that generated every FinBERT and VADER score was not present in the latest uploaded archive. Therefore, the supplied score files can be analysed directly, but regenerating all sentiment scores from raw headlines requires the original scoring pipeline/model setup to be added.

### 5. Stock-only Random Forest benchmark

Open:

`Stock Data Analysis (yfinance)/02_eda_NVDA_onlystocksdata.ipynb`

This notebook explores NVDA price behaviour and cross-market relationships, creates price/volume-derived features, and trains a `RandomForestRegressor` to predict the next trading day's return. It uses a chronological split, with the earlier observations for training and the latest 20% for testing. This provides a benchmark that does not include headline sentiment.

### 6. News-aware Random Forest

Open:

`Models/Random Forest/02_eda_NVDA_stocksandheadlines.ipynb`

The notebook combines the market dataset with daily sentiment data, adds NVDA trading volume through `yfinance`, and creates features from the market and sentiment series.

The feature groups include:

- **Sentiment:** daily sentiment, 3-day and 5-day rolling sentiment, and 1-, 3-, and 5-day sentiment lags.
- **Market movement:** daily and log returns, 3-, 5-, and 10-day volatility, volume change, moving averages, and a price moving-average ratio.
- **Interactions:** sentiment multiplied by daily return and sentiment multiplied by 5-day volatility.

The notebook fits a `RandomForestRegressor` with 300 trees, maximum depth 5, minimum leaf size 10, and `random_state=42`. It uses the oldest 80% of the rows for training and the newest 20% for testing. The target is the next trading day's return (`target_return`).

**Results saved in the notebook output:**

| Metric             | News-aware Random Forest | Persistence baseline |
|--------------------|-------------------------:|---------------------:|
| MAE                |                   0.0183 |               0.0253 |
| RMSE               |                   0.0236 |               0.0338 |
| R²                 |                  -0.0199 |         Not reported |
| Direction accuracy |                    0.488 |                0.545 |
| Direction F1-score |                    0.533 |                0.541 |

The Random Forest has lower reported MAE and RMSE than the notebook's persistence baseline, but its direction accuracy and F1-score are slightly lower than the baseline. The negative R² also indicates that the model does not explain the variation in the held-out returns well under this evaluation. These results should be interpreted as exploratory rather than as evidence of a reliable trading signal.

### 7. Quantile Regression

Open:

`Models/Quantile regressor/Quantile_Regressor_Model.ipynb`

Unlike the Random Forest point forecast, the Quantile Regression notebook fits five separate `QuantileRegressor` models to estimate different parts of the next-day return distribution:

- 10th percentile (`q10`)
- 25th percentile (`q25`)
- 50th percentile / median (`q50`)
- 75th percentile (`q75`)
- 90th percentile (`q90`)

**Input files:**

1.  `Sentiment Analysis/FinBERT_VADER_Combined_Net_Sentiment.csv`
2.  `Data collection/Stocks and returns/stocks_data_clean.csv`

The notebook asks for these files using the Google Colab upload interface. It maps calendar-day headline sentiment to the next available trading date, aggregates the daily sentiment and number of headlines, and builds 13 features. These include lagged returns, rolling return mean and volatility, the gap from the 20-day moving average, gold/oil/USD-INR returns, daily sentiment, rolling sentiment averages, and log headline volume.

The target is the **next trading day's NVDA return expressed as a percentage** (`next_ret`). Features are standardised in a `StandardScaler` pipeline. Each quantile model uses `QuantileRegressor` with L1 regularisation (`alpha=0.01`) and the HiGHS solver. The data is split chronologically: the first 80% is used for training and the final 20% for testing. Predicted quantiles are sorted row-wise to address quantile crossing in the displayed forecast bands.

#### Quantile Regression results

The notebook's saved output reports the following pinball losses (lower is better):

| Quantile | Model pinball loss | Training-quantile baseline | Comparison          |
|-----------------|------------------:|------------------:|-----------------|
| q10      |             0.4162 |                     0.4180 | Slightly better     |
| q25      |             0.7396 |                     0.7263 | Worse               |
| q50      |             0.9164 |                     0.9079 | Worse               |
| q75      |             0.7461 |                     0.7433 | Worse               |
| q90      |             0.4511 |                     0.4511 | Approximately equal |

Additional saved evaluation results:

| Evaluation | Quantile Regression result | Reference baseline / target |
|--------------------|-------------------------:|-------------------------:|
| 10th–90th percentile interval coverage | 82.5% | 80% nominal coverage |
| Median forecast MAE | 1.833 percentage points | 1.809 for a zero-return forecast |
| Median forecast RMSE | 2.3495 percentage points | 2.3540 for a zero-return forecast |
| Median forecast R² | 0.0019 | -0.0020 for a zero-return forecast |
| Up/down accuracy in the main evaluation cell | 0.490 | 0.495 for always predicting up |

The notebook also reports a same-direction-as-today baseline accuracy of 0.539. A later diagnostic cell excludes exactly-zero return observations and reports direction accuracy of 0.508; this differs from the all-observation result above because the evaluation subset changes. The confusion matrix shows that the median-forecast rule predicts an upward direction for 91.7% of test observations, while approximately 49.5% of actual test returns are upward. This class-direction imbalance should be considered when interpreting its F1-score.

A five-fold time-series walk-forward check is also included. Its fold-level MAE values are close to the training-median baseline, with only small and mixed differences. Overall, the stored evaluation suggests that the quantile model produces an interval with near-nominal observed coverage, but its median point forecast does not consistently outperform simple return baselines.

The notebook saves `quantile_predictions.csv` and generates forecast-band, confusion-matrix, and coefficient/feature-effect plots. The plot named `feature_importance_qr.png` represents standardised linear coefficients from the median quantile model; it is not tree-based feature importance.

### 8. LightGBM status

The latest project ZIP contained a `Models/LightGBM/` directory, but it was empty. No LightGBM notebook, Python script, trained model, or evaluation output was available to inspect. For that reason, this README does **not** describe a LightGBM training procedure or report LightGBM performance.

When the LightGBM implementation is added, document its input features, target, chronological validation strategy, hyperparameters, metrics, and comparison baselines here. The `lightgbm` package should also be added to `requirements.txt` if the implementation uses it.

## Important methodology and reproducibility notes

- **News timing matters.** The modelling notebooks map calendar-day news to the next available trading date. This handles weekends and market holidays, but it does not explicitly distinguish pre-market, regular-session, and after-market-close publications. A weekday headline published after the market close may therefore be paired with that same trading day's row. This can introduce look-ahead bias.
- **Improve timestamp alignment before making forecasting claims.** Use the original publication timestamp (`published_utc`) to assign each headline to the first trading session whose defined information cutoff occurs after publication. Then regenerate daily sentiment features and rerun the chronological evaluation.
- **Model results are not a direct head-to-head comparison.** The Random Forest and Quantile Regression notebooks use different feature definitions, sentiment inputs, target scaling conventions, and slightly different train/test date ranges. Their metrics should not be ranked against each other without rerunning them on the same rows, target units, feature availability, and validation protocol.
- **Baselines are essential.** The saved results show mixed performance against simple baselines. A model with a better regression error does not necessarily provide better up/down direction predictions.
- **Sentiment-generation code is incomplete.** The FinBERT/VADER output files are present, but the full original inference/scoring workflow is not included.
- **Data sources can change.** `yfinance` history and Massive/Polygon API availability may vary over time. Re-running collection may not reproduce the committed snapshots exactly.
- **Chronological evaluation is used.** Do not randomly shuffle time-series observations when evaluating forecasting models. Prefer walk-forward validation and keep all learned preprocessing inside each training fold.
- **Outputs are exploratory.** Check target leakage, market-session timing, class balance, feature availability, and out-of-sample stability before drawing conclusions.

## Dependencies

Core dependencies are listed in `requirements.txt`. They include pandas and NumPy for data processing, `yfinance` and `requests` for data collection, Matplotlib/Seaborn/SciPy for analysis and visualisation, and scikit-learn for the Random Forest and Quantile Regression workflows.

The Quantile Regression notebook uses the HiGHS solver through scikit-learn's `QuantileRegressor`. If the solver is unavailable in a local environment, update the SciPy/scikit-learn environment or run the notebook in a compatible Colab runtime.

## Publish or update the GitHub repository

If the repository already exists on GitHub, open `README.md`, choose the edit (pencil) button, replace its contents with this file's contents, and commit the change. No branch setup is needed if the repository already has its `main` branch.

For a local Git workflow, from the repository folder:

``` bash
git add README.md
git commit -m "Update README with Quantile Regression and model results"
git push
```

## Data and attribution

Market data is retrieved from Yahoo Finance through `yfinance`; news data is retrieved through the Massive/Polygon API. Headline tables may retain publisher and article URL fields. Respect the source providers' terms and the rights of publishers when redistributing or republishing headline data.

## Future improvements

- Add the missing LightGBM notebook/script and evaluate it using the same chronological test period and baseline as the existing models.
- Add the complete FinBERT and VADER scoring pipeline, including model identifiers/versions and preprocessing details.
- Align headlines to market sessions using precise publication timestamps.
- Run identical walk-forward validation across the stock-only Random Forest, news-aware Random Forest, Quantile Regression, and LightGBM models once implemented.
- Compare against naïve, linear, and other simple baselines, and report uncertainty across multiple time splits.
