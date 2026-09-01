# BTC Bull & Bear Cycle Analysis

---

## Data

The data used is daily BTC closing prices sourced from `data/btc_daily.csv`, covering Nov 2013 to Aug 2026. Each row represents one day and includes open, high, low, close prices and trading volume.

---

## Section 1: Cycle Detection

To identify bull and bear cycles, use an ex-post method - means, look at the full price history and find the true peaks and troughs. References Bry-Boschan (1971) and Pagan & Sossounov (2003).

There are three steps:

**Step 1 - Find candidate peaks and troughs**

- peak: day where the closing price is higher than all prices within 90 days on either side.
- trough: day where the closing price is lower than all prices within 90 days on either side.

90-day window (3 months each side) was used to avoid picking up short-term noise.

**Step 2 - Enforce strict alternation**

A valid cycle must alternate: peak, trough, peak, trough...

If two peaks appear in a row, keep only the higher one. If two troughs appear in a row, we keep only the lower one.  
This ensures every bear market has exactly one peak and one trough, with no ambiguity.

**Step 3 - Filter by threshold and minimum duration**

2 additional filters were used to better classify a full cycle rather than just a local dip or bounce (due to bitcoin's high volatility). Note that these are debatable and can be adjusted:

- 30% threshold: the move from peak to trough (or trough to peak) must be at least 30% - this is stricter than the standard 20% to account for Bitcoin's high day-to-day volatility
- 60-day minimum: the cycle leg must last at least 60 days - removes short-lived crashes that recover quickly

---

## Section 1 Results

### Bear Market Cycles Detected

| Cycle      | Peak Date  | Trough Date | Peak Price | Trough Price | Drawdown | Duration |
| ---------- | ---------- | ----------- | ---------- | ------------ | -------- | -------- |
| 2013–2015  | 2013-11-29 | 2015-01-14  | $1,165     | $175         | -85.0%   | 411 days |
| 2017–2018  | 2017-12-16 | 2018-12-15  | $19,357    | $3,183       | -83.6%   | 364 days |
| 2019–2020  | 2019-06-26 | 2020-03-12  | $12,876    | $4,857       | -62.3%   | 260 days |
| 2021 (Apr) | 2021-04-13 | 2021-07-20  | $63,587    | $29,796      | -53.1%   | 98 days  |
| 2021–2022  | 2021-11-08 | 2022-11-21  | $67,555    | $15,760      | -76.7%   | 378 days |
| 2025–2026  | 2025-10-06 | 2026-02-05  | $124,720   | $62,791      | -49.7%   | 122 days |

> Note on the April 2021 cycle: Most market observers refer to this as a "mid-cycle correction" rather than a full bear market, because BTC went on to make a new all-time high ($69k) just four months later. However, by the objective criteria of this algorithm (≥30% decline, ≥60 days), it qualifies, flagging it here.

### Bull Market Cycles Detected

| Cycle       | Trough Date | Peak Date  | Trough Price | Peak Price | Gain     | Duration |
| ----------- | ----------- | ---------- | ------------ | ---------- | -------- | -------- |
| 2015 (mini) | 2015-01-14  | 2015-07-12 | $175         | $311       | +77.6%   | 179 days |
| 2015–2017   | 2015-08-24  | 2017-12-16 | $213         | $19,357    | +9001%   | 845 days |
| 2018–2019   | 2018-12-15  | 2019-06-26 | $3,183       | $12,876    | +304.5%  | 193 days |
| 2020–2021   | 2020-03-12  | 2021-04-13 | $4,857       | $63,587    | +1209.2% | 397 days |
| 2021 (mini) | 2021-07-20  | 2021-11-08 | $29,796      | $67,555    | +126.7%  | 111 days |
| 2022–2024   | 2022-11-21  | 2024-03-13 | $15,760      | $73,135    | +364.1%  | 478 days |
| 2024–2025   | 2024-09-06  | 2025-01-21 | $53,950      | $106,159   | +96.8%   | 137 days |
| 2025        | 2025-04-08  | 2025-10-06 | $76,252      | $124,720   | +63.6%   | 181 days |
| 2026        | 2026-02-05  | 2026-05-10 | $62,791      | $82,200    | +30.9%   | 94 days  |

### Average Cycle Duration

| Phase | Average Duration        | Number of Cycles |
| ----- | ----------------------- | ---------------- |
| Bear  | ~272 days (~8.9 months) | 6                |
| Bull  | ~291 days (~9.5 months) | 9                |

---

## Section 2: Post-Breakout Drift Test

Test whether BTC exhibits positive post-breakout drift following a >+5σ weekly return and break above 200 day EMA, where σ is estimated using the preceding 60-day realised volatility and scaled to a weekly horizon. Measure subsequent 30D, 60D, 120D and 365D returns.

---

### Calculations

**Step 1 — Daily log returns**

Each day's return is computed as:

```
r_t = ln(close_t / close_{t-1})
```

**Step 2 — Rolling realised volatility (annualised)**

For each day, compute the rolling standard deviation of daily log returns, then annualise:

```
σ_annual_t = std(r_{t-N}, ..., r_t) × √252
```

**Step 3 - Scale to weekly volatility**

```
σ_weekly_t = σ_annual_t / sqrt(52)
```

Dividing by sqrt(52) converts annual vol to the expected standard deviation of a weekly return, assuming independent daily returns.

**Step 4 — Weekly log returns**

Daily closes resampled to week-end (Sunday).

```
R_week_t = ln(close_t / close_{t-1 week})
```

**Step 5 — Signal: normalised weekly return**

```
signal_t = R_week_t / σ_weekly_{t-1}
```

The prior week's vol is used (shifted back 1 week).
All weeks where `signal > 5` are flagged.

**Step 6 - check > 200 day EMA**

Keep only breakout weeks where the closing price was\*above the 200-day EMA.

**Step 7 - Measure forward returns**

```
fwd_Nd = (close_{t+N} / close_t) - 1    for N ∈ {30, 60, 120, 365}
```

---

### Section 2 Results

#### 60-Day Vol Lookback

**All >5σ weeks (above 200-day EMA):**

| Date       | Log Weekly Ret | Ann. Vol | Weekly Vol | Signal |
| ---------- | -------------- | -------- | ---------- | ------ |
| 2016-05-29 | +17.1%         | 22.3%    | 3.1%       | 5.52σ  |
| 2026-08-23 | +21.3%         | 23.4%    | 3.2%       | 6.56σ  |

**Forward returns: (2026-08-23 excluded - too recent)**

| Date       | Signal | fwd 30D | fwd 60D | fwd 120D | fwd 365D |
| ---------- | ------ | ------- | ------- | -------- | -------- |
| 2016-05-29 | 5.52σ  | +25.1%  | +26.4%  | +17.3%   | +339.2%  |
| 2026-08-23 | 6.56σ  | —       | —       | —        | —        |

---

#### 60-Day Vol Lookback, 4σ Threshold

**All >4σ weeks:**

| Date       | Log Weekly Ret | Ann. Vol | Weekly Vol | Signal |
| ---------- | -------------- | -------- | ---------- | ------ |
| 2016-05-29 | +17.1%         | 22.3%    | 3.1%       | 5.52σ  |
| 2019-04-07 | +23.1%         | 39.3%    | 5.5%       | 4.24σ  |
| 2023-01-15 | +19.8%         | 32.2%    | 4.5%       | 4.44σ  |
| 2026-08-23 | +21.3%         | 23.4%    | 3.2%       | 6.56σ  |

**Forward returns (2026-08-23 excluded - too recent):**

| Date       | Signal | fwd 30D | fwd 60D | fwd 120D | fwd 365D |
| ---------- | ------ | ------- | ------- | -------- | -------- |
| 2016-05-29 | 5.52σ  | +25.1%  | +26.4%  | +17.3%   | +339.2%  |
| 2019-04-07 | 4.24σ  | +11.2%  | +51.0%  | +128.2%  | +41.7%   |
| 2023-01-15 | 4.44σ  | +6.4%   | +20.0%  | +30.1%   | +103.6%  |
| 2026-08-23 | 6.56σ  | —       | —       | —        | —        |

---

**Max drawdown (2026-08-23 excluded - too recent):**

Max drawdown measures the worst peak-to-trough decline within each holding window, starting from the breakout date's close.

| Date       | Signal | MDD 30D | MDD 60D | MDD 120D | MDD 365D | MDD to Bull End | Bull End   |
| ---------- | ------ | ------- | ------- | -------- | -------- | --------------- | ---------- |
| 2016-05-29 | 5.52σ  | -21.4%  | -21.4%  | -29.5%   | -29.6%   | -35.0%          | 2017-12-16 |
| 2019-04-07 | 4.24σ  | -7.4%   | -12.3%  | -26.7%   | -62.3%   | -13.0%          | 2019-06-26 |
| 2023-01-15 | 4.44σ  | -8.9%   | -18.6%  | -18.6%   | -20.1%   | -20.1%          | 2024-03-13 |
