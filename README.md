# Tox21 Toxicity Classification (NR-AhR)

## Overview

This project explores machine learning approaches for toxicity prediction using the Tox21 dataset. The goal is to classify whether a molecule is **toxic or non-toxic** based on its ability to activate specific biological receptors.

Toxicity is defined in terms of receptor activation (e.g., a compound activating the Aryl Hydrocarbon Receptor, AhR).

---

## Dataset

* **Source:** [Tox21 dataset](https://tripod.nih.gov/tox21/challenge/data.jsp)
* **Total molecules:** 7,831 (after preprocessing)
* **Bioassays:** 12 toxicity targets
* **Task type:** Binary classification (active vs inactive)

### Data Cleaning

* Removed molecules that could not be processed by `MolFromSmiles` (RDKit)
* Removed missing values for the selected endpoint

### Missing Values (per target)

Some endpoints contain a high percentage of missing values (up to ~26%), which influenced the choice of target.

### Class Imbalance

The dataset is **highly imbalanced**, with toxic compounds often below 10%.

Example (SR-ATAD5):

* Toxic: **3.74%**
* Non-toxic: **96.26%**

For this reason, **PR-AUC (Precision-Recall AUC)** is used as the primary evaluation metric instead of ROC-AUC.

---

## Target Selection

The **NR-AhR** endpoint was selected because:

* Moderate missing data (~16%)
* Relatively higher proportion of positive samples (~12%) compared to other targets
* Biologically meaningful task

---

## Feature Engineering

Three feature representations were explored:

1. **Molecular Descriptors**
2. **Morgan Fingerprints**
3. **Combined Features (Descriptors + Fingerprints)**

---

## Molecular Descriptors Pipeline

### Exploratory Analysis

* Computed **Mutual Information (MI)** scores
* Visualized top 10 features using **violin plots** (grouped by class)
* Identified **94 highly correlated feature pairs**

Conclusion: Feature selection is necessary.

### Preprocessing Steps

1. Train-test split (stratified)
2. Standard scaling (`StandardScaler`)
3. Removal of:

   * Infinite values
   * Constant features (`VarianceThreshold`)
4. Feature selection:

   * `SelectKBest` (based on MI)
   * `RFE` (Recursive Feature Elimination)

> - Dataset with selected molecular descriptors: [desc_X_train_selected](data/processed/mol_descriptors/desc_X_train_selected.csv), [desc_X_test_selected](data/processed/mol_descriptors/desc_X_test_selected.csv),
> - Outputs of selected molecular descriptors dataset: [y_train](data/processed/mol_descriptors/y_train.csv), [y_test](data/processed/mol_descriptors/y_test.csv)
---

## Cross-Validation Analysis

Logistic Regression with stratified K-fold:

### All descriptors

* 3 splits → PR-AUC: 51.89% ± 1.18%
* 5 splits → PR-AUC: 53.78% ± 2.57%
* 10 splits → PR-AUC: 55.94% ± 5.16%

### Selected descriptors

* 3 splits → PR-AUC: 54.55% ± 3.45%
* 5 splits → PR-AUC: 54.98% ± 4.13%
* 10 splits → PR-AUC: 56.32% ± 5.55%

Feature selection improves performance slightly and reduces computation time (~2x faster).

---

## Morgan Fingerprints

* No feature selection applied
* Each bit encodes molecular substructures
* Important information often lies in **combinations of bits**, not individual ones

> - Dataset with morgan fingerprints: [fp_X_train](data/processed/fingerprints/fp_X_train.csv), [fp_X_test](data/processed/fingerprints/fp_X_test.csv)
> - Outputs of morgan fingerprints dataset: [y_train](data/processed/fingerprints/y_train.csv), [y_test](data/processed/fingerprints/y_test.csv)
---

## Mixed Features

* Combination of selected molecular descriptors and morgan fingerprints applied
* Alignment of indices of train/test sets of both features is crucial

> - Dataset with combined features: [X_train](data/processed/mixed/X_train), [X_test](data/processed/mixed/X_test)
> - Outputs of combined features dataset: [y_train](data/processed/mixed/y_train.csv), [y_test](data/processed/mixed/y_test.csv)

---

## Models

Three models were trained:

* Logistic Regression
* Support Vector Machine (SVM)
* Random Forest

### Training Details

* Stratified cross-validation
* Hyperparameter tuning via `GridSearchCV`
* Class imbalance handled using `class_weight="balanced"`

---

## Evaluation

Metrics used:

* **PR-AUC (primary)**
* ROC-AUC
* F1-score
* MCC (Matthews Correlation Coefficient)

### Saved Outputs

For each feature type:

* Precision-Recall curves 
* ROC curves
* Confusion matrices

> Stored in: ([mol_descriptors](results/plots/mol_descriptors), [morgan_fp](results/plots/morgan_fp), [mixed](results/plots/mixed))

---

## Results

Best models:

| Model                              | PR-AUC | ROC-AUC | F1     | MCC    |
| ---------------------------------- | ------ | ------- | ------ | ------ |
| Logistic Regression (mixed)        | 0.7205 | 0.9185  | 0.6097 | 0.5742 |
| Random Forest (mixed)              | 0.7166 | 0.9296  | 0.5182 | 0.5335 |
| SVM (mixed)                        | 0.7133 | 0.9083  | 0.6316 | 0.5833 |
| Random Forest (descriptors)        | 0.6729 | 0.9186  | 0.4413 | 0.4578 |
| SVM (descriptors)                  | 0.6712 | 0.8908  | 0.6133 | 0.5633 |
| Random Forest (fingerprints)       | 0.6687 | 0.9184  | 0.4955 | 0.5022 |
| SVM (fingerprints)                 | 0.6585 | 0.9123  | 0.6350 | 0.5844 |
| Logistic Regression (descriptors)  | 0.6572 | 0.9068  | 0.5288 | 0.4930 |
| Logistic Regression (fingerprints) | 0.6451 | 0.9072  | 0.5924 | 0.5493 |

> Full results available in: [results](results/best_models.csv))

---

## Tools Used

| Category                 | Libraries              |
|--------------------------|------------------------|
| **Data Handling**        | `pandas`, `numpy`      |
| **Visualization**        | `matplotlib`, `seaborn`|
| **Modeling & Evaluation**| `scikit-learn`         |
| **Chemistry**            | `rdkit`                |

---

## Key Insights

* Combining descriptors + fingerprints yields the best performance
* Feature selection on descriptors:

  * Slightly improves performance
  * Significantly reduces computation time
* PR-AUC is crucial due to strong class imbalance
* Morgan fingerprints alone are competitive but benefit from combination

---

## Potential Limitations

* Single endpoint (NR-AhR) — results may not generalize to other targets
* No external validation dataset

---

## Future Work

* Multi-task learning across all 12 endpoints
* Advanced models (e.g., XGBoost, deep learning)
* Handling imbalance with more advanced techniques (SMOTE, focal loss)
* External validation

---

## Reproducibility

To reproduce:

1. Install dependencies (RDKit, scikit-learn, numpy, pandas, matplotlib, seaborn)
2. Run pipeline
3. Check outputs in `results/`

---

## Author

Marco La Rosa

Project developed for the AML Basic Course of the University of Bologna

📧 marco.larosa4@studio.unibo.it

---
