# Clinical Risk Prediction (Bayesian Heart Disease Modeling)

In this project, Bayesian logistic regression is applied to model the probability of heart disease using clinical data from the canonical UCI Heart Disease dataset. The focus is not only on prediction, but on understanding how different prior structures affect inference, stability, and convergence in a medical context. 

---

## Project Overview

We model a binary outcome (`target`: heart disease *present* vs *absent*) using a set of clinically meaningful predictors:

- Standardized continuous variables: `age_z`, `thalach_z` (max heart rate), `oldpeak_z` (ST depression)
- Categorical risk factors: `sex`, `cp` (chest pain type), `exang` (exercise-induced angina), `ca` (number of major vessels), `thal` (thallium stress test)

The goal is to compare four Bayesian prior specifications and study their impact on:

- Posterior coefficient estimates and credible intervals  
- Convergence diagnostics (Gelman–Rubin PSRF, traceplots, autocorrelation)  
- Model fit by Deviance Information Criterion (DIC)  

---

## Data & Preprocessing

- Source: UCI Heart Disease dataset (1,025 patients, 14 variables).  
- Cleaning:
  - Converted integer codes to labelled factors (e.g., `sex`, `cp`, `restecg`, `exang`, `slope`, `thal`).  
  - Removed 13 records (≈1.3%) with missing values in `ca` or `thal` by listwise deletion → **1,012 patients** retained.  
  - Standardized continuous predictors (`age`, `trestbps`, `chol`, `thalach`, `oldpeak`) to z-scores.  

- Variable selection:
  - Started with all 9 candidate predictors.  
  - Fit a frequentist logistic regression and used backward AIC selection.  
  - Final feature set: `age_z`, `thalach_z`, `oldpeak_z`, `sex`, `cp`, `exang`, `ca`. 

We also constructed age groups (5-year bands) to enable hierarchical modeling over ordered age strata, dropping the two youngest groups where all patients had disease (no variation). 

---

## Bayesian Models

All models are fit in **JAGS** via `rjags` with 3 MCMC chains, 1,000 burn-in iterations, and 5,000 sampling iterations per chain (4,600 post-burn-in draws used for summaries).

We compare four prior structures on the regression coefficients:

1. **Flat Prior**  
   - Essentially uninformative `Uniform(-1e6, 1e6)` priors on all coefficients.

2. **Weakly Informative Normal Prior**  
   - `Normal(0, 1/4)` on each coefficient (mean 0, variance 4).  
   - Encourages shrinkage while still allowing reasonably large effects. 

3. **Hierarchical Normal Prior**  
   - Group-level means and precisions for age effects and factor levels; for example  
     `beta_1[j] ~ Normal(mu_1, tau_1)` with `mu_1 ~ Normal(0, 0.25)`, `tau_1 ~ Gamma(0.01, 0.01)`.

4. **Hierarchical Normal + Ordinal Age Trend**  
   - Extends (3) by modeling age-group coefficients with a **linear trend in age group index**,  
     `beta_1[j] ~ Normal(mu_1 + slope_age * age_group_index[j], tau_1)`, capturing the monotonic increase in risk with age. 

---

## Key Results & Findings

### Convergence & Stability

- **Flat prior** shows poor convergence: many PSRFs > 3 and a multivariate PSRF of **3.42**, indicating unstable chains and high posterior uncertainty.  
- **Normal prior** performs best: PSRFs ≈ 1 with multivariate PSRF **1.03**, and well-mixed traces.  
- **Hierarchical** models (with and without age trend) converge reasonably (multivariate PSRF ≈ 1.23–1.26) but require more iterations for some group-level effects.

### Predictors that are Robust Across Priors

Across all four models, three predictors are consistently important, while the others have wide intervals that overlap 0:

- **`thalach_z` (max heart rate)** – higher exercise capacity is **protective** against heart disease.  
- **`oldpeak_z` (exercise-induced ST depression)** – larger ST depression is associated with **higher risk**.  
- **`ca` (number of vessels)** – more diseased vessels are strongly associated with **higher probability** of heart disease.  

Their 95% credible intervals exclude 0 under all prior settings, highlighting robust clinical relevance. 

<p align="center">
  <img width="750" height="500" alt="Posterior estimates under Normal prior" src="https://github.com/user-attachments/assets/a244f881-08e7-4e91-9a9f-ad25b3f23c8f" />
</p>

### Model Fit (DIC)

Approximate DIC values:

| Prior Specification              |   DIC |
|----------------------------------|------:|
| Flat                             | 383.9 |
| Weakly Informative Normal        | **382.7** |
| Hierarchical Normal              | 383.3 |
| Hierarchical Normal + Age Trend  | 383.2 |

All models fit similarly well, but the **weakly informative Normal prior** delivers the best trade-off between fit and complexity and the cleanest convergence behavior.

---

## Conclusion

This project shows how prior choice and hierarchical structure can meaningfully affect Bayesian logistic regression in a clinical setting. Even though overall model fit (DIC) is similar across priors, the weakly informative Normal prior offers the best balance of stability and flexibility, producing well-converged chains and interpretable coefficients. 

From a practical standpoint, the models consistently highlight clinically sensible risk factors (exercise capacity, ST depression, and number of diseased vessels), illustrating how Bayesian methods can provide both predictive insight and transparent uncertainty quantification for heart disease risk assessment.

---

## Tech Stack

- **Language:** R  
- **Bayesian engine:** JAGS (`rjags`, `coda`)  
- **Data wrangling & visualization:** `dplyr`, `tidyr`, `ggplot2`, `readr`, `janitor`
