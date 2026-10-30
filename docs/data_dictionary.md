# Data dictionary and file inventory

This guide describes the supplied snapshots. Row counts are from the uploaded archive; they may differ after recollection or regeneration.

## Market data

**File:** `Data collection/Stocks and returns/stocks_data_clean.csv`

| Field | Description |
|---|---|
| `date` | Trading-date label. |
| `stock` | NVDA adjusted close price downloaded using `yfinance` with `auto_adjust=True`. |
| `gold` | Gold futures (`GC=F`) adjusted close. |
| `oil` | Crude-oil futures (`CL=F`) adjusted close. |
| `usdinr` | USD/INR exchange-rate series (`USDINR=X`). |
| `stock_ret` | Daily percentage return for NVDA. |
| `gold_ret` | Daily percentage return for gold. |
| `oil_ret` | Daily percentage return for oil. |
| `usdinr_ret` | Daily percentage return for USD/INR. |

Snapshot: 1,515 rows; dates 2021-01-05 through 2026-10-30.

## News data

**CSV:** `Data collection/News headlines/NVIDIA_total_headlines_Massive.csv`

| Field | Description |
|---|---|
| `date` | Calendar date derived from the publication timestamp. |
| `published_utc` | Publication timestamp in UTC. |
| `title` | Headline text. |
| `matched_keywords` | Finance-related keywords that matched the headline. |
| `publisher` | Publisher name, when available. |
| `url` | Article URL, when available. |

Snapshot: 18,499 rows; dates 2022-10-20 through 2026-10-20.

**Workbook:** `Data collection/News headlines/NVIDIA_Finance_Massive_Headlines_Filtered.xlsx`

Contains a `Summary` sheet and a `Filtered Headlines` sheet. The latter has the finance/company relevance metadata used during filtering, including `company_match`, `title_finance_terms`, `matched_metadata_finance_terms`, `match_basis`, and `finance_term_count`.

## Sentiment data

### Daily FinBERT-derived scores

**File:** `Sentiment Analysis/daily_sentiment_scores.csv`

| Field | Description |
|---|---|
| `date` | Calendar date. |
| `sentiment_mean` | Mean headline sentiment score for the day. |
| `sentiment_sum` | Sum of headline sentiment scores for the day. |
| `n_headlines` | Number of scored headlines that day. |
| `avg_positive` | Mean positive probability. |
| `avg_negative` | Mean negative probability. |
| `avg_neutral` | Mean neutral probability. |

Snapshot: 1,342 rows; dates 2022-10-20 through 2026-10-20.

### Headline-level FinBERT/VADER combination

**File:** `Sentiment Analysis/FinBERT_VADER_Combined_Net_Sentiment.csv`

Includes `finbert_score`, `vader_score`, their predicted labels, positive/negative/neutral probabilities, `finbert_net_sentiment`, `vader_net_sentiment`, `combined_net_sentiment`, and `combined_sentiment_label`, alongside date, title, and match status. Dates in this file use `DD-MM-YYYY` format.

### VADER daily output

**File:** `Sentiment Analysis/NVDA_VADER_Sentiment_Analysis.csv`

Contains daily average VADER sentiment, positive/negative/neutral probabilities, headline count, and a 7-day rolling sentiment value. Dates use `DD-MM-YYYY` format.

### FinBERT workbook

**File:** `Sentiment Analysis/NVIDIA_FinBERT_Sentiment_Analysis.xlsx`

Sheets:

- `Headline Sentiment`: headline-level metadata and FinBERT probability/sentiment fields.
- `Daily Sentiment`: daily average sentiment and class probabilities, headline count, and 7-day rolling value.
- `Sentiment Distribution`: headline counts and percentages by sentiment class.

The workbook contains 6,365 headline-level records and 1,342 daily-level records (excluding header rows).

## Model-derived / exploratory outputs

The `Sentiment Analysis/`, `Stock Data Analysis (yfinance)/`, and `Models/Random Forest/` directories contain PNG plots and, in the sentiment directory, a correlation matrix, rolling sentiment data, and other CSV/XLSX outputs. These are supplied result artifacts rather than all being regenerated automatically by the included notebooks.

## Date-format caution

The project currently uses more than one date format:

- Market/news/daily FinBERT CSVs generally use `YYYY-MM-DD`.
- Combined headline sentiment and daily VADER CSVs use `DD-MM-YYYY`.
- Publication timestamps in the news CSV include a UTC offset.

Parse dates explicitly when joining files. Avoid relying on automatic date inference for the `DD-MM-YYYY` files.
