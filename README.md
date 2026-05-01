# Tox21 Toxicity Machine Learning Classification

![Python](https://img.shields.io/badge/Python-3.10%2B-blue?logo=python&logoColor=white)
![RDKit](https://img.shields.io/badge/RDKit-cheminformatics-2ecc71)
![scikit-learn](https://img.shields.io/badge/scikit--learn-ML-orange?logo=scikitlearn)

## Quick Links
- [Technical Report (PDF)](tox21_report.pdf)
- [Processed Datasets](#data-access)
- [Model Performance](#performance-summary)
- [Visualizations Archive](results/plots/)
- [Jupyter Notebook](tox21-toxicity-ML-classification.ipynb)

## Project Overview
This study implements a computational pipeline for toxicity prediction using the Tox21 dataset, focusing on the **NR-AhR** (Aryl Hydrocarbon Receptor) endpoint. The goal is the binary classification of molecules as active (toxic) or inactive (non-toxic) based on their structural properties.

## Data Analysis and Class Distribution

The dataset is highly imbalanced, with toxic (active) samples representing a small fraction of the labeled data and a non-negligible amount of missing values across targets.

For the NR-AhR target, active samples account for ~11.7% of the valid (non-missing) data, with ~16% missing values overall.
This target was selected as a trade-off between data availability and class distribution, and it favors simplicity over more complex approaches such as multi-target learning.

> Detailed distribution plot: [class-distribution.png](results/plots/class-distribution.png)

## Feature Engineering Strategies
Three distinct molecular representation strategies were evaluated to identify the most predictive feature set:

1. Molecular Descriptors: over 200 physicochemical properties were computed from SMILES using RDKit. The pipeline included removal of invalid molecules, filtering of non-finite values, and elimination of constant features using `VarianceThreshold`. Features were then standardized using `StandardScaler`. Due to the high dimensionality of the descriptor space, feature selection was performed using two complementary methods: Mutual Information (`SelectKBest`) and Recursive Feature Elimination (`RFE`), both evaluated through stratified 5-fold cross-validation optimizing PR-AUC. The final feature set was obtained by retaining the intersection of features selected by both methods.
2.  **Morgan Fingerprints:** 2048-bit circular fingerprints. No feature selection was applied as each bit represents a substructure, and the predictive value lies in the **combination of bits** rather than individual features.
3.  **Mixed (Combination):** A hybrid approach concatenating selected descriptors with fingerprints to capture both physical properties and structural motifs.

**Conclusion:** The **Mixed** representation consistently outperformed individual feature sets across all classification metrics.

## Performance Summary
Models were trained using Stratified 5-Fold Cross-Validation with `class_weight="balanced"`.

| Model | Feature Set | PR-AUC | ROC-AUC | F1-Score | MCC |
| :--- | :--- | :---: | :---: | :---: | :---: |
| **Logistic Regression** | **Mixed** | **0.7205** | 0.9185 | 0.6097 | 0.5742 |
| **SVM** | **Mixed** | 0.7133 | 0.9083 | 0.6316 | 0.5833 |
| **Random Forest** | **Mixed** | 0.7166 | **0.9296** | 0.5182 | 0.5335 |
| Random Forest | Descriptors | 0.6729 | 0.9186 | 0.4413 | 0.4578 |
| SVM | Fingerprints | 0.6585 | 0.9123 | **0.6350** | **0.5844** |

> Full results available in: [results](results/best_models.csv)

**Key Findings:**
- **Combination Advantage:** The hybrid approach (Mixed) achieved a significant performance boost over using descriptors or fingerprints in isolation.
- **Top Metrics:** **Logistic Regression (Mixed)** provides the best **PR-AUC**, while **SVM (Mixed)** delivers the highest **F1-Score** and **MCC**, ensuring the most reliable balance for minority class detection.

| Precision-Recall Curve (Top Model) | Confusion Matrix (Best Balance) |
| :---: | :---: |
| ![PR Curve](results/plots/mixed/pr_curve_Logistic%20Regression.png) | ![Confusion Matrix](results/plots/mixed/confusion_matrix_Support%20Vector%20Machine.png) |

## Data Access
Direct links to the generated training and testing sets:

| Representation | Training Set | Testing Set |
| :--- | :--- | :--- |
| **Molecular Descriptors** | [desc_X_train_selected.csv](data/processed/mol_descriptors/desc_X_train_selected.csv) | [desc_X_test_selected.csv](data/processed/mol_descriptors/desc_X_test_selected.csv) |
| **Morgan Fingerprints** | [fp_X_train.csv](data/processed/fingerprints/fp_X_train.csv) | [fp_X_test.csv](data/processed/fingerprints/fp_X_test.csv) |
| **Mixed (Combined)** | [X_train.csv](data/processed/mixed/X_train.csv) | [X_test.csv](data/processed/mixed/X_test.csv) |

## Project Structure
```text
.
├── data/
│   ├── raw/                                      # Original Tox21 dataset
│   └── processed/                                # Engineered datasets
├── results/
│   ├── models/                                   # Serialized trained models (.joblib)
│   ├── plots/                                    # Visualizations
│   ├── best_models.csv                           # Global metrics report
├── tox21-toxicity-ML-classification.ipynb        # Main analysis and pipeline notebook
├── tox21_report.pdf                              # Detailed technical report
└── README.md                                     # Project documentation
```

## Tools Used

| Category                 | Libraries              |
|--------------------------|------------------------|
| **Data Handling**        | `pandas`, `numpy`      |
| **Visualization**        | `matplotlib`, `seaborn`|
| **Modeling & Evaluation**| `scikit-learn`         |
| **Chemistry**            | `rdkit`                |

---

## Future Work

* Threshold tuning
* Multi-task learning across all 12 endpoints
* Advanced models (e.g., XGBoost, deep learning)
* Handling imbalance with more advanced techniques (SMOTE, focal loss)
* External validation

---

## Reproducibility
To replicate the analysis, execute the following commands:
```bash
pip install notebook pandas numpy rdkit scikit-learn matplotlib seaborn joblib
jupyter notebook tox21-toxicity-ML-classification.ipynb
```
The pipeline automatically handles data processing, feature selection, and model evaluation, storing all outputs in the `results/` and `data/processed/` directories.

---

## Author
**Marco La Rosa**  
University of Bologna - AML Basic Course  
Email: [marco.larosa4@studio.unibo.it](mailto:marco.larosa4@studio.unibo.it)

---
*Just someone having fun with chemistry and machine learning*
