# S&P 1500 Momentum & Automated Execution Engine

An end-to-end algorithmic trading pipeline in Python that extracts equities across the S&P 1500, pulls historical market data, ranks assets using risk-adjusted momentum metrics, and autonomously rebalances a paper portfolio via Alpaca's REST API.

---

## Architecture Overview

```
[ Wikipedia Directories ] ──> universe.py       (Dynamic table parsing & symbol regex)
                                  │
[ Alpaca Asset Registry ] ──> ticker_filter.py  (Tradability & fractionability check)
                                  │
[ Alpaca SIP Feed ]       ──> ingestor.py       (Batched OHLCV fetch -> Parquet storage)
                                  │
[ Historical Parquet ]    ──> brain.py          (Vectorized return & volatility ranking)
                                  │
[ Alpaca Orders API ]     ──> executor.py       (Position rebalancing & notional sizing)
```

---

## Pipeline Workflow

* **`universe.py` (Constituent Ingestion):** Scrapes active S&P 500, S&P 400 (MidCap), and S&P 600 (SmallCap) directories. Uses dynamic table extraction with regex filtering to extract and normalize US ticker symbols.
* **`ticker_filter.py` (Asset Validation):** Queries Alpaca's asset registry to filter out illiquid, inactive, or non-fractionable symbols before downloading market history.
* **`ingestor.py` (Historical Ingestion):** Downloads 180 days of daily OHLCV bars using Alpaca's SIP data feed in batches of 100 symbols. Saves the raw dataset to Apache Parquet format (`data/sp1500_history.parquet`) using PyArrow for fast columnar I/O.
* **`brain.py` (Quant Scoring Engine):** Pivots the dataset into a matrix of dates $\times$ assets. Calculates 180-day total return and annualized volatility ($\sigma \times \sqrt{252}$) using Pandas vectorization to derive an Information Ratio / risk-adjusted momentum score.
* **`executor.py` (Portfolio Rebalancer):** Evaluates current portfolio holdings, liquidates positions that have dropped out of the Top 10 rankings, and issues fractional market buy orders with equal notional capital allocation ($95\%$ portfolio equity divided equally among target assets).
* **`main.py` (Orchestration):** Runs the end-to-end pipeline in sequence with configuration loaded via environment variables.

---

## Project Structure

```text
├── data/
│   └── sp1500_history.parquet  # Generated market data cache
├── .env                        # Local API credentials (ignored by git)
├── brain.py                    # Matrix math & momentum scoring
├── executor.py                 # Alpaca order execution & rebalancing
├── ingestor.py                 # Batched market data client
├── main.py                     # Pipeline runner
├── ticker_filter.py            # Tradability validation
├── universe.py                 # Wikipedia constituent scraper
└── requirements.txt            # Project dependencies
```

---

## Quickstart

### 1. Prerequisites
* Python 3.10+
* Alpaca Paper Trading API Keys

### 2. Installation
Clone the repository and install dependencies:
```bash
git clone [https://github.com/your-username/your-repo-name.git](https://github.com/your-username/your-repo-name.git)
cd your-repo-name
pip install pandas numpy requests alpaca-py pyarrow python-dotenv
```

### 3. Environment Variables
Create a `.env` file in the root directory:
```env
APCA_API_KEY_ID=your_alpaca_key_id
APCA_API_SECRET_KEY=your_alpaca_secret_key
```

### 4. Run the Engine
```bash
python main.py
```
