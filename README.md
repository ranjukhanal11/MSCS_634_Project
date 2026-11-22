# MSCS_634_Project
**Advanced Data Mining for Data-Driven Insights and Predictive Modeling**

**Course:** MSCS-634-M50 — Advanced Big Data and Data Mining  
**Institution:** University of the Cumberlands  
**Instructor:** Professor Satish Penmatsa  
**Authors:** Ranju Khanal, Anusha Sharma  
**Term:** Fall 2026

---

## Overview

This repository contains the complete four-deliverable data-mining project for
MSCS-634. The project builds a full analytical pipeline — from data quality
assurance through regression, classification, clustering, and association-rule
mining — on the **Breast Cancer Wisconsin (Diagnostic)** dataset. Each
deliverable adds a modeling paradigm, and the final deliverable consolidates
results, presents integrated insights, and discusses practical recommendations,
ethical considerations, and future directions.

The overarching theme is **disciplined, leakage-aware analytics**: careful
cleaning, stratified sampling, pipeline-contained preprocessing, minority-class
metrics, and honest reporting of limitations. Every technical decision is
justified both statistically and by its clinical plausibility.

---

## Dataset

- **Name:** Breast Cancer Wisconsin (Diagnostic) — WDBC
- **Source:** UCI Machine Learning Repository  
  https://archive.ics.uci.edu/dataset/17/breast+cancer+wisconsin+diagnostic
- **Reproducible access:** `sklearn.datasets.load_breast_cancer(as_frame=True)`
- **Size:** 569 observations × 30 numerical features × 1 binary diagnosis label
- **Subset used:** **10 mean nuclear measurements + diagnosis**
  - `mean radius`, `mean texture`, `mean perimeter`, `mean area`,
    `mean smoothness`, `mean compactness`, `mean concavity`,
    `mean concave points`, `mean symmetry`, `mean fractal dimension`
- **Class balance:** 357 benign (62.7%) / 212 malignant (37.3%) — imbalanced
- **Origin:** Digitized fine-needle aspirate (FNA) images of breast masses
  (Street, Wolberg, & Mangasarian, 1993)

**Why this dataset:** It is well documented, publicly available, meets the
project's ≥8 attributes / ≥500 records requirement, and supports both
supervised and unsupervised analysis without requiring external outcome
linkage. Verified diagnosis labels enable honest evaluation.

---

## Repository Layout

```text
MSCS_634_Project/
├── README.md                                  
├── requirements.txt
├── deliverables/
│   ├── Deliverable_1_Data_Cleaning_EDA.ipynb
│   ├── Deliverable_2_Regression_Modeling.ipynb
│   ├── Deliverable_3_Classification_Clustering_Rules.ipynb
│   └── Deliverable_4_Final_Report.pdf
├── data/
│   └── wdbc_cleaned.csv
├── figures/
│   ├── 01_diagnosis_distribution.png
│   ├── 02_radius_area_histograms.png
│   ├── 03_concave_points_by_diagnosis.png
│   ├── 04_feature_correlation_heatmap.png
│   ├── 05_radius_area_scatter.png
│   ├── 06_predicted_vs_actual.png
│   ├── 07_cv_r2_boxplot.png
│   ├── 08_decision_tree_confusion.png
│   ├── 09_knn_confusion.png
│   ├── 10_roc_curves.png
│   ├── 11_feature_importances.png
│   ├── 12_kmeans_pca.png
│   ├── 13_silhouette_elbow.png
│   └── 14_association_rules.png
├── presentation/
│   ├── Deliverable_4_Presentation.pptx
│   └── generate_presentation.py
└── reports/
    ├── Deliverable_1_Report.pdf
    ├── Deliverable_2_Report.pdf
    ├── Deliverable_3_Report.pdf
    └── Deliverable_4_Final_Report.pdf
```

---

## How to Run

1. Install Python 3.10+ and Jupyter Notebook / JupyterLab.
2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```
3. Open any deliverable notebook in `deliverables/` and select **Run All**.
4. The notebooks load the dataset via `sklearn.datasets.load_breast_cancer`
   (no external download required) and save cleaned CSV and figures
   automatically to `data/` and `figures/`.
5. To regenerate the presentation slides:
   ```bash
   python presentation/generate_presentation.py
   ```

**`requirements.txt`:**
```
numpy>=1.24
pandas>=2.0
matplotlib>=3.7
seaborn>=0.12
scikit-learn>=1.3
mlxtend>=0.22
jupyter>=1.0
python-pptx>=0.6.21
```

---

## Project Steps

### Deliverable 1 — Data Collection, Cleaning, and Exploration
- Loaded the 569-record dataset, selected 10 mean measurements + diagnosis
- Performed quality audit: **zero missing values, zero duplicates, zero
  invalid negatives**
- Screened outliers using the 1.5 × IQR rule but **deliberately retained**
  flagged observations (extreme tumor morphology is real signal, not data error)
- Visualized class distribution, feature histograms, class-wise boxplots,
  correlation heatmap, and radius-vs-area scatterplot
- **Key finding:** extreme multicollinearity — radius ↔ perimeter r ≈ 0.998;
  radius ↔ area r ≈ 0.987

### Deliverable 2 — Regression Modeling and Performance Evaluation
- **Primary target:** `mean area` (excluded from features to prevent leakage)
- Engineered five derived features (ratios, interactions, `radius_squared`)
- Trained three leakage-safe pipelines: **Linear Regression, Ridge, Lasso**
- Evaluated with R², MSE, RMSE, MAE, plus 5-fold cross-validation
- **Ablation study** removing geometric shortcut features
- **Secondary regression** on `mean fractal dimension` (harder target)

### Deliverable 3 — Classification, Clustering, and Pattern Mining
- **Classification:** tuned Decision Tree and k-NN via 5-fold
  StratifiedGridSearchCV (optimizing F1 due to class imbalance)
- **Clustering:** K-Means on standardized features (k = 2 by elbow + silhouette),
  visualized in PCA space
- **Association rules:** Apriori on tertile-discretized features, filtered for
  rules predicting malignancy

### Deliverable 4 — Final Insights, Recommendations, and Presentation
- Consolidated all findings into a comprehensive report
- Discussed ethical considerations: privacy (de-identified public data),
  fairness (no demographic attributes available), bias (single 1990s institution)
- Produced a 5–7 minute video presentation with speaker notes
- Delivered practical recommendations and future-work roadmap

---

## Major Findings

### 1. The dataset is exceptionally clean but strongly multicollinear
No missing data, duplicates, or invalid values. However, the three size features
(`radius`, `perimeter`, `area`) are near-perfect linear combinations of one
another. This structural property dominates every subsequent modeling choice.

### 2. Regression on `mean area` is nearly deterministic — and that is a warning
All three models reached **R² ≈ 0.999** because
`mean area ≈ π · (mean radius)²` for near-circular masses. The ablation study
confirmed that even without explicit squared/ratio features, R² remains ≈ 0.989
due to multicollinearity. The lesson is **not** that the models are excellent —
it is that **R² near 1.0 can indicate target leakage or a trivial geometric
identity**, not predictive skill. A realistic comparison on `mean fractal
dimension` yielded R² ≈ 0.83, where Ridge and Lasso regularization actually
mattered.

### 3. Classification is highly accurate across paradigms
- **k-NN (tuned):** Accuracy 0.956, F1 0.938, ROC-AUC 0.986
- **Decision Tree (tuned):** Accuracy 0.921, F1 0.889, ROC-AUC 0.928
- **`mean concave points`** drives ~84% of the tree's discriminative power,
  matching classical cytological markers of malignancy

### 4. Unsupervised structure matches supervised labels without using them
K-Means with k = 2 (selected by both elbow and silhouette) recovered a
benign-dominant cluster and a malignant-dominant cluster, confirming the ten
measurements carry real class-separating signal. PCA projection (≈ 80%
variance in the first two components) visually aligns the unsupervised and
supervised structure.

### 5. Association rules are clinically intuitive
The strongest rules — e.g., `{high area, high concave points} → malignant` —
reached **confidence ≈ 1.0** and **lift > 2.6**. Large, irregular masses are
classically associated with malignancy, so these rules are directly
interpretable and easy to embed in decision-support dashboards.

### 6. Feature selection mattered more than algorithm sophistication
Ten well-chosen mean measurements succeeded across supervised regression,
supervised classification, unsupervised clustering, and rule-based mining. No
single algorithm "won" the project — disciplined feature engineering and
leakage-safe pipelines did.

---

## Key Results Summary

| Stage | Method | Headline Result |
|---|---|---|
| Cleaning | Audit + IQR screening | 569 / 569 records retained; 0 issues |
| Regression (primary) | OLS / Ridge / Lasso | R² ≈ 0.999 (geometric identity) |
| Regression (secondary) | Same three models | R² ≈ 0.83 (realistic difficulty) |
| Classification (DT) | Tuned Decision Tree | F1 = 0.889, ROC-AUC = 0.928 |
| Classification (k-NN) | Tuned k-NN | F1 = 0.938, ROC-AUC = 0.986 |
| Clustering | K-Means (k = 2) | Recovered diagnosis structure without labels |
| Association Rules | Apriori | Top rules: confidence ≈ 1.0, lift > 2.6 |

---

## Challenges and Resolutions

| Challenge | Resolution |
|---|---|
| Target leakage risk in regression | Explicit exclusion of `mean area` from features |
| Extreme multicollinearity | Ablation study + Ridge/Lasso comparison, not blind deletion |
| Class imbalance (62.7 / 37.3) | Stratified splits + F1/ROC-AUC reporting (accuracy alone rejected) |
| Shared mutable pipeline state | `sklearn.base.clone()` in ablation loops |
| Over-aggressive Lasso on secondary target | Reported as an instructive negative result; alpha tuning deferred |
| Right-skewed continuous targets | Documented; log-transform recommended for future work |

---

## Ethical Considerations

- **Privacy:** Data is de-identified and public; no attempt was made to
  reconstruct patient identities.
- **Fairness:** No demographic attributes available, so subgroup fairness
  cannot be formally audited. This is a documented limitation, not a solved
  issue.
- **Generalization risk:** Original population from a single 1990s institution;
  models may not transfer to modern populations, different imaging equipment,
  or multimodal inputs.
- **Clinical use:** All results are explicitly framed as **educational and
  exploratory** — not clinical diagnostic rules. Real-world deployment would
  require external validation, fairness audits with protected attributes,
  and human-in-the-loop oversight.

---

## Practical Recommendations

1. **Clinical triage:** The Decision Tree's interpretable root split on
   `mean concave points` offers a low-cost triage signal for flagging
   high-suspicion cases.
2. **Case-based reasoning:** k-NN retrieval surfaces morphologically similar
   past cases — useful for tumor-board discussions.
3. **Patient subgroups:** K-Means clusters suggest cohorts that may warrant
   differentiated follow-up surveillance.
4. **Decision-support alerts:** High-lift association rules can be encoded as
   traceable EHR warnings (e.g., "large + irregular → suspect malignancy").
5. **Modeling hygiene:** When predicting area-related outcomes, drop one of
   `{radius, perimeter, area}` or use Ridge/Lasso to control coefficient
   variance.

---

## Limitations and Future Work

**Limitations:**
- Small sample size (N = 569)
- Only 10 mean measurements from one imaging modality
- No demographic, clinical-history, or longitudinal outcome variables
- External validity untested against contemporary multi-institutional data

**Future directions:**
- Log-transform skewed continuous targets
- Gradient boosting and non-linear regressors as a benchmark
- Multimodal and demographic feature integration when available
- Formal fairness audits with protected attributes
- Real clinical workflow validation with external cohorts

---

## Presentation

The final deliverable includes a **5–7 minute recorded video presentation**
supported by `presentation/Deliverable_4_Presentation.pptx`, which contains
bullet-point slides and full speaker notes. To regenerate the deck:

```bash
python presentation/generate_presentation.py
```

The script writes `Deliverable_4_Presentation.pptx` and prints per-slide word
counts plus a total runtime estimate.

---

## References (APA 7)

Belsley, D. A., Kuh, E., & Welsch, R. E. (1980). *Regression diagnostics: Identifying influential data and sources of collinearity*. John Wiley & Sons.

Deepthi, Nayak, A., & Panicker, R. O. (2026). Independent evaluation of machine learning and deep learning models for breast cancer detection. *PLOS Digital Health, 5*(9), e0001747. https://doi.org/10.1371/journal.pdig.0001747

de Vargas, V. W., Aranda, J. A. S., Costa, R. D. S., Pereira, P. R. da S., & Barbosa, J. L. V. (2023). Imbalanced data preprocessing techniques for machine learning: A systematic mapping study. *Knowledge and Information Systems, 65*(1), 31–57. https://doi.org/10.1007/s10115-022-01772-8

Han, J., Pei, J., & Tong, H. (2022). *Data mining: Concepts and techniques* (4th ed.). Morgan Kaufmann.

Hastie, T., Tibshirani, R., & Friedman, J. (2009). *The elements of statistical learning: Data mining, inference, and prediction* (2nd ed.). Springer.

Hoerl, A. E., & Kennard, R. W. (1970). Ridge regression: Biased estimation for nonorthogonal problems. *Technometrics, 12*(1), 55–67. https://doi.org/10.1080/00401706.1970.10488634

James, G., Witten, D., Hastie, T., & Tibshirani, R. (2021). *An introduction to statistical learning: With applications in R* (2nd ed.). Springer.

Kaufman, S., Rosset, S., Perlich, C., & Stitelman, O. (2012). Leakage in data mining: Formulation, detection, and avoidance. *ACM Transactions on Knowledge Discovery from Data, 6*(4), Article 15. https://doi.org/10.1145/2382577.2382579

Kuhn, M., & Johnson, K. (2019). *Feature engineering and selection: A practical approach for predictive models*. Chapman & Hall/CRC.

Kutner, M. H., Nachtsheim, C. J., Neter, J., & Li, W. (2005). *Applied linear statistical models* (5th ed.). McGraw-Hill/Irwin.

Pedregosa, F., Varoquaux, G., Gramfort, A., Michel, V., Thirion, B., Grisel, O., Blondel, M., Prettenhofer, P., Weiss, R., Dubourg, V., Vanderplas, J., Passos, A., Cournapeau, D., Brucher, M., Perrot, M., & Duchesnay, É. (2011). Scikit-learn: Machine learning in Python. *Journal of Machine Learning Research, 12*, 2825–2830.

Street, W. N., Wolberg, W. H., & Mangasarian, O. L. (1993). Nuclear feature extraction for breast tumor diagnosis. *Proceedings of SPIE — International Society for Optical Engineering, 1905*, 861–870. https://doi.org/10.1117/12.148598

Tibshirani, R. (1996). Regression shrinkage and selection via the lasso. *Journal of the Royal Statistical Society: Series B (Methodological), 58*(1), 267–288. https://doi.org/10.1111/j.2517-6161.1996.tb02080.x

Zhu, J., Zhao, Z., Yin, B., Wu, C., Yin, C., Chen, R., & Ding, Y. (2025). An integrated approach of feature selection and machine learning for early detection of breast cancer. *Scientific Reports, 15*, Article 13015. https://doi.org/10.1038/s41598-025-97685-x

---

