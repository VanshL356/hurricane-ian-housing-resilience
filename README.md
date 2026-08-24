# Predicting Housing Market Resilience After Hurricane Ian

Can pre-disaster housing and economic conditions predict which local housing markets will be most resilient after a major shock?

This project combines Zillow housing values, NOAA Storm Events damage estimates, and county economic data to measure and predict housing-market resilience following Hurricane Ian in 2022.

The analysis covers **293 counties across Florida, Alabama Georgia from 2016–2024**, with a focused case study of **20 Hurricane Ian-exposed Florida counties**.

---

## Project Summary

Built a county-level housing resilience index based on four dimensions of
post-September-2022 market performance:

- 24-month home-value appreciation
- maximum drawdown from the September 2022 baseline
- number of months below the pre-event baseline
- post-event housing-price volatility

I then used pre-Ian housing and economic characteristics to predict subsequent
resilience.

The project uses two related prediction designs:

1. **Regional predictive performance:** Linear regression, elastic net, and
   random forest models are evaluated across 281 counties in Florida, Alabama and
   Georgia using state-stratified 10-fold cross-validation.

3. **Hurricane Ian case study:** All 20 designated Ian-exposed Florida counties
   are removed from model training. Benchmark models are trained on 198 Georgia
   and Florida counties outside the designated Ian group, and then used to
   predict resilience for the 20 completely held-out Ian counties.

This separates the general predictive-performance question from the
Ian-specific benchmarking exercise.

Models were evaluated using **state-stratified 10-fold cross-validation**.

The ensemble achieved the strongest out-of-sample performance:

**CV R² = 0.531**

---

## Model Performance

![Model Performance](figures_tables/model_performance.png)

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

Recent housing-market dynamics were more informative than most static economic characteristics.

This suggests that the market's trajectory entering the event contained substantial information about subsequent resilience.

---

## Hurricane Ian Case Study

For the disaster case study, the 20 designated Hurricane Ian-exposed Florida
counties are treated as a completely held-out sample.

The benchmark models are trained on **198 counties in Georgia and Florida
outside the designated Ian group**. None of the 20 Ian counties contribute to
model estimation or elastic-net tuning.

For each Ian county:

**Unexpected Resilience = Actual Resilience − Predicted Resilience**

Positive values indicate greater resilience than predicted from pre-Ian
fundamentals. Negative values indicate less resilience than predicted.

### Actual vs. Predicted Resilience

![Actual vs Predicted Resilience](figures/03_florida_actual_vs_predicted.png)

Several counties substantially underperformed the regional benchmark.
Charlotte, Flagler, Sarasota, Lee, and Manatee exhibited some of the largest
negative prediction errors.

Other exposed counties, including Highlands, Hardee, Okeechobee, Hendry,
Glades, and Seminole, were more resilient than the benchmark predicted.
---

## Does Hurricane Damage Help Explain Unexpected Resilience?

![Damage and Unexpected Resilience](figures/04_damage_unexpected_resilience.png)

Higher NOAA property damage per capita was associated with lower-than-predicted
housing resilience among the 20 Ian-exposed Florida counties.

The continuous relationship was negative but imprecisely estimated:

- Spearman correlation: **ρ = -0.305**
- Spearman p-value: **0.192**
- OLS damage coefficient: **-0.064**
- OLS p-value: **0.126**
- OLS R²: **0.125**

Given the small 20-county sample, these estimates are interpreted as
descriptive rather than causal.

### Exploratory Severe-Damage Comparison

The five counties with at least **$2,000 in reported NOAA property damage per
capita** had substantially lower unexpected resilience than the remaining 15
Ian-exposed counties.

| Group | N | Mean Unexpected Resilience | Median Unexpected Resilience |
|---|---:|---:|---:|
| Other Ian-exposed counties | 15 | -0.047 | -0.183 |
| Severe-damage counties | 5 | -0.781 | -0.891 |

The difference was significant in an exploratory Wilcoxon rank-sum comparison
(**p = 0.029**).

Because the $2,000-per-capita threshold was selected descriptively rather than
preregistered, this result should be interpreted as exploratory.

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

### Economic Features

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

### Regional Prediction

The general modeling sample contains **281 counties** with complete predictor
and resilience information:

- Alabama: 63
- Florida: 63
- Georgia: 155

Linear regression, elastic net, and random forest models are evaluated using
state-stratified 10-fold cross-validation.

The equal-weight ensemble achieves a cross-validated R² of **0.531**.

### Hurricane Ian Holdout

For the Ian case study, all 20 designated Ian-exposed Florida counties are
removed before model estimation.

The primary benchmark contains:

- Florida counties outside the designated Ian group: 43
- Georgia counties: 155
- Total training counties: **198**

Linear regression, elastic net, and random forest models are fit to those 198
counties. Elastic-net tuning uses only the benchmark training sample.

The three predictions are averaged to generate the final benchmark prediction
for each completely held-out Ian county.

---

## Limitations

NOAA Storm Events property-damage estimates are an imperfect measure of total hurricane exposure. A value of zero represents zero reported property damage in the selected NOAA records and should not necessarily be interpreted as zero physical impact.

The Hurricane Ian case study contains only 20 counties, which limits statistical power.

The resilience index is researcher-defined, although alternative weighting schemes and a PCA specification produce very similar county rankings.

The predictive models identify relationships useful for forecasting and benchmarking; they do not establish causal effects of individual predictors or Hurricane Ian damage.

---

## Key Takeaway

Pre-Ian housing-market conditions contain meaningful information about
subsequent county housing resilience. A cross-validated ensemble explains about
**53% of held-out variation** across the broader southeastern sample, with
recent housing appreciation and volatility among the strongest predictors.

When the 20 Hurricane Ian-exposed Florida counties are excluded entirely from
model training, several heavily damaged markets—especially Charlotte, Lee,
Sarasota, and Manatee—substantially underperform their predicted resilience.

Across the Ian counties, greater NOAA property damage is directionally
associated with lower unexpected resilience. The continuous relationship is
imprecisely estimated in the small 20-county sample, while the five
highest-damage counties show substantially lower unexpected resilience in an
exploratory group comparison.

These conclusions remain essentially unchanged when Alabama is removed from
the benchmark training sample.

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
