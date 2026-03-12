# QRT Asset Allocation Prediction

Work in progress. This repository documents my experiments and research notes for the QRT Asset Allocation Prediction challenge.

The objective of this project is to predict the **sign of the next‑day return** for each allocation using tabular market features such as lagged returns, signed volumes, and flow statistics.

The project is treated as a small **quant research notebook**: experiments, observations, and hypotheses are recorded as the work progresses.

---

# Project Structure

```
qrt-asset-allocation-prediction/

notebooks/        experiment notebooks
submissions/      generated submission files
data/             competition dataset
venv/             python virtual environment
README.md         research log and project summary
```

---

# Dataset Overview

Each row corresponds to a pair:

```
(allocation, TS)
```

where `TS` represents the time step.

Main feature groups:

**Lagged returns**

RET_1 ... RET_20

**Signed volume features**

SIGNED_VOLUME_1 ... SIGNED_VOLUME_20

**Flow statistics**

FLOW_MEAN_*
FLOW_IMBALANCE_*
FLOW_STD_*
FLOW_SHOCK

**Turnover**

MEDIAN_DAILY_TURNOVER

**Group indicators**

GROUP_1 ... GROUP_4

Target variable:

TARGET = sign of next‑day return

---

# Feature Engineering

Starting from the benchmark notebook, several additional summary features were constructed.

## Return statistics

Average recent performance

AVERAGE_PERF_3
AVERAGE_PERF_5
AVERAGE_PERF_10
AVERAGE_PERF_15
AVERAGE_PERF_20

Cross‑sectional averages within each TS

ALLOCATIONS_AVERAGE_PERF_*

## Volatility features

STD_PERF_20
ALLOCATIONS_STD_PERF_20

These capture recent return dispersion and cross‑sectional volatility regimes.

## Flow statistics

FLOW_MEAN_3
FLOW_MEAN_5
FLOW_MEAN_10
FLOW_MEAN_20

FLOW_IMBALANCE_5
FLOW_IMBALANCE_10
FLOW_IMBALANCE_20

FLOW_STD_10
FLOW_STD_20

FLOW_SHOCK

Missing values were filled with zero following the benchmark preprocessing.

---

# Models Explored

## Ridge Regression (Baseline)

Ridge regression was used as the primary benchmark model.

Cross‑validation accuracy:

~0.520

Public leaderboard score:

~0.508

Despite its simplicity, Ridge performs competitively and serves as a strong baseline.

---

## Logistic Regression

A logistic regression model with feature standardization was tested.

Pipeline:

StandardScaler → LogisticRegression

Cross‑validation accuracy:

~0.520

Performance is similar to Ridge.

Without feature scaling, convergence becomes significantly slower.

---

## XGBoost

Tree‑based models were explored to capture nonlinear interactions.

Example configuration:

max_depth = 3
learning_rate = 0.05
n_estimators = 200

Cross‑validation accuracy:

~0.523

However, leaderboard performance does not improve significantly (~0.507), suggesting possible overfitting or validation mismatch.

---

# Observations from Ridge Coefficients

Inspecting Ridge coefficients provides insight into which features actually drive predictions.

## Strong signals

RET_1 ... RET_10
AVERAGE_PERF_3
AVERAGE_PERF_10
ALLOCATIONS_AVERAGE_PERF_*

This indicates that **short‑term return structure** and **cross‑sectional averages** dominate predictive power.

## Weak signals

FLOW_*
SIGNED_VOLUME_*
GROUP_*
MEDIAN_DAILY_TURNOVER

These features have coefficients close to zero in the linear model.

This suggests that flow and volume variables provide little linear predictive power in this dataset.

---

# Current Findings

Key observations so far:

1. Short‑term return dynamics dominate predictive performance.

2. Cross‑sectional statistics provide additional signal.

3. Flow and signed volume features appear weak under linear models.

4. More complex models do not necessarily improve leaderboard performance.

5. Feature quality appears more important than model complexity.

---

# Future Experiments

Several directions will be explored next.

## Feature selection

Based on Ridge coefficients, reduce noisy features and focus on the most informative groups:

RET features
return averages
cross‑sectional statistics
volatility features

## Nonlinear feature engineering

Potential transformations to test:

risk‑adjusted returns
(momentum / volatility)

volatility regimes
(short‑term vol / long‑term vol)

cross‑sectional normalization
(return minus cross‑sectional mean)

cross‑sectional ranking

These transformations may reveal nonlinear structures not captured by linear models.

## Model exploration

Possible future models:

LightGBM
regularized logistic regression
model blending

However, improvements are expected to come primarily from better feature design rather than model complexity.

---

# Reflections

A key takeaway from this project is that **alpha discovery is closely related to feature engineering**.

In tabular financial datasets, the main challenge is often identifying meaningful transformations of existing signals rather than building increasingly complex models.

In many cases:

well‑designed features + simple models

outperform

complex models with weak features.

This project is therefore treated as an iterative research process rather than a one‑shot modeling task.
