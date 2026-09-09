# ML-Based Content Review Prioritization

## FlyRank ML Internship — Capstone Project

This project uses historical content-performance signals and content freshness to prioritize which content should be reviewed first.

The goal is to turn historical data into a practical, ranked content-review queue that helps limited review effort focus on content with stronger signals of needing attention.

> **Important:** This project is a decision-support system. It does not claim that refreshing content will automatically improve future performance.

---

## Project Overview

### Research Question

> **Can historical content-performance signals and content freshness be used to prioritize which content should be reviewed first?**

### Decision Supported

The analysis supports a practical content-review decision:

* identify content with stronger signals of review priority;
* rank those items for review;
* help human reviewers focus their limited time on the items most worth examining first.

The workflow is intentionally designed as:

**Historical signals → ranked review queue → human validation → decision**

rather than:

**Historical signals → automatic content change**

---

## Who This Is For

This workflow is intended for teams that need to prioritize content review or refresh work across a large collection of pages.

A reviewer can use the resulting queue to answer:

> "Which content should we look at first?"

The model provides a stronger prioritization signal than a simple hand-written rule, while the final decision remains with a human reviewer.

---

## Dataset

The project uses the public-safe anonymized FlyRank content-refresh dataset used in the earlier ML tasks.

**Raw data file:**

`data/raw/content_refresh_anonymized.csv`

### Dataset size

* **30,000 records**
* **44 columns**

Important variables include:

* `content_id`
* `client_id`
* `search_volume`
* `competition`
* `cpc`
* `word_count`
* `char_count`
* `impressions_90d`
* `clicks_90d`
* `pageviews_90d`
* `sessions_90d`
* `users_90d`
* `engaged_sessions_90d`
* `ai_sessions_90d`
* `days_with_impressions`
* `days_with_sessions`
* `content_age_days`
* `days_since_last_update`
* `ctr`
* `avg_position`
* `engagement_rate`
* `scroll_rate`
* `ai_traffic_pct`
* `trend_direction`
* `trend_pct`

### Important data definitions

`impressions_90d` represents observed impressions during the preceding 90-day period represented by the dataset record.

`days_since_last_update` represents the elapsed time since the content was last updated.

The available project materials do not provide a separately verified public release number or exact calendar start/end dates, so no release number or calendar dates are claimed here.

### Public-safe handling

The dataset is anonymized.

This project does not expose:

* private client names;
* private search queries;
* confidential identifiers;
* private domains or URLs;
* other sensitive client information.

---

## Machine Learning Task

The target is a binary classification problem.

The notebook defines the target as:

```python
declining_target = (trend_direction == "down").astype(int)
```

Therefore:

* `1` = content is classified as declining;
* `0` = content is not classified as declining.

The purpose of the classification model is to provide a prioritization signal for content review.

---

## Approach

The project compares two approaches:

1. A simple, interpretable baseline rule.
2. A Random Forest classifier.

### Overall workflow

```text
Anonymized FlyRank Dataset
          │
          ▼
Data Validation & Feature Preparation
          │
          ▼
     Target Definition
          │
          ├──────────────────────┐
          ▼                      ▼
Simple Baseline           Random Forest
Freshness + Visibility    Classifier
          │                      │
          └──────────┬───────────┘
                     ▼
       Grouped-by-Client Evaluation
                     │
                     ▼
              Model vs Baseline
                     │
                     ▼
          Ranked Review Queue
                     │
                     ▼
           Human Validation
                     │
                     ▼
                Decision
```

---

## Baseline

The baseline is intentionally simple and interpretable.

It uses two signals:

* `days_since_last_update`
* `impressions_90d`

The validated thresholds are:

```text
Staleness threshold: 104 days
Visibility threshold: 3,616 impressions
```

The baseline identifies content as a high-priority review candidate when:

```text
days_since_last_update >= 104
AND
impressions_90d >= 3,616
```

This produces the reason code:

```text
stale_and_visible
```

and the recommended action:

```text
review
```

The baseline is useful because a human reviewer can easily understand why an item was selected.

---

## Random Forest Model

The capstone uses a `RandomForestClassifier`.

The final grouped-client model uses:

* `n_estimators = 300`
* `random_state = 42`
* `class_weight = "balanced"`

The model uses the documented feature set from the ML-09 validation work, including historical search, content, traffic, engagement, freshness, and performance signals.

The model is evaluated against the same declining-content target as the baseline.

---

# Evaluation v2 — Final Grouped-by-Client Evaluation

The final evaluation uses an **80/20 grouped-by-client split**.

This means records belonging to the same client remain in the same partition instead of being split between training and testing.

This was selected as the more conservative evaluation design because it tests performance on clients that were not represented in the training partition.

The baseline and Random Forest are evaluated on the **same grouped-client test set**.

## Results

| Method            |  Precision |     Recall |         F1 |
| ----------------- | ---------: | ---------: | ---------: |
| ML-07 Baseline    |     0.6389 |     0.0130 |     0.0255 |
| **Random Forest** | **0.6798** | **0.7989** | **0.7345** |

### What the results show

The Random Forest achieved:

* **67.98% precision**
* **79.89% recall**
* **0.7345 F1**

The baseline achieved:

* **63.89% precision**
* **1.30% recall**
* **0.0255 F1**

The largest difference is recall:

> The baseline identified only **1.30%** of the declining-content cases, while the Random Forest identified **79.89%** on the same grouped-client test split.

F1 also increased substantially:

> **0.0255 → 0.7345**

These results indicate that the Random Forest provides a substantially stronger prioritization signal than the simple baseline on this evaluation.

However, these are **dataset- and validation-specific results**, not a guarantee of production performance.

---

## Validation Design Matters

The notebook also compares different validation designs.

An earlier random stratified split produced:

| Validation design       | Precision | Recall |     F1 |
| ----------------------- | --------: | -----: | -----: |
| Random stratified split |    0.6932 | 0.7599 | 0.7250 |

A previous grouped-by-client validation measured:

| Validation design       | Precision | Recall |     F1 |
| ----------------------- | --------: | -----: | -----: |
| Grouped-by-client split |    0.6275 | 0.6884 | 0.6566 |

The final capstone evaluation then reproduced the grouped-client setup and evaluated the ML-07 baseline on the **same test partition**, producing the final comparison shown above.

The difference between validation designs demonstrates that measured performance depends on how the data is split.

For this reason, the grouped-client evaluation is treated as the more conservative result for the final documentation.

---

## Key Finding

The main result is not simply that the Random Forest has a higher score.

The more useful result is that the model can convert historical signals into a **ranked content-review queue**.

Compared with the simple baseline, the Random Forest produced substantially stronger recall and F1 on the final grouped-client evaluation.

This makes it more useful as a decision-support signal for prioritizing content review.

---

# Ranked Recommendations

The final playbook translates the analysis into action-oriented categories.

## 1. Review stale and visible content first

**Highest priority**

Prioritize content satisfying:

```text
days_since_last_update >= 104
AND
impressions_90d >= 3,616
```

Reason code:

```text
stale_and_visible
```

Recommended action:

```text
review
```

These items are both sufficiently old and sufficiently visible to justify human review.

---

## 2. Monitor stale content

Content that is stale but does not meet the visibility threshold should be monitored rather than automatically changed.

Reason:

```text
stale
```

Action:

```text
monitor
```

---

## 3. Monitor visible but recently updated content

Content with strong visibility but without the staleness signal should also be monitored.

Reason:

```text
visible
```

Action:

```text
monitor
```

---

## 4. Monitor content with no strong signal

Content that satisfies neither strong prioritization signal should not automatically be escalated.

Reason:

```text
no_strong_signal
```

Action:

```text
monitor
```

---

## Top-20 Review Queue

The notebook generates a ranked **Top-20 content-review candidate queue**.

In the final output, all 20 top candidates satisfy the stale-and-visible condition and receive the `review` action.

The queue is therefore designed to give a reviewer a concrete starting point rather than automatically changing content.

---

# Outputs and Artifacts

The capstone produces the following artifacts:

### Notebook

`work/notebooks/capstone.ipynb`

This contains:

* research question;
* data documentation;
* data validation;
* target definition;
* baseline;
* Random Forest model;
* validation;
* v2 evaluation;
* model-vs-baseline results;
* recommendations;
* limitations;
* reproducibility notes;
* demo outline.

### Baseline Action Score

`work/outputs/baseline_action_score.csv`

Contains the ranked baseline output, including fields such as:

* rank;
* content ID;
* days since last update;
* impressions;
* score;
* reason code;
* recommended action.

### Content Action Playbook Queue

`work/outputs/content_action_playbook_queue.csv`

Contains the ranked action-oriented content-review queue used by the playbook.

### Figure

`work/figures/action_queue_by_action.png`

Visualizes the content action queue by recommended action.

---

# Reproducibility

The notebook is designed to run from top to bottom.

## Option 1 — Google Colab

The notebook is Colab-ready.

1. Open `work/notebooks/capstone.ipynb`.
2. Open it in Google Colab.
3. Make sure the repository is available in the Colab environment.
4. Run the notebook from the beginning.
5. Use **Runtime → Run all**.
6. Confirm that the dataset loads successfully.
7. Confirm that the model evaluation completes.
8. Confirm that the output CSV files and figure are generated.

The notebook expects the dataset at:

```text
data/raw/content_refresh_anonymized.csv
```

and generates artifacts under:

```text
work/outputs/
work/figures/
```

## Option 2 — Local Repository

Clone this repository from GitHub and open the project in VS Code or another Python/Jupyter environment.

Install the packages listed in:

```text
requirements.txt
```

Then open:

```text
work/notebooks/capstone.ipynb
```

Run the notebook from top to bottom.

The notebook uses standard Python data-science libraries including:

* pandas;
* NumPy;
* Matplotlib;
* scikit-learn.

---

# Repository Structure

The repository contains both the original FlyRank internship foundation and the completed work area.

```text
ML-Internship-Assignments/
│
├── README.md
├── SETUP.md
├── GUIDE.md
├── DATA_USE.md
├── AGENTS.md
├── CLAUDE.md
├── requirements.txt
│
├── data/
│   └── raw/
│       └── content_refresh_anonymized.csv
│
├── docs/
│   ├── data-dictionary.md
│   ├── ml-core-foundation-framework.md
│   ├── ml-intern-dataset-and-lane-guide.md
│   └── intern-free-tooling-guide.md
│
├── work/
│   ├── README.md
│   ├── notebooks/
│   │   └── capstone.ipynb
│   │
│   ├── outputs/
│   │   ├── baseline_action_score.csv
│   │   └── content_action_playbook_queue.csv
│   │
│   └── figures/
│       └── action_queue_by_action.png
│
└── submission/
    └── paper_url.txt
```

The `work/` directory is the main area for the internship lane experiments and capstone work.

---

# Data Safety

This project follows the public-safe data handling requirements of the internship.

* Only anonymized data is used.
* No private client names are included.
* No private queries are included.
* No confidential client identifiers are intentionally exposed.
* Results are described as observed, measured, directional, and decision-support evidence.
* The model is not presented as a system that predicts or controls a search engine's algorithm.
* Recommendations are not treated as automatic content changes.

---

# Limitations

This project provides **directional, dataset-specific evidence rather than causal proof**.

## 1. Observational data

The data is observational.

Relationships between freshness, visibility, and performance do not establish that one causes another.

Therefore, the project cannot claim:

> "Refreshing this content will cause its future performance to improve."

---

## 2. Validation design affects the result

Performance changes depending on the validation design.

The random split and grouped-client split produce different metrics.

Therefore, the reported metrics should not be interpreted as universal production performance.

---

## 3. Possible temporal leakage requires further verification

The dataset does not independently provide a verified prediction timestamp and a fully separated future outcome window for every feature.

Some rolling 90-day performance variables may therefore require additional temporal verification before production use.

---

## 4. Thresholds are dataset-specific

The following thresholds:

```text
104 days
3,616 impressions
```

were validated for this analysis.

They should be revalidated if:

* the data distribution changes;
* the content environment changes;
* the business context changes;
* the system is moved to production.

---

## 5. Limited signals

The final action playbook intentionally focuses on freshness and visibility.

It does not fully represent:

* business value;
* strategic importance;
* content quality;
* brand considerations;
* editorial priorities;
* other business context.

A human reviewer may have information that is not represented in the dataset.

---

## 6. Classification errors

The Random Forest can produce both:

* false positives;
* false negatives.

Therefore, predictions should support human review rather than replace it.

---

## 7. No causal refresh experiment

The project does not include a controlled experiment measuring the effect of refreshing content.

Therefore, the project cannot establish that following the recommendations will improve future content performance.

---

## 8. Human review is required

The final queue should not directly trigger automatic content changes.

The recommended workflow is:

```text
Historical signals
        ↓
Ranked review queue
        ↓
Human validation
        ↓
Decision
```

not:

```text
Historical signals
        ↓
Automatic content change
```

---

# Guardrails / No-Go Decisions

The model and queue should **not** be used to automatically:

* delete content;
* publish substantial rewrites;
* launch campaigns;
* promote content;
* change business rules or policies;
* assume that a high score guarantees improvement;
* act when required data is missing or invalid.

The system is intended to help a reviewer decide **what to examine first**, not to replace the reviewer.

---

# Demo Video

A 3–5 minute live demonstration accompanies this project.

The demo should show:

1. The research question.
2. The dataset and important signals.
3. The baseline and Random Forest approach.
4. The grouped-client evaluation.
5. The model-vs-baseline results.
6. The ranked review queue.
7. One design decision.
8. One important limitation or guardrail.

### Demo link

**YouTube:** `PASTE_YOUR_UNLISTED_YOUTUBE_LINK_HERE`

The video should demonstrate the actual notebook/workflow rather than relying on presentation slides.

---

# Suggested Demo Flow

The capstone notebook already contains a five-minute demo outline.

### 1. Question — ~45 seconds

Explain:

> Can historical content-performance signals and content freshness be used to prioritize which content should be reviewed first?

Then explain that the practical goal is to focus limited review effort on pages with stronger signals of needing attention.

### 2. Data and Method — ~1 minute

Show:

* 30,000 anonymized records;
* 44 columns;
* `days_since_last_update`;
* `impressions_90d`;
* `clicks_90d`;
* `trend_direction`;
* `trend_pct`.

Explain the declining-content target and the baseline thresholds.

### 3. Results — ~1 minute

Show the model-vs-baseline table:

| Model         | Precision | Recall |     F1 |
| ------------- | --------: | -----: | -----: |
| Baseline      |    0.6389 | 0.0130 | 0.0255 |
| Random Forest |    0.6798 | 0.7989 | 0.7345 |

### 4. Honest Result — ~1 minute

Highlight:

* baseline recall: **1.30%**
* Random Forest recall: **79.89%**
* baseline F1: **0.0255**
* Random Forest F1: **0.7345**

Then explain that these results are encouraging for prioritization but are not a production guarantee.

### 5. Recommendation — ~1 minute

Show the ranked queue and explain:

```text
days_since_last_update >= 104
AND
impressions_90d >= 3,616
```

means:

```text
reason = stale_and_visible
action = review
```

Finish with:

**Historical signals → ranked review queue → human validation → decision**

---

# Future Improvements

Possible next steps include:

1. Perform stronger temporal validation using verified prediction and future-outcome windows.
2. Revalidate the 104-day and 3,616-impression thresholds on new data.
3. Add business-value and strategic-priority features.
4. Perform deeper error analysis of false positives and false negatives.
5. Test additional models and compare them under the same grouped and temporal validation designs.
6. Evaluate ranking quality directly at useful review-queue sizes.
7. Run a controlled experiment to determine whether recommended content refreshes actually improve future performance.
8. Monitor model performance and data distribution if the workflow is ever used in production.

---

# Project Takeaway

The key lesson from this capstone is that machine learning is useful here not simply because it produces a higher metric.

Its practical value is in turning historical content signals into a **ranked, explainable review queue**.

The final result shows that the Random Forest substantially outperformed the simple baseline on the final grouped-client evaluation:

```text
Random Forest
Precision: 0.6798
Recall:    0.7989
F1:        0.7345
```

versus:

```text
ML-07 Baseline
Precision: 0.6389
Recall:    0.0130
F1:        0.0255
```

The appropriate interpretation is:

> Historical content-performance signals can provide useful evidence for prioritizing content review, but recommendations should remain human-validated and should not be treated as causal guarantees of future performance.

---

# Original Internship Repository Resources

This repository began as the FlyRank ML Internship starter repository.

Useful project documentation includes:

* `SETUP.md` — environment, GitHub, Colab, and data-access setup.
* `GUIDE.md` — repository structure and guidance on what to edit.
* `DATA_USE.md` — data-safety requirements.
* `docs/data-dictionary.md` — definitions for the dataset columns.
* `docs/ml-core-foundation-framework.md` — ML workflow and foundations.
* `docs/ml-intern-dataset-and-lane-guide.md` — dataset, lane, and capstone guidance.
* `docs/intern-free-tooling-guide.md` — free tooling guidance.
* `work/README.md` — documentation for the personal work area.
* `work/notebooks/capstone.ipynb` — the completed capstone analysis.

---

## License and Data

The repository code follows the repository's stated MIT licensing terms.

Dataset usage remains subject to the project's `DATA_USE.md` requirements.

---

## Status

**Capstone analysis:** Complete
**Final v2 grouped-client evaluation:** Complete
**Ranked recommendation queue:** Complete
**README documentation:** Complete
**Demo video:** Add final unlisted YouTube link after recording
**Paper deployment URL:** Maintained separately in `submission/paper_url.txt`
