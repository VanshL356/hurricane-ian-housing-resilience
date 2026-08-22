# Predicting Housing Market Resilience After Hurricane Ian

Can pre-disaster housing and economic conditions predict which county housing markets are most resilient following a major shock?

This project combines county-level housing, economic, demographic, and disaster-damage data to study housing-market resilience following Hurricane Ian.

The analysis covers counties in Florida, Alabama and Georgia from 2016–2024 and includes a focused case study of 20 Hurricane Ian-exposed Florida counties.

## Project Overview

1. Build a monthly county-level housing and economic panel.
2. Construct Hurricane Ian property-damage exposure scale using NOAA Storm Events data.
3. Develop a multidimensional housing-market resilience index.
4. Predict post-2022 resilience using only pre-Ian housing and economic fundamentals.

An ensemble of linear regression, elastic net, and random forest models is evaluated using state-stratified 10-fold cross-validation.

## Results

- Built a monthly panel covering 293 counties across Florida, Georgia and Alabama
- Constructed a housing resilience index using:
  - 24-month housing-price performance
  - maximum post-event drawdown
  - months below the September 2022 housing-value baseline
  - post-event housing-price volatility
- Predictive modeling used only information available before Hurricane Ian.
- The ensemble model achieved an out-of-sample CV R² of **0.531**.
- Random forest, elastic net, and linear regression individually achieved CV R² values of approximately **0.48–0.50**.
- Recent housing-price appreciation and pre-hurricane volatility were the strongest predictors of subsequent resilience.
- Among Ian-exposed Florida counties, greater NOAA property damage was directionally associated with lower-than-predicted resilience, although the continuous relationship was imprecisely estimated in the small 20-county sample.
- Alternative resilience-index specifications produced nearly identical county rankings, with Spearman correlations of approximately **0.98–1.00**.

## Model Performance

| Model | CV RMSE | CV MAE | CV R² |
|---|---:|---:|---:|
| Ensemble | 0.536 | 0.423 | 0.531 |
| Random Forest | 0.555 | 0.435 | 0.498 |
| Elastic Net | 0.558 | 0.444 | 0.491 |
| Linear Regression | 0.562 | 0.444 | 0.485 |

## Hurricane Ian Case Study

For each county, the predictive models generate an expected resilience score using only pre-Ian housing and economic information.

Unexpected resilience is defined as:

**Actual Resilience − Predicted Resilience**

Among the 20 Ian-exposed Florida counties, the association between NOAA property damage per capita and unexpected resilience was negative, but statistically imprecise.

The highest-damage group also exhibited lower unexpected resilience in an exploratory Wilcoxon rank-sum comparison.

## Data Structure

The project uses data from:

Zillow Home Value Index (ZHVI)
NOAA/NCEI Storm Events
County-level population data (U.S Census)
Median household income (U.S Census)
Unemployment rates (BLS)

## Repository Structure

```text
R/
    Data cleaning, feature engineering, modeling, and visualization scripts

figures/
    Final figures used in the project

results/
    Model performance and final analytical tables

data/
    Documentation for external data sources

archive/
    Supplemental causal-inference diagnostics and earlier analyses
