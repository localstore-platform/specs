# Pull Request Template — ML / Models

## Summary

- Short description of model/data change and objective (e.g., new ranking model v2).

## Related issue / ticket

- Issue: [JIRA/Issue #]

## Type of change

- [ ] Model training / architecture change
- [ ] Data pipeline change
- [ ] Evaluation / metrics only
- [ ] Docs / model card

## Data and Reproducibility

- Dataset(s) touched (paths / versions)
- Training code commit / experiment id
- Random seeds and environment used

## Metrics & Evaluation

- Provide evaluation metrics (validation/test) and compare to baseline.

## Artifacts

- Model artifact location (S3/GCS path) and tag
- Inference / serving changes and backward compatibility notes

## How to validate

1. Unit tests for data transforms
2. Run a small-local training with provided seed
3. Smoke test inference against staging endpoint

## Checklist

- [ ] Unit tests for data/feature code
- [ ] Reproducible training script included
- [ ] Model card updated
- [ ] Monitoring and drift metrics added for production
