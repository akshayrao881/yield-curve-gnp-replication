# Does the Yield Curve Still Predict Growth? A Replication of Estrella & Hardouvelis (1991), Extended to 2023

**Business question:** If the 10-year minus 3-month Treasury spread predicted U.S. output growth in 1955-1988, does it still predict it through 2023, and does it beat richer macro models out-of-sample?

**Headline results** (200 quarterly observations, 1974Q1-2023Q4, 4-quarter-ahead real GNP growth)
- SPREAD stays significant: coefficient **0.64** (Newey-West SE 0.18, p = 0.0005), vs **1.30** in the original paper (Table I, k = 4)
- Explanatory power fell sharply: adj. R² **0.13** vs **0.35** in the original
- Adding controls does not help out-of-sample: expanding-window RMSE **2.40** (SPREAD only) vs **2.76** (full model)
- The four controls are jointly insignificant: Wald χ² = 2.48, df = 4, p = 0.65
- Structural break at 2008Q1 (Newey-West Wald): full model rejects stability (p = 0.001); SPREAD-only model does not at 5% (p = 0.09)

## Replication vs original (like-for-like, k = 4)
| Specification | Original SPREAD coef. | This study | Original adj. R² | This study |
|---|---|---|---|---|
| SPREAD only (Table I) | 1.30 | 0.64 | 0.35 | 0.13 |
| + real fed funds rate (Table III) | 1.04 | 0.67 | 0.42 | 0.13 |
| + all controls (Table IV) | 0.81 | 0.56 | 0.49 | 0.15 |

The real fed funds coefficient flips from significantly negative (original, -0.28) to small and insignificant (here, +0.04; Newey-West p = 0.70).

## Models
1. GNP.FG = a0 + a1 SPREAD
2. Model 1 + RFF.G (ex-ante real fed funds rate; expected inflation from a recursive AR(12), as in the paper)
3. Model 2 + GLI.G (leading-indicator growth) + LDEP.G (lagged growth) + GNPDEF.G (lagged inflation)

GNP.FG = (400/4) x [log(GNP t+4) - log(GNP t)]. Inference uses Newey-West standard errors with lag 3 (equal to k - 1, and consistent with the residual ACF).

## Model selection
| Model | Adj. R² | AIC | BIC | CV RMSE | CV MAE |
|---|---|---|---|---|---|
| 1: SPREAD | 0.128 | 863.3 | **873.2** | **2.399** | **1.478** |
| 2: + RFF.G | 0.126 | 864.7 | 877.9 | 2.433 | 1.488 |
| 3: full | 0.150 | **861.9** | 885.0 | 2.760 | 1.584 |

Cross-validation: expanding window, first 120 quarters as the initial training set, one-step-ahead evaluation.

## Diagnostics (Model 1)
| Test | Result |
|---|---|
| Breusch-Godfrey (order 3) | Autocorrelation present, p = 1.6e-21, so Newey-West is required |
| Shapiro-Wilk | Non-normal residuals (W = 0.932), fat tails |
| RESET (Newey-West Wald) | No misspecification, p = 0.18 |
| Chow / Wald at 2008Q1 | chi-sq 4.81, df 2, p = 0.09 |
| CUSUM | Stays inside bounds until about 2019, then touches the lower bound |
| VIF (Model 3) | Max 1.26, no multicollinearity |

## Takeaways
1. The term spread remains a statistically significant predictor, but its slope and R² are less than half of the 1955-1988 values.
2. Extra macro controls add in-sample fit (lowest AIC) but hurt out-of-sample accuracy and are unstable across the 2008 crisis.
3. GLI.G looks significant under plain OLS (p = 0.004) and is not once autocorrelation is handled (p = 0.28), a reminder to check standard errors before trusting a t-stat.

## Data
All series are quarterly, from FRED, in the data files listed below. The raw files span 1947-2025; the study window is 1955Q1-2024Q4, and the usable regression sample is **1974Q1-2023Q4 (200 observations)**. It starts in 1974Q1 because the recursive AR(12) inflation forecast needs a burn-in period, and ends in 2023Q4 because the dependent variable looks four quarters ahead.

| Variable | Role | How it is built | Source (file) |
|---|---|---|---|
| `GNP.FG` | Dependent (Y) | Forward 4-quarter real GNP growth, annualized: `(400/4) x [log(GNP t+4) - log(GNP t)]`. Underlying series is real GNP in billions of dollars | Real GNP (`GNP.csv`) |
| `SPREAD` | Main predictor (X) | 10-year Treasury yield minus 3-month T-bill yield, in percent | `GS10` (`GS10.csv`), `TB3MS` (`TB3MS.csv`) |
| `RFF.G` | Control (Z1) | Ex-ante real fed funds rate: nominal rate minus expected inflation, where expected inflation is a one-quarter-ahead forecast from an AR(12) re-estimated recursively on the GNP deflator | `FEDFUNDS` (`FEDFUNDS.csv`), `GNPDEF` (`GNPDEF.csv`) |
| `GLI.G` | Control (Z2) | Annualized quarterly growth of the leading-indicators index: `400 x [log(GLI t) - log(GLI t-1)]` | OECD composite leading indicator for the U.S., FRED `USALOLITOAASTSAM` (`GLI.csv`) |
| `LDEP.G` | Control (Z3) | Lagged dependent variable: past 4-quarter real GNP growth, to capture persistence | Real GNP (`GNP.csv`) |
| `GNPDEF.G` | Control (Z4) | Past 4-quarter inflation from the GNP deflator, annualized | `GNPDEF` (`GNPDEF.csv`) |

`USREC.csv` (NBER recession indicator) is used only to shade recession periods on the charts; no regression result depends on it. It is FRED's quarterly series `USRECQ`, trimmed to 1855Q1–2025Q2 (the range the code expects) with the column renamed to `USREC`.

## Run it
```
# R >= 4.2
install.packages(c("tidyverse","zoo","lmtest","sandwich","broom","car","strucchange","caret","kableExtra","patchwork","gridExtra"))
rmarkdown::render("yield_curve_gnp_replication.Rmd")   # needs LaTeX (xelatex)
```

## Limitations
- Overlapping 4-quarter forecast windows make errors serially correlated; Newey-West corrects standard errors but the effective sample is smaller than 200
- Current FRED vintage of GNP is more heavily revised than the paper's 1988 data, and the leading-indicator series is the OECD index rather than the U.S. composite
- The paper's survey-forecast comparison and recession probit are not replicated

## Authors
Akshay Rao and Reshma Thomas. Original paper: Estrella, A. and Hardouvelis, G., 1991, *The Term Structure as a Predictor of Real Economic Activity*, Journal of Finance 46(2), 555-576.
