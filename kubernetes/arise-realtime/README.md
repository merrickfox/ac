# Arise Realtime Service Kubernetes Manifests

This directory contains Kubernetes manifests for deploying the Arise Realtime WebSocket service built with Elixir/Phoenix and using libcluster for node discovery.

## Components

- **Deployment**: Manages the Elixir application pods with proper environment variables for libcluster
- **Service**: Provides internal network access to the pods
- **Ingress**: Exposes the WebSocket endpoints publicly with proper WebSocket configuration
- **ConfigMap**: Stores configuration for libcluster and the Phoenix app
- **Secret**: Stores the Erlang cookie used for node communication
- **HorizontalPodAutoscaler**: Automatically scales pods based on CPU/memory usage

## Prerequisites

- Kubernetes cluster with Nginx Ingress Controller
- Container registry with your Elixir application image
- Proper DNS configuration for the WebSocket endpoint

## Deployment Instructions

1. **Update the Docker image reference**:
   Edit `deployment.yaml` and replace `${DOCKER_REGISTRY}/arise-realtime:latest` with your actual image path.

2. **Generate a secure Erlang cookie**:
   Generate a secure Erlang cookie and update the `secret.yaml` file:

   ```bash
   # Generate a secure cookie and encode it
   COOKIE=$(openssl rand -base64 32 | tr -d '\n')
   ENCODED_COOKIE=$(echo -n "$COOKIE" | base64)

   # Update the secret.yaml file
   sed -i "s/UGxlYXNlQ2hhbmdlVGhpc1RvQVByb3BlckNvb2tpZVZhbHVl/$ENCODED_COOKIE/g" secret.yaml
   ```

3. **Update domain configuration**:
   Edit `ingress.yaml` and `configmap.yaml` to use your actual domain instead of `realtime.arise.example.com`.

4. **Update namespace if needed**:
   If you're not deploying to the `default` namespace, update the namespace in `configmap.yaml`.

5. **Apply the manifests**:

   ```bash
   # Create namespace if needed
   kubectl create namespace your-namespace  # Optional

   # Apply the manifests
   kubectl apply -f kubernetes/arise-realtime/ -n your-namespace
   ```

6. **Verify deployment**:

   ```bash
   # Check if pods are running
   kubectl get pods -l app=arise-realtime -n your-namespace

   # Check if service is created
   kubectl get service arise-realtime -n your-namespace

   # Check if ingress is properly configured
   kubectl get ingress arise-realtime -n your-namespace
   ```

## libcluster Configuration

The deployment is configured to use libcluster with the Kubernetes strategy. The pods will automatically discover each other using the Kubernetes API based on the label selector `app=arise-realtime`.

This configuration assumes your Elixir application is configured to use libcluster with environment variables. Ensure your application's libcluster configuration reads from these environment variables.

## Scaling

The service is configured to automatically scale between 3 and 20 replicas based on CPU and memory utilization. You can modify the HPA configuration in `hpa.yaml` to adjust scaling behavior.

## Troubleshooting

If pods are not clustering properly:

1. Check if the service account has permissions to list pods in the namespace
2. Verify the Erlang cookie is consistent across all pods
3. Check the logs for any clustering-related errors:
   ```bash
   kubectl logs -l app=arise-realtime -n your-namespace
   ```
