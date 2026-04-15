# Network Intrusion Detection — Brute-Force Classification with Machine Learning

Binary classification of malicious network traffic using flow-based features from the CIC IoT-DIAD 2024 dataset.  
Focus: Brute-Force attack detection in IoT environments.

> **Authors:** Doersing, Huke  
> **Context:** University project — Master's programme, Data Science & AI

---

## Overview

This project develops a machine learning pipeline to classify network flows as either benign or anomalous (Brute-Force attacks). The dataset is sourced from the Canadian Institute for Cybersecurity and covers simulated traffic across 105 IoT devices with 33 attack scenarios.

**Key design decisions:**
- Flow-based feature selection (no payload inspection) → applicable to unknown and zero-day patterns
- Imbalanced dataset preserved intentionally (~95.7% benign) to reflect realistic network conditions
- F1-Score (macro) used as primary evaluation metric due to class imbalance

---

## Project Structure

```
├── Doersing_Huke_KI_Model.ipynb   # Main notebook
├── requirements.txt
├── README.md
└── data/                          # Not included — see Dataset section below
    ├── BenignTraffic.pcap_Flow.csv
    ├── BenignTraffic2.pcap_Flow.csv
    └── DictionaryBruteForce.pcap_Flow.csv
```

---

## Dataset

**CIC IoT-DIAD 2024** — Canadian Institute for Cybersecurity  
Download: [https://www.unb.ca/cic/datasets/iot-diad-2024.html](https://www.unb.ca/cic/datasets/iot-diad-2024.html)

Place the following files in a `data/` directory at the project root before running the notebook:

| File | Label assigned |
|---|---|
| `BenignTraffic.pcap_Flow.csv` | benign |
| `BenignTraffic2.pcap_Flow.csv` | benign |
| `DictionaryBruteForce.pcap_Flow.csv` | anomaly |

---

## Features

The following flow-based features were selected for training:

| Feature | Description |
|---|---|
| Flow Duration | Total connection duration |
| Total Fwd Packet | Packet count in forward direction |
| Total Length of Fwd Packet | Total bytes sent forward |
| Total Length of Bwd Packet | Total bytes sent backward |
| Flow IAT Mean | Mean inter-arrival time between packets |
| Flow Bytes/s | Average throughput in bytes per second |
| SYN Flag Count | TCP connection initiation flags |
| ACK Flag Count | TCP acknowledgement flags |
| FIN Flag Count | TCP connection termination flags |

---

## Methodology

### 1 — Exploratory Data Analysis (EDA)
- Correlation matrix (Pearson) to identify feature relationships
- Boxplots and KDE plots per class to visualise distributional differences
- Key finding: anomalous flows cluster at low values across most features, indicating short, automated, connection patterns

### 2 — Preprocessing
- Baseline: `SimpleImputer (median)` + `StandardScaler`
- Extended grid: 5 scalers × 3 imputing strategies × `SimpleImputer` / `KNNImputer`
- Best configuration for LightGBM: `KNNImputer` + `RobustScaler`

### 3 — Model Comparison (baseline)

| Model | Accuracy | Precision | Recall | F1-Score |
|---|---|---|---|---|
| **Random Forest** | 0.9894 | 0.9382 | 0.9318 | **0.9350** |
| Decision Tree | 0.9869 | 0.9222 | 0.9166 | 0.9194 |
| LightGBM | 0.9774 | 0.8296 | 0.9730 | 0.8868 |
| Gradient Boosting | 0.9794 | 0.8934 | 0.8401 | 0.8646 |

### 4 — Hyperparameter Tuning (BayesSearchCV)
Random Forest and LightGBM were selected for tuning. Bayesian optimisation with 50 iterations, 4-fold cross-validation, scoring on `f1_macro`.

Best LightGBM parameters found:
- `n_estimators`: 500, `learning_rate`: 0.033, `max_depth`: 15, `num_leaves`: 150, `min_child_samples`: 5, `subsample`: 1.0, `colsample_bytree`: 0.936

### 5 — Final Results

| Model | Accuracy | Precision | Recall | F1-Score |
|---|---|---|---|---|
| **RandomForestClassifier** | **0.9896** | **0.9390** | **0.9332** | **0.9361** |
| LGBMClassifier | 0.9857 | 0.8912 | 0.9516 | 0.9190 |

**Selected model: RandomForestClassifier** — highest F1-Score, best balance of precision and recall, lowest false-positive rate.

---

## Setup

**Requirements:** Python 3.9+

```bash
pip install -r requirements.txt
jupyter notebook Doersing_Huke_KI_Model.ipynb
```

---

## Limitations & Outlook

- Dataset is synthetic (lab environment) — real-world traffic distributions may differ
- Only Brute-Force attack type included; multi-class extension across all 33 attack scenarios is a natural next step
- Sequential or graph-based models (RNN, GCN) could further improve detection of complex attack patterns
- Hybrid rule-based + ML systems worth exploring for production environments

---

## License

Dataset: [CIC IoT-DIAD 2024](https://www.unb.ca/cic/datasets/iot-diad-2024.html) — Canadian Institute for Cybersecurity (not included in this repository). 
