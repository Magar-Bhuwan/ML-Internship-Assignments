# FlyRank Capstone — Content Review Prioritization

## 1. Project Overview

This project builds a machine-learning workflow to help prioritize which content should be reviewed first.

The project uses historical content-performance signals and content freshness information from an anonymized FlyRank dataset. The goal is to turn these signals into a practical review-prioritization workflow that can help focus limited review effort on content that appears more likely to require attention.

This is a **decision-support system**. It does not claim to predict Google's algorithm or guarantee that changing or refreshing content will improve future performance.

---

## 2. Research Question

**Can historical content-performance signals and content freshness be used to prioritize which content should be reviewed first?**

### Decision Supported

The analysis supports a practical content-review decision:

> Which content should be examined first when review resources are limited?

The model produces a classification signal that can be used as an input to a human review process.

---

## 3. Who Is This For?

This project is intended for people who need to prioritize content-review work using available performance data, including:

* Content and SEO teams
* Data analysts
* Performance teams
* ML practitioners working with ranking or prioritization problems
* Teams that need a data-driven review queue

The output should be treated as **decision support**, not as an automatic replacement for human review.

---

## 4. Dataset

The project uses the anonymized FlyRank content-refresh dataset.

### Dataset characteristics

* Records: approximately 30,000
* Features: 44 columns
* Data type: anonymized content and performance information
* Data includes historical search, traffic, engagement, content-age, freshness, position, and trend-related signals.

Examples of available fields include:

* `search_volume`
* `competition`
* `cpc`
* `content_type`
* `main_intent`
* `word_count`
* `impressions_90d`
* `clicks_90d`
* `pageviews_90d`
* `sessions_90d`
* `content_age_days`
* `days_since_last_update`
* `ctr`
* `avg_position`
* `engagement_rate`
* `trend_direction`
* `trend_pct`

No private client information should be added to this public repository.

---

## 5. Workflow

The project follows this workflow:

```text
Historical Content Data
          ↓
Data Inspection & Cleaning
          ↓
Feature Preparation
          ↓
Target Definition
          ↓
Baseline Model
          ↓
Random Forest Model
          ↓
Grouped-by-Client Validation
          ↓
Performance Comparison
          ↓
Content Review Prioritization
          ↓
Human Review / Decision Support
```

The key idea is that the machine-learning model supports a review queue rather than automatically changing content.

---

## 6. Model

The main machine-learning model is a **Random Forest Classifier**.

The final model uses:

* 300 trees
* `random_state=42`
* balanced class weights

The model is evaluated using a grouped-by-client split so that records belonging to the same client are not placed in both the training and testing sets.

This provides a more conservative estimate of how the model performs on clients that were not represented in training.

---

## 7. Evaluation

The final grouped-by-client evaluation produced the following results:

| Method         |  Precision |     Recall |         F1 |
| -------------- | ---------: | ---------: | ---------: |
| ML-07 Baseline |     0.6389 |     0.0130 |     0.0255 |
| Random Forest  | **0.6798** | **0.7989** | **0.7345** |

The Random Forest achieved higher precision, substantially higher recall, and a much higher F1 score than the simple baseline on the same grouped-client test split.

The Random Forest achieved:

* Precision: **67.98%**
* Recall: **79.89%**
* F1: **73.45%**

The baseline achieved:

* Precision: **63.89%**
* Recall: **1.30%**
* F1: **2.55%**

These results are specific to this dataset and validation design and should not be interpreted as guaranteed production performance.

---

## 8. Why Grouped-by-Client Validation?

A key design decision was to use a grouped-by-client train/test split.

A random split can allow records from the same client to appear in both training and testing data. That can make the model appear stronger than it would be when applied to a genuinely unseen client.

The grouped-by-client split prevents this overlap and therefore provides a more conservative test of generalization.

This makes the evaluation more useful for understanding whether the workflow can transfer beyond the exact client records used during training.

---

## 9. Usage

### Option A — Google Colab

1. Open the repository on GitHub.
2. Open:

```text
work/notebooks/capstone.ipynb
```

3. Open the notebook in Google Colab.
4. Run the cells from top to bottom.
5. Review the dataset inspection and feature preparation.
6. Review the baseline and Random Forest results.
7. Review the grouped-by-client evaluation.
8. Review the final comparison and recommendations.

### Option B — Local Environment

Clone the repository:

```bash
git clone https://github.com/Magar-Bhuwan/ML-Internship-Assignments.git
cd ML-Internship-Assignments
```

Create and activate a virtual environment:

```bash
python -m venv .venv
```

Windows:

```bash
.venv\Scripts\activate
```

Linux/macOS:

```bash
source .venv/bin/activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Then open:

```text
work/notebooks/capstone.ipynb
```

and run the notebook.

---

## 10. Reproducibility

The notebook uses a fixed random seed for the Random Forest:

```text
random_state = 42
```

The evaluation results reported above are generated from the notebook's grouped-by-client evaluation.

The exact results may depend on the Python and library versions used when the notebook is executed.

---

## 11. Limitations

This project has several important limitations.

### 1. Observational data

The dataset is observational. The model identifies patterns associated with the target but does not establish that changing or refreshing content causes future performance improvement.

### 2. Dataset-specific thresholds

The signals and thresholds used in the workflow are based on the available dataset. They may not transfer directly to another dataset or production environment.

### 3. Historical information

The model relies on historical performance signals. Future search behavior, competition, content quality, or other external conditions may differ.

### 4. Human review is still required

The model should be used to prioritize review, not to automatically change content without human validation.

### 5. Generalization

Although grouped-by-client validation provides a more conservative evaluation, further testing on additional data and future time periods would be needed before treating the approach as a production system.

---

## 12. AI Transparency

I built this project with assistance from AI tools, including Claude. AI assistance was used for tasks such as brainstorming, explaining code and concepts, debugging, structuring documentation, and reviewing parts of the workflow.

I remained responsible for the final project decisions, reviewed the implementation, checked the notebook outputs, and verified the reported evaluation results against the project data.

The final repository and conclusions should therefore be understood as work that I developed with AI assistance and personally reviewed and validated.

---

## 13. Recommended Decision Workflow

The intended workflow is:

```text
Historical Signals
       ↓
ML Prioritization
       ↓
Ranked Review Queue
       ↓
Human Validation
       ↓
Content Decision
```

The model should support human decision-making rather than automatically applying content changes.

---

## 14. Repository

GitHub repository:

https://github.com/Magar-Bhuwan/ML-Internship-Assignments

Main capstone notebook:

```text
work/notebooks/capstone.ipynb
```

Research paper URL:

https://magar-bhuwan.github.io/ML-Internship-Assignments/


Demo video:

https://youtu.be/kKvHFwMvViU
