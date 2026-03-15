---
title: "Outcome Structure in Polymarket Crypto Markets: Why 15-Minute Windows Matter"
date: 2026-03-11 13:00:00 +0800
categories: [Weekly Research, Polymarket]
tags: [polymarket, microstructure, markov-chain, random-walk]
math: true
toc: true
---

At first glance, these binary outcomes look close to a fair coin. The interesting part is that the 15-minute layer still shows a repeatable short-memory pattern.

## 1. Motivation

Before touching a full signal stack (price features, execution, risk overlays), I wanted to answer a simpler question: is there any structure in the resolved outcomes themselves?

This is a diagnostics-first exercise, not a "we can predict everything" claim. If outcomes are fully memoryless, later models must rely almost entirely on external features. If mild structure exists, it can still serve as a useful conditioning prior.

So the target here is intentionally modest: check whether short-horizon dependence is present, persistent, and practically non-trivial.

## 2. Data and scope

- Symbols: `btc`, `eth`, `sol`, `xrp`
- Timeframes: `5m`, `15m`, `1h`
- Variable: `resolved_outcome ∈ {Up, Down}`
- `NOT_EXISTS` rows are excluded from directional tests
- Continuity rule: if timestamp gaps or `NOT_EXISTS` occur, start a new contiguous segment

This post stays at the outcome-process level. It is not yet a price-feature model or an execution-aware trading backtest.

All timestamps below are in UTC.

**Quick coverage by timeframe**

| timeframe | start_utc | end_utc | symbols covered | `NOT_EXISTS` ratio |
|---|---|---|---|---|
| 5m | 2026-02-13 00:00 (BTC); 2026-02-18 19:40 (ETH/SOL/XRP) | 2026-03-10 23:55 | btc/eth/sol/xrp | ~0% |
| 15m | 2025-12-11 00:00 | 2026-03-10 23:45 | btc/eth/sol/xrp | ~0.10%-0.12% |
| 1h | 2025-03-11 04:00 | 2026-03-11 03:00 | btc/eth/sol/xrp | ~26.8%-28.8% |

One practical note on platform effects: 1h has a large and synchronized `NOT_EXISTS` block across symbols (roughly 27%–29%), while 15m is around 0.10%–0.12% and 5m is near zero. That pattern is much more consistent with listing/availability constraints on the platform than with random fetch failures.

<details>
<summary>Appendix: full symbol-level coverage table</summary>

| symbol | timeframe | start_utc | end_utc | N_valid | N_NOT_EXISTS |
|---|---|---|---|---:|---:|
| btc | 5m | 2026-02-13 00:00 | 2026-03-10 23:55 | 7488 | 0 |
| eth | 5m | 2026-02-18 19:40 | 2026-03-10 23:55 | 5812 | 0 |
| sol | 5m | 2026-02-18 19:40 | 2026-03-10 23:55 | 5812 | 0 |
| xrp | 5m | 2026-02-18 19:40 | 2026-03-10 23:55 | 5811 | 0 |
| btc | 15m | 2025-12-11 00:00 | 2026-03-10 23:45 | 8631 | 9 |
| eth | 15m | 2025-12-11 00:00 | 2026-03-10 23:45 | 8631 | 9 |
| sol | 15m | 2025-12-11 00:00 | 2026-03-10 23:45 | 8631 | 9 |
| xrp | 15m | 2025-12-11 00:00 | 2026-03-10 23:45 | 8630 | 10 |
| btc | 1h | 2025-03-11 04:00 | 2026-03-11 03:00 | 6410 | 2349 |
| eth | 1h | 2025-03-11 04:00 | 2026-03-11 03:00 | 6410 | 2349 |
| sol | 1h | 2025-03-11 04:00 | 2026-03-11 03:00 | 6394 | 2365 |
| xrp | 1h | 2025-03-11 04:00 | 2026-03-11 03:00 | 6238 | 2521 |

</details>

## 3. Research questions

1. Is unconditional `P(Up)` approximately 0.5?
2. Are outcomes independent over time, or does first-order dependence exist?
3. Which timeframe carries the strongest dependence signal?
4. Does second-order context add practical value beyond first-order transitions?
5. Is the dependence stable in rolling windows?
6. Are regime-split findings robust under strict information timing (`shift`) versus descriptive timing (`noshift`)?

## 4. Method overview

At a high level, the analysis pipeline is:

- unconditional symmetry diagnostics by timeframe
- formal symmetry tests by symbol/timeframe
- transition-based independence tests (P_up_given_up vs P_up_given_down)
- run statistics and weekday/month stability checks
- 15m first-order and second-order Markov summaries
- rolling stability diagnostics (window = 200)
- simple chronological 70/30 benchmark versus naive baselines
- gap-regime split diagnostics
- explicit `shift` vs `noshift` timing-sensitivity check

The guiding principle is simple: descriptive structure is not automatically tradable alpha.

### 4.1 Notation and core formulas

Let \(X_t \in \{0,1\}\), where \(1=\text{Up}\), \(0=\text{Down}\).

Symmetry benchmark:

\[
H_0: P(X_t=1)=0.5
\]

First-order Markov benchmark:

\[
P(X_t \mid X_{t-1},X_{t-2},\dots)=P(X_t \mid X_{t-1})
\]

Transition MLEs (from counts \(N_{UU},N_{UD},N_{DU},N_{DD}\)):

\[
\hat P(\text{Up}\mid \text{Up})=\frac{N_{UU}}{N_{UU}+N_{UD}},\quad
\hat P(\text{Up}\mid \text{Down})=\frac{N_{DU}}{N_{DU}+N_{DD}}
\]

\[
\Delta_P=\hat P(\text{Up}\mid \text{Up})-\hat P(\text{Up}\mid \text{Down})
\]

Second-order context MLE:

\[
\hat P(X_t=1\mid X_{t-1}=i,X_{t-2}=j)=
\frac{N_{(j,i)\rightarrow 1}}{N_{(j,i)\rightarrow 0}+N_{(j,i)\rightarrow 1}}
\]

Rolling gap definition (window \(W\)):

\[
\text{Gap}_t^{(W)}=\hat P_t^{(W)}(\text{Up}\mid \text{Down})-\hat P_t^{(W)}(\text{Up}\mid \text{Up})
\]

with rolling MLEs computed from transition counts inside the rolling window.

Standardized gap in the notebook:

\[
\text{SE}_t=
\sqrt{
\frac{\hat p_{UD,t}(1-\hat p_{UD,t})}{n_{\text{prev down},t}}
+
\frac{\hat p_{UU,t}(1-\hat p_{UU,t})}{n_{\text{prev up},t}}
}
,\quad
z\_{{gap},t}=\frac{\text{Gap}_t}{\text{SE}_t}
\]

Simple reversal predictor used for benchmark:

\[
\hat X_t=
\begin{cases}
0,& X_{t-1}=1\\
1,& X_{t-1}=0
\end{cases}
\]

Timing discipline note:
- `noshift`: regime features at \(t\) may include transition information ending at \(t\).
- `shift`: regime features used at \(t\) are shifted to use information available by \(t-1\) only.

## 5. Main findings

### 5.1 Unconditional symmetry is very close to fair

**Table 1. Unconditional Up Ratio by Timeframe**

| timeframe | N | up_ratio | ci95_low | ci95_high |
|---|---:|---:|---:|---:|
| 5m | 24923 | 0.5013 | 0.4951 | 0.5076 |
| 15m | 34523 | 0.4971 | 0.4918 | 0.5024 |
| 1h | 25452 | 0.5034 | 0.4973 | 0.5096 |

The first takeaway is straightforward: unconditional direction is balanced across all three horizons. If edge exists, it does not come from a raw Up/Down imbalance.

### 5.2 Formal symmetry test mostly confirms the same story

**Table 2. Symmetry Test by Symbol and Timeframe**

| symbol | timeframe | N | up_ratio | z | p_value |
|---|---|---:|---:|---:|---:|
| btc | 15m | 8631 | 0.4980 | -0.3767 | 0.7064 |
| eth | 15m | 8631 | 0.4988 | -0.2260 | 0.8212 |
| sol | 15m | 8631 | 0.4975 | -0.4628 | 0.6435 |
| xrp | 15m | 8630 | 0.4941 | -1.0980 | 0.2722 |
| btc | 1h | 6410 | 0.4997 | -0.0500 | 0.9602 |
| eth | 1h | 6410 | 0.5042 | 0.6745 | 0.5000 |
| sol | 1h | 6394 | 0.5131 | 2.1010 | 0.0356 |
| xrp | 1h | 6238 | 0.4965 | -0.5571 | 0.5775 |
| btc | 5m | 7488 | 0.5065 | 1.1325 | 0.2574 |
| eth | 5m | 5812 | 0.4991 | -0.1312 | 0.8956 |
| sol | 5m | 5812 | 0.5048 | 0.7346 | 0.4626 |
| xrp | 5m | 5811 | 0.4934 | -1.0101 | 0.3124 |

Only `sol-1h` is marginally significant in this sample (`p≈0.0356`). I treat that as a local exception, not the main story.

### 5.3 The strongest dependence appears at 15m, and it is reversal-like

**Figure 1. 15m P(Up given Up) vs P(Up given Down) by symbol**

![Figure 1](/assets/images/2026-03-11-polymarket-random-walk/outcome/figure1_transition_15m.png)

**Table 3. Transition-Independence Test Summary**

| symbol | timeframe | N_pairs | P_up_given_up | P_up_given_down | delta_P | z | p_value |
|---|---|---:|---:|---:|---:|---:|---:|
| btc | 15m | 8627 | 0.4675 | 0.5282 | -0.0606 | -5.6324 | 1.78e-08 |
| eth | 15m | 8627 | 0.4633 | 0.5344 | -0.0711 | -6.6003 | 4.10e-11 |
| sol | 15m | 8627 | 0.4743 | 0.5205 | -0.0463 | -4.2981 | 1.72e-05 |
| xrp | 15m | 8625 | 0.4586 | 0.5284 | -0.0698 | -6.4852 | 8.86e-11 |
| btc | 5m | 7487 | 0.4916 | 0.5218 | -0.0302 | -2.6153 | 0.0089 |
| eth | 5m | 5811 | 0.4810 | 0.5170 | -0.0360 | -2.7420 | 0.0061 |
| sol | 5m | 5811 | 0.4944 | 0.5153 | -0.0209 | -1.5943 | 0.1109 |
| xrp | 5m | 5809 | 0.4895 | 0.4968 | -0.0072 | -0.5518 | 0.5811 |
| btc | 1h | 6401 | 0.4859 | 0.5136 | -0.0277 | -2.2124 | 0.0269 |
| eth | 1h | 6400 | 0.4904 | 0.5180 | -0.0276 | -2.2054 | 0.0274 |
| sol | 1h | 6374 | 0.5050 | 0.5221 | -0.0170 | -1.3584 | 0.1743 |
| xrp | 1h | 6214 | 0.4955 | 0.4968 | -0.0013 | -0.1056 | 0.9159 |

Reading this table row by row, the center of gravity is clear:
- 15m is the cleanest and strongest horizon: all four symbols reject independence, and the sign is reversal-like.
- 5m has partial evidence, but not the same consistency.
- 1h is weaker and noisier, with an added availability problem.

### 5.4 Availability matters: 1h is materially distorted by `NOT_EXISTS`

**Table 4. `NOT_EXISTS` Availability Diagnostics**

| symbol | timeframe | N_total | N_not_exists | ratio |
|---|---|---:|---:|---:|
| btc | 15m | 8640 | 9 | 0.0010 |
| eth | 15m | 8640 | 9 | 0.0010 |
| sol | 15m | 8640 | 9 | 0.0010 |
| xrp | 15m | 8640 | 10 | 0.0012 |
| btc | 5m | 7488 | 0 | 0.0000 |
| eth | 5m | 5812 | 0 | 0.0000 |
| sol | 5m | 5812 | 0 | 0.0000 |
| xrp | 5m | 5811 | 0 | 0.0000 |
| btc | 1h | 8759 | 2349 | 0.2682 |
| eth | 1h | 8759 | 2349 | 0.2682 |
| sol | 1h | 8759 | 2365 | 0.2700 |
| xrp | 1h | 8759 | 2521 | 0.2878 |

So for structure work, 15m is a cleaner substrate. 1h can still be studied, but continuity distortions are too large to ignore.

### 5.5 Stability/runs are directionally consistent with the transition evidence

From weekday/month range diagnostics and run summaries:
- 15m appears comparatively stable.
- 1h shows larger month-level variation.
- Run behavior is directionally consistent with the transition-based dependence signal.

I view this as supporting context rather than a headline result; the transition evidence carries more weight.

### 5.6 15m first-order predictor beats naive baselines, but edge is modest

**Figure 2. 15m out-of-sample accuracy: Markov vs baselines**

![Figure 2](/assets/images/2026-03-11-polymarket-random-walk/outcome/figure2_accuracy_vs_baselines.png)

**Table 5. 15m Markov Predictor vs Naive Baselines**

| symbol | accuracy_markov | accuracy_50/50 | accuracy_majority | logloss_markov | logloss_50/50 | logloss_majority |
|---|---:|---:|---:|---:|---:|---:|
| btc | 0.5245 | 0.5000 | 0.4936 | 0.6980 | 0.6931 | 0.6934 |
| eth | 0.5346 | 0.5000 | 0.4975 | 0.7006 | 0.6931 | 0.6932 |
| sol | 0.5238 | 0.5000 | 0.4948 | 0.6965 | 0.6931 | 0.6933 |
| xrp | 0.5332 | 0.5000 | 0.5135 | 0.7000 | 0.6931 | 0.6930 |

Accuracy beats both naive baselines, but the gain is modest. Also, logloss does not beat 50/50, so this is weak classification edge, not strong probability calibration.

### 5.7 Second-order context adds little practical classification gain

**Table 6. First-Order vs Second-Order Comparison at 15m**

| symbol | acc_first | acc_second | delta_acc_second_minus_first | logloss_first | logloss_second | delta_logloss_second_minus_first |
|---|---:|---:|---:|---:|---:|---:|
| btc | 0.5302 | 0.5302 | 0.0000 | 0.6913 | 0.6906 | -0.000716 |
| eth | 0.5354 | 0.5354 | 0.0000 | 0.6906 | 0.6902 | -0.000464 |
| sol | 0.5231 | 0.5231 | 0.0000 | 0.6921 | 0.6916 | -0.000483 |
| xrp | 0.5349 | 0.5349 | 0.0000 | 0.6906 | 0.6906 | -0.000003 |

Second-order context is not "zero information," but in practical classification terms it barely moves the needle.

### 5.8 Rolling stability supports persistence of reversal ordering

**Figure 3. Share of rolling windows with negative transition difference**

![Figure 3](/assets/images/2026-03-11-polymarket-random-walk/outcome/figure3_rolling_stability_share_negative.png)

Shares of rolling windows with negative `P_up_given_up - P_up_given_down`:
- btc: 0.859
- eth: 0.864
- sol: 0.797
- xrp: 0.870

This makes the 15m reversal ordering look persistent rather than a one-off full-sample artifact.

**Figure 3b. Rolling transition paths with upward crossover dates (window=200)**

![Figure 3b](/assets/images/2026-03-11-polymarket-random-walk/outcome/figure8_rolling_transition_upward_crossovers.png)

This chart helps reconcile two facts at once: full-sample dependence is reversal-like, but local windows can still flip temporarily.

I call these flip points **upward crossovers**, defined as dates where:
\[
\text{roll }P(\text{Up}\mid \text{Up}) - \text{roll }P(\text{Up}\mid \text{Down})
\]
crosses from non-positive to positive.

The overlap coverage is not small:
- crossover dates (unique) by symbol: BTC 21, ETH 19, SOL 38, XRP 22
- union of crossover dates across all symbols: 67 days
- dates shared by at least 2 symbols: 30 days, i.e. **44.8%** of union dates
- per-symbol share of crossover dates that overlap with at least one other symbol: BTC 61.9%, ETH 63.2%, SOL 57.9%, XRP 72.7%

So even though the baseline ordering is mostly reversal-like, there are synchronized windows where this ordering weakens or locally inverts across multiple symbols at the same time.

### 5.9 Gap regime splits are interesting under shift spec

**Figure 4. Pooled high-minus-low by window (shift spec)**

![Figure 4](/assets/images/2026-03-11-polymarket-random-walk/outcome/figure4_shift_spec_pooled_high_minus_low.png)

At window 200, all symbols satisfy `acc_high > acc_low`. Pooled high-minus-low declines with larger windows:
- 100: 0.0961
- 200: 0.0681
- 300: 0.0555
- 400: 0.0494

Under this specification, higher-gap windows look more predictable, though the lift decays as smoothing gets heavier.

### 5.10 Shift vs no-shift is the key caution point

**Figure 5. Pooled high-minus-low by window: shift vs no-shift**

![Figure 5](/assets/images/2026-03-11-polymarket-random-walk/outcome/figure5_shift_vs_noshift_pooled_high_minus_low.png)

If you've read this far, you probably saw the trap coming: timing alignment can quietly make a regime story look much stronger than it really is.

I did not catch this immediately in the first pass. My initial `Gap_t` construction (the no-shift version) used rolling transition estimates that include the transition ending at timestamp \(t\). Then I evaluated prediction quality at that same timestamp. That is exactly the kind of subtle timing leak that can inflate regime separation.

After adding the strict `shift` check (features at \(t\) must be available by \(t-1\) only), the gap-regime uplift changed direction.

Under `noshift`, pooled high-minus-low is strongly positive across windows; under `shift`, it turns slightly negative:

- No-shift symbol-level mean high-minus-low: about 0.196 to 0.205
- Shift symbol-level mean high-minus-low: about -0.018 to -0.009
- No-shift strict ordering (`high > medium > low`) holds across all tested windows
- Shift mostly fails this strict ordering criterion

So no-shift remains useful for descriptive pattern reading, but I treat shift as the conservative predictive reference.

### 5.11 Why one timing adjustment can still move the result a lot

**Figure 6. Regime switch matrix and transition-wise delta z**

![Figure 6](/assets/images/2026-03-11-polymarket-random-walk/outcome/figure6_regime_switch_and_delta_z.png)

The shift operation changes only one bar in alignment, but it can still move a material share of regime assignments near thresholds.

In this sample:
- about 5.2% of rows switch regime bucket after applying shift
- most switches are `medium <-> high` or `medium <-> low`, while `low <-> high` is almost absent
- transition-wise `delta_z` is asymmetric: `UU`/`DD` move down, `UD`/`DU` move up

That is enough to reshuffle the composition of high/low buckets, and the pooled accuracy spread can flip even without dramatic visual change in the raw series.

### 5.12 Overlay remains informative, but it is descriptive evidence

**Figure 7. z_gap and rolling prediction accuracy overlay (window=200)**

![Figure 7](/assets/images/2026-03-11-polymarket-random-walk/outcome/figure7_zgap_rolling_accuracy_overlay.png)

Visually, `z_gap_t` and rolling reversal accuracy co-move a lot. This is still useful because it says the dependence-strength proxy is not random noise.

But I would frame this as descriptive coherence, not causal proof by itself. The conservative takeaway remains:
- the outcome process has a weak, persistent 15m dependence layer,
- and any regime-based uplift claim must pass strict timing alignment first.

## 6. What this means for strategy design

The outcome-layer evidence is better treated as a weak structural signal than a standalone production alpha.

Practical use case:
- as a conditioning prior (state filter) in broader models,
- especially when combined with price, volatility, and execution features,
- with strict causal timing constraints.

In short: useful as structure, not sufficient as a one-layer deployable strategy.

## 7. Limitations and next steps

Main limitations:
- outcome-only framing (no direct price microstructure/execution costs yet)
- modest effect sizes
- heavy 1h availability distortion (`NOT_EXISTS`)
- timing sensitivity in regime diagnostics (`shift` vs `noshift`)

Next steps:
- integrate outcome structure with Binance-derived features,
- enforce strict feature timing and walk-forward evaluation,
- add execution-aware feasibility checks (costs, slippage, turnover constraints).

## 8. Conclusion

Unconditionally, the process is close to fair. But it is not fully memoryless.

The cleanest repeatable structure appears at the 15-minute horizon. It is mostly first-order, reversal-like, and modest in size. That makes it a credible structural layer for future models, not a standalone alpha claim.

The most important practical lesson is methodological: regime conclusions can change materially under timing treatment, so causal alignment discipline is non-negotiable.

---

## Data / analysis provenance

Source notebooks:
- `root_docs/Outcome_Analysis.ipynb`
- `root_docs/Outcome_15m_Markov_Analysis.ipynb`

Source docs:
- `root_docs/Outcome_Analysis_Summary.md`
- `root_docs/Outcome_Plan.md`

Source CSV directory:
- `analysis_csv/`

Figure generator:
- `make_outcome_blog_figures.py`
