# Cookie Cats A/B Test — Gate Placement and Player Retention

A data science portfolio project analyzing a real mobile game A/B test: does moving the first "gate" from level 30 to level 40 change player retention?

## Business Problem

Cookie Cats is a mobile puzzle game where players hit a gate that forces them to wait or pay to continue. The company ran an A/B test comparing two gate placements:

- **Control (`gate_30`):** gate appears at level 30
- **Treatment (`gate_40`):** gate appears at level 40

**Hypothesis:** Moving the gate from level 30 to level 40 changes 7-day player retention.

## Experiment Design

| Parameter | Value |
|---|---|
| Primary metric | `retention_7` (7-day retention) |
| Secondary metric | `retention_1` (1-day retention) |
| Guardrail metric | `sum_gamerounds` (engagement) |
| Significance level (α) | 0.05 |
| Power | 0.8 |
| Test type | Two-sided |

## Dataset

**Source:** [Mobile Games A/B Testing - Cookie Cats](https://www.kaggle.com/datasets/yufengsui/mobile-games-ab-testing) (Kaggle)

| Column | Description |
|---|---|
| `userid` | Unique player ID |
| `version` | `gate_30` (control) or `gate_40` (treatment) |
| `sum_gamerounds` | Rounds played in the first 14 days |
| `retention_1` | Returned on day 1 (True/False) |
| `retention_7` | Returned on day 7 (True/False) |

~90,000 rows, no missing values, no duplicate users.

## Methodology

1. **Data quality checks** — nulls, duplicates, and a Sample Ratio Mismatch (SRM) check via chi-square goodness-of-fit, to confirm randomization worked before trusting any result.
2. **Exploratory analysis** — distribution of `sum_gamerounds`, outlier identification (one extreme value removed), and a decision to keep zero-round users (legitimate "churned on day 0" data points).
3. **Primary analysis** — two-proportion z-test with confidence intervals for `retention_1` and `retention_7`.
4. **Robustness check** — 5,000-iteration bootstrap on `retention_7`, independent of the z-test's normal approximation.
5. **Guardrail check** — Mann-Whitney U test on `sum_gamerounds` (right-skewed, so a non-parametric test is used instead of a t-test).
6. **Power analysis** — minimum detectable effect (MDE) at 80% power, to confirm the sample size could reliably detect the observed effect.

## Results

| Metric | gate_30 | gate_40 | Difference | p-value | Significant? |
|---|---|---|---|---|---|
| `retention_1` | 44.82% | 44.23% | −0.59 pp | 0.074 | No |
| `retention_7` (primary) | 19.02% | 18.20% | −0.82 pp | 0.0016 | **Yes** |
| `sum_gamerounds` (guardrail) | mean 51.34 | mean 51.30 | — | ≈0.05 | No |

The bootstrap analysis independently confirms the `retention_7` result: a 95% CI of [−0.0133, −0.0033] and only a 0.1% chance that `gate_40` actually performs better.

## Recommendation

**Keep the gate at level 30.** Moving it to level 40 produces a statistically significant drop in 7-day retention — the pre-specified primary metric — with no offsetting benefit to day-1 retention or player engagement. At scale, this translates to meaningfully fewer weekly active players per million installs, with no measurable upside.

## Limitations

- The dataset has no timestamps, so novelty/primacy effects and the "peeking" problem could not be checked.
- This is a retrospective analysis; the experiment design is stated as if pre-registered but was not locked in before data collection.
- Only two retention snapshots (day 1, day 7) are available.
- The SRM check is borderline (p = 0.0086 against a strict α = 0.001 threshold) — worth monitoring if this test were repeated.

## Project Structure

```
.
├── README.md
├── A-B testing .ipynb   # Full analysis notebook
└── cookie_cats.csv       # Dataset (not included — download from Kaggle)
```

## Requirements

```
pandas
numpy
matplotlib
seaborn
scipy
statsmodels
```

## How to Run

1. Download `cookie_cats.csv` from [Kaggle](https://www.kaggle.com/datasets/yufengsui/mobile-games-ab-testing) and place it in the project folder.
2. Install dependencies: `pip install pandas numpy matplotlib seaborn scipy statsmodels`
3. Open and run `A-B testing .ipynb` top to bottom.
