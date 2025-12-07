# Helm for Llamator MCP

### Setup:
1. Setup .env according to example.
2. Add secrets to k8s cluster
```
kubectl create secret generic llamator-openai-keys --from-env-file=.env
```
3. Run helm chart
```
helm install llamator ./
```