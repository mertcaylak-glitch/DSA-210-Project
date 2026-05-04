# 📈 AI Research Activity as a Market Signal: Predicting BOTZ ETF Returns

> **Does the daily volume of AI research publications on arXiv carry any predictive signal for the BOTZ ETF's financial performance?**

This project investigates whether the pace of AI research — as measured by daily arXiv submission counts in the `cs.AI` category — can be used to predict the returns of the Global X Robotics & Artificial Intelligence ETF (BOTZ). The analysis spans September 2016 to December 2023 and follows a rigorous five-stage analytical pipeline.

---

##  Motivation

Artificial Intelligence has arguably been the defining technological force of the past decade. As research output in AI has grown exponentially — doubling roughly every few years — so too has investor interest in AI-adjacent assets. This raises a natural question: does the *intensity* of academic AI innovation, as proxied by daily arXiv submission volumes, leave any measurable fingerprint on the financial performance of an AI-focused ETF?

The question sits at the intersection of two well-studied ideas: the *Efficient Market Hypothesis* (public information should already be priced in) and the *technology diffusion hypothesis* (breakthroughs take time to reach markets). Testing this with real data offers a concrete, reproducible way to explore both.

On a personal level, I was motivated by the observation that many landmark AI papers — the Transformer, BERT, GPT-3 — coincided with notable periods of market excitement in AI stocks. Rather than relying on anecdote, I wanted to subject this intuition to rigorous statistical scrutiny.

---

##  Project Structure

```
├── 00_ETFDataAcquisition.ipynb          # BOTZ ETF price data collection (yfinance)
├── 00_PaperDataAcquisition.ipynb        # arXiv cs.AI paper counts (OAI-PMH)
├── 00_MasterDatasetAcquisition.ipynb    # Dataset merging & accumulation logic
├── 01_EDA_improved.ipynb                # Exploratory Data Analysis
├── 02_HT_improved.ipynb                 # Hypothesis Testing (Granger Causality)
├── 03_EDAHT-2_improved.ipynb            # Event-Driven Market Reaction Analysis
├── 04_ML_improved.ipynb                 # Machine Learning Models
├── ai_market_master_dataset.csv         # Pre-built master dataset (ready to use)
├── botz_daily_data.csv                  # Raw BOTZ ETF daily prices
├── ai_daily_papers.csv                  # Raw arXiv daily paper counts
└── requirements.txt                     # Python dependencies
```

---

##  Data Sources

### BOTZ ETF (Financial Data)
- **Source:** Yahoo Finance via the `yfinance` Python library
- **Ticker:** BOTZ — Global X Robotics & Artificial Intelligence ETF
- **Coverage:** September 2016 (ETF inception) – December 2023
- **Variables:** Adjusted daily closing price, daily percentage return
- **Collection:** Programmatic API pull; see `00_ETFDataAcquisition.ipynb`

### arXiv cs.AI (Research Activity Data)
- **Source:** arXiv.org via the OAI-PMH protocol (`sickle` library)
- **Category filter:** `cs.AI` (Artificial Intelligence sub-category only)
- **Coverage:** September 2016 – December 2023
- **Variable:** Number of new paper submissions per calendar day
- **Collection:** Server-side filtered harvest; see `00_PaperDataAcquisition.ipynb`

### Master Dataset Construction
The two raw sources operate on incompatible time grids: BOTZ data exists only on trading days, while arXiv receives submissions every day of the week including weekends and public holidays. The `00_MasterDatasetAcquisition.ipynb` notebook documents the merging strategy in full detail.

**Key design decision — weekend accumulation:** For each trading day `t`, the assigned paper count is the **sum** of all arXiv submissions from all calendar days since the previous trading day (exclusive) up to and including `t`. This means Monday's count = Saturday + Sunday + Monday submissions. A naive calendar-date merge would silently discard all weekend submissions, introducing a systematic downward bias on every Monday.

---

##  Research Pipeline

```
Data Acquisition → Dataset Preparation → EDA → Hypothesis Testing → Event Study → Machine Learning
```

### Notebook 1 — Exploratory Data Analysis (`01_EDA_improved`)

Establishes the foundational understanding of both time series before any modelling.

**Key steps:**
- Data loading & quality validation (missing values, zero-paper days, extreme return days)
- Extended descriptive statistics: skewness, kurtosis, variance for paper counts and ETF returns
- Time-series visualisation of arXiv submission volumes with 30-day moving average and ±1 std band
- BOTZ ETF price, daily returns, and rolling annualised volatility panels with event annotations (COVID crash, rate-hike bear)
- Distribution analysis with Shapiro-Wilk normality tests and KDE overlays

**Key finding:** The raw correlation between paper counts and ETF price (r ≈ 0.49) is **spurious** — both series share an upward trend driven by the passage of time. After removing trends, the stationary correlation collapses to r ≈ −0.02.

---

### Notebook 2 — Hypothesis Testing (`02_HT_improved`)

Tests whether paper counts contain *any* statistically significant predictive information about future ETF movements using Granger causality.

**Hypotheses:**
- **H_A:** Past AI paper volumes Granger-cause BOTZ returns
- **H_B:** Past BOTZ returns Granger-cause AI paper volumes

**Methodology:**
- Stationarity transformation: ETF price → percentage return; paper count → log-difference `Δlog(1+count)`
- Augmented Dickey-Fuller (ADF) unit root testing with stationarity dashboards (rolling mean, rolling std, ACF/PACF)
- VAR-based optimal lag selection via AIC and BIC criteria (searched over 1–30 lags)
- Granger F-tests across multiple horizons: 3, 5, 14, 30, 60, 90, 120, 180 days plus AIC/BIC-optimal lags
- Results visualised as p-value heatmaps and −log₁₀(p) bar charts

**Key finding:** A statistically significant but economically negligible 3-day Granger precedence was detected (p < 0.05), but it explains less than **0.2% of return variance** — insufficient for any practical trading signal.

---

### Notebook 3 — Event Study (`03_EDAHT-2_improved`)

Shifts from aggregate to individual: do specific landmark AI publications trigger measurable abnormal trading activity around the event date?

**Event set (11 papers, 2016–2023):**

| Paper | Date |
|-------|------|
| Grad-CAM | 2016-10-07 |
| GCN | 2017-03-26 |
| Transformer (Attention Is All You Need) | 2017-06-12 |
| BERT | 2018-10-11 |
| EfficientNet | 2019-05-28 |
| GPT-3 | 2020-05-28 |
| ViT | 2020-10-22 |
| CLIP | 2021-02-26 |
| LoRA | 2021-06-17 |
| AlphaFold 2 | 2021-07-15 |
| GPT-4 Technical Report | 2023-03-15 |

**Metrics computed:**
- **SCAV** (Standardised Cumulative Abnormal Volume): z-score relative to a 120-day estimation window, normalised by √n
- **CAR** (Cumulative Abnormal Return): market-model adjusted returns (BOTZ excess over QQQ β-adjusted benchmark), tested with t-statistics

**Methodological safeguards:**
- Minimum 60-trading-day inter-event spacing filter to prevent contamination
- Multi-window robustness check: [−1,+1], [−3,+3], [−5,+10] day windows
- Benjamini-Hochberg FDR correction for multiple comparisons
- AlphaFold 3 removed (date falls outside dataset range)

---

### Notebook 4 — Machine Learning (`04_ML_improved`)

Tests whether non-linear ML models can extract a predictive signal that simple correlation analysis misses.

**Problem framing:**

| Task | Target | Metrics |
|------|--------|---------|
| Regression | `daily_return` (continuous) | RMSE, R² |
| Classification | `direction` (Up/Down) | Accuracy, Macro F1 |
| Clustering | — (unsupervised) | Silhouette score, ARI |

**Models:**
- Regression: Linear Regression · Random Forest · XGBoost · SVR · LSTM (PyTorch)
- Classification: Logistic Regression · Random Forest
- Clustering: K-Means with elbow method and silhouette selection

**Feature engineering (16 features):**

| Feature | Rationale |
|---------|-----------|
| `paper_lag_1/3/5/10` | Granger-motivated lags |
| `paper_ma_7 / paper_ma_30` | Short vs. long-run publication trend |
| `paper_momentum` | Acceleration in research output (7d MA − 30d MA) |
| `log_paper_diff` | Stationary paper count (preferred by HT results) |
| `return_lag_1/5` | Price momentum |
| `abs_return_lag1` | Volatility clustering proxy |
| `dow_Mon/Tue/Wed/Thu` | Day-of-week dummies (arXiv deadline effect) |

**Methodological safeguards:**
- Chronological 80/20 train/test split — no future data leaks into training
- TimeSeriesSplit (k=5) cross-validation for all hyperparameter searches
- StandardScaler fitted on train set only; `transform()` applied separately to test set

---

##  Key Results

| Stage | Finding |
|-------|---------|
| EDA | Raw r ≈ 0.49 (price vs. count) is spurious; stationary r ≈ −0.02 |
| Granger Test | Weak 3-day precedence (p < 0.05), explains < 0.2% of variance |
| Event Study | Some landmark papers show abnormal volume; price impact is inconsistent across event windows |
| Regression | All five models yield **R² < 0** on the held-out test set |
| Classification | No model significantly exceeds the 50% random baseline |
| Clustering | K-Means finds coherent structure (Silhouette > 0.3) but clusters do **not** align with market direction (ARI ≈ 0) |

---

##  Interpretation

The consistently near-zero or negative predictive performance across all model types is consistent with the **Efficient Market Hypothesis**: arXiv paper counts are publicly available information that is rapidly absorbed into asset prices. No model — linear or non-linear, shallow or deep — reliably extracts an exploitable signal from this data alone.

The counterintuitive clustering finding deserves note: *low-publication days tend to have slightly higher mean returns than high-publication days*, suggesting that publication bursts may lag behind market pricing of underlying AI developments.

**A note on negative results:** The failure of all models to outperform a naive baseline is itself the scientific contribution. It provides rigorous, reproducible evidence against a hypothesis that might seem intuitive on the surface — and that has real implications for anyone considering research-activity-based investment strategies.

---

##  Limitations & Suggested Extensions

| Limitation | Suggested Improvement |
|------------|----------------------|
| Only paper *count* used | Sentiment / topic modelling of abstracts |
| No macroeconomic controls | Add VIX, interest rates, sector indices |
| Daily frequency | Weekly aggregation may reduce noise |
| Single ETF (BOTZ) | Extend to ROBO, AIQ, or individual AI stocks |
| Linear Granger only | Non-linear Granger / transfer entropy |
| Small event set (n=11) | Expand to broader set of high-citation papers |
| No post-2023 validation | Validate 3-day Granger result on 2024 data |

---

##  Requirements

Install all dependencies via:

```bash
pip install -r requirements.txt
```

Core packages: `numpy`, `pandas`, `matplotlib`, `seaborn`, `scipy`, `statsmodels`, `scikit-learn`, `xgboost`, `torch`, `yfinance`, `sickle`

---

##  Reproducing the Analysis

Run the notebooks in the following order. Each notebook saves its output for the next stage.

```bash
# Step 1 — Data collection (optional: pre-built CSVs are already included)
jupyter notebook 00_ETFDataAcquisition.ipynb
jupyter notebook 00_PaperDataAcquisition.ipynb
jupyter notebook 00_MasterDatasetAcquisition.ipynb

# Step 2 — Analysis pipeline
jupyter notebook 01_EDA_improved.ipynb
jupyter notebook 02_HT_improved.ipynb
jupyter notebook 03_EDAHT-2_improved.ipynb
jupyter notebook 04_ML_improved.ipynb
```

> **Note:** If you skip Step 1, `ai_market_master_dataset.csv` is already included in the repository and Steps 2–4 can be run directly.

---

##  Methodological Notes

- All analysis is conducted on **stationary** transformations of both series. Using raw price levels inflates correlations via shared trends — this is demonstrated explicitly in `01_EDA_improved`.
- **Granger causality is not true causality.** A significant result means predictive precedence, not mechanism.
- The event study uses **QQQ-adjusted returns** (market model) to isolate BOTZ-specific reactions beyond broad tech sentiment.
- LSTM results should be interpreted with caution: without explicit sequence windowing, the model may not fully exploit temporal dependencies.

---

##  AI Assistance Disclosure

**Specific areas where AI assistance was used:**
- Refining notebook cell structure and markdown documentation
- Debugging data pipeline code
- Improving plot formatting and panel layouts

AI assistance was not used to generate hypotheses, select statistical tests, interpret findings, or draw conclusions.

---
