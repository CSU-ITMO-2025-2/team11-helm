# LLAMATOR MCP Server - Helm Chart

[![Helm](https://img.shields.io/badge/Helm-3.0+-blue.svg)](https://helm.sh)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-1.25+-blue.svg)](https://kubernetes.io)

Production-ready Helm chart for deploying **LLAMATOR MCP Server** - a platform for LLM Red Teaming and security testing.

## Quick Start

### Prerequisites

- Kubernetes 1.25+
- Helm 3.0+
- External Secrets Operator (ESO) installed in cluster
- SecretStore `vault-team11` configured in your namespace

### 1. Create Secrets in Vault

```bash
# OpenAI API Key
vault kv put kv/team11/llamator/openai OPENAI_API_KEY="sk-..."

# API Key for client authentication
vault kv put kv/team11/llamator/api API_KEY="your-api-key"

# S3 credentials (optional)
vault kv put kv/team11/llamator/s3 \
  ACCESS_KEY_ID="..." \
  SECRET_ACCESS_KEY="..."
```

### 2. Install the Chart

```bash
# Install with default values
helm install llamator ./ -n team11-ns

# Or upgrade existing release
helm upgrade --install llamator ./ -n team11-ns
```

### 3. Verify Installation

```bash
# Check ExternalSecrets synced
kubectl get externalsecret -n team11-ns

# Check secrets created
kubectl get secret -n team11-ns

# Check pods
kubectl get pods -n team11-ns

# Run Helm tests
helm test llamator -n team11-ns
```

## Configuration

### Key Configuration Options

| Parameter | Description | Default |
|-----------|-------------|---------|
| `global.namespace` | Target namespace | `team11-ns` |
| `api.replicaCount` | Number of API replicas | `2` |
| `api.autoscaling.enabled` | Enable HPA for API | `true` |
| `worker.replicaCount` | Number of Worker replicas | `3` |
| `worker.autoscaling.enabled` | Enable HPA for Worker | `true` |
| `redis.enabled` | Enable Redis | `true` |
| `redis.persistence.enabled` | Enable Redis persistence | `true` |
| `ingress.enabled` | Enable Ingress | `true` |
| `externalSecrets.enabled` | Enable ESO integration | `true` |

### External Secrets Operator Configuration

Secrets are automatically synced from Vault via External Secrets Operator:

```yaml
vault:
  enabled: true
  secretStoreRef: "vault-team11"
  secretStoreKind: "SecretStore"
  refreshInterval: "1m"
  secretPath: "team11/llamator"
  remoteRef:
    openai:
      key: "team11/llamator/openai"
      property: "OPENAI_API_KEY"
    api:
      key: "team11/llamator/api"
      property: "API_KEY"
    s3:
      key: "team11/llamator/s3"
      accessKeyIdProperty: "ACCESS_KEY_ID"
      secretAccessKeyProperty: "SECRET_ACCESS_KEY"
```

### S3 Storage Configuration

```yaml
storage:
  s3:
    enabled: true
    endpoint: "https://s3.regru.cloud"
    bucket: "team11"
    region: ""
    existingSecret: "llamator-s3-keys"
```

## Monitoring

### Prometheus ServiceMonitor

```yaml
monitoring:
  serviceMonitor:
    enabled: true
    interval: 30s
    path: /metrics
    labels:
      release: prometheus
```

## Troubleshooting

### Check ExternalSecret status

```bash
kubectl describe externalsecret -n team11-ns
```

### Check pod logs

```bash
kubectl logs -n team11-ns -l app.kubernetes.io/name=llamator-mcp
```

### Verify secrets

```bash
kubectl get secret llamator-openai-keys -n team11-ns -o yaml
```

## Maintainers

- Roman Neronov - roman.nieronov@mail.ru
- Timur Nizamov - abc@nizamovtimur.ru
- Ilia Tambovtsev - tambovtsev.io@phystech.edu
- Alina Nikitina - alinanikitten@gmail.com

## Links

- [LLAMATOR MCP Server](https://github.com/CSU-ITMO-2025-2/team11-llamator-mcp)
- [Helm Documentation](https://helm.sh/docs/)
