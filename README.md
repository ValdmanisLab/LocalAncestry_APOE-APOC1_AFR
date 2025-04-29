# Local African Ancestry Prediction Pipeline (Chr19 APOE-APOC1) using 1KGP

## Overview

This pipeline predicts local African ancestry (AFR vs. non-AFR) using SNP data from chromosome 19.  
The model is trained on 1000 Genomes Project samples and can be applied to any external VCF file overlapping the trained SNP positions.  
The focus region for SNP extraction is:  
**chr19:44903121–44921336**.

---

## Input Files and Structure

All files are organized under:

### Folder Structure

- `data/Files_to_prepare_the_model/`
    - `chr19_44903121_44921336_1KGP.vcf.gz`: Extracted VCF file for the APOE region on chromosome 19 from the 1000 Genomes Project.
    - `chr19_44903121_44921336_1KGP.vcf.gz.csi`: CSI index file for the corresponding VCF.
    - `snp_positions_chr19_44903121_44921336.tsv`: List of selected SNPs (CHROM, POS) used to build and filter the model input features.
    - `igsr_samples.tsv`: Metadata file containing sample names and superpopulation labels (AFR, EUR, EAS, SAS, AMR).
    - `list_of_no_relatives_1KGP.tsv`: Filtered list of unrelated samples from the 1000 Genomes Project used for model training.

- `code/`
    - `LocalAncestry_APOE-APOC1_AFR.ipynb`: Main Jupyter notebook implementing the end-to-end local ancestry prediction pipeline.

- `results/`
    - `Model/`: Folder where the trained model (`.pkl`) and SNP importance table (`.tsv`) are saved.
    - `Graphs/`: Contains visual outputs such as confusion matrix, PCA plot, and ROC curve.

---

## Model Methodology

- **Model Type**: Random Forest Classifier (`sklearn.ensemble.RandomForestClassifier`)
- **Parameters**:
    - `n_estimators=200`
    - `max_depth=15`
    - `min_samples_leaf=2`
    - `class_weight='balanced'`
    <br></br>

- **SNP Features Used**: 205 SNPs common between model list and input VCF
- **Genotype Encoding**:
    - 0 = homozygous reference (e.g., A\|A)
    - 1 = heterozygous (e.g., A\|G or G\|A)
    - 2 = homozygous alternative (e.g., G\|G)
    - -1 = missing/uninterpretable
- **Filtering Criteria**:
    - SNPs retained if ≥90% of samples have valid genotype calls
    - Only non-related samples used for model training
- **Population Labels**:
    - 1 = AFR (African ancestry)
    - 0 = non-AFR (EUR, SAS, EAS, AMR)

---

## Model Performance Summary

### Dataset

- **Training Samples**: 2,356 (non-related 1000 Genomes samples)
- **Train/Test Split**: 80% training (n=1,884), 20% testing (n=472)
- **SNPs Used**: 204 (after filtering)

### Classifier Settings

- **Algorithm**: Random Forest



### Cross-Validation Results (cv=6)

| Fold | F1-Score |
|:----:|:--------:|
|  1   | 0.854    |
|  2   | 0.870    |
|  3   | 0.921    |
|  4   | 0.929    |
|  5   | 0.931    |

**Average F1-Score**: 0.901

---

### Confusion Matrix (Test Set)

|               | Predicted non-AFR | Predicted AFR |
|---------------|-------------------|---------------|
| **True non-AFR** | 362               | 9             |
| **True AFR**     | 7                 | 94            |

---

### Test Set Performance

| Metric         | Class 0 (non-AFR) | Class 1 (AFR) |
|----------------|-------------------|--------------|
| Precision      | 0.98               | 0.91         |
| Recall         | 0.98               | 0.93         |
| F1-Score       | 0.98               | 0.92         |

| Overall Metric      | Value |
|---------------------|-------|
| Accuracy             | 96.61% |
| Macro Avg F1-Score   | 95%   |
| Weighted Avg F1-Score| 97%   |

---

## Graphs and Visualizations

- **ROC Curve** (Receiver Operating Characteristic, calculated AUC = 0.99)
- **PCA Plot** (Principal Component Analysis)
- **Confusion Matrix** (visualization)

---

## Model Files

- **Model Export**:  
  `afr_ancestry_model_<date>.pkl`
- **Feature Importance Table**:  
  `importance_df_model_<date>.tsv`

---

## Prediction Process

When running predictions on a new VCF file:

1. **SNP Matching**:
    - Extract and encode only the SNPs that overlap with the trained model SNPs.
2. **Prediction**:
    - Classify each sample as AFR or non-AFR.

---

## Thresholds and Decision Criteria

- If predicted AFR probability ≥ 0.5, label as **AFR**.
