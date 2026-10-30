# NVIDIA Financial News Sentiment & Stock Analysis

A data-science project exploring whether financial-news sentiment adds information when analysing NVIDIA (NVDA) stock returns. The repository combines market data, keyword-filtered financial headlines, supplied FinBERT/VADER sentiment outputs, exploratory analysis, and Random Forest modelling.

> **Research / educational project only.** This repository is not investment advice and does not provide a trading recommendation. Historical relationships and model scores do not guarantee future performance.

## Project overview

The project has two modelling tracks:

1. **Stock-only analysis:** explores NVDA and selected cross-market price series, computes return-based features, and trains a Random Forest using market data.
2. **Stock + news analysis:** combines market features with daily headline sentiment and headline-volume information, then predicts the next trading day's NVDA return with a Random Forest.

The supplied sentiment results include FinBERT and VADER scores, plus a combined net-sentiment dataset. The repository also includes the filtered headline and daily sentiment tables used by the modelling workflow.

## Repository structure

```text
nvidia-financial-news-sentiment-analysis/
├── README.md
├── requirements.txt
├── .gitignore
├── .gitattributes
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
│   ├── LightGBM/                 # reserved for future work
│   └── Quantile regressor/       # reserved for future work
└── docs/
    ├── data_dictionary.md
    └── reproducibility_notes.md
```

Folder names have been lightly standardised (for example, `Random forrest` → `Random Forest`); the original project grouping and workflow are retained. A few obvious `NVDIA` filename typos were corrected to `NVIDIA`.

## Workflow

### 1. Set up the environment

Use Python 3.10 or newer. From the repository root:

```bash
python -m venv .venv
```

Activate the environment, then install the dependencies:

```bash
# Windows PowerShell
.venv\Scripts\Activate.ps1

# macOS / Linux
source .venv/bin/activate

python -m pip install --upgrade pip
pip install -r requirements.txt
```

You can run the notebooks in Jupyter, VS Code, or Google Colab. The supplied notebooks contain Google Colab upload/download helpers; those specific `google.colab` cells need to be skipped or adapted when running locally.

### 2. Collect market prices

Open:

`Data collection/Stocks and returns/01_data_collection_NVDA_yfonly.ipynb`

Run the notebook from top to bottom. It downloads adjusted closing-price series through `yfinance` for:

| Asset field | Yahoo Finance symbol |
|---|---|
| `stock` | `NVDA` |
| `gold` | `GC=F` |
| `oil` | `CL=F` |
| `usdinr` | `USDINR=X` |

The notebook exports `stocks_data_clean.csv` and a local SQLite database (`market.db`). The committed CSV is already supplied, so this step can be skipped when using the included snapshot.

### 3. Collect and filter finance headlines

The script is:

`Data collection/News headlines/nvidia_total_headlines.py`

It queries the Massive/Polygon reference-news endpoint for ticker `NVDA` and filters headlines using configurable finance-related keywords. It prompts for the API key with hidden input when `API_KEY` is blank. **Do not commit API keys or place them in source code.** Check your provider plan, rate limits, and terms before making a new collection request.

The script currently writes `nvda_headlines.csv` to the working directory. The repository also contains the supplied historical collection snapshot, `NVIDIA_total_headlines_Massive.csv`.

### 4. Use the supplied sentiment outputs

The `Sentiment Analysis/` directory contains headline-level and daily-level FinBERT/VADER result files, combined net-sentiment values, summary tables, and plots. These are precomputed outputs supplied with the project.

The original end-to-end code that generated every FinBERT and VADER score was **not present** in the uploaded archive. Therefore, the sentiment result files can be analysed directly, but regenerating them from raw headlines requires the corresponding scoring pipeline/model setup to be added separately.

### 5. Build the merged dataset and train the news-aware model

Open:

`Models/Random Forest/02_eda_NVDA_stocksandheadlines.ipynb`

The first modelling cell combines `stocks_data_clean.csv` and `daily_sentiment_scores.csv`, adds NVDA volume via `yfinance`, creates lagged/rolling sentiment and price features, and exports `final_dataset.csv`. It then:

- uses the oldest 80% of rows for training and the newest 20% for testing;
- fits a `RandomForestRegressor` to predict `target_return` (the next trading day's return);
- reports MAE, RMSE and R² alongside a simple persistence baseline;
- evaluates up/down direction calls and generates feature-importance and prediction plots.

For the supplied Colab workflow, upload these two files when prompted:

- `Data collection/Stocks and returns/stocks_data_clean.csv`
- `Sentiment Analysis/daily_sentiment_scores.csv`

The notebook expects them in the Colab `/content/` directory. When running locally, set the notebook's input paths to the corresponding repository paths and skip the `google.colab.files` download cell.

### 6. Run the stock-only benchmark

Open:

`Stock Data Analysis (yfinance)/02_eda_NVDA_onlystocksdata.ipynb`

This notebook explores the market data and trains a stock-only Random Forest. It provides a useful comparison against the news-aware model.

## Included data at a glance

The supplied snapshots contain approximately:

- **1,515** daily market rows, from 5 January 2021 to 30 October 2026;
- **18,499** filtered finance-news headlines, from 20 October 2022 to 20 October 2026;
- **6,365** headline-level FinBERT sentiment records;
- **1,342** daily FinBERT sentiment records.

See [`docs/data_dictionary.md`](docs/data_dictionary.md) for key fields, date formats, and file descriptions.

## Important methodology and reproducibility notes

- **News timing matters.** The current news-aware notebook maps calendar-day sentiment to the next available trading date. It handles weekends and market holidays, but does not explicitly shift after-market-close weekday headlines to the next session. That can introduce look-ahead bias. For defensible forecasting, align each headline using its publication timestamp and a clearly defined market-session cutoff, then rerun the evaluation.
- **The data sources can change.** `yfinance` history and Massive/Polygon API availability can vary over time. Re-running data collection may not reproduce the exact committed snapshots.
- **Sentiment-generation code is incomplete in the supplied archive.** FinBERT/VADER outputs are present, but the original scoring notebook/script is not.
- **No LightGBM or quantile-regression implementation was included.** Their folders are retained as placeholders for future extensions.
- **Chronological splitting is used.** Do not randomly shuffle time-series rows when evaluating forecasting performance.
- **Model outputs are exploratory.** Compare against baselines and inspect temporal alignment, class balance, leakage, and out-of-sample stability before drawing conclusions.

## Dependencies

Core libraries are listed in [`requirements.txt`](requirements.txt). The project uses pandas/NumPy for data processing, yfinance for price downloads, Matplotlib/Seaborn for plots, SciPy for correlation analysis, and scikit-learn for modelling and metrics.

## Publish this project to GitHub

1. Create a new **empty** repository on GitHub named `nvidia-financial-news-sentiment-analysis` (do not initialise it with a README, licence, or `.gitignore`).
2. Open a terminal in the extracted project folder and run:

```bash
git init
git add .
git commit -m "Initial commit: NVIDIA news sentiment and stock analysis"
git branch -M main
git remote add origin https://github.com/<YOUR-USERNAME>/nvidia-financial-news-sentiment-analysis.git
git push -u origin main
```

Replace `<YOUR-USERNAME>` with your GitHub username. GitHub may ask you to authenticate through your browser or credential manager.

## Data and attribution

Market data is retrieved from Yahoo Finance through `yfinance`; news data is retrieved through the Massive/Polygon API. The headline tables retain publisher and article URL fields where supplied. Respect the source providers' terms and the rights of publishers when redistributing or republishing headline data.

## Contributing / extending

Useful next steps include adding the FinBERT/VADER scoring pipeline, testing time-aware headline alignment, adding walk-forward validation, comparing with naïve and linear baselines, and implementing the planned LightGBM and quantile-regression tracks.
