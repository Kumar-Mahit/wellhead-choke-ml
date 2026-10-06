# Prediction of Two-Phase Flow Rate Through Wellhead Chokes in Oil Wells

## Project Overview

This project uses machine learning to predict the **gross liquid flow rate (QL)** through oil-well wellhead chokes.

Accurate flow-rate estimation is useful because installing and maintaining individual flow meters for every well can be expensive. The goal of this project is to investigate whether machine-learning models can estimate flow rate from measurements available around the wellhead choke.

The project follows the general experimental approach of a Stanford CS229 project on two-phase flow through wellhead chokes, while using a **publicly available replacement dataset** because the original `RawData.xlsx` dataset was not publicly available.

---

## Problem Statement

Given wellhead/choke measurements, predict the continuous target:

**QL — Gross Liquid Flow Rate (bbl/day)**

Since the target is continuous, this is formulated as a **supervised regression problem**.

---

## Dataset

The project uses a public **Sorush Oil Field** dataset containing:

- **7,245 records initially**
- **6,883 records after 3σ outlier filtering**

### Features used

| Feature | Description |
|---|---|
| `Pwh` | Wellhead pressure |
| `D64` | Choke-related D64 measurement |
| `GLR` | Gas-liquid ratio |
| `QL` | Gross liquid flow rate — target |

### Feature configurations

**Model A**
```text
Pwh, D64, GLR
```

**Model B**
```text
D64, GLR
```

**Model C**
```text
D64, GLR
```

Models B and C are identical in this experiment because the public replacement dataset does not contain all of the additional variables required to reproduce the original Model C definition.

---

## Methodology

The project follows this workflow:

```text
Dataset Loading
      ↓
Data Audit
      ↓
Exploratory Data Analysis
      ↓
3σ Outlier Removal
      ↓
Feature Engineering
      ↓
Train/Test Split
      ↓
10 Regression Models
      ↓
Model A/B/C Comparison
      ↓
Normalized-Target Experiment
      ↓
Training-Only Cross-Validation
      ↓
Top 3 Model Selection
      ↓
Extra Trees Hyperparameter Tuning
      ↓
Final Evaluation
      ↓
Residual & Feature-Importance Analysis
```

### Data preprocessing

A **3σ rule** was used to remove extreme observations from the selected numerical features and target.

The experiment uses **random seed 42**.

For scale-sensitive models, scaling is fitted using the training data rather than the complete dataset.

---

## Models Evaluated

Ten regression approaches were compared:

1. Linear Regression
2. Ridge Regression
3. Bayesian Ridge
4. Polynomial Linear Regression
5. Polynomial Ridge Regression
6. MLP Regression
7. K-Nearest Neighbors (KNN)
8. Random Forest
9. Gradient Tree Boosting
10. Extra Trees

An adapted **Gilbert-form baseline** was also evaluated for comparison.

---

## Model Selection

The initial 80/20 screening showed strong performance from nonlinear models.

The three strongest candidates were selected using **5-fold cross-validation on the training data only**:

| Rank | Model | Mean CV R² |
|---:|---|---:|
| 1 | Extra Trees | 0.961575 |
| 2 | KNN | 0.960948 |
| 3 | Random Forest | 0.959699 |

The final model was selected as **Extra Trees** because it achieved the best cross-validation performance and provides useful feature-importance information.

---

## Hyperparameter Tuning

Extra Trees was tuned using `RandomizedSearchCV` with 5-fold cross-validation.

### Best parameters

```text
n_estimators = 200
max_depth = 40
min_samples_split = 10
min_samples_leaf = 2
max_features = 1.0
```

Best cross-validation R²:

```text
0.962511
```

The final test set was kept separate from the hyperparameter-selection process.

---

## Final Results

The tuned Extra Trees model achieved:

| Metric | Test Result |
|---|---:|
| **R²** | **0.948751** |
| **MAE** | **220.25 bbl/day** |
| **RMSE** | **720.10 bbl/day** |
| **Correlation** | **0.974067** |
| **MAPE** | **3.58%** |

### Interpretation

An R² of approximately **0.949** means that the final model explains about 94.9% of the variance in the held-out QL values in this experiment.

The average absolute prediction error is approximately **220 bbl/day**.

---

## Feature Importance

The tuned Extra Trees model produced the following feature-importance values:

| Feature | Importance |
|---|---:|
| `GLR` | **0.453755** |
| `D64` | **0.320018** |
| `Pwh` | **0.226227** |

GLR was the most important feature according to the fitted Extra Trees model.

These values represent model-based feature importance and should **not** be interpreted as causal effects.

---

## Normalized Target Experiment

A separate experiment predicted:

```text
PressureNormalizedTarget = QL / Pwh
```

The best normalized-target screening result was obtained by KNN:

```text
R² = 0.968349
```

These results are reported separately because the normalized target is a different prediction problem from raw QL. Therefore, the normalized-target R² should not be directly compared with the final raw-QL test R².

---

## Project Structure

The recommended GitHub repository structure is:

```text
wellhead-choke-ml/
│
├── README.md
├── requirements.txt
│
├── notebook/
│   └── Wellhead_Choke_ML_PROJECT_FINAL.ipynb
│
├── src/
│   └── Python source code
│
├── data/
│   └── dataset
│
├── figures/
│   ├── EDA/
│   ├── model_comparison/
│   └── diagnostics/
│
└── report/
    └── Wellhead_Choke_ML_Project_Report.pdf
```

A saved trained model is not included in the repository. The notebook contains the complete training and tuning workflow needed to reproduce the model.

---

## How to Run

### 1. Clone the repository

```bash
git clone <YOUR-GITHUB-REPOSITORY-URL>
cd wellhead-choke-ml
```

### 2. Create a virtual environment

```bash
python -m venv .venv
```

Activate it on macOS/Linux:

```bash
source .venv/bin/activate
```

On Windows:

```bash
.venv\Scripts\activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Open the notebook

```bash
jupyter notebook
```

Open:

```text
notebook/Wellhead_Choke_ML_PROJECT_FINAL.ipynb
```

Run the notebook from top to bottom.

---

## Evaluation Metrics

The project reports:

- **R²** — coefficient of determination
- **MAE** — mean absolute error
- **MSE** — mean squared error
- **RMSE** — root mean squared error
- **Correlation** — correlation between predictions and actual values
- **MAPE** — mean absolute percentage error

---

## Limitations

### 1. Replacement dataset

The original Stanford CS229 `RawData.xlsx` dataset was not publicly available. Therefore, the numerical results in this repository are from the public Sorush Oil Field replacement dataset.

They should **not** be presented as an exact reproduction of the original CS229 experiment.

### 2. Different available variables

The replacement dataset does not contain all variables used by the original project, including variables such as downstream pressure, temperature, water cut, GOR and explicit flow-regime information.

Therefore, some of the original feature-engineering experiments cannot be reproduced exactly.

### 3. Adapted Gilbert baseline

The Gilbert-form baseline was adapted to the available public variables. Its result is therefore a project baseline rather than the original Gilbert correlation result.

### 4. Model B and Model C

Models B and C are identical for the replacement dataset because the additional variables required for the original Model C formulation are unavailable.

### 5. Generalization

The dataset represents a particular oil-field environment. Additional validation on different wells and fields would be required before using the model in a production environment.

---

## Future Work

Possible future improvements include:

- Testing the model on additional oil fields
- Adding downstream pressure and fluid-property variables
- Performing well-wise/group-wise validation
- Testing generalization to previously unseen wells
- Comparing additional boosting algorithms
- Adding prediction uncertainty estimates
- Investigating domain-informed feature engineering
- Deploying the trained model as an API or monitoring tool

---

## References

1. Nazari, S. and Alshafloot, M. (2019). *Prediction of Two-Phase Flow Rate Through Wellhead Chokes in Oil Wells*. Stanford CS229 project report.

2. Nazari, S. and Alshafloot, M. (2019). *CS229 Project Implementation*. Stanford CS229 GitHub repository.

3. Barjouei, H.S. et al. (2021). *Prediction performance advantages of deep machine learning algorithms for two-phase flow rates through wellhead chokes*. Journal of Petroleum Exploration and Production Technology, 11, 1233–1261. DOI: 10.1007/s13202-021-01087-4.

4. Scikit-learn documentation for the regression algorithms and model-selection tools used in this project.

---

## Project Summary

**Problem:** Predict gross liquid flow rate through oil-well wellhead chokes.

**Dataset:** 7,245 public Sorush Oil Field samples.

**Cleaned dataset:** 6,883 samples.

**Models tested:** 10 regression algorithms.

**Final model:** Tuned Extra Trees Regressor.

**Final test R²:** **0.948751**

**Final test RMSE:** **720.10 bbl/day**

**Final test MAE:** **220.25 bbl/day**

**Final test correlation:** **0.974067**

**Best CV R²:** **0.962511**

**Random seed:** **42**
