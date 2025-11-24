README Template — local-store-infra

Purpose
Infrastructure and deployment repository. Hosts IaC, Helm charts, k8s manifests, and environment promotion scripts.

Quick summary

- Tech: Terraform, Helm, Kubernetes, GitHub Actions (or similar)
- Audience: DevOps/SRE, backend engineers
- Responsibility: Environment provisioning, deployment pipelines, shared infra modules

Suggested layout

- /terraform/
- /helm/
- /k8s/
- /ci/ (pipeline templates and actions)

Guidelines

- Keep secrets in org secret store; reference them via CI.
- Use blue/green or canary deployments for production traffic.

Ownership

- CODEOWNERS: SRE lead

Notes

- Use immutable image tags and promote using an automated pipeline rather than manual edits to manifests.
