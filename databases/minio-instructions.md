# Minio Instructions

This document provides instructions for setting up Minio, a high-performance, distributed object storage system, in a Kubernetes environment. Minio is compatible with Amazon S3 APIs and is ideal for storing unstructured data such as photos, videos, log files, backups, and container images.

## Architecture Overview

- **Minio Server**: Deployed as a StatefulSet with multiple replicas for high availability.
- **Persistent Volumes**: Each Minio pod uses Persistent Volume Claims (PVCs) for data storage.
- **Services**: A headless service for pod communication and a regular service for application access.
- **Access Credentials**: Managed via Kubernetes secrets.

## Deployment Steps

### 1. Create the secrets from the environment file

```bash
# Create Kubernetes secret from the environment file
kubectl create secret generic minio-secrets \
  --from-env-file=minio-secrets.env
```

### 2. Deploy the Minio cluster

```bash
# Deploy the StatefulSet and services
kubectl apply -f minio-app.yaml
```

### 3. Wait for the cluster to be ready

```bash
# Check pod status
kubectl get pods -l app=minio-app
# Check logs
kubectl logs -f minio-app-0
kubectl logs -f minio-app-1
```

### 4. Access the MinIO Console

```bash
# Port-forward to access the MinIO console locally
kubectl port-forward svc/minio-console 9001:9001
```

Then open your browser to `http://localhost:9001` and login with:

- **Username**: `minio` (from minio-secrets.env)
- **Password**: `minio123` (from minio-secrets.env)

### 5. Connect your application to MinIO

Use the following connection details in your application configuration:

#### For S3-compatible API access:

- **Endpoint**: `minio:9000` (API port)
- **Access Key**: `minio` (from your secrets)
- **Secret Key**: `minio123` (from your secrets)
- **Bucket Name**: (create via console or API)
- **Use SSL**: false (unless you have configured TLS)
- **Region**: us-east-1 (default, can be changed as needed)

#### For console/web UI access:

- **Internal**: `minio-console:9001`
- **External**: Use port-forward as shown above

## Services Overview

The MinIO deployment creates three services:

1. **`minio`** (ClusterIP): For S3 API access from applications (port 9000)
2. **`minio-console`** (NodePort): For web console access (port 9001, external port 30901)
3. **`minio-headless`** (Headless): For StatefulSet pod-to-pod communication

## Example Usage

### Create a bucket using MinIO Client (mc):

```bash
# Install mc client in a pod
kubectl run mc-client --image=minio/mc --rm -it --restart=Never -- /bin/bash

# Inside the pod:
mc alias set myminio http://minio:9000 minio minio123
mc mb myminio/river-images
mc ls myminio/
```

### Python example for uploading files:

```python
from minio import Minio

# Initialize MinIO client
client = Minio(
    "minio:9000",
    access_key="minio",
    secret_key="minio123",
    secure=False
)

# Upload a file
client.fput_object("river-images", "sample.jpg", "/path/to/sample.jpg")
```

## Monitoring

### Check cluster status:

```bash
kubectl get pods -l app=minio-app
kubectl get svc | grep minio
kubectl get pvc | grep minio
```

### View logs:

```bash
kubectl logs minio-app-0
kubectl logs minio-app-1
```

## Troubleshooting

### Pod not starting:

1. Check logs: `kubectl logs minio-app-0`
2. Verify secrets: `kubectl get secret minio-secrets -o yaml`
3. Check storage: `kubectl get pvc`

### Cannot access console:

1. Verify port-forward: `kubectl port-forward svc/minio-console 9001:9001`
2. Check service: `kubectl get svc minio-console`
3. Try NodePort access: `<node-ip>:30901`

### Application connection issues:

1. Verify service name: Use `minio:9000` (not `minio-console`)
2. Check credentials match minio-secrets.env
3. Ensure bucket exists before accessing
