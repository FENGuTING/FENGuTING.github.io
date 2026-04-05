---
title: "Price Structure in Binance Crypto Markets: From Outcome Validation to Event-Regime Alpha"
date: 2026-04-05 12:00:00 +0800
categories: [Weekly Research, Polymarket]
tags: [polymarket, random-walk, volatility, event-study]
math: true
toc: true
---

The outcome-layer post left one unresolved issue: did the 15-minute pattern come from the underlying market, or from the way Polymarket contracts resolve? This follow-up moves one layer down to Binance spot prices and asks a narrower question than "can prices be predicted?" It asks where the structure actually sits, and which parts of it survive contact with costs.

## 1. Motivation

The outcome post was useful, but it still treated the contract outcome as the object of interest. That is not the same as studying the underlying market.

So I moved to Binance spot prices for two reasons:

- first, to check whether Polymarket `15m` outcomes are close enough to Binance `15m` return signs to treat Binance as a valid external benchmark;
- second, to separate three different claims that often get blurred together: raw direction is predictable, volatility is persistent, and conditional event slices may still contain tradable structure even if the unconditional process does not.

The argument of this post is narrower than "crypto prices are predictable." It is:

- the benchmark is valid,
- raw returns are close to weak-form efficiency,
- volatility is materially more persistent than direction,
- and the few patterns worth keeping are event- and regime-conditioned rather than always-on.

## 2. Data and scope

- Assets: `BTCUSDT`, `ETHUSDT`, `SOLUSDT`, `XRPUSDT`
- Raw source: Binance spot `1m` OHLCV
- Sample window: `2025-03-11 00:00:00 UTC` to `2026-03-10 23:59:00 UTC`
- Unified rows per asset: `525,600`
- Aggregated horizons used downstream: `1m`, `5m`, `15m`, `1h`
- Outcome alignment layer: Polymarket `15m` resolved outcomes

**Table 1. Raw price-data coverage**

| symbol | rows | dup_removed | start_utc | end_utc |
|---|---:|---:|---|---|
| btc | 525600 | 0 | 2025-03-11 00:00:00 | 2026-03-10 23:59:00 |
| eth | 525600 | 0 | 2025-03-11 00:00:00 | 2026-03-10 23:59:00 |
| sol | 525600 | 0 | 2025-03-11 00:00:00 | 2026-03-10 23:59:00 |
| xrp | 525600 | 0 | 2025-03-11 00:00:00 | 2026-03-10 23:59:00 |

All timestamps below are in UTC.

## 3. Research Questions and Working Hypotheses

This post answers six linked questions:

1. Are Polymarket `15m` outcomes close enough to Binance `15m` return signs to justify using Binance as the external benchmark?
2. Are raw short-horizon returns close to a random walk, or do they contain meaningful unconditional directional structure?
3. If unconditional direction is weak, is volatility persistence still strong?
4. Do large up and down moves leave different forward-return footprints?
5. Does BTC lead other assets in a way that is strong enough to matter in practice?
6. After conditioning on events and regimes, which patterns remain economically interesting after a simple cost screen?

I treat those as the following working hypotheses.

**Table 2. Working hypotheses**

| ID | Working hypothesis | What would support it |
|---|---|---|
| H1 | Polymarket `15m` outcomes are a good proxy for Binance `15m` sign returns | high pointwise match rate and similar transition ordering |
| H2 | Raw short-horizon returns are close to weak-form efficiency | small return ACF and no large, stable directional dependence |
| H3 | Any departure from a random walk is more likely mild mean reversion than strong continuation | variance ratios modestly below 1 |
| H4 | Volatility is more persistent than direction | `abs_return` and `sq_return` ACF far larger than raw-return ACF |
| H5 | Large moves create conditional asymmetry | forward returns differ after up-shocks and down-shocks |
| H6 | BTC lead-lag is weak unless tightly conditioned on state | low explanatory power and unstable coefficients across regimes |
| H7 | Only a small subset of event/regime patterns survives costs | a few `15m` slices pass the cost filter; most do not |

## 4. Research Design

The notebook order is part of the research logic, not just presentation.

**Table 3. Research workflow**

| Phase | Notebook | Role in the argument | Why it comes before the next step |
|---|---|---|---|
| I | `price_outcome_analysis.ipynb` | validate the external benchmark | without this, the earlier outcome results are hard to interpret |
| II | `price_deep_research.ipynb` | establish the unconditional baseline for price behavior | this prevents local event results from being mistaken for general predictability |
| III | `price_phase_3_event_regime_research.ipynb` | search for conditional, cost-aware pockets | this is only credible after the unconditional baseline is already known |

That sequencing matters. If I had started directly from phase III, it would have been easy to find a few local effects and overstate them. Starting with benchmark validity and unconditional diagnostics makes the later conditional results harder to overclaim.

The main empirical tests are:

- alignment tests at `15m`: sign match rate, rolling match stability, and first-order transition comparison;
- weak-form diagnostics: short-horizon ACF and variance-ratio tests;
- volatility diagnostics: ACF of `abs_return` and `sq_return`;
- event studies: forward returns after `2σ` and `3σ` moves;
- cross-asset transfer tests: lagged BTC-to-ETH relationships by regime;
- cost-aware filtering: retain only event/regime patterns that still matter after simple trading-cost assumptions.

## 5. Results

### 5.1 RQ1 / H1: Is Binance a valid external benchmark for the outcome layer?

**Question.**
Do resolved Polymarket outcomes track the sign of underlying Binance `15m` returns closely enough to justify using Binance as the benchmark?

**Test.**
I align Binance `15m` sign returns with resolved Polymarket outcomes, then check pointwise match rates, rolling stability, and first-order transition probabilities.

**Figure 1. Match rate by symbol**

![Figure 1](/assets/images/2026-03-11-polymarket-random-walk/price_01/figure1_match_rate_by_symbol.png)

**Table 4. Price-outcome match rate at 15m**

| symbol | match_rate | N |
|---|---:|---:|
| btc | 0.9826 | 8631 |
| eth | 0.9857 | 8631 |
| sol | 0.9783 | 8631 |
| xrp | 0.9838 | 8630 |

Binance `15m` sign returns and resolved Polymarket outcomes agree about 98% of the time across all four assets. That is high enough to treat the outcome layer as a close proxy for the underlying price-sign process for the rest of this study.

**Answer to RQ1.**
Yes. H1 is supported.

What matters here is not only the pointwise agreement, but also whether the dependence structure lines up.

### 5.2 RQ1 continued: Does the transition structure also line up?

**Question.**
If the outcome layer is a good proxy point by point, does it also preserve the same first-order dependence pattern?

**Test.**
I compare Binance and outcome transition matrices at `15m`, focusing on whether the ordering of `P(Up|Up)` and `P(Up|Down)` matches across assets.

**Figure 2. Binance vs outcome transition matrices**

![Figure 2](/assets/images/2026-03-11-polymarket-random-walk/price_01/figure2_transition_matrices_binance_vs_outcome.png)

**Table 5. 15m transition comparison: Binance vs outcome**

| symbol | P(Up\|Up) Binance | P(Up\|Down) Binance | P(Up\|Up) Outcome | P(Up\|Down) Outcome |
|---|---:|---:|---:|---:|
| btc | 0.4767 | 0.5204 | 0.4675 | 0.5280 |
| eth | 0.4742 | 0.5378 | 0.4633 | 0.5342 |
| sol | 0.4804 | 0.5280 | 0.4743 | 0.5206 |
| xrp | 0.4704 | 0.5212 | 0.4587 | 0.5285 |

Not only do pointwise signs align, the first-order transition logic aligns as well. Both layers show the same broad reversal-like ordering:

$$
P(\text{Up} \mid \text{Down}) > P(\text{Up} \mid \text{Up})
$$

That is the same qualitative pattern that appeared in the outcome post. The price layer therefore validates the earlier structural story rather than replacing it with a different one.

**Answer to RQ1, part 2.**
Yes. The agreement is structural, not only pointwise.

### 5.3 RQ2 / H2-H3: Are raw returns predictably directional at short horizons?

**Question.**
Once I move to the price layer directly, do raw returns show enough unconditional dependence to support a broad directional model?

**Test.**
I use two weak-form diagnostics:

- short-horizon return ACF, summarized by mean absolute ACF across lags `1` to `20`;
- variance-ratio tests across `1m -> 5m`, `1m -> 15m`, and `1m -> 1h`.

**Figure 3. 15m short-return ACF across assets**

![Figure 3](/assets/images/2026-03-11-polymarket-random-walk/price_01/figure3_acf_short_return_15m.png)

**Table 6. Short-horizon return ACF summary**

| symbol | 1m mean abs ACF | 5m mean abs ACF | 15m mean abs ACF |
|---|---:|---:|---:|
| btc | 0.0034 | 0.0073 | 0.0083 |
| eth | 0.0038 | 0.0056 | 0.0062 |
| sol | 0.0040 | 0.0060 | 0.0092 |
| xrp | 0.0125 | 0.0146 | 0.0140 |

These are small values. XRP is the least close to zero, but even there the effect is not large enough to justify an always-on sign predictor.

**Table 7. Variance-ratio diagnostics**

| symbol | VR 1m to 5m | VR 1m to 15m | VR 1m to 1h |
|---|---:|---:|---:|
| btc | 0.9849 | 0.9680 | 0.9399 |
| eth | 0.9633 | 0.9483 | 0.9381 |
| sol | 0.9849 | 0.9667 | 0.9323 |
| xrp | 0.9701 | 0.9436 | 0.8078 |

All ratios are below 1. That points to mild mean reversion rather than persistent continuation, but the departure from 1 is not large enough to support a strong unconditional alpha claim.

**Answer to RQ2.**
Raw returns are not perfectly memoryless, but they are close enough to weak-form efficiency that unconditional directional alpha looks weak. H2 and H3 are mostly supported.

### 5.4 RQ3 / H4: Is volatility persistence stronger than directional dependence?

**Question.**
If raw sign predictability is weak, does more persistent structure survive in volatility instead?

**Test.**
I repeat the ACF exercise on `abs_return` and `sq_return`, using the same assets and horizons.

**Figure 4. 15m absolute-return ACF across assets**

![Figure 4](/assets/images/2026-03-11-polymarket-random-walk/price_01/figure4_acf_abs_return_15m.png)

**Table 8. Volatility-clustering summary**

| horizon | feature | btc | eth | sol | xrp |
|---|---|---:|---:|---:|---:|
| 1m | abs_return | 0.2882 | 0.2310 | 0.2257 | 0.2758 |
| 5m | abs_return | 0.2361 | 0.1956 | 0.1839 | 0.2124 |
| 15m | abs_return | 0.1968 | 0.1537 | 0.1415 | 0.1815 |
| 1m | sq_return | 0.0903 | 0.0790 | 0.1321 | 0.0860 |
| 5m | sq_return | 0.0529 | 0.0999 | 0.0776 | 0.0513 |
| 15m | sq_return | 0.1070 | 0.0728 | 0.0562 | 0.0600 |

Compared with the near-zero raw-return ACFs, these persistence levels are materially larger and broadly stable across assets.

**Answer to RQ3.**
Yes. H4 is strongly supported. The most stable regularity in this dataset is not direction; it is volatility persistence.

That distinction matters for strategy design. A weak directional process can still produce useful state information if volatility clusters hard enough.

### 5.5 RQ4 / H5: Do large moves create conditional asymmetry?

**Question.**
Even if the unconditional process is close to weak-form efficiency, do large shocks produce asymmetric forward returns?

**Test.**
I label large moves using `2σ` and `3σ` thresholds, split them into up-jumps and down-jumps, and then measure forward `1`, `3`, and `5` bar returns.

**Figure 5. 15m event study after `2σ` jumps**

![Figure 5](/assets/images/2026-03-11-polymarket-random-walk/price_01/figure5_event_study_15m_z2.png)

**Table 9. 15m jump aftermath examples**

| symbol | threshold | event | fwd bars | mean_fwd_return |
|---|---:|---|---:|---:|
| eth | 3 | down_jump | 5 | 0.001280 |
| sol | 3 | down_jump | 5 | 0.001405 |
| xrp | 3 | down_jump | 5 | 0.005385 |
| xrp | 3 | up_jump | 5 | -0.001066 |
| sol | 2 | up_jump | 5 | -0.000592 |
| btc | 2 | down_jump | 5 | 0.000698 |

The figure uses the simpler `2σ` event-study view because it is easier to read at a glance. The table keeps the broader set of `2σ` and `3σ` examples. Both point in the same direction.

At `15m`, large down moves are often followed by positive forward returns, while large up moves are more likely to be followed by flat or negative forward returns. The effect is not uniform across every asset and threshold, but the asymmetry is strong enough to reject a purely symmetric aftermath story.

**Answer to RQ4.**
Yes. H5 is supported, especially on the downside. The important point is that the asymmetry appears after conditioning on large moves, not in the unconditional process.

### 5.6 RQ5 / H6: Does BTC lead ETH in a practically meaningful way?

**Question.**
Can BTC be treated as a leading information asset for ETH once the data are split by regime?

**Test.**
I run lagged BTC-to-ETH correlations and simple predictive regressions at `15m`, split between `high_vol` and `other` regimes.

**Table 10. BTC-ETH lead-lag at 15m by regime**

| regime | lag | corr | beta | tstat | r2 |
|---|---:|---:|---:|---:|---:|
| high_vol | 1 | 0.0063 | 0.0091 | 0.53 | 0.00004 |
| high_vol | 2 | -0.0073 | -0.0106 | -0.61 | 0.00005 |
| high_vol | 3 | -0.0058 | -0.0084 | -0.48 | 0.00003 |
| other | 1 | -0.0195 | -0.0339 | -3.26 | 0.00038 |
| other | 2 | 0.0059 | 0.0103 | 0.99 | 0.00004 |
| other | 3 | -0.0057 | -0.0100 | -0.96 | 0.00003 |

There is one statistically notable coefficient in the `other` regime at lag `1`, but the effect size is still small and the `r^2` is essentially zero. In `high_vol`, the relationship is negligible.

**Answer to RQ5.**
Not in a way that supports an always-on cross-asset signal. H6 is supported. BTC is more useful as a market-state variable than as a strong directional leader.

### 5.7 RQ6 / H7: Which conditional patterns survive a simple cost screen?

**Question.**
Once the unconditional baseline is established as weak, which event/regime patterns still look meaningful after accounting for simple trading costs?

**Test.**
The phase-III notebook defines past-based event and regime labels, estimates post-event effects, and keeps only patterns that remain interesting after `2`, `5`, or `10` bps cost assumptions.

**Figure 6. `down_2sigma` aftermath conditioned on volatility regime at 15m**

![Figure 6](/assets/images/2026-03-11-polymarket-random-walk/price_01/figure6_regime_conditioned_down_2sigma_15m.png)

**Table 11. Economically interesting event/regime patterns**

| pattern | asset | horizon | regime | fwd | raw effect (bps) | cost tolerance (bps) |
|---|---|---|---|---:|---:|---:|
| `down_2sigma` aftermath | xrp | 15m | high_vol | 5 | 18.06 | 10 |
| `up_3sigma` aftermath | eth | 15m | unconditional | 3 | 12.27 | 10 |
| `vol_burst_12_48` aftermath | xrp | 15m | unconditional | 5 | -12.06 | 10 |
| `local_shock_99` aftermath | xrp | 15m | unconditional | 5 | 9.10 | 5 |
| `down_2sigma` aftermath | sol | 15m | unconditional | 5 | -7.91 | 5 |
| `vol_burst_12_48` aftermath | xrp | 15m | unconditional | 3 | -6.81 | 5 |

I chose the `down_2sigma` regime plot here because it shows the phase-III logic clearly: once the same event is split by volatility state, the forward-return profile can change materially by asset.

The surviving candidates are concentrated around `15m`, and several of the strongest ones sit in XRP rather than appearing evenly across all four assets. That is exactly what a conditional story should look like: narrow, state-dependent, and not broadly portable.

**Answer to RQ6.**
Only a small subset survives. H7 is supported.

### 5.8 How robust are those conditional patterns?

The answer is mixed, and that is an important part of the story rather than an inconvenience to hide.

**Table 12. Robustness summary**

| diagnostic | result | takeaway |
|---|---|---|
| `down_2sigma` cross-asset sign consistency | `2` positive, `2` negative | mixed; not a universal pattern |
| `up_2sigma` cross-asset sign consistency | `0` positive, `4` negative | more directionally consistent, but not always economically large |
| `local_shock_99` cross-asset sign consistency | `1` positive, `3` negative | partial consistency only |
| first-half vs second-half splits | several effects weaken or change size | subsample stability is incomplete |
| threshold sensitivity | effect size moves with the `z` cutoff | some patterns are fragile to event definition |

This is why the sequence of the post matters. Once the unconditional baseline has already been shown to be weak, these conditional patterns can be read for what they are: plausible local opportunities, not general proofs of broad predictability.

## 6. What This Implies for Strategy Design

The strategy ranking is fairly clear after the full sequence of tests.

1. Do not start from an unconditional sign model. The data do not support that as the first prototype.
2. Put volatility and regime filters near the front of the stack. That is where the most stable structure lives.
3. Test event-triggered `15m` strategies before anything slower or more elaborate. That is where the cost-aware candidates are concentrated.
4. Treat BTC more as a state signal and risk flag than as a strong transfer signal.

That is a different conclusion from "fit a classifier on all bars and hope the average effect survives."

## 7. Limitations and Next Steps

The main limitations are straightforward:

- several supporting phase-II and phase-III outputs still live as tables rather than polished figures;
- several conditional effects are narrow and do not generalize cleanly across assets;
- robustness is mixed across subsamples and threshold definitions;
- and the economic filter is still a coarse screen rather than a full execution-aware backtest.

The next steps follow directly from those limits:

- turn the strongest phase-III tables into publication-quality figures;
- run strict chronological walk-forward tests on the top `15m` candidates;
- add turnover, slippage, and fill-risk assumptions;
- and combine price-state features with the earlier outcome-state results instead of treating the two layers separately.

## 8. Conclusion

This post does not argue that crypto prices are broadly predictable. It makes a narrower claim, but a more useful one.

Binance validates the outcome layer as a benchmark. Raw returns still look close to weak-form efficiency. The strongest stable regularity is volatility clustering, not unconditional sign predictability. The only places where directional edge starts to look interesting are the event- and regime-conditioned subsets, especially around `15m`, and even there the evidence is selective rather than universal.

That is the real value of the notebook sequence. It turns a loose collection of tests into a disciplined argument:

1. verify the benchmark;
2. establish the unconditional baseline;
3. only then keep the conditional pockets that still survive costs.

---

## Data / analysis provenance

Research sequence:
- `Price_Data/Price_Plan.md`
- `price_outcome_analysis.ipynb`
- `Price_Data/price_deepresearch_plan.md`
- `price_deep_research.ipynb`
- `price_phase_3_event_regime_research.ipynb`

Primary data directory:
- `Price_Data/`

Key output folders:
- `Price_Data/analysis_outputs/`
- `Price_Data/deep_research_outputs/`
- `Price_Data/phase_3_event_regime_outputs/`
