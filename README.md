# Tail-Aware Residual Alpha Engine

**Residual equity alpha validation with hidden-risk and hidden-luck diagnostics.**

This repository is best read as an **alpha-referee / alpha-validation framework**, not as a production trading strategy.  
The project asks whether residual equity signals represent:

1. **H1 — genuine stock-specific predictability**;
2. **H2 — hidden risk premia**, especially crash/tail/drawdown exposure not captured by linear factor models;
3. **H0 — inconclusive evidence**, where the signal is not clean enough to certify;
4. **H3 — hidden luck / no alpha**, where apparent performance fails the luck gate or behaves like noise.

The core contribution is not “finding alpha at all costs.”  
It is building a framework that can **refuse to certify weak, risky or lucky signals as alpha**.

---

## Research question

> Do residual equity signals reflect genuine stock-specific predictability, or are they compensation for hidden crash/tail risk or artifacts of backtest luck?

The framework evaluates this question through:

- a residualization ladder;
- residual momentum and reversal signals;
- investible long-short portfolio construction;
- hidden-risk diagnostics;
- placebo and deflated-Sharpe-style luck gates;
- four-state verdict classification;
- synthetic worlds with known ground truth;
- public-data equity applications.

---

## Why this project exists

Most alpha projects ask:

> “Can I produce a positive backtest?”

This project asks:

> “Can I tell when a positive backtest is not alpha?”

That distinction is the point.  
A residual signal can look attractive because it forecasts stock-specific returns, but it can also be:

- hidden market beta;
- nonlinear beta;
- downside beta;
- drawdown beta;
- crash-risk premia;
- coskewness exposure;
- volatility/tail exposure;
- a lucky specification selected from many trials.

The project applies a market-risk mindset to alpha research: **try to destroy the signal before calling it alpha**.

---

## Architecture

### 1. Residualization ladder

The same signals are evaluated after progressively removing common-risk structure:

| Level | Description |
|---|---|
| `L0_raw` | Raw returns |
| `L1_market` | Rolling market-beta residuals |
| `L2_FF3` / factor residuals | Rolling academic-factor residuals |
| `L3_PCA` | Rolling PCA statistical residuals |

The broader research specification allows extensions to IPCA and autoencoders, but the current public-data prototype focuses on the core L0–L3 ladder.

---

### 2. Core signals

The core V1 signal set is deliberately small:

| Signal | Description |
|---|---|
| `S1_resmom` | Residual momentum |
| `S2_resrev` | Short-term residual reversal |

A small signal set is intentional: the project prioritizes adversarial validation over broad specification search.

---

### 3. Hidden-risk diagnostics

For each candidate long-short portfolio, the referee evaluates:

- market beta;
- downside beta gap;
- drawdown / CDaR-style beta;
- skewness;
- coskewness;
- tail score;
- hard tail flags;
- max drawdown;
- realized tail behavior of the strategy P&L.

The goal is to detect whether a signal that appears profitable is actually monetizing hidden crash/tail exposure.

---

### 4. Hidden-luck diagnostics

The luck gate combines:

- deflated-Sharpe-style z-score approximation;
- signal placebo tests;
- placebo percentile;
- pre-registered verdict rules.

The framework is designed so that a signal must pass both the performance/luck gate and the tail-risk gate before it can be certified as H1.

---

## Four-state verdict rule

| Verdict | Meaning |
|---|---|
| `H1_genuine_alpha` | Passes luck gate and remains sufficiently tail-clean |
| `H2_hidden_risk` | Passes luck gate but exhibits material hidden/tail-risk exposure |
| `H0_inconclusive` | Some evidence, but not clean enough to certify |
| `H3_hidden_luck_or_no_alpha` | Fails the luck gate or has no robust evidence of alpha |

The addition of `H0_inconclusive` is important: the framework is allowed to say **“not enough evidence”** instead of forcing every result into alpha or non-alpha.

---

## Synthetic validation: V1.4 referee

Before applying the framework to real/public data, the four-state referee is tested on synthetic worlds with known ground truth:

| World | Ground truth |
|---|---|
| `H1_idio_alpha` | Designed idiosyncratic alpha |
| `H2_hidden_crash` | Designed crash-risk premium masquerading as alpha |
| `H3_pure_noise` | No signal |

### Held-out Monte Carlo results

V1.4 uses a conservative tail score and validates the frozen verdict rule on held-out synthetic Monte Carlo worlds.

Key validation outcomes:

| Metric | Result |
|---|---:|
| H3 noise certified as H1 | `0 / 400 = 0.0%` |
| H2 hidden-crash momentum certified as H1 | `2 / 200 = 1.0%` |
| H1 momentum detected as H1 | `166 / 200 = 83.0%` |
| H1 reversal detected as H1 | `162 / 200 = 81.0%` |
| Pre-registered validation targets passed | `3 / 4` |

The main improvement from V1.3 to V1.4 was reducing dangerous hidden-crash-to-alpha certifications:

| Metric | V1.3 | V1.4 |
|---|---:|---:|
| H2 momentum → H1 | `10 / 200 = 5.0%` | `2 / 200 = 1.0%` |
| H3 noise → H1 | `1 / 400 = 0.2%` | `0 / 400 = 0.0%` |

Interpretation:

> V1.4 is intentionally conservative. It sharply reduces dangerous false alpha certifications at the cost of quarantining or rejecting some true-alpha cases.

---

## Public-data applications

The frozen V1.4 referee is then applied to public-data equity universes. These are not CRSP-grade tests; they are **public-data prototypes** using yfinance, current constituents, and simplified costs.

### F0b — S&P 500 current-universe public-data run

- Data source: yfinance.
- Universe: current S&P 500 names, 498 usable stocks.
- Period: 2007–2024.
- Verdict level: `L2_FF3`.

Headline result:

| Signal | L2 net Sharpe | Verdict |
|---|---:|---|
| Residual momentum | `-0.02` | `H3_hidden_luck_or_no_alpha` |
| Residual reversal | `-0.58` | `H3_hidden_luck_or_no_alpha` |

Interpretation:

> Residual momentum improved relative to raw momentum, but not enough to pass the frozen luck/tail referee. The framework rejects the signal rather than overclaiming weak alpha.

---

### F0c-yf — Small/mid-cap public-data robustness run

- Data source: yfinance.
- Attempted S&P600 fetch failed; a hardcoded small/mid-cap fallback universe was used.
- Universe: 202 usable stocks.
- Period: 2007–2024.
- Verdict level: `L2_FF3`.

Headline result:

| Signal | L2 net Sharpe | Verdict |
|---|---:|---|
| Residual momentum | `-0.59` | `H3_hidden_luck_or_no_alpha` |
| Residual reversal | `-0.34` | `H3_hidden_luck_or_no_alpha` |

Interpretation:

> On this public-data small/mid-cap robustness universe, residual momentum does not improve over raw momentum and is rejected.

---

### F0d — Pre-declared hidden-risk harvesters

F0d tests whether declared hidden-risk harvesters would be classified as H2 if they earned enough to pass the luck gate.

Signals:

| Signal | Intended test |
|---|---|
| `S1_downside_risk` | Co-crash/downside-risk harvester |
| `S2_lowvol` | Low-vol / junk-rally-style tilt |

Results:

| Signal | L2 net Sharpe | DSR z | Tail score | Verdict |
|---|---:|---:|---:|---|
| `S1_downside_risk` | `-0.23` | `-3.01` | `2` | `H3_hidden_luck_or_no_alpha` |
| `S2_lowvol` | `-0.80` | `-5.48` | `2` | `H3_hidden_luck_or_no_alpha` |

Interpretation:

> These declared hidden-risk harvesters exhibit some tail-risk diagnostics, but they do not earn enough to pass the luck gate. The frozen referee therefore rejects them as H3 rather than over-interpreting tail exposure as a profitable hidden-risk premium.

---

## Final interpretation

The strongest current claim is:

> A conservative residual-alpha referee was validated on controlled synthetic worlds and then applied to public equity data, where weak residual momentum/reversal and declared hidden-risk harvesters were rejected rather than misclassified as alpha.

This is a **successful alpha-validation project**, not an empirical alpha-discovery claim.

The framework demonstrates that:

- noise is not certified as alpha;
- hidden crash-risk premia are rarely misclassified as genuine alpha;
- public-data residual momentum/reversal signals are not strong enough to certify;
- the system can say H0/H3 instead of forcing a positive result;
- negative results are part of the deliverable.

---

## Relationship to the Factor Momentum project

The ETF Factor Momentum project is a tradable factor-timing case study: it finds a modest defensive timing effect, but the result is substantially explained by static factor/market exposures and does not survive full data-snooping controls.

This project is different:

| Project | Main contribution |
|---|---|
| ETF Factor Momentum | Implements and stress-tests an investable factor-timing idea |
| Tail-Aware Residual Alpha Engine | Builds a general alpha-referee to distinguish alpha from hidden risk and hidden luck |

Factor Momentum is more directly investable.  
Tail-Aware Residual Alpha is more original and more directly aligned with risk-aware alpha research.

---

## What this project does **not** claim

This project does **not** claim that:

- residual momentum is a production-ready alpha on public data;
- public-data S&P500 or small/mid-cap runs are CRSP-grade evidence;
- yfinance/current-constituent results are free from survivorship bias;
- the synthetic thresholds automatically transfer perfectly to real markets;
- the framework has exhausted all possible residual-alpha signals.

The public-data results are deliberately described as prototypes.

---

## Limitations

- Current-constituent universes introduce survivorship bias.
- No delisting returns.
- yfinance data are not institutional-grade.
- Transaction costs are simplified.
- Residualization is beta-only in the public-data prototype.
- V1.4 thresholds are calibrated on synthetic worlds; transfer to real data is a model assumption.
- No IPCA/autoencoder extension in the current public prototype.
- No claim of production readiness.

---

## Repository structure recommendation

```text
.
├── notebooks/
│   ├── 01_v1_4_synthetic_referee_validation.ipynb
│   ├── 02_f0b_sp500_public_data_application.ipynb
│   ├── 03_f0c_small_mid_public_data_robustness.ipynb
│   └── 04_f0d_declared_hidden_risk_harvesters.ipynb
├── README.md
├── PROJECT_SUMMARY.md
├── RESUME_BULLETS.md
├── requirements.txt
└── outputs/
```

---

## Suggested short résumé bullet

> Built a tail-aware residual-alpha validation framework distinguishing genuine stock-specific predictability from hidden crash-risk premia, inconclusive evidence and backtest luck; validated the four-state referee on held-out synthetic Monte Carlo worlds and applied it to public S&P500/small-cap equity universes plus pre-declared hidden-risk harvesters, rejecting weak or risky signals rather than misclassifying them as alpha.

---

## Not investment advice

This repository is a research project and does not constitute investment advice or a production trading system.
