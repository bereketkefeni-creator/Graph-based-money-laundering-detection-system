# Crypto-TMS — Money-Laundering Detection on a Transaction Graph

## Group 6

Crypto-TMS is a compliance/forensics dashboard built around the Elliptic Bitcoin transaction dataset. The project compares:

1. **Method 1 — Tabular Random Forest:** transaction-level features only.
2. **Method 2 — Graph-Feature Random Forest:** transaction features plus hand-built graph features.
3. **Optional GCN:** not required for the core submission.

The dashboard is a demonstration interface for inspecting real precomputed test-set predictions and transaction-network structure.

![Crypto-TMS dashboard screenshot](dashboard-screenshot.png)
*Live view: 16,670 real test-set transactions loaded, with per-model verdicts, transaction lookup, and a 1-hop neighbor graph.*

---

## 1. Dataset

The project uses the Elliptic Bitcoin transaction dataset.

- Transaction nodes: **203,769**
- Graph edges: **234,355**
- Training transactions: **29,894**
- Test transactions: **16,670**
- Test labels:
  - Licit: **15,587**
  - Illicit: **1,083**

Because the task is temporal, the project uses a temporal split rather than a random split:

- **Training:** time steps 1–34
- **Testing:** time steps 35–49

This prevents future transactions from being randomly mixed into the training data.

---

## 2. Machine-Learning Methods

### Method 1 — Tabular Random Forest

Uses the original transaction-level features.

Test performance for the illicit class:

- Precision: **0.8026**
- Recall: **0.7322**
- F1: **0.7658**

### Method 2 — Graph-Feature Random Forest

Adds structural information from the transaction graph:

- In-degree
- Out-degree
- PageRank
- Neighborhood illicit fraction

Test performance for the illicit class:

- Precision: **0.8731**
- Recall: **0.7304**
- F1: **0.7954**

### Main finding

Adding graph features improved illicit precision from **80.26% to 87.31%** and illicit F1 from **76.58% to 79.54%**. Recall remained approximately unchanged at 73%.

This means graph structure helped the model make more precise illicit predictions, but it did not substantially increase the number of illicit transactions detected. A likely reason recall didn't move: the illicit transactions both models still miss tend to look structurally ordinary (typical in-degree/out-degree/PageRank), so graph features sharpen confidence on cases that already look suspicious rather than surfacing new ones.

### Reading the F1-over-time chart (`f1_over_time.png`)

The chart shows illicit F1 falling to near zero after time step 43 for both methods. **This is not the models breaking** — it's a small-sample artifact. The count of illicit-labeled transactions per test time step drops sharply in the same window:

| Time step | 40 | 41 | 42 | 43 | 44 | 45 | 46 | 47 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| Illicit count | 112 | 116 | 239 | **24** | **24** | **5** | **2** | 22 |

With as few as 2–5 positive examples in a step, a single misclassification swings F1 by 20–50 percentage points, so F1 becomes statistically unstable rather than meaningful. This scarcity likely reflects a genuine drop in illicit activity late in the Elliptic timeline, not a bug in the pipeline. Treat the whole-test-set numbers in the table above (dominated by the earlier, well-populated steps) as the headline result, and treat the post-step-43 portion of the F1 chart as evidence of *when metrics stop being trustworthy*, not evidence of model failure.

---

# 3. How to Run the Dashboard — IMPORTANT

## Option B: Standalone Dashboard

The submitted standalone dashboard is designed to work without Live Server and without requiring the browser to fetch a local JSON file automatically.

### Files that must be together

Put these two files in the same folder:

```text
Crypto-TMS-dashboard.html
dashboard_data.json
```

You may also keep the notebook, CSV, image, and report in the same project folder.

### Step 1 — Open the dashboard

Double-click:

```text
Crypto-TMS-dashboard.html
```

It will open in a browser.

### Step 2 — Load the real project data

At the top of the dashboard, click:

```text
Choose File
```

Select:

```text
dashboard_data.json
```

from the project folder.

The dashboard should then show:

```text
16670 records loaded
```

and populate the summary cards.

### Step 3 — Select a transaction

There are two ways:

#### Random Transaction

Click:

```text
Random Transaction
```

The dashboard selects one test transaction automatically.

#### Transaction Lookup

Enter a transaction ID in the Transaction Lookup box and click:

```text
Scan
```

The transaction must exist in the exported test-set records.

### Step 4 — Inspect the model verdicts

The dashboard displays:

- Method 1 — Tabular Random Forest verdict
- Method 2 — Graph-Feature Random Forest verdict
- Ground-truth test label
- Illicit probability for each model

### Step 5 — Inspect graph information

The dashboard displays:

- Time step
- In-degree
- Out-degree
- PageRank
- One-hop transaction neighbors

The graph visualization shows the selected transaction and its exported neighboring transactions.

---

# 4. Important Dashboard Interpretation

The dashboard is a **forensic demonstration interface**, not an autonomous financial decision system.

The model predictions are **precomputed** from the notebook and exported into `dashboard_data.json`.

The dashboard does not retrain the Random Forest models in the browser.

Therefore:

```text
Notebook
   ↓
Train models
   ↓
Generate test predictions
   ↓
Export dashboard_data.json
   ↓
Dashboard loads the exported results
   ↓
User investigates transactions
```

---

# 5. How the Instructor Can Demonstrate the Project

The instructor can demonstrate the project in approximately 2–3 minutes:

### Step 1
Open:

```text
Crypto-TMS-dashboard.html
```

### Step 2
Click **Choose File** and select:

```text
dashboard_data.json
```

Confirm that:

```text
16670 records loaded
```

appears.

### Step 3
Click **Random Transaction**.

### Step 4
Show the:

- Transaction ID
- Tabular RF prediction
- Graph-Feature RF prediction
- Ground truth

### Step 5
Show the graph statistics:

- Degree
- PageRank
- Time step

### Step 6
Explain the comparison:

> The tabular model uses transaction features only. The graph-enhanced model adds structural information from the transaction network. Graph features increased illicit precision from 0.803 to 0.873 and illicit F1 from 0.766 to 0.795.

---

# 6. How to Reproduce the Machine-Learning Experiment

Open:

```text
elliptic_money_laundering_detection.ipynb
```

Run the notebook cells in order.

The main workflow is:

```text
Load Elliptic data
       ↓
Clean and align transactions
       ↓
Temporal train/test split
       ↓
Train Tabular Random Forest
       ↓
Build transaction graph
       ↓
Compute graph features
       ↓
Train Graph-Feature Random Forest
       ↓
Evaluate precision / recall / F1
       ↓
Evaluate F1 over time
       ↓
Export dashboard predictions
```

The core models are evaluated on future test time steps rather than a random sample.

---

# 7. Why Temporal Evaluation Matters

Random splitting would allow transactions from later periods to appear in training while earlier transactions appear in testing. That can produce an unrealistically optimistic estimate.

A temporal split better represents deployment:

```text
Past transactions → training
Future transactions → testing
```

This is especially important for illicit-transaction detection because transaction behavior can change over time.

The notebook evaluates illicit F1 by test time step and examines the distribution-shift behavior in the later timeline.

---

# 8. Evaluation

The test set is imbalanced:

```text
Licit:    15,587
Illicit:   1,083
```

Therefore, accuracy alone is not sufficient.

The most important metrics are:

- Illicit precision
- Illicit recall
- Illicit F1

### Final comparison

| Method | Illicit Precision | Illicit Recall | Illicit F1 |
|---|---:|---:|---:|
| Tabular RF | 0.8026 | 0.7322 | 0.7658 |
| Graph-Feature RF | **0.8731** | 0.7304 | **0.7954** |

---

# 9. Project Limitations

- The illicit class is substantially smaller than the licit class.
- Future transaction behavior can differ from historical training behavior. In particular, illicit-labeled transactions per test time step drop to single digits after time step 43, which makes illicit F1 highly volatile late in the test window — a small-sample effect layered on top of genuine temporal shift, not a sign the models fail outright.
- Graph features are hand-engineered rather than learned end-to-end.
- The dashboard uses precomputed predictions rather than performing live model inference.
- The system is a research/educational prototype and should not be treated as a production AML decision system.
- A GCN is an optional extension and is not required for the core two-model comparison.

---

# 10. Recommended Submission Files

```text
Crypto-TMS/
│
├── elliptic_money_laundering_detection.ipynb
├── Crypto-TMS-dashboard.html
├── dashboard_data.json
├── comparison.csv
├── f1_over_time.png
├── Crypto-TMS_Final_Report.docx
└── README.md
```

---

## Final conclusion

The project demonstrates that transaction-network structure provides additional predictive information beyond tabular transaction features. The graph-feature Random Forest improved illicit-class precision and F1 while maintaining approximately the same recall. Temporal evaluation is essential because financial transaction behavior can shift over time.
