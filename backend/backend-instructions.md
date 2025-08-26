# Backend Instructions

This document provides instructions for setting up and deploying the backend component of the river segmentation application using Kubernetes.

## Initial Setup

1. Create a ConfigMap for the backend configuration:

   ```bash
   kubectl create configmap backend-config --from-env-file=backend-config.env
   ```

   This command creates a ConfigMap named `backend-config` using the environment variables defined in the `backend-config.env` file.
   You can check the contents of the ConfigMap with:
   `bash
kubectl get configmap backend-config -o yaml
`

2. Create the secrets from backend-secrets.env file:

   ```bash
   kubectl create secret generic backend-secrets --from-env-file=backend-secrets.env
   ```

   This command creates a secret named `backend-secrets` using the environment variables defined in the `backend-secrets.env` file.
   You can check the contents of the secret with:

   ```bash
   kubectl get secret backend-secrets -o yaml
   ```

## Deployment

1. Apply the Kubernetes deployment and service configurations for the backend:
   ```bash
   kubectl apply -f backend-app.yaml
   ```
   These commands will create the necessary deployment and service for the backend application.
2. Verify that the backend pods are running:

   ```bash
    kubectl get pods -l app=river-backend
   ```

   Ensure that the pods are in the `Running` state.
   To check the logs of a specific pod, use:

   ```bash
   kubectl logs <pod-name>
   ```

## HPA Configuration

1. To set up Horizontal Pod Autoscaling (HPA) for the backend, apply the HPA configuration:
   ```bash
   kubectl apply -f backend-hpa.yaml
   ```
   This command will create an HPA resource that automatically scales the number of backend pods based on CPU utilization.
2. Verify the HPA status:

   ```bash
   kubectl get hpa backend-hpa
   ```

   This command will show the current status of the HPA, including the number of replicas and CPU utilization.
