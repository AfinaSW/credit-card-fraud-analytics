# 💳 Hunting Card Fraud: From Raw Logs to $31.9K in Saved Capital
### An End-to-End Payment Risk Analysis & Rule-Based Fraud Defense

[![Python](https://img.shields.io/badge/Python-3.9+-3776AB?style=flat&logo=python&logoColor=white)](https://www.python.org/)
[![Pandas](https://img.shields.io/badge/Pandas-Data_Analysis-150458?style=flat&logo=pandas&logoColor=white)](https://pandas.pydata.org/)
[![NumPy](https://img.shields.io/badge/NumPy-Vectorized_Math-013243?style=flat&logo=numpy&logoColor=white)](https://numpy.org/)
[![Seaborn](https://img.shields.io/badge/Seaborn-Visual_Storytelling-4c72b0?style=flat)](https://seaborn.pydata.org/)

---

## 🎯 The Hook & Project Mission

When an unauthorized transaction goes through, the bank loses twice: first to the chargeback refund, and second to customer trust. But blocking everything indiscriminately is worse—it alienates honest users and buries support teams in angry calls.

**The Mission:**  
Instead of blindly throwing a black-box ML model at an imbalanced Kaggle dataset, I took the perspective of an in-house **Fraud Risk Analyst**.  
My goal was to investigate 283k+ transactions, uncover the actual operational playbook used by fraudsters, and design a transparent, **heuristic rule engine** that cuts fraud losses without overwhelming the fraud operations team.

---

## ⚡ What Was Done (At a Glance)

1. **Hygiene & Leakage Prevention:** Audited 284k records, identifying and eliminating 1,081 technical retry duplicates (0.38%) that would have artificially inflated detection accuracy.
2. **Behavioral Forensics:** Disproved standard industry assumptions around ticket size and transaction timing through quantile mapping (`np.percentile`) and temporal grouping.
3. **Anomaly & Correlation Screening:** Analyzed statistical divergence across 28 anonymized PCA components to isolate high-conviction risk features.
4. **Heuristic Engine Design:** Built a production-ready, 3-tier vectorized alert system in Python (`np.select`) using empirical percentile thresholds.
5. **Scenario Cost Modeling:** Modeled a real-world financial cost-benefit matrix balancing prevented capital loss against operational review overhead.

---

## 💡 Top 3 Counterintuitive Discoveries: Insight ➔ Decision

### 1. The Low-Ticket Trap (Amount Analysis)
* **The Common Myth:** *"Fraudsters try to drain credit limits with massive purchases."*
* **The Reality:** The median fraudulent transaction was **$9.82**, compared to **$22.00** for legitimate users. Furthermore, fraudulent tickets strictly capped at **$2,125.87**, while honest purchases reached up to **$25,691.16**.
* **Why it matters:** Criminals intentionally run automated micro-charges to verify stolen cards without triggering SMS alerts or 3D-Secure challenges.
* **The Business Decision:** **Never use transaction amount as a standalone blocking rule.** Doing so blinds the system to card-testing rings while harassing high-value legitimate cardholders.

---

### 2. The Overnight Vulnerability Window (Temporal Dynamics)
* **The Discovery:** While total transaction volume drops heavily between 01:00 and 05:00, the **relative fraud rate spikes dramatically** during these hours.
* **Why it matters:** Fraudsters exploit the hours when victims are asleep and unable to immediately freeze their cards or notice push notifications.
* **The Business Decision:** **Use time as a contextual multiplier, not a hard block.** Flag overnight operations for mandatory silent step-up verification (e.g., in-app biometric confirmation) only if secondary risk indicators are present.

---

### 3. Divergent Behavioral Signatures (Statistical Screening)
* **The Discovery:** Out of 28 latent vectors, four features exhibited severe negative divergence during attacks: **`V14` (-0.29)**, **`V17` (-0.31)**, **`V12` (-0.25)**, and **`V10` (-0.21)**, while **`V4` (+0.13)** spiked positively.
* **Why it matters:** These features capture extreme operational deviations from typical cardholder behavior.
* **The Business Decision:** Calibrated empirical cutoff boundaries around these specific levers to build deterministic, explainable alert rules that comply with regulatory audit standards.

---

## 🛠️ The 3-Tier Alert Engine: Performance & Routing

I designed three intuitive alert rules and evaluated their precision vs. capture rate trade-offs:

| Rule Layer | Target Behavioral Pattern | Alerts Generated | Frauds Caught | False Alarms | Precision | Recall | Recommended Action |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: | :--- |
| **Rule 1** | Severe `V14` (< -6) or `V17` (< -5) drop | 466 | 324 | 142 | **69.5%** | **68.5%** | **Auto-Challenge:** Immediate 3DS / SMS verification |
| **Rule 2** | `V12` anomaly (< -4) + `V4` spike (> 3) | 329 | 250 | 79 | **76.0%** | **52.9%** | **Priority Queue:** High-confidence manual review |
| **Rule 3** | Overnight window (01–05h) + `V10` (< -2.5) | 202 | 96 | 106 | **47.5%** | **20.3%** | **Secondary Signal:** Risk scoring weight |
| **COMBINED**| **Any of the 3 rules triggered** | **606** | **336** | **270** | **55.5%** | **71.0%** | **Unified First-Line Defense** |

> **Key Analytical Takeaway:** With zero machine learning complexity, this rule framework caught **71.04% of all fraud instances** while generating only **270 false alarms across 283,000+ legitimate transactions** (a False Positive Rate below 0.1%).

---

## 💰 The Bottom Line: Financial Scenario Analysis

In risk operations, technical metrics do not matter unless they translate into preserved capital.

| Metric | Financial Volume | % of Exposure / Share |
| :--- | :--- | :--- |
| **Total Fraud Exposure at Risk** | **$58,591.39** | 100.0% |
| **[+] Fraud Capital Protected (True Positives)** | **$31,953.74** | **54.5%** |
| **[-] Fraud Capital Missed (False Negatives)** | $26,637.65 | 45.5% |
| **[-] Operational Review Overhead** (270 FP × $3.00) | $810.00 | — |
| **[=] ESTIMATED NET BENEFIT DELIVERED** | **$31,143.74** | **ROI: ~3,845%** |

### The Strategic Lesson:
While the rule engine intercepted **71% of fraud incidents**, it captured **54.5% of the dollar volume**.  
This gap reveals that the remaining 29% of uncaught fraud contained larger, sophisticated, evasive tickets. **This specific $26.6K blind spot—not algorithmic curiosity—is the exact business case for where an advanced ML scoring model should be introduced next.**

---
## 📊 Operational Risk Dashboard (Power BI)

To bridge data analysis with daily risk operations, I engineered an operational monitoring dashboard in Power BI. It provides the Fraud Operations team with live tracking of queue volume, rule saturation, and net financial impact.

![Operational Risk Dashboard](images/dashboard.png)

### Core Dashboard Capabilities:
* **Alert Volume & Queue Ingestion:** Tracks hourly transaction capacity against active heuristic rule alerts.
* **Rule Saturation Monitoring:** Visualizes the True Positive vs. False Positive footprint for each rule layer to identify operational fatigue.
* **Financial Ledger & Scenario Impact:** Real-time visibility into prevented chargeback exposure versus review operational overhead.
## 🔭 Next Steps for the Risk Team

* [ ] **Temporal Backtesting:** Run the rule engine on out-of-time transaction windows to track threshold decay and evasion drift.
* [ ] **Refine Cost Assumptions:** Replace the baseline $3.00 investigation scenario with live telephony/SMS vendor fees and estimated customer churn friction.
* [ ] **Error Cohort Deep-Dive:** Perform exploratory clustering on the $26.6K in missed high-ticket fraud to identify shared merchant categories.

