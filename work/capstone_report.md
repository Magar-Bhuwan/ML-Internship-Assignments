# Can Historical Content Signals Help Prioritize Content for Review?

## Abstract

Can historical content-performance signals and content freshness be used to prioritize which content should be reviewed first?

Using 30,000 anonymized FlyRank content records, this study combines freshness, visibility, and historical performance signals to identify content with a declining-performance target.

A Random Forest classifier was evaluated against a simple freshness-and-visibility baseline using a grouped-by-client test split to reduce client-level overlap between training and testing data.

On that shared test split, the Random Forest achieved 0.6798 precision, 0.7989 recall, and 0.7345 F1, compared with 0.6389 precision, 0.0130 recall, and 0.0255 F1 for the baseline.

The result supports using historical signals to prioritize a human content-review queue, while the observational data and lack of a controlled refresh experiment mean the model should be treated as decision support rather than proof that refreshing content will improve future performance.

## 1. Problem Framing

Content teams rarely have unlimited time to review every existing page.

When a large content portfolio contains pages with different levels of freshness, visibility, and performance change, a practical question is which pages deserve human review first.

This capstone addresses that content-prioritization problem in the FlyRank Product Performance context.

The research question is:

> Can historical content-performance signals and content freshness be used to prioritize which content should be reviewed first?

The goal is not to automatically decide which content should be changed. The goal is to provide stronger evidence for deciding where limited review effort should be focused.

## 2. Data Safety

The analysis uses 30,000 anonymized FlyRank content records with 44 columns.

Important signals include:

* `days_since_last_update`
* `impressions_90d`
* `clicks_90d`
* `trend_direction`
* `trend_pct`

No client names, private URLs, search queries, or other client-identifying information are used in the report.

The dataset is treated as observational data.

The results are therefore framed as observed and directional rather than causal.

## 3. Baseline

The baseline uses two simple signals:

* `days_since_last_update >= 104`
* `impressions_90d >= 3,616`

The purpose of the baseline is to provide a transparent comparison against the machine-learning approach.

On the grouped-client test split:

| Model         | Precision | Recall |     F1 |
| ------------- | --------: | -----: | -----: |
| Baseline      |    0.6389 | 0.0130 | 0.0255 |
| Random Forest |    0.6798 | 0.7989 | 0.7345 |

## 4. Model / Analysis

The target is:

`declining_target = (trend_direction == "down").astype(int)`

A Random Forest classifier was trained using historical content and performance signals.

The analysis uses grouped-by-client validation so that records from the same client are not placed in both the training and testing partitions.

The goal is prioritization rather than automatic content modification.

## 5. Evaluation

The final comparison uses the same grouped-by-client test split for both the baseline and Random Forest.

The Random Forest achieved:

* Precision: 0.6798
* Recall: 0.7989
* F1: 0.7345

The baseline achieved:

* Precision: 0.6389
* Recall: 0.0130
* F1: 0.0255

The strongest difference is recall: the Random Forest identified substantially more declining-content cases than the simple baseline on this test split.

## 6. Interpretation

The result suggests that combining multiple historical content-performance signals can provide a stronger prioritization signal than the simple freshness-and-visibility rule.

However, the result should not be interpreted as proof that the identified content will improve after being refreshed.

The analysis is observational, the thresholds are dataset-specific, and the evaluation does not establish that changing content causes future performance improvement.

## 7. Recommendation

The highest-priority review group is stale and visible content:

* `days_since_last_update >= 104`
* `impressions_90d >= 3,616`

These items should enter a human review queue.

The recommended workflow is:

**Historical signals → ranked review queue → human validation → decision**

The model should not automatically delete content, publish rewrites, or make major content changes without human review.

## 8. Reproducibility

Main notebook:

`work/notebooks/capstone.ipynb`

Capstone report:

`work/capstone_report.md`

The analysis uses fixed random-state settings for the validation process.

The notebook should be run from top to bottom to reproduce the analysis.

The deployed research paper URL is stored in:

`submission/paper_url.txt`

## Limitations

This analysis provides directional, dataset-specific evidence rather than causal proof.

The dataset does not provide a fully verified prediction timestamp and completely separated future outcome window for production forecasting.

The 104-day freshness threshold and 3,616-impression visibility threshold are dataset-specific and should be revalidated before operational use.

The final queue should therefore be treated as decision support for human reviewers.

## Conclusion

Historical content-performance signals can provide useful evidence for prioritizing content review.

The Random Forest performed substantially better than the simple baseline on the grouped-client evaluation.

The practical recommendation is to use the model as a review-prioritization tool, followed by human validation, rather than as an automatic content-change system.
