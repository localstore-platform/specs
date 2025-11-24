# Pull Request Template — Infra / Terraform

## Summary

- Short description of infra change (resources affected, cloud provider).

## Related issue / ticket

- Issue: [JIRA/Issue #]

## Type of change

- [ ] Terraform change
- [ ] Helm / K8s manifests
- [ ] Cloud resource change

## Plan & State

- Attach or link to `terraform plan` (inline snippet or URL to run output).
- State changes: list resources to be created/updated/destroyed.

## Risk & Rollback

- Downtime required? (yes/no)
- Rollback steps (how to revert state)

## Security & Cost

- IAM / network scope changes
- Cost impact estimate

## Checklist

- [ ] `terraform fmt` and `terraform validate` passed
- [ ] `terraform plan` attached and reviewed
- [ ] State locking and secrets handling verified
- [ ] Monitoring and alerting updated where needed
