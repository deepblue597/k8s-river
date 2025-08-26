# Frontend Instructions

This document provides instructions for setting up and deploying the frontend component of the river segmentation application using Kubernetes.

## Initial Setup

1. Create a ConfigMap for the frontend configuration:

   ```bash
   kubectl create configmap frontend-config --from-env-file=frontend-config.env
   ```

   This command creates a ConfigMap named `frontend-config` using the environment variables defined in the `frontend-config.env` file.

You can check the contents of the ConfigMap with:

```bash
kubectl get configmap frontend-config -o yaml
```

2. Ensure that the `frontend-config.env` file contains the correct backend service details:
   ```env
   BACKEND_IP=river-backend
   BACKEND_PORT=8000
   ```
   Replace `river-backend` and `8000` with the actual service name and port of your backend if they differ.

## Deployment

1. Apply the Kubernetes deployment and service configurations for the frontend:
   ```bash
   kubectl apply -f frontend-app.yaml
   ```
   These commands will create the necessary deployment and service for the frontend application.
2. Verify that the frontend pods are running:

   ```bash
   kubectl get pods -l app=river-frontend
   ```

   Ensure that the pods are in the `Running` state.

   To check the logs of a specific pod, use:

   ```bash
    kubectl logs <pod-name>
   ```

3. Access the frontend application:
   - If you are using a LoadBalancer service, get the external IP:
     ```bash
     kubectl get svc river-frontend
     ```
     Access the application via `http://<EXTERNAL-IP>:8501`.
   - If you are using a NodePort service, access it via `http://<NODE-IP>:<NODE-PORT>`.
   - If you are using port forwarding, set it up with:
     ```bash
     kubectl port-forward svc/river-frontend 8501:8501
     ```
     Then access the application at `http://localhost:8501`.
     or use minikube service command:
     ```bash
        minikube service river-frontend
     ```

## HPA Configuration

1. If you have set up Horizontal Pod Autoscaling (HPA) for the frontend, ensure that the HPA configuration is applied:
   ```bash
   kubectl apply -f frontend-hpa.yaml
   ```
   This command will create or update the HPA for the frontend deployment.
2. Check the status of the HPA:
   ```bash
   kubectl get hpa -l app=river-frontend
   ```
   Ensure that the HPA is correctly scaling the frontend pods based on the defined metrics.
