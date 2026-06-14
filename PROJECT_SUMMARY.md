# Project Summary — Tail-Aware Residual Alpha Engine

## One-line description

A risk-aware alpha-validation framework that tests whether residual equity signals represent genuine stock-specific predictability, hidden crash-risk premia, inconclusive evidence or backtest luck.

---

## Best positioning

This is not an “I found alpha” project.  
It is an “I built an alpha referee” project.

The project is strongest when described as:

> a disciplined alpha-validation framework that applies market-risk thinking to alpha research.

It is especially useful for Quant Research / Systematic Investing because it connects:

- factor modelling;
- PCA/residualization;
- residual momentum/reversal;
- downside and drawdown risk;
- coskewness/tail diagnostics;
- placebo tests;
- deflated-Sharpe-style luck gates;
- public-data equity validation.

---

## Why it is strong

The project shows research maturity in three ways:

1. **It does not overclaim.**  
   Weak public-data signals are rejected instead of being marketed as alpha.

2. **It separates performance from explanation.**  
   A signal must pass both a luck gate and a hidden-risk gate.

3. **It allows quarantine.**  
   The `H0_inconclusive` state prevents the false binary “alpha vs no alpha.”

---

## Core evidence

### V1.4 synthetic validation

| Validation item | Result |
|---|---:|
| H3 noise certified as H1 | 0 / 400 |
| H2 hidden-crash momentum certified as H1 | 2 / 200 |
| H1 momentum detected as H1 | 166 / 200 |
| H1 reversal detected as H1 | 162 / 200 |
| Pre-registered targets passed | 3 / 4 |

Main interpretation:

> The referee is conservative: it nearly eliminates noise-to-alpha and hidden-crash-to-alpha misclassification, while accepting a measured detection cost.

---

### F0b — S&P500 public-data prototype

| Signal | L2 net Sharpe | Verdict |
|---|---:|---|
| Residual momentum | -0.02 | H3 |
| Residual reversal | -0.58 | H3 |

Interpretation:

> Residual momentum improves relative to raw momentum but remains too weak to certify.

---

### F0c-yf — small/mid-cap public-data robustness

| Signal | L2 net Sharpe | Verdict |
|---|---:|---|
| Residual momentum | -0.59 | H3 |
| Residual reversal | -0.34 | H3 |

Interpretation:

> The small/mid-cap robustness universe does not support residual momentum or reversal.

---

### F0d — declared hidden-risk harvesters

| Signal | L2 net Sharpe | DSR z | Tail score | Verdict |
|---|---:|---:|---:|---|
| Downside-risk harvester | -0.23 | -3.01 | 2 | H3 |
| Low-vol harvester | -0.80 | -5.48 | 2 | H3 |

Interpretation:

> The signals show some tail-risk diagnostics but do not earn enough to force the H2 dilemma.

---

## Relationship to Factor Momentum

Factor Momentum is more investable and empirically concrete.  
Tail-Aware Residual Alpha is more original and more aligned with alpha research judgement.

| Dimension | Factor Momentum | Tail-Aware Residual Alpha |
|---|---|---|
| Tradable strategy | Stronger | Weaker |
| Empirical positive result | Stronger | Weaker |
| Originality | Lower | Higher |
| Risk-to-alpha narrative | Moderate | Strong |
| QR/Systematic research fit | Good | Very strong |
| Current résumé readiness | Good | Good if packaged correctly |

Best combined narrative:

> Factor Momentum shows that I can implement and stress-test an investable systematic timing idea. Tail-Aware shows that I can build a conservative alpha-referee that does not confuse weak performance, hidden risk or luck with genuine alpha.

---

## What to say in interviews

The strongest answer:

> I wanted an alpha project that did not just optimize Sharpe. The framework asks whether a residual signal is genuine stock-specific predictability, hidden crash-risk compensation or backtest luck. I validated the referee on synthetic worlds where the ground truth is known, then applied it to public equity universes. The public runs did not certify alpha, which is exactly the point: the machine is designed to reject weak signals rather than overclaim.

---

## What not to say

Avoid:

- “I found alpha.”
- “Residual momentum works.”
- “This is a production strategy.”
- “The public-data results are institutional-grade.”
- “The V1.4 thresholds are universally valid.”

Say instead:

- “I built an alpha-validation framework.”
- “The public-data tests rejected weak residual signals.”
- “The project is about research discipline and hidden-risk diagnostics.”
- “The next step would be CRSP/PIT data and richer cost/capacity modelling.”

---

## Final status

**Status:** strong alpha-related project, currently best described as an alpha-validation framework rather than an alpha-generating strategy.

**Résumé readiness:** yes, in a project list or selected research section.

**One-page CV priority:** high for Quant Research/Systematic roles if space allows; otherwise use Factor Momentum for investable strategy evidence and Tail-Aware for risk-aware alpha research.
