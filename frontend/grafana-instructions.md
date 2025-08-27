# Grafana Instructions

This document provides instructions for setting up Grafana, an open-source platform for monitoring and observability, in a Kubernetes environment. Grafana allows you to visualize and analyze metrics from various data sources like timescaledb

## Architecture Overview

- **Grafana Server**: Deployed 2 replicas for high availability.
- **Persistent Volume Claim (PVC)**: For storing Grafana data such as dashboards and user settings.
- **Services**: A regular service for accessing Grafana.
- **Access Credentials**: Managed via Kubernetes secrets.

## Deployment Steps

### 1. Create the secrets from the environment file

```bash
# Create Kubernetes secret from the environment file
kubectl create secret generic grafana-secrets \
  --from-env-file=grafana-secrets.env
```

### 2. Deploy the Grafana application

```bash
# Deploy the StatefulSet and services
kubectl apply -f grafana-app.yaml
```

### 3. Wait for the Grafana pods to be ready

```bash
# Check pod status
kubectl get pods -l app=grafana-app
```

### 4. Access the Grafana UI

```bash
# Port-forward to access the Grafana UI locally
kubectl port-forward svc/grafana-service 3000:3000
# or via minikube
minikube service grafana-service
```

Then open your browser to `http://localhost:3000` and login with:

- **Username**: `admin` (from grafana-secrets.env)
- **Password** : `secretPassword!2#` (from grafana-secrets.env)
