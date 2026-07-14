# 🏦 German Credit Risk Analysis
 
**Skills demonstrated:** Data Cleaning · Missing Value Imputation · Feature Engineering · Classification · Risk Segmentation · Threshold Optimisation · Business Storytelling

---

## Project Summary

End-to-end credit risk analysis of 1,000 loan applicants from the German Credit dataset (Prof. Hofmann, UCI ML Repository). The project covers risk target engineering, missing value treatment, feature engineering, predictive modelling, and a three-tier applicant risk segmentation framework — delivering a production-ready underwriting decision tool.

---

## Two Real-World Data Challenges Solved

Most portfolio projects start with a clean, labelled dataset. This one didn't.

**Challenge 1 — No target column.**  
The dataset had no Risk label. The binary target was engineered from scratch using weighted scoring rules derived from the original Hofmann credit scoring methodology.

**Challenge 2 — Heavy missing values.**  
Saving accounts: 183 missing (18.3%). Checking account: 394 missing (39.4%). Rather than imputing with a statistical average, missing values were treated as a distinct `none` category — because an applicant with no savings account is a fundamentally different risk profile from one with little savings. The `none` group became the highest-risk category in both features.

---

## Key Findings

- **23.6% bad risk rate** across 1,000 applicants — consistent with the original dataset benchmark
- Applicants with **no savings AND no checking account** have a **50%+ bad rate** — the strongest single underwriting signal
- **Vacation and education loans** carry 44–67% bad rates vs 15% for furniture and radio/TV loans
- **Renters** (41.9%) and free-housing applicants (48.1%) carry dramatically higher risk than homeowners (15.3%)
- Bad risk applicants borrow **2.6× more** on average (DM 6,225 vs DM 2,359) over **76% longer** loan terms

---

## Model Performance

| Model | ROC-AUC | Precision (Bad) | Recall (Bad) | F1 (Bad) |
|---|---|---|---|---|
| Logistic Regression | 0.970 | 0.750 | 0.830 | 0.788 |
| Random Forest | 0.970 | 0.813 | 0.830 | 0.821 |
| **Gradient Boosting ★** | **0.990** | **0.909** | **0.851** | **0.879** |

All metrics on the bad risk class (minority). Gradient Boosting selected for deployment.

---

## Threshold Optimisation

The default 0.50 threshold is not optimal for credit risk — approving a bad loan costs far more than declining a good applicant. Three operating points evaluated:

| Threshold | Recall | Bad Loans Caught | Good Apps Declined |
|---|---|---|---|
| **0.30 (recommended)** | **95.7%** | **45 of 47** | **6** |
| 0.50 (default) | 85.1% | 40 of 47 | 4 |
| 0.70 (strict) | 72.3% | 34 of 47 | 3 |

The threshold is a **business decision**, not a statistical one. The model supports all three operating points.

---

## Feature Engineering

Seven features engineered beyond the nine raw columns:

| Feature | Logic | Rationale |
|---|---|---|
| `credit_duration_ratio` | Credit ÷ Duration | Monthly debt burden — affordability proxy |
| `credit_to_age_ratio` | Credit ÷ Age | Credit relative to earning years |
| `sav_enc` | none=0 → rich=4 | Ordinal: preserves savings hierarchy |
| `chk_enc` | none=0 → rich=3 | Ordinal: preserves checking hierarchy |
| `no_savings` | sav_enc ≤ 1 | Binary flag: little or no savings |
| `no_checking` | chk_enc ≤ 1 | Binary flag: little or no checking |
| `double_vulnerability` | no_savings AND no_checking | Both accounts weak — highest risk combination |

---

## Three-Tier Risk Framework

| Risk Tier | Applicants | Bad Rate | Avg Credit | Action |
|---|---|---|---|---|
| Low Risk (prob < 0.30) | 760 | 0% | DM 2,352 | Auto-approve |
| Medium Risk (0.30–0.60) | 8 | 75% | DM 3,840 | Manual review |
| High Risk (prob > 0.60) | 232 | 98% | DM 6,264 | Decline / collateral |

---

## Savings × Checking Underwriting Matrix

Bad loan rate (%) for every combination of savings and checking account level.  
Deployable as a policy rule table — no ML infrastructure required.

| Savings \ Checking | None | Little | Moderate | Rich |
|---|---|---|---|---|
| **none** | 51% | 44% | 38% | 0% |
| **little** | 31% | 22% | 14% | 0% |
| **moderate** | 18% | 10% | 5% | 0% |
| **quite rich** | 5% | 3% | 2% | 0% |
| **rich** | 0% | 0% | 0% | 0% |

---

## Business Recommendations

1. **Deploy two-gate underwriting model** — Auto-approve prob < 0.30, manual review 0.30–0.60, decline > 0.60. Catches 95.7% of bad loans.
2. **Purpose-based pricing** — Require deposit or co-signer for vacation and education loans (44–67% bad rate).
3. **Mandate account verification** — Require one active account at origination. Eliminates the double-vulnerability segment.
4. **Monthly burden threshold** — Cap credit ÷ duration at DM 250/month for unsecured loans.

---

## Files

| File | Description |
|---|---|
| `german_credit_data.csv` | Raw dataset (UCI ML Repository) |
| `CR_CreditRisk_Report.docx` | Full 9-section findings report |
| `CR_Business_Deck.pptx` | 9-slide business presentation with speaker notes |
| `CR_Analytics_Dashboard.html` | Interactive 6-page analytics dashboard |
| `CR_01_eda_dashboard.png` | Exploratory data analysis charts |
| `CR_02_model_evaluation.png` | Model evaluation dashboard |
| `CR_03_risk_segmentation.png` | Risk segmentation dashboard |

---

## Methodology

- Stratified 80/20 train/test split to preserve the 23.6% bad risk ratio in both sets
- `class_weight='balanced'` in Logistic Regression and Random Forest to handle class imbalance
- Ordinal encoding for savings and checking — preserving the financial strength order (none < little < moderate < rich)
- Missing values treated as `none` category — industry-standard, signal-preserving approach
- StandardScaler applied to Logistic Regression only — tree models are scale-invariant
- Threshold 0.30 recommended for conservative lending; 0.50 for volume lending

---

## Dataset

[German Credit Data](https://archive.ics.uci.edu/ml/datasets/statlog+(german+credit+data)) · Prof. Hans Hofmann · UCI Machine Learning Repository · 1,000 instances · 20 original attributes (9 used after cleaning)
