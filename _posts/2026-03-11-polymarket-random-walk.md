---
title: "Polymarket Crypto Markets: Outcome Dependence and Random-Walk Exploration"
date: 2026-03-11 13:00:00 +0800
categories: [Weekly Research, Polymarket]
tags: [polymarket, microstructure, markov-chain, random-walk]
math: true
toc: true
---

## Objective

This research is divided into two stages.

### Phase I: Outcome-level research

To study whether the resolved outcomes of Polymarket crypto markets exhibit:

1. directional symmetry (Up vs Down),
2. serial dependence across adjacent markets,
3. simple Markov predictability.

The key question is:

Can the sequence of market resolutions be treated as memoryless, or does short-horizon dependence exist?

---

### Phase II: Price-level research

After the outcome sequence is established, extend the study to the underlying crypto price process itself:

1. test whether returns across 5min / 15min / 1h horizons follow a random-walk benchmark,
2. test whether return distributions are symmetric,
3. compare outcome process vs underlying price process.

This stage requires external price data because Gamma API does not directly provide underlying asset prices.

---

## Phase I Data Plan (Outcome-level)

### Symbols

```text
[btc, eth, sol, xrp]
```

### Timeframes

```text
5min / 15min / 1hour
```

### Time ranges

5min markets

Fetch all available data from launch date onward.

Known:
```text
BTC 5min launched on 2026-02-13
```
For symbols with unknown launch dates:

- iterate candidate slugs,

- query Gamma API,

- skip if null / false.
---
15min markets

Fetch:

```text
2025-12-11 → 2026-03-10
```
(approximately 3 months)

---
1h markets

Fetch:
```text
2025-03-11 → 2026-03-10
```
(approximately 1 year)

---
Slug Construction

Slug format prototype:

```text
xrp-updown-5m-1773216300
```
The timestamp part is Unix time.
The symbol and timeframe parts change accordingly.

---
Data Fields to Store

For each market:

- symbol

- timeframe

- slug

- unix timestamp

- human-readable timestamp

- resolved outcome (Up / Down)

Recommended final schema:
```text
symbol | timeframe | slug | timestamp_unix | timestamp_dt | resolved_outcome
```
---

## Phase I Statistical Questions

### 1. Symmetry
Check whether:
```text
P(Up) ≈ P(Down)
```
for each symbol and timeframe.

---
### 2. Serial dependence
Check:
```text
P(X_t | X_{t-1})
```
Transition matrix:
```text
Up → Up
Up → Down
Down → Up
Down → Down
```
---

### 3. Markov prediction
Test whether first-order Markov transition improves prediction beyond naive baseline.

---
## Phase II Preview (Price-level)
Later merge external crypto price data:

Recommended fields:
```text
open | close | return | sign(return) | abs(return)
```
Data sources:
- Chainlink
- Binance / Coinbase OHLC
- other stable crypto market feed

This stage allows:
- random walk tests,
- variance ratio,
- runs test,
- return symmetry,
- horizon comparison.
---
