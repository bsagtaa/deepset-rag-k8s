# Addressing Advanced Deployment Requirements

## Airgapped Deployment

### Solution
Local Container Registry

How it works:
Set up a self-hosted registry (e.g., Docker Registry) within your airgapped network. Pull required images from public repositories using an internet-connected machine, then push them to your local registry. Update the Helm chart’s values.yaml to point to the local registry URLs.


How it works:
Mirror relevant Git repositories within the airgapped network. Set up GitOps tools like ArgoCD pointing to these internal repositories, and configure synchronization with a private image registry for a smooth workflow.

## Integration with Alternative Password Stores
### Solution
External Secrets Operator:

Add the ExternalSecret CRDs to your templates and configure SecretStore resources to point to your chosen secret management backend (e.g., HashiCorp Vault, AWS Secrets Manager).
Replace Kubernetes Secret objects with ExternalSecret references.

HashiCorp Vault:

Implement Vault Agent sidecar pattern to inject secrets into your pods, and use annotations for secret path references.
Why it’s useful:

Centralizes credentials management and streamlines secret rotation without restarting apps.

Monitoring and Logging Integration
1. Monitoring Integration
Deploy kube-prometheus-stack for a comprehensive open-source monitoring solution. For higher availability, you can integrate Thanos or Mimir.
Use dedicated nodes for monitoring to ensure they have enough resources to scale.
2. Logging Pipeline
Use Grafana Loki or ELK stack for logging, and set up Fluent Bit/Fluentd DaemonSets to collect and structure logs.
Forward logs to either Loki or Elasticsearch, and set up log retention policies.