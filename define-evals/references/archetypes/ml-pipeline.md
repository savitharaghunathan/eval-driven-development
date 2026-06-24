# ML Pipeline Archetype

## Signature Patterns

Training, inference, model serving, feature engineering, data preprocessing, model versioning. Explicitly non-LLM or mixed LLM/traditional ML.

**Disambiguation:** Use ml-pipeline when there is a trained model (inference, training, fine-tuning). Use data-workflow when there is no trained model — pure ETL, reporting, analytics. If a system has both, match both archetypes; model-specific items come from ml-pipeline, data plumbing items from data-workflow.

**Examples:** Classification service, recommendation engine, anomaly detection, forecasting, computer vision pipeline.

## Extraction Checklist

When analyzing a spec for ML pipeline systems, look for:

- Model type and architecture (what kind of model?)
- Training data requirements (source, size, labeling, splits)
- Feature engineering steps (transformations, derived features)
- Inference interface (API, batch, streaming?)
- Performance metric requirements (accuracy, precision, recall, F1, AUC, etc.)
- Data quality requirements (schema validation, missing values, outliers)
- Drift detection requirements (data drift, concept drift)
- Model versioning and rollback criteria
- Training reproducibility requirements
- Bias and fairness requirements

## Default Eval Dimensions

- Model performance (accuracy/precision/recall/F1/AUC as specified)
- Data quality compliance (schema valid, no unexpected nulls/outliers)
- Drift detection rate (catches distribution shifts)
- Training reproducibility (same data + config = same model within tolerance)
- Feature correctness (transformations produce expected outputs)
- Inference latency and throughput
- Rollback trigger accuracy (correctly identifies when to revert)
- Bias/fairness metrics (if specified in requirements)

## Default Grader Patterns

| Requirement Type | Grader Type | Notes |
| ---------------- | ----------- | ----- |
| Model performance | statistical validation | Confidence intervals on metrics |
| Data quality compliance | code-based | Schema validation, null checks |
| Drift detection rate | statistical validation | KS test, distribution comparison |
| Training reproducibility | statistical validation | Variance across runs |
| Feature correctness | code-based | Compare transformed values |
| Inference latency/throughput | code-based | Measure timing |
| Rollback trigger accuracy | outcome verification | Inject regression, check trigger |
| Bias/fairness metrics | statistical validation | Compare across groups |

## Common Failure Modes

- Model accuracy reported on test set doesn't hold on production distribution (data leakage)
- Drift goes undetected because monitoring only checks input distributions, not predictions
- Training is non-reproducible due to random seeds, data ordering, or library version differences
- Feature engineering introduces subtle bugs that only manifest on edge cases
- Rollback criteria are too sensitive (false alarms) or too lax (missed regressions)
- Bias in training data propagates to model predictions undetected

## Example Eval Tasks

```yaml
- id: accuracy-within-threshold
  requirement: Model accuracy meets specified threshold with statistical significance
  description: Evaluate model on held-out test set and verify accuracy with confidence interval
  input: Test dataset of 500 labeled examples
  expected_behavior: Accuracy ≥ 92% with 95% confidence interval
  reference_solution: Model scores 94.2% accuracy, 95% CI [92.8%, 95.6%], lower bound ≥ 92%
  grader_type: statistical
  grader_logic: Compute accuracy on test set. Calculate 95% confidence interval. PASS if lower bound of CI ≥ 92%.
  category: capability
  priority: P0

- id: drift-detection-on-shifted-data
  requirement: Pipeline detects distribution shift in incoming data
  description: Feed pipeline data with a gradual shift in feature distributions
  input: 1000 records where feature_a distribution has shifted by 0.5 standard deviations from training data
  expected_behavior: Drift detection triggers an alert
  reference_solution: KS test on feature_a returns p < 0.05, pipeline logs drift alert
  grader_type: statistical
  grader_logic: Run KS test between training and test distributions. PASS if p-value < 0.05 and alert was generated.
  category: capability
  priority: P1
```
