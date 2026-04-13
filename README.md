# DSA-210-Project
# Analysis of the Relationship Between Academic AI Research and AI Stock Market Performance

## Project Overview
This project investigates the relationship between academic Artificial Intelligence (AI) research output and the financial performance of the AI sector. By analyzing daily AI publication counts and the price movements of the **Global X Robotics & Artificial Intelligence ETF (BOTZ)**, we aim to determine if academic trends can predict market behavior.

## Motivation
The rapid advancement in AI has created a massive financial surge. While market movements are often attributed to news and earnings, the role of pure academic output—often the precursor to commercial technology—is less studied. This project seeks to quantify whether academic "hype" or productivity acts as a leading indicator for financial markets.

## Objectives
* **Trend Identification:** Analyze historical price movements of BOTZ and daily AI paper counts to find long-term connections.
* **Correlation Analysis:** Quantify the statistical link between research volume and market prices.
* **Causality Testing:** Use the **Granger Causality Test** to determine the direction of the relationship: Does research drive the market, or does market performance drive academic interest?

## Data Sources
* **BOTZ ETF Data:** Daily historical prices representing the AI and robotics sector.
* **AI Academic Papers:** Daily publication counts filtered for AI-related topics.
* **Master Dataset:** A merged version of these two sources (`ai_market_master_dataset.csv`), synchronized by date.

## Data Analysis

### 1. Data Collection and Cleaning
* Merged financial and academic datasets.
* Handled missing values (`NaN`) arising from public holidays and non-trading days.
* Computed a **30-day moving average** for AI paper counts to smooth out daily noise and visualize the broader academic trend.

### 2. Stationarity & Transformation
To perform valid statistical testing, the data was transformed to achieve stationarity:
* **ETF Price:** Converted to **daily percentage returns**.
* **AI Paper Count:** A **Log-Difference** transformation was applied. This was crucial to stabilize the increasing variance (heteroskedasticity) and handle the exponential growth in publication volume observed over time.

### 3. Exploratory Data Analysis (EDA)
* Plotted rolling statistics (mean and standard deviation) to verify the stationarity of the transformed series.
* Visualized the correlation using scatter plots and regression lines, revealing a moderate positive relationship (Pearson Correlation: ~0.48).

## Hypothesis & Causality Analysis
The **Augmented Dickey-Fuller (ADF) Test** was used to confirm that both time series were stationary ($p < 0.05$) before proceeding to causality testing.

### Granger Causality Test
We tested the predictive relationship across multiple time horizons (lags of 3, 14, 30, and 60 days):
* **Hypothesis A (Research -> Market):** Do AI publication trends help forecast BOTZ ETF prices?
* **Hypothesis B (Market -> Research):** Does market performance help forecast subsequent AI academic interest?

## Findings and Results
The analysis yielded a specific and statistically significant insight:

* **Short-Term Predictive Power (Lag 3):** Hypothesis A was **accepted** only at the 3-day lag ($p = 0.0310$). This suggests that the market reacts very quickly to academic output; a surge in research papers provides a predictive signal for market movements within a 72-hour window.
* **Market Efficiency:** At longer horizons (14, 30, and 60 days), the relationship became insignificant. This indicates that the market is efficient and "prices in" the information from research breakthroughs almost immediately.
* **Unidirectional Relationship:** Hypothesis B was **rejected** across all horizons (p-values $> 0.05$). Market performance does not appear to drive academic publication volume, confirming that academic research cycles are independent of daily stock market volatility.

**Conclusion:** Academic AI research is a short-term leading indicator for the AI market, reflecting high market sensitivity to technological progress.