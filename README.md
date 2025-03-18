# Haystack RAG Kubernetes Deployment

This repository contains Helm charts for deploying the Haystack RAG application in a Kubernetes production-like environment.

## Prerequisites

- Kubernetes cluster (v1.21+)
- Helm v3
- kubectl configured to communicate with your cluster
- Docker for building and pushing images
- Storage class that supports ReadWriteMany access mode for shared file storage
- Storage class that supports ReadWriteOnce access mode for OpenSearch data

## Building the Docker Images

Before deploying to Kubernetes, you need to build and push the Docker images:

```bash
# Clone the original repository
git clone https://github.com/deepset-ai/haystack-rag-app.git
cd haystack-rag-app

# Build indexing service image
docker build -t registry-arn/haystack-indexing:latest -f backend/Dockerfile.indexing ./backend

# Build query service image
docker build -t registry-arn/haystack-query:latest -f backend/Dockerfile.query ./backend

# Build frontend image
docker build -t registry-arn/haystack-frontend:latest -f frontend/Dockerfile.frontend ./frontend

# Push images to your registry(I used the local images to set up k8s)
docker push registry-arn/haystack-indexing:latest
docker push registry-arn/haystack-query:latest
docker push registry-arn/haystack-frontend:latest
```

## Deploying with Helm

1. Clone this repository:
```bash
git clone https://github.com/bsagtaa/deepset-rag-k8s.git
cd deepset-rag-k8s/haystack-rag
```

2. Create a .env file for storing below secrets(since we are using kind secret which requires to manually inject credentials to the secret object):
      - OPENSEARCH_USER
      - OPENSEARCH_PASSWORD
      - OPENAI_API_KEY

```bash

cat > .env << EOF
OPENSEARCH_USER=admin
OPENSEARCH_PASSWORD=your-secure-password
OPENAI_API_KEY=sk-proj-999
EOF
```
Note: For production setup, we should always go for external-secrets operator, Hashicorp Valuts(more on Readme-bonus.md)

3. Update `values.yaml` with your specific configurations:
```bash
nano haystack-rag/values.yaml
```

Update the following values:
- Image repositories to point to your container registry
- Ingress host to match your domain
- Resource limits based on your cluster capacity
- OpenSearch configuration (replicas, storage, etc.)
- Any additional environment variables

4. Install the ingress-controller:
In order for an Ingress to work in your cluster, there must be an ingress controller running. Since in this project we are required to expose frontend and API so we will deploy ingress.
Helm chart to install: https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx

5. Install the Helm chart:
```bash
# Create namespace
kubectl create namespace haystack

# Install secrets from .env file
kubectl -n haystack create secret generic haystack-rag-secrets --from-env-file=.env

# Install the helm chart
helm install haystack-rag ./haystack-rag -n haystack

# In case you make any changes to your chart, you can simply run below command to update
```bash
helm upgrade haystack-rag ./haystack-rag -n haystack
```


## Kubernetes Resources Created

The Helm chart creates the following Kubernetes resources:

- **Deployments**:
  - haystack-frontend
  - haystack-indexing
  - haystack-query
  - haystack-nginx
- **StatefulSet**:
  - haystack-opensearch
- **Services**:
  - haystack-frontend
  - haystack-indexing
  - haystack-query
  - haystack-opensearch
  - haystack-opensearch-headless (for StatefulSet pod discovery)
  - haystack-nginx (acting as a reverse proxy that handle all traffic and internally route to backend, since we are not using any cloud provider hence this can be used as an alternative)
- **Persistent Volume Claims**:
  - haystack-file-storage (shared between indexing and query services)
  - Dynamic PVCs for each OpenSearch pod (created by the StatefulSet)
- **ConfigMaps**:
  - haystack-config (contains nginx configurations)
- **Ingress**:
  - haystack-ingress (NGINX)
