# VIX regime detection and asset rotation

Fit volatility regimes to the VIX with a hidden Markov model, rotate between equities,
bonds and gold conditional on the inferred state, and test whether the rotation is worth
the complexity.

**It isn't — not against the right benchmark.** The strategy comfortably beats buy-and-hold
SPY. It loses to an equal-weighted portfolio of the same three assets. That result is the
point of the repository.

---

## Method

**Regime model.** Daily VIX levels, 18 Nov 2004 → present, from Yahoo Finance. Two
candidate specifications are fitted and compared on information criteria:

| Model | Log-likelihood | AIC | BIC |
|---|---|---|---|
| 2-state HMM | −9,103.75 | 18,221.50 | 18,267.59 |
| 3-state HMM | **−8,746.45** | **17,520.91** | **17,613.08** |

The three-state model wins on both criteria, and the states are economically legible
rather than statistical artefacts — mean daily ETF returns conditioned on the filtered
state:

| State | TLT | GLD | SPY | SPY vol | Reading |
|---|---|---|---|---|---|
| 0 | −0.0000% | 0.047% | **0.099%** | 0.0054 | Calm, equity-favourable |
| 1 | 0.020% | 0.039% | 0.022% | 0.0123 | Transitional |
| 2 | **0.098%** | 0.061% | **−0.435%** | 0.0357 | Stress — equities sell off, bonds bid |

State 2 carries a −0.435% mean daily SPY return at 3.6% daily volatility. That is a crisis
state, and it is recovered from the VIX alone without ever being told what a crisis is.

**Rotation rule.** Allocations fixed per state:

```
State 0 (calm)        SPY 60%   GLD 40%   TLT  0%
State 1 (transition)  SPY 40%   GLD 60%   TLT  0%
State 2 (stress)      SPY  0%   GLD 40%   TLT 60%
```

**Backtest.** Regime labels are lagged one day before being traded. The HMM filters the
state using information available at time `t`, so acting on that label at `t` would
assume same-close execution on a signal derived from that close. The lag is the difference
between a defensible backtest and a flattering one.

## Results

| | Cumulative | Ann. return | Ann. vol | Sharpe | Max DD |
|---|---|---|---|---|---|
| Regime rotation (lagged) | 822.39% | 11.04% | 0.1221 | 0.9045 | **−25.74%** |
| Equal weight (SPY/GLD/TLT) | 559.60% | 9.30% | 0.0962 | **0.9665** | −23.04% |
| Buy & hold SPY | 769.64% | 10.74% | 0.1898 | 0.5655 | −55.19% |

### Reading this honestly

**Against SPY, the model looks excellent.** Sharpe 0.9045 against 0.5655, and maximum
drawdown cut from −55.19% to −25.74%. The regime machinery is doing real work: it is out
of equities during the stress state, which is where SPY's drawdown is manufactured.

**Against equal weight, it isn't.** A naive one-third split of the same three assets,
rebalanced and requiring no model at all, delivers a *higher* Sharpe (0.9665) and a
*shallower* drawdown (−23.04%). The rotation earns more in absolute terms — 822% against
560% cumulative — but it does so by carrying more risk, not by timing regimes better.

So the honest conclusion is that most of the apparent benefit comes from **holding bonds
and gold at all**, not from knowing *when* to hold them. SPY is simply the wrong
benchmark for a three-asset strategy, and comparing against it is how this kind of model
usually gets published looking better than it is.

The HMM is still worth something — the drawdown profile differs, and State 2 is a genuine
crisis detector — but it has not earned the claim that regime timing adds risk-adjusted
return over naive diversification. On this evidence, it doesn't.

## Sensitivity

The notebook tests robustness along two axes: **number of states** (2 vs 3 — the 2-state
fit collapses the transitional regime and degrades both the economic reading and the
information criteria) and **sample window** (results are sensitive to whether 2008 is in
the estimation sample, which is the usual problem with regime models fitted on data
containing one dominant crisis).

## Limitations

- **No transaction costs.** The rotation trades on every state change; equal weight does
  not. Costs would widen the gap *against* the strategy, not close it.
- **Allocations are chosen, not optimised.** The 60/40/0 style weights per state are
  reasonable but discretionary. They were not fitted, which avoids one kind of
  overfitting and introduces another kind of arbitrariness.
- **One regime variable.** Only the VIX drives state inference. Credit spreads or term
  structure would be the obvious additions.
- **Single historical path.** No bootstrap, no confidence intervals on the Sharpe
  differences. The 0.9045 vs 0.9665 gap is not tested for significance, and with one
  sample path it probably could not be.

## Reproducing

```bash
pip install -r requirements.txt
jupyter notebook notebooks/vix_regime_rotation.ipynb
```

Data is pulled from Yahoo Finance and FRED at runtime. Results will shift as the series
extends.

## Provenance and contribution

Originally a group work project for **MScFE 622 Stochastic Modeling** at WorldQuant
University, February 2026.

The regime modelling, state interpretation, rotation design, lagged backtest and
sensitivity analysis in this notebook are mine. The repository contains no WorldQuant
University assignment text, rubric, or submitted document, and no personal information
belonging to my coursemates.

## Licence

MIT — see [LICENSE](LICENSE).
