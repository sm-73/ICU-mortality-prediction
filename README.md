# ICU Mortality Prediction

## Project Overview

This project builds and evaluates machine learning models to predict in-hospital mortality for ICU patients using the PhysioNet 2012 Challenge dataset. Early and accurate mortality prediction in the ICU can help medical teams prioritize care, allocate resources effectively, and intervene before a patient's condition deteriorates (all while managing the high cost of critical care).



## Dataset

**Source:** [Predict Mortality of ICU Patients - PhysioNet](https://www.kaggle.com/datasets/msafi04/predict-mortality-of-icu-patients-physionet)

- **Set-A** (~4,000 patient records): used for training and evaluation in this project
- **46 variables** per patient: demographic info + 36 physiological measurements recorded over the first 48 hours of ICU admission
- **Target variable:** `In-hospital_death` (binary: 0 = survived, 1 = deceased)

> The dataset exhibits severe class imbalance: **86.1% survivors (3,396)** vs **13.9% deceased (547)**.



## Methodology

### 1. Exploratory Data Analysis (EDA)
- Analyzed distributions of all 46 features
- Identified and visualized class imbalance
- Examined feature correlations (e.g., PaO2 & SaO2: r = 0.35; GCS & Lactate: r = -0.24)



### 2. Data Preprocessing
- **Missing value handling:** Features with >70% missing data were dropped (Troponin I: 94.95%, Cholesterol: 92.32%, Troponin T: 78.29%, RespRate: 72.58%)
- **Imputation:** KNN imputation used to fill remaining gaps (preserves relationships between physiological markers better than simple mean imputation)
- **Outlier treatment:** Outliers were detected via IQR method (e.g., Creatinine: 10.73%, BUN: 8.22%) but **retained**, as they may reflect real critical patient conditions
- **Class imbalance:** SMOTE (Synthetic Minority Oversampling Technique) applied exclusively to training data to generate synthetic samples of the minority (deceased) class
- **Normalization:** StandardScaler applied uniformly across all models for fair comparison



### 3. Feature Selection
- Applied **MCSPCA** (Mean-Centering Subspace Principal Component Analysis): a cost-sensitive extension of PCA that assigns higher weights to the minority class
- ~28 principal components needed to explain 95% of variance
- Top 15 features selected based on MCSPCA loading scores:

| # | Feature | # | Feature |
|---|---------|---|---------|
| 1 | Urine | 9 | WBC |
| 2 | Na | 10 | K |
| 3 | Mg | 11 | ICUType |
| 4 | FiO2 | 12 | Platelets |
| 5 | Lactate | 13 | Glucose |
| 6 | SaO2 | 14 | GCS |
| 7 | Weight | 15 | PaO2 |
| 8 | Temperature | | |



## Models

All three models were trained under identical conditions:
- **80/20 train-test split**
- **5-fold cross-validation** during training
- Same SMOTE-balanced training data and StandardScaler pipeline

| Model | Key Configuration |
|-------|------------------|
| Decision Tree | Max depth = 5 |
| K-Nearest Neighbors (KNN) | k = 5, with standardization |
| Random Forest | 200 estimators, max depth = 10, class_weight = 'balanced' |



## Results

| Model | Accuracy | Precision | Recall | F1-Score | AUC |
|-------|----------|-----------|--------|----------|-----|
| **Random Forest** | **0.8861** | **0.8619** | **0.9208** | **0.8891** | **0.78** |
| KNN | 0.8555 | 0.7830 | 0.9838 | 0.8719 | 0.72 |
| Decision Tree | 0.7472 | 0.7243 | 0.8004 | 0.7572 | 0.71 |



### Best Model: Random Forest
- Highest accuracy (88.6%) and F1-score (0.889)
- Best AUC (0.78): strongest ability to distinguish survivors from non-survivors
- Ensemble nature makes it robust to noise and complex clinical relationships



### Notable Observations
- **KNN** achieved the highest recall (0.984), meaning it was the most sensitive at identifying patients who would die (valuable in a clinical setting) but at the cost of more false alarms (lower precision)
- **Decision Tree** was the weakest performer; its limited depth caused underfitting and missed complex patterns in the data



## References

1. Remeseiro, B., & Bolon-Canedo, V. (2019). A review of feature selection methods in medical applications. *Computers in Biology and Medicine*, 112, 103375. https://doi.org/10.1016/j.compbiomed.2019.103375

2. Liu, J., et al. (2018). Mortality prediction based on imbalanced high-dimensional ICU big data. *Computers in Industry*, 98, 218–225. https://doi.org/10.1016/j.compind.2018.01.017

3. Chia, A. H. T., et al. (2021). Explainable machine learning prediction of ICU mortality. *Informatics in Medicine Unlocked*, 25, 100674. https://doi.org/10.1016/j.imu.2021.100674

4. Monteiro, F., et al. (2020). Prediction of mortality in Intensive Care Units: a multivariate feature selection. *Journal of Biomedical Informatics*, 107, 103456. https://doi.org/10.1016/j.jbi.2020.103456
