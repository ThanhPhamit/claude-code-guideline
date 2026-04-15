---
globs:
  - '**/k8s/**/*.yaml'
  - '**/k8s/**/*.yml'
  - '**/kubernetes/**/*.yaml'
  - '**/kubernetes/**/*.yml'
  - '**/helm/**/*.yaml'
  - '**/helm/**/*.yml'
  - '**/charts/**/*.yaml'
  - '**/charts/**/*.yml'
---
# Kubernetes Rules

## Resource Management
- Always set `resources.requests` and `resources.limits` for CPU and memory
- Use `LimitRange` and `ResourceQuota` for namespace-level defaults

## Health Checks
- Always include `livenessProbe` and `readinessProbe`
- Add `startupProbe` for slow-starting applications
- Use appropriate probe types: httpGet for HTTP services, tcpSocket for TCP, exec for custom checks

## Security
- Never run containers as root without explicit justification
- Set `securityContext.runAsNonRoot: true` and `readOnlyRootFilesystem: true`
- Use dedicated `ServiceAccount` per workload (never `default`)
- Apply RBAC with least-privilege: prefer `Role` over `ClusterRole`
- Use `NetworkPolicy` for network segmentation (default-deny + explicit allow)
- Use `Secret` for sensitive data, never `ConfigMap`

## Image Management
- Never use `latest` tag — pin exact version or SHA digest
- Always pull from private registry (ECR, GCR, etc.)
- Set `imagePullPolicy: IfNotPresent` for tagged images

## Deployment
- Use `Deployment` for stateless, `StatefulSet` for stateful workloads
- Set `PodDisruptionBudget` for critical services
- Use `topologySpreadConstraints` or `podAntiAffinity` for high availability
- Always specify `namespace`

## Helm
- Pin chart versions in `Chart.yaml` dependencies
- Use `values.yaml` for defaults, environment-specific overrides via `-f`
- Validate with `helm template` and `helm lint` before install/upgrade
