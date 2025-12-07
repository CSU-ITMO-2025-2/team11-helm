# Setup steps

Minikube:
```sh
eval "$(minikube docker-env)"
docker build -t llamator-api:local .

# Same image for worker, if it's the same Dockerfile
docker build -t llamator-worker:local .
```

