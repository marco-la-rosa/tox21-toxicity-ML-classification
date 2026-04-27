# Tox21 Toxicity Machine Learning Classification

## Quick Links
- [Processed Datasets](#data-access)
- [Model Performance](#performance-summary)
- [Visualizations Archive](results/plots/)
- [Jupyter Notebook](tox21-toxicity-ML-classification.ipynb)

## Project Overview
This study implements a computational pipeline for toxicity prediction using the Tox21 dataset, focusing on the **NR-AhR** (Aryl Hydrocarbon Receptor) endpoint. The goal is the binary classification of molecules as active (toxic) or inactive (non-toxic) based on their structural properties.

## Data Analysis and Class Distribution
The dataset exhibits significant class imbalance. For the NR-AhR target, active samples constitute approximately **12%** of the total. This endpoint was selected for its biological significance and relatively higher positive sample rate compared to other targets.

Detailed distribution plot: [class-distribution.png](results/plots/class-distribution.png)

## Feature Engineering Strategies
Three distinct molecular representation strategies were evaluated to identify the most predictive feature set:

1.  **Molecular Descriptors:** 200+ physicochemical properties. The pipeline included cleaning invalid SMILES, filtering infinite features, scaling, `VarianceThreshold` for constant features, and optimization via `SelectKBest` and **Recursive Feature Elimination (RFE)**.
2.  **Morgan Fingerprints:** 2048-bit circular fingerprints. No feature selection was applied as each bit represents a substructure, and the predictive value lies in the **combination of bits** rather than individual features.
3.  **Mixed (Combination):** A hybrid approach concatenating selected descriptors with fingerprints to capture both physical properties and structural motifs.

**Conclusion:** The **Mixed** representation consistently outperformed individual feature sets across all classification metrics.

## Performance Summary
Models were trained using Stratified K-Fold Cross-Validation with `class_weight="balanced"`.

| Model | Feature Set | PR-AUC | ROC-AUC | F1-Score | MCC |
| :--- | :--- | :---: | :---: | :---: | :---: |
| **Logistic Regression** | **Mixed** | **0.7205** | 0.9185 | 0.6097 | 0.5742 |
| **SVM** | **Mixed** | 0.7133 | 0.9083 | **0.6316** | **0.5833** |
| **Random Forest** | **Mixed** | 0.7166 | **0.9296** | 0.5182 | 0.5335 |
| Random Forest | Descriptors | 0.6729 | 0.9186 | 0.4413 | 0.4578 |
| SVM | Fingerprints | 0.6585 | 0.9123 | 0.6350 | 0.5844 |

> Full results available in: [results](results/best_models.csv))

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
│   ├── raw/                # Raw Tox21 data
│   └── processed/          # Engineered datasets (Descriptors, Fingerprints, Mixed)
├── results/
│   ├── plots/              # ROC, PR, CM, and Distribution plots
│   ├── models/             # Serialized .joblib models
│   └── best_models.csv     # Global metrics report
└── tox21-toxicity-ML-classification.ipynb
```

## Tools Used

| Category                 | Libraries              |
|--------------------------|------------------------|
| **Data Handling**        | `pandas`, `numpy`      |
| **Visualization**        | `matplotlib`, `seaborn`|
| **Modeling & Evaluation**| `scikit-learn`         |
| **Chemistry**            | `rdkit`                |

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
To replicate the analysis, execute the following commands:
```bash
pip install pandas numpy rdkit scikit-learn matplotlib seaborn joblib
jupyter notebook tox21-toxicity-ML-classification.ipynb
```
The pipeline automatically handles data processing, feature selection, and model evaluation, storing all outputs in the `results/` directory.

---

## Author
**Marco La Rosa**  
University of Bologna - AML Basic Course  
Email: [marco.larosa4@studio.unibo.it](mailto:marco.larosa4@studio.unibo.it)

---
*Developed with ❤️ for Computational Medicinal Chemistry*
