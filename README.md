# Tail-Aware Residual Alpha Engine

**A risk-aware *alpha referee*: a framework that decides whether a residual equity signal is genuine stock-specific predictability, a hidden crash-risk premium dressed as alpha, inconclusive, or backtest luck — and that is built to *refuse to certify* weak, risky, or lucky signals.**

This repository is best read as an **alpha-validation framework**, not a production trading strategy. Its contribution is not "finding alpha at all costs." It is a referee that, across two synthetic design→validation cycles and four real-data applications, behaved exactly as a disciplined referee should: it never certified noise or hidden crash-risk as alpha, it beat a naive placebo screen on real data, it measured real tail risk when present, and it reported its own limits.

---

## The question

> Do residual equity signals reflect genuine stock-specific predictability, or are they compensation for hidden crash/tail risk, or artifacts of backtest luck?

A positive backtest is not enough. A residual signal can look attractive while actually monetizing hidden market beta, *non-linear/downside* beta, drawdown beta, coskewness/tail exposure — or simply being a lucky specification picked from many trials. This project applies a **market-risk mindset to alpha research: try to destroy the signal before calling it alpha.**

---

## Four-state verdict

| Verdict | Meaning |
|---|---|
| `H1_genuine_alpha` | Passes the luck gate **and** is sufficiently tail-clean |
| `H2_hidden_risk` | Passes the luck gate **but** carries material hidden/tail-risk exposure |
| `H0_inconclusive` | Some evidence, not clean enough to certify — **quarantine** |
| `H3_hidden_luck_or_no_alpha` | Fails the luck gate, or behaves like noise |

The `H0_inconclusive` quarantine is deliberate: the referee is allowed to say *"not enough evidence"* instead of forcing every result into a false alpha/no-alpha binary. A signal must clear **both** a luck gate and a hidden-risk gate before it can be `H1`.

---

## Architecture

**Residualization ladder** — the same signals are re-evaluated after progressively removing common-risk structure: `L0_raw` → `L1_market` (rolling market-beta residuals) → `L2_FF3` (rolling Fama-French-3 residuals, the verdict level) → `L3_PCA` (rolling statistical residuals). Residuals are **beta-only** (factor slopes removed, intercept kept) to strip exposure without demeaning — see *Design notes*.

**Signals** — deliberately small: residual momentum (`S1`, Blitz-Huij-Martens information-ratio style) and short-term residual reversal (`S2`). The project prioritizes adversarial validation over specification search.

**Hidden-risk gate** — for each long-short P&L: market beta, downside-beta gap (worst market decile), CDaR-style drawdown beta, skewness, Harvey-Siddique coskewness, combined into a **graded tail score** (each metric 0/1/2 against soft/hard thresholds).

**Hidden-luck gate (double)** — a deflated-Sharpe-style hurdle (`√(2·ln N)` multiple-testing adjustment) **and** a placebo distribution built by permuting the signal cross-section through the *entire* pipeline (same selection, holding, costs). Both must clear.

---

## Synthetic validation — building and stress-testing the referee (V1.1 → V1.4)

The referee was developed against three synthetic worlds with **known ground truth**: `H1_idio_alpha` (designed idiosyncratic alpha), `H2_hidden_crash` (a crash-risk premium engineered to look like alpha — constant carry per unit crash-beta, repaid violently in market co-crashes, near-zero variance otherwise so it is *invisible to linear/PCA residualization*), and `H3_pure_noise`.

The process — not just the result — is the point:

- **Deterministic regression test.** Six fixed-seed asserts lock the designed verdict in each world; they have been reproduced bit-for-bit across different machines/BLAS (synthetic generators are fully seeded).
- **Monte Carlo characterization with a frozen rule.** Verdict thresholds were selected on a *design* seed range, frozen together with four **pre-registered** targets, then evaluated **once** on a *virgin held-out* seed range.
- **An hypothesis that failed first.** A mechanism-targeted "fifth condition" (loss on market-tail days) was the principled first fix for V1.3's residual error rate; tested on the design set it could not separate the dangerous cases from genuine alpha, so it was rejected in favor of the graded soft-score — which *dominated* a blunt margin rule.
- **Pre-registration that caught a miss.** V1.4 met **3 of 4** held-out targets; the momentum-detection target (≥85%) missed at 83%. The miss is reported, not hidden — that is what pre-registration is for.

### V1.4 held-out results (200 seeds/world, virgin seeds)

| Metric | Result |
|---|---:|
| H3 noise certified as `H1` (false positive on pure luck) | **0 / 400 = 0.0%** |
| H2 hidden-crash momentum certified as `H1` (dangerous) | **2 / 200 = 1.0%** |
| H1 momentum detected as `H1` | 166 / 200 = 83.0% |
| H1 reversal detected as `H1` | 162 / 200 = 81.0% |
| Pre-registered targets met | 3 / 4 |

### What V1.4 fixed vs V1.3 (paired, on the *same* held-out worlds)

| Metric | V1.3 | V1.4 |
|---|---:|---:|
| H2 momentum → `H1` (dangerous) | 10 / 200 = 5.0% | **2 / 200 = 1.0%** |
| H3 noise → `H1` | 1 / 400 = 0.2% | **0 / 400 = 0.0%** |

The graded score cut dangerous hidden-risk certifications five-fold (8 cases intercepted, 0 worsened; McNemar exact one-sided *p* ≈ 3.9×10⁻³) at a measured ~3–6 pp detection cost. **Interpretation: the referee is intentionally conservative — it nearly eliminates noise-to-alpha and hidden-crash-to-alpha errors, accepting that it will quarantine some true alpha.**

---

## Real-data applications — the frozen referee meets data it has never seen (F0 → F0d)

The **frozen** V1.4 referee (no re-tuning) was applied to public equity data. These are **public-data prototypes** — yfinance adjusted closes, current index constituents (survivorship-biased), Ken French daily factors, linear 10 bps costs — *not* CRSP-grade studies. Signals are formed monthly (BHM convention) but **held daily**, so the daily-calibrated referee applies directly.

### Residual momentum across the size spectrum — a monotone null

| Phase | Universe | `L2` residual-momentum net Sharpe | Placebo pct | DSR z | Verdict |
|---|---|---:|---:|---:|---|
| **F0** | S&P 100 (mega-cap) | **+0.10** | 98.5 | −1.6 | `H3` |
| **F0b** | S&P 500 (large/mid) | **−0.02** | 100.0 | −2.1 | `H3` |
| **F0c** | small/mid-cap (~202 names) | **−0.59** | 31.2 | −4.6 | `H3` |

Factor-neutral residual momentum is **not investable in clean public US equity data over 2007–2024 after costs**, and — against the classic "momentum lives in small caps" prior — it gets *worse* as capitalization falls (the post-2009 small-cap momentum regime). Survivorship bias inflates momentum *upward*, so these nulls are **conservative-strong**.

### The headline real-data finding: the double gate beats a naive screen

At F0 and F0b the residual-momentum strategy **beat ~98–100% of its own cost-shifted placebo distribution** — a placebo-only screen would have certified it — **yet failed the deflated-Sharpe hurdle** (DSR z < 0) and was correctly rejected. Transaction costs push the no-information placebo distribution well below zero, so beating it is easy; the DSR hurdle is the gate that matters. **This is the strongest real-data evidence that the framework's design choices are not cosmetic.**

### F0d — a pre-registered hidden-risk test (the honest way to probe `H2`)

`H2` can only fire on a strategy that *passes the luck gate first*, which none of the momentum nulls did. Rather than hunt universes until an `H2` appears (the p-hacking the project guards against), F0d **declares two hidden-risk harvesters ex ante** and asks whether the referee classifies them correctly:

- `S1_downside_risk` — long high `(downside-beta − plain-beta)`, the *non-linear* tail amplification that survives linear residualization while co-crashing with the market (the synthetic-H2 archetype; harvests the Ang-Chen-Xing downside-risk premium).
- `S2_lowvol` — the low-volatility anomaly, a *different* archetype that crashes in junk-rallies (market **up**), not market co-crashes.

| Signal | `L2` residual Sharpe | DSR z | downside-gap | skew | coskew | cdar | Verdict |
|---|---:|---:|---:|---:|---:|---:|---|
| `S1_downside_risk` | −0.23 | −3.01 | +0.20 | +0.22 | −0.05 | 0.52 | `H3` |
| `S2_lowvol` | −0.80 | −5.48 | +0.08 | −0.48 | −0.02 | 0.85 | `H3` |

Three findings, two of them positive and unique to F0d:

1. **The `H2` path is validated end-to-end.** In the offline demo (a synthetic panel with a downside-risk premium injected), `S1` earns a premium *and* co-crashes → the referee returns **`H2_hidden_risk`**, refusing to certify a profitable-but-co-crashing strategy as alpha. The detector engages exactly as designed.
2. **On real data, the tail battery measures real risk.** `S1`'s realized P&L shows an elevated drawdown beta (cdar 0.52) and a positive downside-gap — the tail machinery *sees* the risk. There is simply no premium to compensate it (residual Sharpe −0.23): in the liquid S&P 500 post-2007, bearing downside risk was not paid. Correct verdict: `H3`.
3. **Archetype-specificity, measured.** `S2_lowvol` has real drawdowns (cdar 0.85) but **no market co-crash** (coskew ≈ 0, downside-gap ≈ 0) — the predicted junk-rally profile. The market-co-crash-tuned battery therefore fires too few conditions to flag it. This is the first *empirical* measurement, on real data, of the battery's archetype-generalization boundary — a limitation we had only stated in words.

*Honesty note:* `S1` sorts on trailing downside-beta-gap and the verdict measures realized-P&L tails — related but out-of-sample, not mechanically identical. It is unsurprising that its tails light up; the non-circular decider is that `S1` **does not earn**, so it is `H3` regardless. The test did its job: no premium → no `H2`, and the two archetypes are distinguished.

---

## What this project demonstrates

- **Noise is never certified as alpha** (0/400 held-out; 0 luck-gate passes on pure noise).
- **Hidden crash-risk is rarely misclassified as alpha** (1% held-out, down from 5%).
- **The double luck gate beats a naive placebo screen on real data** — the cost-shifted-placebo trap is real and the DSR hurdle catches it.
- **The tail battery measures real crash/tail risk** when a strategy carries it (momentum's negative skew; the downside-risk harvester's drawdown beta).
- **The referee reports its own limits** — a pre-registered target missed honestly (83% vs 85%), and a measured archetype-generalization boundary (`S2`).
- **Negative results are part of the deliverable** — weak public-data signals are rejected, not marketed.

---

## Design notes (and why they matter)

- **Beta-only residuals.** Factor betas are estimated *with* an intercept (unbiased slopes) but the residual subtracts *only* slope×factor, never the intercept. This strips exposure without demeaning, avoiding a rolling-intercept artifact found in V1 (where residual-vs-future-residual IC went spuriously negative). It also keeps the residual convention identical between synthetic calibration and real-data use. Literal BHM additionally removes the intercept; this factor-neutral variant is a declared, minor deviation in favor of project-wide consistency.
- **Frequency consistency.** The referee's thresholds were calibrated on *daily* long-short returns, so real strategies are formed monthly but held daily and evaluated on daily P&L.
- **Reproducibility.** Synthetic runs are fully seeded and reproduce bit-for-bit across machines; every real-data notebook ships executed in an offline DEMO mode and switches to live data with a single config flag.

---

## How to run

Each notebook runs end-to-end (`Restart & Run All`). They ship in **DEMO mode** (`data_source='shaped'`) — offline, no internet, realistic synthetic panels — so the pipeline is visibly working. For the real study set `data_source='yfinance'` and run on Colab (internet required for prices + Ken French factors). Real-data results are vendor-dependent: what should be robust is the **methodology and the verdict logic**, not a specific Sharpe to the digit.

```text
.
├── notebooks/
│   ├── 01_synthetic_referee_validation.ipynb   # frozen referee + held-out MC
│   ├── 02_f0_sp100_public_data.ipynb                # first real run; double-gate demo
│   ├── 03_f0_sp500_public_data.ipynb               # broad universe
│   ├── 04_f0_smallcap_public_data.ipynb            # small/mid-cap robustness
│   └── 05_f0_declared_hidden_risk_harvesters.ipynb # pre-registered H2 test
├── README.md
├── PROJECT_SUMMARY.md
└── outputs/
```

---

## Limitations

- Current-constituent universes → survivorship bias (inflates momentum/alpha upward).
- No delisting returns; yfinance data are not institutional-grade.
- Linear 10 bps costs; no market impact, borrow, or capacity modelling.
- Residualization is beta-only in the public prototype; no IPCA/autoencoder levels yet.
- The deflated-Sharpe gate is an approximation (exact Lo/Mertens SE, `√(2·ln N)` hurdle), not the full Bailey-López de Prado PSR/DSR.
- **V1.4 thresholds were calibrated on synthetic worlds; transfer to real data is a model assumption** — F0d gives encouraging but not conclusive evidence on the tail side, and shows the battery is tuned to a market-co-crash archetype.
- No claim of production readiness or certified empirical alpha.

---

## Roadmap

Point-in-time universe + delisting returns (removes survivorship); a real-data threshold-recalibration split using the V1.3–V1.4 protocol *once a profitable real strategy exists to calibrate against*; broader hidden-risk archetypes for the tail battery (smooth short-vol bleed, liquidity, slowly-varying beta); and the spec's extensions (option-based tail tests, capacity curves, IPCA/autoencoder residualization).

---

*This repository is a research project and does not constitute investment advice or a production trading system.*
