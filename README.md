# LLAMATOR MCP Server - Helm Chart

[![Helm](https://img.shields.io/badge/Helm-3.0+-blue.svg)](https://helm.sh)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-1.25+-blue.svg)](https://kubernetes.io)

Production-ready Helm chart for deploying **LLAMATOR MCP Server** - a platform for LLM Red Teaming and security testing.

## Quick Start

### 1. Create Secrets

```bash
# Create OpenAI API keys secret
kubectl create secret generic llamator-openai-keys \
  --from-literal=LLAMATOR_MCP_ATTACK_OPENAI_API_KEY='YOUR_ATTACK_KEY' \
  --from-literal=LLAMATOR_MCP_JUDGE_OPENAI_API_KEY='YOUR_JUDGE_KEY' \
  --from-literal=LLAMATOR_MCP_TARGET_OPENAI_API_KEY='YOUR_TARGET_KEY' \
  --namespace <your-namespace>
```

### 2. Install the Chart

```bash
# Install with default values
helm install llamator ./ --namespace <your-namespace> --create-namespace

# Or with custom values
helm install llamator ./ -f values-example-production.yaml --namespace <your-namespace>
```

### 3. Verify Installation

```bash
# Check pods
kubectl get pods -n <your-namespace>

# Run Helm tests
helm test llamator -n <your-namespace>
```

## Configuration

### Key Configuration Options

| Parameter | Description | Default |
|-----------|-------------|---------|
| `api.enabled` | Enable API server | `true` |
| `api.replicaCount` | Number of API replicas | `1` |
| `api.autoscaling.enabled` | Enable HPA for API | `false` |
| `worker.enabled` | Enable Worker | `true` |
| `worker.replicaCount` | Number of Worker replicas | `1` |
| `redis.enabled` | Enable Redis | `true` |
| `redis.persistence.enabled` | Enable Redis persistence | `false` |
| `ingress.enabled` | Enable Ingress | `true` |
| `networkPolicy.enabled` | Enable NetworkPolicy | `true` |
| `vault.enabled` | Enable Vault integration | `false` |

### Example: Custom Configuration

```yaml
# custom-values.yaml
api:
  replicaCount: 3
  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 10

worker:
  replicaCount: 5
  autoscaling:
    enabled: true

redis:
  persistence:
    enabled: true
    size: 10Gi

ingress:
  hosts:
    - host: llamator.example.com
      paths:
        - path: /
          pathType: Prefix
```

```bash
helm install llamator ./ -f custom-values.yaml
```

### Configuration with Vault

```yaml
# values-vault.yaml
vault:
  enabled: true
  address: "https://vault.example.com"
  
  vaultSecretsOperator:
    enabled: true
    authMount: "kubernetes"
    role: "llamator-mcp"
    mount: "secret"
    secretPath: "llamator-mcp"
    refreshAfter: "5m"
    vaultConnectionRef: "vault-connection"
    createConnection: true

security:
  createExampleSecrets: false  # Important!
```

### Vault Secrets Structure

Create the following secrets in Vault:

```bash
# OpenAI API Keys
vault kv put secret/llamator-mcp/openai \
  LLAMATOR_MCP_ATTACK_OPENAI_API_KEY="..." \
  LLAMATOR_MCP_JUDGE_OPENAI_API_KEY="..." \
  LLAMATOR_MCP_TARGET_OPENAI_API_KEY="..."

# API Key (optional)
vault kv put secret/llamator-mcp/api \
  LLAMATOR_MCP_API_KEY="..."

# S3 Storage (optional)
vault kv put secret/llamator-mcp/storage \
  S3_ACCESS_KEY_ID="..." \
  S3_ACCESS_KEY="..."
```

## 🏭 Production Deployment

For production deployments, use `values-example-production.yaml` as a starting point:

```bash
# Copy and customize production values
cp values-example-production.yaml values-production.yaml
# Edit values-production.yaml with your settings

# Deploy
helm install llamator ./ -f values-production.yaml -n production
```

### Production Checklist

- [ ] Enable API key authentication (`security.apiKey.enabled: true`)
- [ ] Configure HPA for API and Worker
- [ ] Enable PodDisruptionBudgets
- [ ] Enable Redis persistence
- [ ] Configure proper resource limits
- [ ] Enable NetworkPolicy
- [ ] Configure TLS via cert-manager
- [ ] Set up monitoring (ServiceMonitor)
- [ ] Consider Vault for secrets management

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

### Pod Annotations for Prometheus

```yaml
monitoring:
  podAnnotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "8000"
    prometheus.io/path: "/metrics"
```

## Maintainers

- Roman Neronov - roman.nieronov@mail.ru
- Timur Nizamov - abc@nizamovtimur.ru
- Ilia Tambovtsev - tambovtsev.io@phystech.edu
- Alina Nikitina - alinanikitten@gmail.com

## Links

- [LLAMATOR MCP Server](https://github.com/CSU-ITMO-2025-2/team11-llamator-mcp)
- [Helm Documentation](https://helm.sh/docs/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
