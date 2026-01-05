# Currency Pairs Trading Engine (FX Statistical Arbitrage)

A reproducible **research + backtesting** pipeline for **FX pairs trading** using **cointegration (Engle–Granger)** and **mean-reversion** signals.

The goal is not to “prove” a strategy works in one backtest, but to demonstrate a disciplined workflow:
**data ingestion → systematic screening → diagnostics → leak-safe evaluation → cost & robustness checks → saved artifacts**.

---

## Main findings (from this universe)

- **USDCNH–USDCNY** emerges as the highest-conviction pair by both **statistics** and **market structure**:
  - extremely low Engle–Granger cointegration p-value and residual ADF p-value  
  - fast mean-reversion behavior (≈ **2-day half-life**)

- **AUDUSD–NZDUSD** is included as a widely used **benchmark pair** (similar macro exposure) to compare the pipeline’s behavior on a “classic” candidate.

> FX costs vary by venue/liquidity/size. This project reports **sensitivity across bps scenarios** rather than assuming one “true” trading cost.

---

## What’s inside

### 1) Data ingestion (Alpha Vantage)
- Pulls daily FX spot series for a curated universe (G10 anchors, select EM basket, EU crosses, CNH/CNY control pair).
- Optional **guardrail** to restrict the analysis window (e.g., **2018+**) for regime relevance.
- Local caching to make runs reproducible and reduce API calls.

### 2) Pair screening (Engle–Granger)
For each candidate pair `(Y, X)` in log space:
- Estimate hedge ratio via OLS: `log(Y) = alpha + beta*log(X) + residual`
- Compute screening statistics:
  - `p_coint` — Engle–Granger cointegration p-value  
  - `p_adf` — ADF p-value on the residual spread  
  - `half_life_days` — mean-reversion half-life estimate  
- Rank pairs by **residual stationarity** + **tradeability** (half-life)

### 3) Diagnostics (visual sanity checks)
For selected pairs we visualize:
- normalized prices (co-movement)
- residual spread (hedged deviation)
- spread z-score (signal behavior)

### 4) Backtest (single-timeline, leak-safe)
- One continuous position series over time (no stitched overlapping windows)
- Rolling refits of `alpha/beta` (monthly by default)
- Z-score trading rule:
  - enter when `|z| > entry_z`
  - exit when `|z| < exit_z`
- Walk-forward structure with training window + **purge gap** to reduce leakage
- Turnover-based transaction costs (bps)

### 5) Cost sensitivity + robustness
- Cost sweeps across bps scenarios (e.g., 0.5 → 10 bps)
- Parameter sweeps that prioritize **cost-robust** performance (e.g., Sharpe@3bps and Sharpe@5bps)

---

## Why focus on USDCNH–USDCNY?

This pair stands out in the screening table for three reasons:

1) **Strongest statistical evidence of stationarity**  
   Residual stationarity is far stronger than the rest of the universe (very low `p_coint` and `p_adf`).

2) **Fast mean reversion (tradeable speed)**  
   Estimated half-life ≈ **2 days**, while many other candidates revert on the order of **weeks** (~50 days), increasing regime risk and reducing practical signal quality at daily frequency.

3) **Clear structural explanation**  
   **CNY** is onshore RMB and **CNH** is offshore RMB. The two markets can deviate due to segmentation and policy constraints, but are structurally linked—making this relationship more defensible than arbitrary cross-pairs.
