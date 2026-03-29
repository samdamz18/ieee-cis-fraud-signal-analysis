![Cover](assets/cover.png)

# IEEE-CIS Fraud Signal Analysis
### The Fraud Signal Was Never Where I Thought It Would Be

[![Python](https://img.shields.io/badge/Python-3.11-blue?style=flat-square&logo=python)](https://www.python.org/)
[![XGBoost](https://img.shields.io/badge/XGBoost-2.0-red?style=flat-square)](https://xgboost.readthedocs.io/)
[![scikit-learn](https://img.shields.io/badge/scikit--learn-1.3-orange?style=flat-square)](https://scikit-learn.org/)
[![License: CC BY-SA 4.0](https://img.shields.io/badge/Dataset-CC%20BY--SA%204.0-lightgrey?style=flat-square)](https://creativecommons.org/licenses/by-sa/4.0/)

> Part 3 of a series on imbalanced financial data.
> Velocity features failed. Relational signals held.

---

## Overview

This project tests three hypotheses on 590,540 real e-commerce transactions from Vesta Corporation:

1. **Behavioural fingerprinting** — does measuring how unusual a transaction is relative to *that customer's own baseline* add ranking power over raw features?
2. **Velocity features** — do rolling time-window features capturing burst behaviour improve on the behavioural baseline?
3. **Relational signal** — do graph-inspired entity features capturing shared infrastructure (one device → many cards, one address → many identities) add stable signal?

Each hypothesis is validated on both a random split and a temporal split — training on the first half of the dataset, evaluating on the second. The temporal split is the test that matters in practice.

---

## Key Results

| Model | Random AUPRC | Temporal AUPRC | Drop |
|---|---|---|---|
| Raw features (111) | 0.6917 | 0.5062 | -18.54 pp |
| Raw + Behavioural (139) | 0.7754 | 0.6440 | -13.14 pp |
| Raw + Beh + Velocity (152) | 0.7738 | 0.6442 | -12.96 pp |
| Raw + Beh + Relational (154) | **0.7829** | 0.6512 | -13.17 pp |
| Full — all families (167) | 0.7792 | **0.6532** | -12.60 pp |

**Behavioural lift over raw: +8.37 pp**
**Velocity lift over behavioural: -0.16 pp** ← failed the hypothesis test
**Relational lift over behavioural: +0.75 pp** ← held on temporal split

### Precision at Top-K (Temporal Split — Full Model)

| Top Fraction | Alerts | Precision |
|---|---|---|
| 0.1% | 295 | **98.98%** |
| 0.5% | 1,476 | 94.99% |
| 1.0% | 2,952 | 93.73% |

---

## Why Velocity Failed

The burst behaviour narrative is compelling — fraudsters burn through cards quickly, counts spike, spend escalates. But on this dataset, two things made velocity features redundant:

1. **The behavioural baseline already captured the signal.** Deviation from the customer's own history already encodes the relevant timing information. Rolling windows added nothing on top.
2. **This is a low-and-slow fraud dataset.** The fraud that survives into this data is deliberate — attackers who mimic the victim's behaviour to avoid triggering alerts. Their burst signatures are weak by design.

---

## Feature Engineering Summary

### Behavioural Features (28)
| Feature Group | Description |
|---|---|
| Amount deviation | `amt_zscore`, `amt_to_mean_ratio`, `amt_to_max_ratio`, `is_new_cust_max` |
| Timing deviation | `D1_zscore`, `cust_velocity`, `card_age_norm`, `hour_sin/cos` |
| Frequency deviation | `C1–C14` deviation z-scores per customer |
| Target encoding | OOF-encoded `customer_uid`, `card1`, `card2`, `addr1`, email domains |

### Velocity Features (13)
Rolling aggregates over 1h, 24h, 7d windows: transaction count, amount sum, amount max, unique emails, plus acceleration ratios.

### Relational Features (15)
Entity-level counts and uniqueness measures (`card1_txn_count`, `device_unique_cards`, `p_email_unique_cards`, etc.) plus OOF-encoded entity risk for card, address, device, and email domain.

---

## Dataset

**IEEE-CIS Fraud Detection** — published by Vesta Corporation via Kaggle
- 590,540 transactions, 20,663 fraud cases (3.50%)
- Imbalance ratio: 1:28
- Source: [Kaggle](https://www.kaggle.com/competitions/ieee-fraud-detection)

> The dataset CSV files are not included in this repo due to file size. Download from Kaggle and place in a `datasets/` folder before running the notebook.

Expected files:
```
datasets/
├── train_transaction.csv
└── train_identity.csv
```

---

## Project Structure

```
ieee-cis-fraud-signal-analysis/
│
├── ieee_cis.ipynb          # Full analysis notebook
├── README.md
├── assets/
│   └── cover.png           # Cover image
└── datasets/               # Place CSV files here (not committed)
    ├── train_transaction.csv
    └── train_identity.csv
```

---

## How to Run

```bash
# 1. Clone the repo
git clone https://github.com/samdamz18/ieee-cis-fraud-signal-analysis.git
cd ieee-cis-fraud-signal-analysis

# 2. Install dependencies
pip install pandas numpy scikit-learn xgboost matplotlib seaborn jupyter

# 3. Download dataset from Kaggle and place in datasets/ folder

# 4. Launch notebook
jupyter notebook ieee_cis.ipynb
```

---

## Series

This is Part 3 of a series on imbalanced financial data:

| Part | Dataset | Core Finding |
|---|---|---|
| [Part 1](https://medium.com/@sammzy/classifying-128-million-financial-transactions-63ba864c0c9e) | ZIMS (128M records) | Signal in text — weighted F1 hid a 0.47 Macro F1 disaster |
| [Part 2](https://medium.com/@sammzy) | PaySim (6.3M transactions) | 3 accounting features drove 70% of model power — AUPRC 0.9981 |
| **Part 3** | IEEE-CIS (590K transactions) | Velocity failed, relational held — temporal stability as the verdict |

---

## Tech Stack

`Python` · `XGBoost` · `scikit-learn` · `pandas` · `NumPy` · `matplotlib` · `seaborn`

---

## Article

Read the full write-up: [**Medium**](https://medium.com/@sammzy/the-fraud-signal-was-never-where-i-thought-it-would-be-8b128f057b95) 

---

## Author

**Samuel Ojo** · Data Analyst · Nigeria
[LinkedIn](https://www.linkedin.com/in/samuel-adedamola-ojo/) · [Medium](https://medium.com/@sammzy)
