# Wellhead Choke ML

## Prediction of Two-Phase Flow Rate Through Wellhead Chokes in Oil Wells

This project investigates the use of machine learning regression models to predict the **gross liquid flow rate through wellhead chokes in oil wells**.

In oil production systems, individual wells may not always have dedicated flow meters because installing and maintaining flow-measurement equipment can be expensive. Estimating flow rate from measurements already available around the wellhead can therefore be useful for production monitoring and analysis.

The objective of this project is to build and compare multiple regression models and determine how accurately they can estimate the **gross liquid flow rate (QL)**.

The project follows the general methodology of the Stanford CS229 project *Prediction of Two-Phase Flow Rate Through Wellhead Chokes in Oil Wells*. However, the original `RawData.xlsx` dataset was not publicly available, so this implementation uses a publicly available **Sorush Oil Field dataset** as a replacement. Therefore, the results in this repository should be considered an independent implementation inspired by the original methodology rather than an exact reproduction of the original experiment.

---

## 1. Problem Statement

The main problem addressed by this project is:

> Can machine learning models accurately predict the gross liquid flow rate through an oil-well wellhead choke using measurements available at the wellhead?

The target variable is:

```text
QL = Gross Liquid Flow Rate
```

measured in:

```text
barrels/day (bbl/day)
```

Since QL is continuous, the problem is formulated as a **supervised regression problem**.

---

## 2. Background

A wellhead choke is used to control the flow of fluids produced from an oil well.

The produced fluid can contain multiple phases, including:

- Oil
- Water
- Gas

Because the flow is multiphase, accurately estimating total liquid flow rate is more difficult than measuring a single-phase flow.

Traditional empirical correlations can be used to estimate flow rate from pressure, choke size, and fluid properties. One of the correlations considered in the original project is the **Gilbert correlation**.

A simplified Gilbert-type relationship can be represented as:

```text
QL = A × P^D × S^B / GLR^C
```

where:

- `QL` = liquid flow rate
- `P` = upstream pressure
- `S` = choke size
- `GLR` = gas-liquid ratio
- `A, B, C, D` = empirical constants

Although empirical correlations can be useful, their performance can depend on operating conditions and the dataset on which they are applied.

This motivates the use of machine learning models that can learn nonlinear relationships between wellhead measurements and observed flow rate.

---

## 3. Dataset

This implementation uses the publicly available **Sorush Oil Field** dataset.

### Dataset Size

Initial dataset:

```text
7,245 samples
```

After preprocessing and 3σ outlier removal:

```text
6,883 samples
```

The dataset contains measurements associated with multiple wells in the Sorush Oil Field.

---

## 4. Input Features

The public replacement dataset provides the following variables used in this project:

| Variable | Description |
|---|---|
| `Pwh` | Wellhead pressure |
| `D64` | Choke-related D64 measurement |
| `GLR` | Gas-liquid ratio |
| `QL` | Gross liquid flow rate — prediction target |

The primary model configuration uses:

```text
Pwh
D64
GLR
```

to predict:

```text
QL
```

---

## 5. Feature Configurations

Multiple feature configurations were considered to investigate the importance of pressure-related information.

### Model A — Full Available Feature Set

```text
Pwh
D64
GLR
```

This is the main model configuration using the relevant variables available in the replacement dataset.

### Model B — Pressure Removed

```text
D64
GLR
```

Wellhead pressure is removed to investigate how much predictive information remains without directly using `Pwh`.

### Model C — Pressure-Independent Configuration

```text
D64
GLR
```

For the public replacement dataset, Models B and C are identical because the additional variables required to reproduce the original Model C formulation are not available.

---

## 6. Data Preprocessing

Before model training, the dataset was inspected for:

- Missing values
- Duplicate records
- Data types
- Unique values
- Well distribution
- Feature ranges
- Target distribution
- Potential outliers

### 6.1 Outlier Removal

A **3σ outlier filtering approach** was used.

For a numerical variable:

```text
Lower limit = mean - 3 × standard deviation
Upper limit = mean + 3 × standard deviation
```

Observations outside the selected range were removed.

This reduced the dataset from:

```text
7,245 samples
```

to:

```text
6,883 samples
```

The purpose of this step is to reduce the influence of extreme observations that could disproportionately affect regression models.

---

## 7. Exploratory Data Analysis

Exploratory data analysis was performed before model training to understand:

- Feature distributions
- Target distribution
- Relationships between variables
- Possible outliers
- Correlations between variables
- Well-level sample distribution

The notebook contains the generated exploratory plots and statistical summaries.

The analysis was used to understand the structure of the dataset before applying machine learning models.

---

## 8. Gilbert Baseline

A Gilbert-type empirical correlation was implemented as a baseline for comparison with the machine-learning models.

For the replacement dataset, the correlation was adapted to the variables that are actually available:

```text
QL_pred = 0.1 × Pwh × D64^1.89 / GLR^0.546
```

This is an **adapted baseline for the replacement dataset**, rather than a claim that the original Stanford experiment used exactly this implementation on the same data.

The purpose of including the baseline is to provide a traditional correlation against which the machine-learning approaches can be compared.

---

## 9. Machine Learning Models

Ten regression approaches were evaluated.

### 9.1 Linear Regression

Linear Regression provides a simple baseline by assuming that the target can be represented as a linear combination of the input variables.

### 9.2 Ridge Regression

Ridge Regression adds L2 regularization to Linear Regression, helping control large model coefficients and improve stability.

### 9.3 Bayesian Ridge

Bayesian Ridge uses a Bayesian formulation of linear regression and provides another regularized linear baseline.

### 9.4 Polynomial Linear Regression

Polynomial features allow the model to represent nonlinear relationships and feature interactions.

### 9.5 Polynomial Ridge Regression

Polynomial features are combined with Ridge regularization to control the complexity introduced by the polynomial expansion.

### 9.6 MLP Regression

A Multi-Layer Perceptron is a neural-network-based regression model capable of learning nonlinear relationships.

### 9.7 K-Nearest Neighbors

KNN predicts a sample using the target values of nearby observations in feature space.

### 9.8 Random Forest

Random Forest combines multiple decision trees and averages their predictions to reduce variance and model nonlinear relationships.

### 9.9 Gradient Tree Boosting

Gradient Tree Boosting builds trees sequentially, with later trees attempting to correct errors made by earlier trees.

### 9.10 Extra Trees

Extra Trees is an ensemble of randomized decision trees. It can model complex nonlinear relationships and interactions between features.

---

## 10. Experimental Workflow

The complete project workflow is:

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
Residual Analysis
      ↓
Feature Importance Analysis
```

The project uses a fixed random seed:

```text
42
```

This improves reproducibility of the experiments.

---

## 11. Initial Model Comparison

The initial 80/20 experiment compared all ten regression models.

For Model A, KNN achieved the highest initial test R²:

```text
R² = 0.953972
```

Other strong models included Random Forest, Gradient Tree Boosting, and Extra Trees.

However, the project does not select the final model simply by choosing the model with the highest single test-set score. Instead, training-only cross-validation is used to select the strongest candidate before final tuning.

This reduces the risk of selecting a model based on one particular test split.

---

## 12. Normalized Target Experiment

A separate experiment was performed using a pressure-normalized target:

```text
PressureNormalizedTarget = QL / Pwh
```

The purpose was to investigate whether removing the direct scale effect of pressure from the target could improve prediction performance.

The best result in the normalized-target screening was obtained using KNN:

```text
R² = 0.968349
```

This is a different prediction problem from predicting raw QL. Therefore, the normalized-target R² should not be directly compared with the final raw-QL test R².

---

## 13. Top Model Selection

The strongest candidate models were selected using **5-fold cross-validation on the training data only**.

The results were:

| Rank | Model | Mean CV R² | CV Std |
|---:|---|---:|---:|
| 1 | Extra Trees | 0.961575 | 0.015749 |
| 2 | KNN | 0.960948 | 0.017526 |
| 3 | Random Forest | 0.959699 | 0.016857 |
| 4 | Gradient Tree Boosting | 0.950392 | — |

The top three models were therefore:

1. Extra Trees
2. KNN
3. Random Forest

Extra Trees was selected for further hyperparameter tuning because it achieved the highest mean cross-validation R².

---

## 14. Hyperparameter Tuning

Extra Trees was tuned using `RandomizedSearchCV` with 5-fold cross-validation.

The search considered parameters controlling:

- Number of trees
- Maximum tree depth
- Minimum samples required for a split
- Minimum samples per leaf
- Number of features considered by each tree

### Best Parameters

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

The final test set was not used to select these hyperparameters.

---

## 15. Final Model

The final selected model is:

**Tuned Extra Trees Regressor**

The final model was trained using the selected training data and evaluated on held-out test and validation sets.

### Final Test Performance

| Metric | Result |
|---|---:|
| **R²** | **0.948751** |
| **MAE** | **220.25 bbl/day** |
| **MSE** | **518,540.94** |
| **RMSE** | **720.10 bbl/day** |
| **Correlation** | **0.974067** |
| **MAPE** | **3.58%** |

The final test R² of approximately **0.949** indicates that the model explains a large proportion of the variation in the held-out gross liquid flow-rate observations.

The MAE of approximately **220 bbl/day** represents the average absolute difference between predicted and observed flow rate in the test set.

---

## 16. Train, Test and Validation Performance

For the tuned Extra Trees model, the results were:

| Dataset | R² |
|---|---:|
| Training | **0.973154** |
| Test | **0.948751** |
| Validation | **0.963289** |

The training score is higher than the held-out scores, which is expected for a flexible tree ensemble.

The important metric for final model reporting is the held-out test performance:

```text
Test R² = 0.948751
```

---

## 17. Feature Importance

The tuned Extra Trees model provides feature-importance values.

| Feature | Importance |
|---|---:|
| `GLR` | **0.453755** |
| `D64` | **0.320018** |
| `Pwh` | **0.226227** |

The model therefore identified `GLR` as the most important feature among the three available inputs.

The approximate contribution ranking is:

```text
GLR  >  D64  >  Pwh
```

These are model-based importance values and should not be interpreted as proof of causal relationships.

---

## 18. Model Diagnostics

Several diagnostic plots were generated for the final Extra Trees model.

### Predicted vs Actual

The predicted-versus-actual plot is used to determine how closely the model predictions follow the observed flow rates.

A strong model should produce predictions concentrated around the ideal:

```text
Predicted = Actual
```

line.

### Residual Analysis

Residuals are calculated as:

```text
Residual = Actual - Predicted
```

Residual plots help identify systematic prediction errors and possible regions where the model performs poorly.

### Residual Distribution

The residual distribution is examined to determine whether errors are approximately centered around zero and whether large systematic deviations are present.

### Absolute Error Distribution

Absolute errors are examined to understand the typical magnitude of prediction errors.

### Feature Importance

Feature importance provides an interpretation of which available input variables contributed most strongly to the fitted Extra Trees model.

The final repository contains the generated diagnostic figures.

---

## 19. Why Extra Trees?

Extra Trees was selected because it achieved the strongest mean cross-validation performance among the screened models.

The model is also suitable for this problem because the relationship between:

```text
Pwh
D64
GLR
```

and:

```text
QL
```

is unlikely to be purely linear.

Tree-based ensemble methods can capture:

- Nonlinear relationships
- Feature interactions
- Threshold effects
- Complex relationships without requiring an explicit mathematical equation

The final Extra Trees model therefore provides a strong balance between predictive performance and interpretability through feature importance.

---

## 20. Project Structure

The recommended repository structure is:

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

The repository does not require a saved trained-model file because the notebook contains the complete training and hyperparameter-tuning workflow.

---

## 21. How to Run the Project

### Step 1 — Clone the repository

```bash
git clone <YOUR-GITHUB-REPOSITORY-URL>
cd wellhead-choke-ml
```

### Step 2 — Create a virtual environment

```bash
python -m venv .venv
```

### Step 3 — Activate the environment

On macOS/Linux:

```bash
source .venv/bin/activate
```

On Windows:

```bash
.venv\Scriptsctivate
```

### Step 4 — Install dependencies

```bash
pip install -r requirements.txt
```

### Step 5 — Open Jupyter Notebook

```bash
jupyter notebook
```

Open:

```text
notebook/Wellhead_Choke_ML_PROJECT_FINAL.ipynb
```

Run the notebook from top to bottom.

---

## 22. Evaluation Metrics

The project uses several regression metrics.

### R² — Coefficient of Determination

R² measures how much of the variation in the target is explained by the model.

Higher values generally indicate better predictive performance.

### MAE — Mean Absolute Error

MAE represents the average absolute difference between predicted and actual flow rates.

```text
MAE = mean(|Actual - Predicted|)
```

It is reported in bbl/day.

### MSE — Mean Squared Error

MSE calculates the average squared prediction error.

Large errors receive greater weight because the residuals are squared.

### RMSE — Root Mean Squared Error

RMSE is the square root of MSE.

It is also expressed in bbl/day and penalizes large errors more strongly than MAE.

### Correlation

Correlation measures the strength of the relationship between predicted and actual values.

### MAPE — Mean Absolute Percentage Error

MAPE expresses prediction error as a percentage of the actual value.

---

## 23. Limitations

### 23.1 Replacement Dataset

The original Stanford CS229 `RawData.xlsx` dataset was not publicly available.

Therefore, the numerical results in this repository are based on the public Sorush Oil Field replacement dataset.

The results should **not** be presented as an exact reproduction of the original Stanford CS229 experiment.

### 23.2 Different Available Variables

The replacement dataset does not contain all variables used in the original project, including variables such as:

- Downstream pressure
- Temperature
- Water cut
- GOR
- Explicit flow-regime information
- Original field/group labels

Consequently, some of the original feature-engineering experiments cannot be reproduced exactly.

### 23.3 Adapted Gilbert Baseline

The Gilbert-form baseline was adapted to the variables available in the replacement dataset.

Its result should therefore be interpreted as an adapted baseline and not as the original Gilbert-correlation result reported for the original dataset.

### 23.4 Models B and C

Models B and C are identical in this implementation because the public replacement dataset does not provide the additional variables needed to reproduce the original Model C definition.

### 23.5 Generalization

The dataset represents a specific oil-field environment.

A model that performs well on this dataset may not automatically generalize to wells from other fields.

Testing on additional fields and previously unseen wells would be required before considering production deployment.

---

## 24. Future Work

Several improvements could be investigated in future work.

### Additional Features

Future datasets could include:

- Upstream pressure
- Downstream pressure
- Temperature
- Water cut
- GOR
- Fluid properties
- Flow-regime indicators
- Additional choke measurements

### Cross-Field Generalization

The model could be evaluated on completely different oil fields to determine whether it has learned general flow relationships rather than field-specific patterns.

### Well-Wise Validation

Instead of randomly splitting individual observations, future experiments could hold out entire wells.

This would provide a stronger test of whether the model can generalize to a well that was not represented during training.

### Additional Machine Learning Models

Future work could compare:

- XGBoost
- LightGBM
- CatBoost
- Support Vector Regression
- More advanced neural networks

### Uncertainty Estimation

A production system would benefit from knowing not only the predicted flow rate but also how confident the model is in that prediction.

Prediction intervals or uncertainty-aware models could therefore be investigated.

### Deployment

The final model could eventually be deployed as:

- A REST API
- A production-monitoring dashboard
- A real-time prediction service
- An engineering decision-support tool

---

## 25. References

1. Nazari, S. and Alshafloot, M. (2019). *Prediction of Two-Phase Flow Rate Through Wellhead Chokes in Oil Wells*. Stanford CS229 Project.

2. Nazari, S. and Alshafloot, M. (2019). *CS229 Project Implementation*. Stanford CS229 GitHub Repository.

3. Barjouei, H.S. et al. (2021). *Prediction performance advantages of deep machine learning algorithms for two-phase flow rates through wellhead chokes*. Journal of Petroleum Exploration and Production Technology, 11, 1233–1261.

4. Scikit-learn documentation for the regression algorithms and model-selection techniques used in this project.

---

## 26. Project Summary

| Category | Details |
|---|---|
| **Project** | Wellhead Choke ML |
| **Problem** | Two-phase flow-rate prediction |
| **Task** | Supervised regression |
| **Target** | Gross Liquid Flow Rate (QL) |
| **Target Unit** | bbl/day |
| **Dataset** | Sorush Oil Field |
| **Initial Samples** | 7,245 |
| **Cleaned Samples** | 6,883 |
| **Input Features** | Pwh, D64, GLR |
| **Models Tested** | 10 regression models |
| **Top CV Model** | Extra Trees |
| **Final Model** | Tuned Extra Trees Regressor |
| **Best CV R²** | **0.962511** |
| **Test R²** | **0.948751** |
| **Test MAE** | **220.25 bbl/day** |
| **Test RMSE** | **720.10 bbl/day** |
| **Test Correlation** | **0.974067** |
| **Test MAPE** | **3.58%** |
| **Random Seed** | **42** |

---

## Conclusion

This project demonstrates that machine-learning regression models can effectively estimate gross liquid flow rate from available wellhead choke measurements in the selected public dataset.

Ten regression approaches were compared, with tree-based ensemble methods and KNN generally providing stronger predictive performance than the simpler linear models.

Using training-only cross-validation, **Extra Trees achieved the highest mean CV R² of 0.961575** among the screened models. After hyperparameter tuning, the final Extra Trees model achieved a **test R² of 0.948751**, with an MAE of approximately **220 bbl/day** and a correlation of **0.974067**.

The results demonstrate the potential of machine learning as an alternative approach for estimating wellhead flow rates. However, further validation on additional wells, fields, and operating conditions would be necessary before using such a model for real-world production decisions.
