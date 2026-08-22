# Predicting Housing Market Resilience After Hurricane Ian

Can pre-disaster housing and economic conditions predict which local housing markets will be most resilient after a major shock?

This project combines Zillow housing values, NOAA Storm Events damage estimates, and county economic data to measure and predict housing-market resilience following Hurricane Ian in 2022.

The analysis covers **293 counties across Florida, Georgia, and Alabama from 2016–2024**, with a focused case study of **20 Hurricane Ian-exposed Florida counties**.

---

## Project Summary

Built a county-level housing resilience index based on four dimensions of post-September-2022 market performance:

- 24-month home-value appreciation
- maximum drawdown from the September 2022 baseline
- number of months below the pre-event baseline
- post-event housing-price volatility

Used only **pre-Ian information** to predict resilience using:

- Linear regression
- Elastic net
- Random forest
- Equal-weight ensemble

Models were evaluated using **state-stratified 10-fold cross-validation**.

The ensemble achieved the strongest out-of-sample performance:

**CV R² = 0.531**

---

## Model Performance

![Model Performance](figures/01_model_performance.png)

| Model | CV RMSE | CV MAE | CV R² |
|---|---:|---:|---:|
| Ensemble | **0.536** | **0.423** | **0.531** |
| Random Forest | 0.555 | 0.435 | 0.498 |
| Elastic Net | 0.558 | 0.444 | 0.491 |
| Linear Regression | 0.562 | 0.444 | 0.485 |

The ensemble explains approximately **53% of held-out variation in county housing resilience**.

Performance differences across the three individual models are relatively small, suggesting that the predictive signal is not dependent on a single modeling approach.

---

## What Predicts Housing Resilience?

![Feature Importance](figures/02_feature_importance.png)

Random-forest permutation importance indicates that the strongest predictive features were:

1. 12-month pre-Ian housing return
2. 12-month pre-Ian housing-market volatility
3. 6-month pre-Ian housing return
4. State
5. 24-month pre-Ian housing return

Recent housing-market dynamics were more informative than most static socioeconomic characteristics.

This suggests that the market's trajectory entering the event contained substantial information about subsequent resilience.

---

## Hurricane Ian Case Study

The predictive models provide an expected resilience score for each county based only on conditions observed before Hurricane Ian.

I define:

**Unexpected Resilience = Actual Resilience − Out-of-Fold Predicted Resilience**

Positive values indicate that a county performed better than expected.

Negative values indicate that a county was less resilient than its pre-Ian characteristics predicted.

### Actual vs. Predicted Resilience

![Actual vs Predicted Resilience](figures/03_florida_actual_vs_predicted.png)

Several counties experienced substantially lower resilience than predicted by their pre-Ian fundamentals.

Charlotte, Sarasota, Lee, Manatee, and Flagler were among the more notable negative prediction errors, while counties including Highlands, Hardee, Hendry, Okeechobee, Seminole, and Orange performed better than expected.

---

## Does Hurricane Damage Help Explain Unexpected Resilience?

![Damage and Unexpected Resilience](figures/04_damage_unexpected_resilience.png)

Among the 20 Ian-exposed Florida counties, higher NOAA property damage per capita was associated with lower-than-predicted housing resilience.

The continuous relationship was directionally negative but statistically imprecise:

- Spearman correlation: **ρ = -0.315**
- p-value: **0.176**
- OLS damage coefficient: approximately **-0.047**
- OLS p-value: approximately **0.257**

The small 20-county case-study sample limits statistical precision, so these estimates are interpreted as descriptive rather than causal.

An exploratory comparison between the five counties with at least **$2,000 in reported NOAA property damage per capita** and the remaining exposed counties produced a Wilcoxon rank-sum p-value of approximately **0.036**.

Because this threshold was chosen descriptively rather than preregistered, the result is treated as exploratory.

---

## Resilience Index

The primary index averages four standardized components:

\[
Resilience_i =
\frac{
Z(Return_{24m}) +
Z(MaxDrawdown) -
Z(MonthsBelowBaseline) -
Z(Volatility)
}{4}
\]

All components are oriented so that higher scores represent greater resilience.

The primary measure uses equal weights for interpretability.

### Index Robustness

I tested whether county rankings depended on those weighting choices.

| Alternative Specification | Spearman Correlation with Primary Index |
|---|---:|
| Return-heavy | 0.979 |
| Drawdown-heavy | 0.997 |
| Recovery-heavy | 0.986 |
| PCA-based index | 0.994 |

The resulting county rankings are nearly identical across specifications.

Among Ian-exposed counties, the damage-resilience relationship also remained negative under every alternative index construction.

---

## Data Engineering

The project integrates multiple county-level datasets with different geographic and temporal structures.

### Zillow

Monthly Zillow Home Value Index data provide the primary housing-market outcome.

### NOAA Storm Events

NOAA Storm Events data were used to construct Hurricane Ian property-damage exposure.

This required additional geographic cleaning because NOAA records include both counties and forecast zones such as:

- `COASTAL LEE`
- `INLAND CHARLOTTE`

These zone names were normalized back to their underlying counties before damage was aggregated.

NOAA damage strings such as `7.00B`, `206.00M`, and `1000.00K` were converted to numeric dollar values.

County damage intensity was then defined as:

**Reported Ian Property Damage / 2022 County Population**

### Socioeconomic Features

Pre-Ian predictors include:

- population
- median household income
- unemployment
- population growth
- income growth
- unemployment change
- pre-Ian housing appreciation
- pre-Ian housing volatility
- September 2022 housing-value level

---

## Modeling Strategy

The final modeling sample contains **281 counties** with complete predictor and resilience information:

- Alabama: 63
- Florida: 63
- Georgia: 155

All predictive features are measured before Hurricane Ian.

Out-of-fold predictions were generated using state-stratified 10-fold cross-validation so that each county's prediction was produced by a model that did not train on that county.

Elastic-net hyperparameters were selected using an inner cross-validation loop within each outer training fold.

The final prediction for each county is the equal-weight average of the linear regression, elastic-net, and random-forest predictions.

---

## Why This Is Not Presented as a Causal DiD Study

An earlier version of the project evaluated difference-in-differences, synthetic difference-in-differences, matched event-study designs, and pretrend adjustments.

Those diagnostics consistently showed substantial differential pre-treatment housing trends between highly affected Florida counties and available comparison markets.

Because those patterns undermine the parallel-trends assumption required for credible causal interpretation, I did not present the post-Ian differences as causal effects.

Instead, the project was reframed around:

- measurement of housing-market resilience
- out-of-sample prediction
- prediction-error analysis
- descriptive disaster-damage heterogeneity

This avoids imposing a causal interpretation that the data do not support.

---

## Limitations

NOAA Storm Events property-damage estimates are an imperfect measure of total hurricane exposure. A value of zero represents zero reported property damage in the selected NOAA records and should not necessarily be interpreted as zero physical impact.

The Hurricane Ian case study contains only 20 counties, which limits statistical power.

The resilience index is researcher-defined, although alternative weighting schemes and a PCA specification produce very similar county rankings.

The predictive models identify relationships useful for forecasting and benchmarking; they do not establish causal effects of individual predictors or Hurricane Ian damage.

---

## Repository Structure

```text
R/
├── 01_build_panel.R
├── 02_build_ian_damage.R
├── 03_build_resilience_index.R
├── 04_predictive_models.R
├── 05_ian_case_study.R
└── 06_figures_tables.R

figures/
├── 01_model_performance.png
├── 02_feature_importance.png
├── 03_florida_actual_vs_predicted.png
└── 04_damage_unexpected_resilience.png

results/
├── model_performance.csv
├── resilience_by_county.csv
├── resilience_index_robustness.csv
├── random_forest_feature_importance.csv
├── ian_damage_by_county.csv
├── ian_case_study.csv
└── additional robustness results

data/
└── README.md
