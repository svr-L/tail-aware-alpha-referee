# Project Summary — Tail-Aware Residual Alpha Engine

## One line

A risk-aware **alpha-validation framework** that decides whether a residual equity signal is genuine stock-specific predictability (`H1`), a hidden crash-risk premium dressed as alpha (`H2`), inconclusive (`H0`), or backtest luck (`H3`).

## Positioning

This is not an *"I found alpha"* project. It is an *"I built an alpha referee — and showed it works"* project.

Strongest one-liner:

> A disciplined alpha-validation framework that applies market-risk thinking to alpha research — validated out-of-sample on synthetic worlds with known ground truth, then stress-tested on real public equity data, where it behaved exactly as a conservative referee should.

It is well-suited to Quant Research / Systematic Investing because it connects, in one coherent pipeline: factor modelling, PCA/residualization, residual momentum/reversal, downside & drawdown risk, coskewness/tail diagnostics, placebo tests, a deflated-Sharpe luck gate, pre-registration, and public-data equity validation.

---

## Why it is strong (the research-maturity case)

1. **It does not overclaim.** Weak public-data signals are *rejected*, not marketed as alpha.
2. **It separates performance from explanation.** A signal must pass both a luck gate *and* a hidden-risk gate.
3. **It allows quarantine.** The `H0_inconclusive` state avoids the false "alpha vs no alpha" binary.
4. **It follows a real validation protocol.** Two design→freeze→held-out cycles; thresholds frozen with **pre-registered** targets before the held-out run; an honest **3/4** outcome (one target missed and reported).
5. **It tested a hypothesis that failed.** A mechanism-targeted fifth tail condition was tried, failed on the design set, and was discarded in favor of a graded score that empirically dominated a blunt rule — a documented "I tried the obvious thing, it didn't work, here's why" arc.
6. **It is reproducible.** Synthetic runs reproduce bit-for-bit across machines; every notebook ships executed.

---

## Core evidence

### Synthetic referee — V1.4 held-out (200 seeds/world, virgin seeds)

| Validation item | Result |
|---|---:|
| H3 noise certified as `H1` | 0 / 400 = 0.0% |
| H2 hidden-crash momentum certified as `H1` (dangerous) | 2 / 200 = 1.0% |
| H1 momentum detected as `H1` | 166 / 200 = 83.0% |
| H1 reversal detected as `H1` | 162 / 200 = 81.0% |
| Pre-registered targets met | 3 / 4 |
| Dangerous-rate improvement V1.3 → V1.4 (paired, same worlds) | 5.0% → 1.0% (McNemar *p* ≈ 3.9e-3) |

> The referee is intentionally conservative: it nearly eliminates noise-to-alpha and hidden-crash-to-alpha misclassification, at a measured ~3–6 pp cost in true-alpha detection.

### Real public-data applications (frozen referee, yfinance + Ken French, 2007–2024)

| Phase | Universe | `L2` residual-momentum net Sharpe | Verdict |
|---|---|---:|---|
| F0 | S&P 100 | +0.10 | `H3` |
| F0b | S&P 500 | −0.02 | `H3` |
| F0c | small/mid-cap (~202) | −0.59 | `H3` |

Plus **F0d**, a pre-registered hidden-risk test: declared downside-risk and low-vol harvesters, both `H3` on real data (no premium survives costs), **but** (i) the offline demo confirms the `H2` path engages when a premium *does* exist, and (ii) the real `S1` vs `S2` contrast measures the tail battery's archetype-specificity (co-crash vs junk-rally).

> Three real findings: a monotone conservative-strong null across the size spectrum; the double luck gate beating a naive placebo screen (a flat strategy fooled the placebo but not the deflated-Sharpe hurdle); and the tail battery correctly measuring real crash/tail risk while withholding the `H2` label absent a surviving premium.

---

## What to say in interviews

> I wanted an alpha project that doesn't just optimize Sharpe. The framework asks whether a residual signal is genuine stock-specific predictability, hidden crash-risk compensation, or backtest luck — a four-state verdict with a quarantine state. I validated the referee on synthetic worlds with known ground truth, froze the rule with pre-registered targets, and confirmed it held out-of-sample. Then I applied the frozen referee to public equity data. It didn't certify alpha — which is the point — but it did three things I can defend: it beat a naive placebo screen (a near-flat strategy that fooled the placebo failed my deflated-Sharpe hurdle), it measured real tail risk when a strategy carried it, and it exposed its own limit — a tail battery tuned to market co-crashes generalizes only partly to a junk-rally archetype.

**Have ready:** why the double gate matters (cost-shifted placebo trap); why beta-only residuals (the intercept artifact); why `H0` exists (asymmetric loss of certifying hidden risk); the failed fifth-condition story; the pre-registered miss (83% vs 85%) as evidence of honesty.

---

## What *not* to say

| Avoid | Say instead |
|---|---|
| "I found alpha." | "I built an alpha-validation framework." |
| "Residual momentum works." | "The public-data tests rejected weak residual signals — by design." |
| "This is a production strategy." | "These are public-data prototypes; the contribution is research discipline." |
| "The public results are institutional-grade." | "Next step is CRSP/PIT data and richer cost/capacity modelling." |
| "The V1.4 thresholds are universally valid." | "Thresholds are synthetic-calibrated; transfer to real data is an assumption I tested partially in F0d." |

---

## Final status

**Status:** a complete, defensible alpha-validation framework — synthetic out-of-sample validation plus real-data stress tests — best described as an *alpha referee*, not an alpha-generating strategy. The real-data nulls are confirmations the referee is calibrated correctly, not shortcomings.

**Résumé readiness:** yes — in a project list or a "selected research" section. High priority for Quant Research / Systematic roles when space allows.
