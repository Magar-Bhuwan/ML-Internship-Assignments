# Prioritizing Content Review with Historical Search Performance Signals

## Google Search Ranking & Discoverability Capstone

**Lane:** Refresh / Content Opportunity Scoring
**Project:** FlyRank ML Internship
**Primary notebook:** `work/notebooks/capstone.ipynb`

---

## Abstract

Content teams often have more pages to review than they can manually evaluate, creating a need for a repeatable way to prioritize content-review effort. This project uses anonymized FlyRank search-performance data to investigate whether historical content visibility, engagement, freshness, and related search signals can help identify content associated with declining performance and prioritize review candidates. A Random Forest classifier is compared with a transparent baseline using a grouped-by-client validation design intended to reduce the risk of having pages from the same client represented in both training and testing data. The evaluation shows that the learned model provides stronger classification performance than the simple baseline on the held-out evaluation data, while the transparent action playbook identifies stale and visible content as the highest-priority review group. The resulting system is intended as directional decision-support evidence for human content reviewers, not as proof of Google's ranking algorithm or as evidence that refreshing a page will cause future search-performance improvement.

---

## 1. Introduction / Problem Statement

### 1.1 Problem

Content teams may manage a large number of pages while having limited time available for content review and refresh work. Reviewing every page with the same priority is inefficient.

The practical decision addressed by this project is:

> **Which content should be reviewed first based on observable historical search-performance and freshness signals?**

The goal is therefore not to predict Google's ranking algorithm. Instead, the goal is to build a repeatable analysis that helps a human reviewer prioritize where to spend attention.

### 1.2 Search Intelligence Lane

This project follows the **Refresh / Content Opportunity Scoring** lane.

The workflow combines:

1. historical search and content-performance signals,
2. a supervised classification model,
3. a transparent baseline,
4. validation designed to reduce client-level leakage,
5. and a rule-based action playbook for producing ranked review recommendations.

### 1.3 Decision Supported

The output supports a simple operational decision:

> **Review high-priority content first, then manually determine whether a refresh, improvement, monitoring action, or no change is appropriate.**

A high priority score does not automatically mean that content must be refreshed.

---

# 2. Data

## 2.1 Dataset

The analysis uses an anonymized FlyRank content-performance dataset containing:

* **30,000 records**
* **44 columns**

The dataset contains search, content, visibility, engagement, and freshness-related signals.

The analysis was performed on public-safe/anonymized data and does not expose client names, domains, URLs, private search queries, credentials, or raw private exports.

## 2.2 Relevant Signals

Examples of signals used in the analysis include:

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
* `content_age_days`
* `days_since_last_update`
* `ctr`
* `avg_position`
* `engagement_rate`
* `scroll_rate`
* `ai_traffic_pct`

These variables provide different views of content demand, visibility, engagement, and freshness.

## 2.3 Target Definition

The supervised task defines a binary declining-performance target from the observed trend direction:

```text
declining_target = (trend_direction == "down")
```

The target therefore identifies records associated with a downward trend in the available dataset.

## 2.4 Exclusions and Leakage Protection

The fields used to define the target were not used as model predictors.

In particular:

* `trend_direction` was excluded from the feature set.
* `trend_pct` was excluded from the feature set.
* Other label-derived fields were excluded where appropriate.

This prevents the model from directly receiving the information it is supposed to predict.

The project also avoids presenting private client-level information in the public report.

---

# 3. Methodology

## 3.1 Overall Workflow

The analysis follows this workflow:

```text
FlyRank search/content data
        ↓
Data inspection and cleaning
        ↓
Feature selection
        ↓
Target definition
        ↓
Baseline construction
        ↓
Random Forest model
        ↓
Grouped validation
        ↓
Model vs baseline comparison
        ↓
Transparent action scoring
        ↓
Ranked review recommendations
```

## 3.2 Baseline

The baseline is deliberately simple and interpretable.

Two observable signals are used:

* `days_since_last_update`
* `impressions_90d`

The observed thresholds are:

```text
STALE_DAYS = 104
VISIBLE_IMPRESSIONS = 3616
```

A content item is considered stale when:

```text
days_since_last_update >= 104
```

It is considered visible when:

```text
impressions_90d >= 3616
```

The baseline score is:

```text
score =
    stale
    × visible
    × impressions_90d
```

Therefore, content receives a positive priority score when it is both stale and visible, with higher historical impressions producing a higher ranking.

This produces a transparent rule that can be explained to a non-technical content reviewer.

## 3.3 Action Labels

The baseline produces four reason categories:

| Condition       | Reason code         | Action     |
| --------------- | ------------------- | ---------- |
| Stale + visible | `stale_and_visible` | **Review** |
| Stale only      | `stale`             | Monitor    |
| Visible only    | `visible`           | Monitor    |
| Neither signal  | `no_strong_signal`  | Monitor    |

The top-ranked review queue is therefore based on a transparent combination of freshness and historical visibility.

## 3.4 Machine Learning Model

The supervised model is a **Random Forest Classifier**.

The model uses historical search-performance, content, engagement, and freshness features.

The model was configured with a balanced class weighting approach to account for the classification imbalance.

The model is used as a predictive decision-support component rather than as evidence of causality.

## 3.5 Validation Design

The evaluation uses a grouped-by-client holdout design.

Approximately 80% of clients are used for training and the remaining clients are held out for testing.

The purpose is to reduce the possibility that pages from the same client appear in both training and testing data.

This provides a more conservative estimate of how well the model may generalize across clients compared with a simple row-level random split.

## 3.6 Leakage Checks

The analysis explicitly checks that:

* the target is not included as a model feature;
* target-derived trend fields are excluded;
* the baseline does not use the target;
* the action queue is based on observable snapshot signals;
* future outcome information is not intentionally introduced into the baseline.

The dataset does not provide a fully verified future-window experiment for a causal refresh-impact claim. Therefore, the project is framed as historical association and decision support rather than future causal prediction.

---

# 4. Results

## 4.1 Model vs Baseline

The project compares the Random Forest against the simple baseline using the same evaluation framework.

The evaluation recorded the following held-out results in the capstone analysis:

| Method         | Precision | Recall |     F1 |
| -------------- | --------: | -----: | -----: |
| ML-07 Baseline |    0.5492 | 0.1097 | 0.1829 |
| Random Forest  |    0.6932 | 0.7599 | 0.7250 |

The Random Forest therefore produced substantially higher recall and F1 than the simple baseline in this evaluation.

The largest difference is recall: the baseline identified only a small fraction of declining items, while the Random Forest identified a substantially larger proportion.

These results support treating the learned model as a stronger predictive signal than the simple baseline on this held-out evaluation.

However, model performance should not be interpreted as proof that the model will improve content performance in production.

> **Important:** If the final evaluation cell of `capstone.ipynb` contains different final metrics, those final notebook values should replace the table above. The paper and notebook must report the same final evaluation.

## 4.2 Interpretation

The result suggests that declining-performance status is associated with a combination of search, content, visibility, engagement, and freshness signals rather than only one simple freshness threshold.

The baseline remains useful because it is easy to explain and operationalize.

The Random Forest provides a stronger predictive signal, while the transparent baseline provides an interpretable action mechanism.

These components therefore serve different purposes:

```text
Random Forest
    ↓
Predictive signal

Transparent baseline
    ↓
Operational prioritization

Human review
    ↓
Final content decision
```

---

# 5. Ranked Recommendations

## 5.1 Priority Framework

The final action playbook prioritizes content according to observable freshness and visibility.

### Priority 1 — Review

**Condition:**

```text
days_since_last_update >= 104
AND
impressions_90d >= 3616
```

**Reason code:**

```text
stale_and_visible
```

**Recommended action:**

> Review the content manually for freshness, accuracy, completeness, search intent alignment, and potential improvement opportunities.

This is the highest-priority group because it combines two observable signals:

* the content has not been updated recently;
* the content has meaningful historical visibility.

## 5.2 Priority 2 — Monitor Stale Content

**Condition:**

```text
days_since_last_update >= 104
AND
impressions_90d < 3616
```

**Reason code:**

```text
stale
```

**Recommended action:**

> Monitor and consider review if additional evidence indicates declining performance or strategic importance.

## 5.3 Priority 3 — Monitor Visible Content

**Condition:**

```text
days_since_last_update < 104
AND
impressions_90d >= 3616
```

**Reason code:**

```text
visible
```

**Recommended action:**

> Continue monitoring performance because the content currently has meaningful visibility but does not meet the freshness threshold.

## 5.4 Priority 4 — Monitor

**Condition:**

```text
days_since_last_update < 104
AND
impressions_90d < 3616
```

**Reason code:**

```text
no_strong_signal
```

**Recommended action:**

> Do not prioritize immediate review based on this baseline alone.

---

# 6. Top-20 Review Queue

The analysis produces a ranked Top-20 review queue.

The highest-ranked items satisfy the `stale_and_visible` condition and receive the `review` action.

The ranking is primarily driven by historical impressions among items that meet the same freshness threshold.

The Top-20 queue is intentionally treated as a **review candidate list**, not as an automatic refresh list.

A high score means that an item matches the prioritization rule strongly.

It does not prove that:

* the content is incorrect;
* the content must be refreshed;
* a refresh will improve rankings;
* or a refresh will increase clicks or traffic.

A human reviewer should determine whether the content is actually outdated, incomplete, inaccurate, or strategically important.

---

# 7. Limitations & Honest Framing

## 7.1 Observational Data

The analysis is based on historical observational data.

It does not constitute a randomized experiment.

Therefore, associations between features and declining performance should not be interpreted as causal effects.

## 7.2 No Causal Refresh Experiment

This project does not test:

> "If we refresh this page, will its Google performance improve?"

That would require a properly designed intervention and future outcome measurement.

The current project instead answers:

> "Which historical content-performance patterns can help prioritize content for human review?"

## 7.3 Temporal Limitation

The available dataset does not provide a fully independently verified prediction timestamp and future outcome window for a production forecasting claim.

Some historical 90-day metrics therefore require caution when interpreted as predictors of future performance.

For that reason, the model is presented as directional decision-support evidence rather than as a production forecasting guarantee.

## 7.4 Threshold Generalization

The baseline thresholds:

```text
104 days
3616 impressions
```

were derived from the observed dataset.

They should not automatically be treated as universal thresholds for every website or future dataset.

They should be recalibrated when the data distribution changes.

## 7.5 Model Errors

The Random Forest will produce false positives and false negatives.

A predicted decline does not guarantee that a page actually needs a refresh.

Likewise, a page not identified by the model may still deserve human review.

## 7.6 No Claim About Google's Algorithm

This project does not claim to discover, reverse-engineer, or prove Google's ranking algorithm.

The signals are treated as observed search-performance and content signals in the available FlyRank dataset.

The appropriate language is:

* observed;
* measured;
* associated;
* directional;
* decision-support.

---

# 8. Reproducibility

The complete analysis is available in the repository.

### Repository

`https://github.com/Magar-Bhuwan/ML-Internship-Assignments`

### Main capstone notebook

```text
work/notebooks/capstone.ipynb
```

### Supporting weekly notebooks

```text
work/notebooks/w01_research_question.ipynb
work/notebooks/w02_ml_task_framing.ipynb
work/notebooks/w03_data_contract.ipynb
work/notebooks/w03_feature_leakage_check.ipynb
work/notebooks/w04_signal_audit.ipynb
work/notebooks/w04_baseline_score.ipynb
work/notebooks/w05_model.ipynb
work/notebooks/w06_validation_audit.ipynb
work/notebooks/w07_action_playbook.ipynb
```

### Main reproducibility path

```text
Research question
→ task framing
→ data contract
→ leakage check
→ signal audit
→ baseline
→ model
→ validation
→ action playbook
→ capstone
```

The notebook contains the code and analysis required to reproduce the reported workflow using the approved project data.

---

# 9. Public-Safety / Data Handling

This public paper intentionally does not expose:

* client names;
* client domains;
* client URLs;
* private search queries;
* credentials;
* raw private exports;
* sensitive client-level information.

The recommendations are represented as anonymized content-level analytical outputs.

---

# 10. Conclusion

This project demonstrates a repeatable approach for prioritizing content-review effort using historical search and content-performance signals.

The Random Forest provided stronger predictive performance than the simple baseline in the held-out evaluation, while the transparent baseline converted freshness and visibility signals into an interpretable review queue.

The most actionable recommendation is to prioritize human review of content that is both sufficiently stale and historically visible, while treating other groups as monitoring candidates.

The overall result should be used as a decision-support workflow rather than an automated content-refresh system.

The next useful production step would be to validate the recommendations against a properly defined future performance window or controlled refresh experiment.

---

# 11. Acknowledgments & Data Credit

Built on the **FlyRank ML Internship dataset**.

[FlyRank](https://flyrank.ai)

This project was completed as part of the FlyRank ML Internship and follows the public-safe data handling requirements of the internship.
