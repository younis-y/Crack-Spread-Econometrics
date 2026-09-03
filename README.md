# Gasoline demand pulls crude, not the other way round

A vector error correction model of the 3-2-1 crack spread — WTI crude, NY Harbor
gasoline, NY Harbor heating oil — on **1,361 weekly observations** spanning
2000-01-07 to 2026-01-30, with a confirmatory ADF/KPSS unit-root strategy,
Johansen cointegration, Bonferroni-corrected Granger causality, and
error-correction adjustment speeds read off the loading matrix.

**The textbook reading does not survive the data.** Refining is normally
described as cost-push: crude sets the input price and products follow. On weekly
data the causality runs the other way — gasoline Granger-causes crude
(F = 8.72, adjusted p < 0.001) while crude does not cause gasoline
(F = 1.94, adjusted p = 0.61) — and crude carries the only equilibrium-restoring
loading on the first cointegrating vector (α = −0.042, p = 0.044, half-life
16.2 weeks). Crude is the series doing the adjusting, which is the signature of
a demand-pull chain: gasoline → crude → heating oil.

The 3-2-1 spread is the theoretical gross refining margin from cracking three
barrels of crude into two of gasoline and one of distillate:

```
S_t = (2 * P_gasoline,t + 1 * P_heating_oil,t) - 3 * P_crude,t
```

## Findings

The direction runs from products to crude, not the other way round.

**Granger causality** (bivariate, on first differences, 4 lags, SSR F-test,
Bonferroni-corrected over the six directed pairs, α* = 0.05/6 = 0.0083):

| Direction | F | Adjusted p | Significant |
|---|---|---|---|
| Gasoline → crude | 8.721 | < 0.001 | yes |
| Crude → gasoline | 1.937 | 0.612 | no |
| Crude → heating oil | 5.450 | 0.0014 | yes |
| Heating oil → crude | 2.593 | 0.210 | no |
| Gasoline → heating oil | 8.238 | < 0.001 | yes |
| Heating oil → gasoline | 0.643 | 1.000 | no |

Gasoline Granger-causes crude; crude does not Granger-cause gasoline. Crude does
Granger-cause heating oil. The chain is gasoline → crude → heating oil, with a
direct gasoline → heating oil channel alongside it.

**Error correction.** Johansen selects rank r = 2, and the VECM is estimated with
five lagged differences (AIC), a constant inside the cointegrating relation, and
two exogenous break dummies. Loadings on the first cointegrating vector:

| Equation | α₁ | z | p | Sign |
|---|---|---|---|---|
| Crude | −0.0420 | −2.018 | 0.044 | negative: equilibrium-restoring |
| Gasoline | +0.0298 | 1.368 | 0.171 | positive, not significant |
| Heating oil | +0.0407 | 2.115 | 0.034 | positive |

Crude is the only series with a negative, and therefore error-correcting,
loading on the first cointegrating vector: it does the work of closing the gap.
Its implied half-life is ln(0.5)/ln(1 − 0.042) = 16.2 weeks. Heating oil's
loading is also significant at 5% but is positive, so it is not adjusting back
towards the relation; gasoline's is neither significant nor negative. Reading
this as "crude is the only series that adjusts" is only correct in the sense of
sign; the heating oil loading is significant and should be reported as such.

**Supporting results.** Trace and maximum-eigenvalue statistics both reject r = 0
and r ≤ 1 comfortably (trace 107.15 and 32.52 against 5% critical values of 29.80
and 15.49). CUSUM on the crude-oil equation residuals does not reject
parameter stability at 5% across the COVID and 2022 episodes; the gasoline and
heating oil equations are not tested (see Scope). Ljung-Box(12) passes in all three equations (p = 0.188, 0.699,
0.884); Jarque-Bera and ARCH(5) reject in all three. Adjustment coefficients keep
their signs under a BIC lag choice and across pre-COVID and post-COVID
subsamples.

**Forecast error variance decomposition** is not reported as a finding. The
shipped figure is superseded, and the corrected shares depend on the Cholesky
ordering. Scope states what was wrong, what the corrected shares are, and why
they still should not be read as identified.

## Reproducing

There is no pinned environment file. The notebook's stored kernel metadata
records Python 3.13.5, and the VECM specification and the numbers above were
re-run against statsmodels 0.14.4, pandas 2.3.3, numpy 2.2.6, scipy 1.15.3.

```bash
pip install pandas numpy matplotlib seaborn statsmodels scipy jupyter
cd Essay_Model
jupyter notebook Essay_Model.ipynb
```

The notebook reads its CSVs with paths relative to `Essay_Model/`, so run it with
that directory as the working directory. Run all cells top to bottom; there is no
other entry point. Figures are written to `Essay_Model/figures/` at 600 dpi. The
impulse response cell runs a 500-replication moving-block bootstrap and is the
slow step.

```
Essay_Model/
  Essay_Model.ipynb          full pipeline: unit roots → cointegration → VECM → IRF/FEVD → diagnostics
  presentation.html          slide deck (pre-dates the FEVD correction, see Scope)
  data/                      FRED CSVs as downloaded
  figures/                   fig1-fig6 and appendix figures A-D
essay_draft.md               essay text as submitted, January 2026
```

## Data

Three daily spot price series from the Federal Reserve Economic Data (FRED)
service, stored unmodified as downloaded (`observation_date` plus one value
column). Days with no published price — market holidays — are present as rows
with an empty value field, not the `.` placeholder used in some FRED vintages:
368 such rows in `DCOILWTICO.csv`, 385 in `DGASNYH.csv`, 387 in `DHOILNYH.csv`.
The loader's `df.replace(['.', '', ' '], np.nan)` is defensive; only the empty
string does any work on these files.

| File | FRED series | Description | Native units |
|---|---|---|---|
| `DCOILWTICO.csv` | DCOILWTICO | WTI crude, Cushing OK, spot | USD/barrel |
| `DGASNYH.csv` | DGASNYH | Regular gasoline, NY Harbor, spot | USD/gallon |
| `DHOILNYH.csv` | DHOILNYH | No. 2 heating oil, NY Harbor, spot | USD/gallon |

The notebook resamples to Friday close, converts the two product series to
USD/barrel (× 42), takes natural logs, and truncates to 2000-01-01 onward. That
leaves 1,361 weekly observations, 2000-01-07 to 2026-01-30. The files carry daily
history back to 1986; only the post-2000 window is used. FRED is not called at
run time, so the analysis is reproducible offline and fixed at the download date.

## Scope

What this repository does not do, and where it is known to be wrong.

- **The FEVD is corrected in code but not in the shipped figure.** The
  decomposition was built from `irf.irfs`, the non-orthogonalised moving-average
  representation, whose shocks are correlated; the tell is 100% self-explained
  variance for every series at h = 1. The cell now uses `irf.orth_irfs`. The
  figure `fig4_fevd.png`, the essay text and the slide deck all pre-date the fix
  and still show the old shares (gasoline 11% of crude variance, crude 19% of
  heating oil variance at 24 weeks). Those numbers should not be cited.
  The corrected cell carries no stored output in the shipped notebook. Re-running
  it on this data gives materially different shares — at 24 weeks, gasoline
  explains 5.2% of crude forecast error variance and crude 76.1% of heating oil
  variance — but the figure has not been regenerated here, and the orthogonalised
  decomposition is ordering-dependent: with crude ordered first in the Cholesky
  factorisation it takes all contemporaneous common variance by construction.
  Regenerate the figure, and report the ordering, before using any FEVD number.
- **The impulse response figures have the same identification question and were
  left alone.** `fig3_irf.png` plots point estimates from `irf.irfs` while its
  bootstrap band is built by rescaling with a Cholesky ratio, and the dashboard
  panel in `fig5_dashboard.png` labels an `irf.irfs` path as a one-standard-
  deviation shock. The Cholesky ordering-sensitivity cell (cell 34) has the same
  defect: it contracts `irf.irfs[h, 1, :]` with a Cholesky impulse vector and
  prints the result as a "Gas→Crude shock" percentage. Its printed numbers are
  not quoted anywhere in this README or in the essay. Only the FEVD was diagnosed
  and fixed; all three code paths need the same review and are flagged rather
  than quietly changed.
- **The I(1) premise is not clean for gasoline.** In cell 6 the confirmatory
  ADF/KPSS table classifies `ln_Gas` in levels as "Ambiguous" under the
  notebook's own decision rule: ADF = -3.098, p = 0.0267, which rejects the unit
  root at 5% (KPSS rejects stationarity, hence the ambiguity). The same cell then
  prints "Conclusion: All log-price series are I(1)". Crude (ADF p = 0.0784) and
  heating oil (p = 0.0632) are unambiguous. The VECM's I(1) premise therefore
  rests on reading the gasoline series as I(1) on the KPSS side of a split
  verdict; that reading is defensible for a log spot price but is a judgement,
  and the printed conclusion overstates it.
- **Granger causality here is short-run only.** The tests are bivariate, run on
  first differences, and exclude the error-correction term. They are not the
  VECM Wald test of joint short-run causality, and they say nothing about the
  long-run channel that the loading matrix describes.
- **Cointegration rank is a judgement at the 1% level.** The third trace
  statistic is 6.35 against a 5% critical value of 3.84 and a 1% value of 6.63.
  Strictly at 5% the trace sequence points to r = 3; r = 2 is the 1% reading and
  is what is estimated. The subsample check (notebook cell 36) returns rank = 2
  on the pre-GFC window (2000-2007, 417 obs) and rank = 2 post-COVID (2020-2026,
  318 obs), matching the full sample; it is the pre-COVID window (2000-2019,
  1,043 obs) that returns rank = 3 at 5%, the same reading the full-sample trace
  sequence gives at 5%. That cell's printed closing narrative attributes the
  divergence to the post-COVID subsample, which its own table contradicts; the
  table governs.
- **ARCH is present in all three equations and is not modelled.** Bootstrap
  confidence intervals and Granger p-values are likely over-optimistic during
  high-volatility episodes. No GARCH, DCC or NARDL specification is implemented,
  and no asymmetric ("rockets and feathers") transmission is tested.
- **Structural breaks are handled with two dummies**, Russia-Ukraine from
  2022-02-24 and COVID from 2020-03-01 to 2020-12-31, entered as exogenous
  regressors outside the cointegrating relation. No endogenous break test (Gregory-
  Hansen, Bai-Perron) is run.
- **The lag-order robustness cell is mislabelled.** Its heading reads "AIC (k=8)
  vs BIC (k=2)"; AIC selects k = 5 on this sample, and k = 5 is what the reported
  model estimates. The label is stale, the estimate is not.
- **The stability test covers one equation.** Cell 40 computes the CUSUM and the
  rolling residual variance from `vecm_result.resid[:, 0]`, the crude-oil
  equation residuals only, and `fig6_stability.png` is titled accordingly. The
  gasoline and heating oil equations are never tested for parameter stability.
- **`essay_draft.md` is the submitted text, not a maintained document.** Only the
  coursework cover sheet was removed and the figure paths repointed. Where it
  disagrees with the notebook, the notebook governs: it reports α_crude as
  p < 0.01 where the notebook prints p = 0.044, gives heating oil → gasoline as
  F = 1.19 where the notebook prints 0.643, and describes an 8-week bootstrap
  block length where the code uses 5 (= k_ar_diff). Its embedded images have been
  repointed to the byte-identical files in `Essay_Model/figures/`, which preserves
  one fault of the original: the Figure 5 caption describing the CUSUM and
  rolling-variance panels is attached to the dashboard figure
  (`fig5_dashboard.png`) rather than to `fig6_stability.png`.
- **No test suite, no CI, no packaged module.** The notebook is the deliverable.
  There is no out-of-sample forecast, no backtest, and no trading rule; nothing
  here is an investment claim.

## Provenance

Written as a master's econometrics coursework essay, January 2026. The essay text
is included as `essay_draft.md`; the submitted PDF is not distributed.

## Licence

MIT. See `LICENSE`.
