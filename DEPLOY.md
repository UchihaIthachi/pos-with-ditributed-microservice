# Deployment Guide

This document outlines the process for deploying the application to a Kubernetes cluster, leveraging Google Cloud Platform (GCP) for image building and registry.

## Overview

The deployment strategy involves the following key stages:

1.  **Building Docker Images:** Application components are containerized using Docker. The build process is typically managed on a dedicated GCP Compute Engine instance.
2.  **Pushing Images to Registry:** Built Docker images are pushed to Google Container Registry (GCR), a private Docker image registry on GCP.
3.  **Deploying to Kubernetes:** The application is deployed to a Kubernetes cluster using `kubectl` and predefined YAML manifest files.

## Prerequisites

*   **Google Cloud Platform (GCP) Account:** With a configured project, billing enabled, and necessary APIs (Compute Engine, Container Registry, Kubernetes Engine).
*   **`gcloud` CLI:** Installed and authenticated.
*   **`kubectl` CLI:** Installed and configured to communicate with your target Kubernetes cluster.
*   **Docker:** Installed locally if you need to inspect or manually manage images (though the primary build process is on a GCP VM).
*   **Git:** For version control.

## Step 1: Building Docker Images

The primary script for managing the build environment and triggering builds is `bin/gcp-build-images.sh`. This script automates:

1.  **Starting a GCP Compute Engine instance** (named `build-1` by default) that acts as a dedicated build server.
2.  **Updating the codebase** on the build server (`git pull`).
3.  **Executing the main build script** (`./build.sh` located in the root of the repository on the build server). This `build.sh` script is responsible for:
    *   Building the individual Docker images for each service (Java microservices, web application, SPA, etc.).
    *   Tagging these images appropriately.
4.  **Stopping the build server instance.**
5.  **Creating a new GCP machine image** from the build server's disk (e.g., `minisys-build-1`). This can be used to quickly launch new build servers with all dependencies pre-installed.

**To initiate the build process (execute with caution, ensure you understand the script's actions and have appropriate GCP permissions):**

```bash
./bin/gcp-build-images.sh
```

*Note: The central `./build.sh` script (not detailed here) likely contains the `docker build` commands for each service defined in their respective `Dockerfile`s.*

## Step 2: Pushing Images to Google Container Registry (GCR)

Once the Docker images are built (either on the GCP build VM and available locally, or built locally if you've adapted the process), they need to be pushed to GCR.

The `kubernetes/gcp-push-images.sh` script automates this:

1.  It identifies local Docker images matching the project's naming convention (e.g., prefixed with `ntw/`).
2.  It tags these images with the GCR path (`<REGISTRY_HOST>/<PROJECT_NAME>/ntw/<image_name>:<REVISION_ID>`).
    *   `REGISTRY_HOST`: Defaults to `asia.gcr.io` but can be overridden.
    *   `PROJECT_NAME`: Your GCP project ID.
    *   `REVISION_ID`: A version tag, defaults to `latest`.
3.  It pushes the tagged images to GCR.

**To push images to GCR:**

Ensure Docker is logged into GCR:
```bash
gcloud auth configure-docker <REGISTRY_HOST>
```
(e.g., `gcloud auth configure-docker asia.gcr.io`)

Then run the script:
```bash
# Set environment variables if needed
# export REGISTRY_HOST="your-gcr-host" # e.g., eu.gcr.io, us.gcr.io
# export REVISION_ID="your-image-tag"  # e.g., v1.0.0

./kubernetes/gcp-push-images.sh
```

## Step 3: Deploying to Kubernetes

With the images available in GCR, the application can be deployed to your Kubernetes cluster using the `kubernetes/kube-deploy.sh` script.

This script:
*   Takes a path to Kubernetes configuration files (defaults to `kubernetes/config/`) and an action (`apply` or `delete`).
*   Processes Kubernetes YAML manifest files (`*.yaml`).
*   Replaces placeholders `IMAGE_REGISTRY_PATH` and `REVISION_ID` in the YAML files with the correct values (derived from environment variables or defaults).
*   Uses `kubectl apply -f -` or `kubectl delete -f -` to deploy or remove the resources.

**To deploy the application:**

1.  Ensure `kubectl` is configured to point to your target Kubernetes cluster.
2.  Set necessary environment variables for the script:
    ```bash
    export REGISTRY_HOST="asia.gcr.io" # Or your GCR host
    export REVISION_ID="latest"        # Or the specific image tag you pushed
    # PROJECT_NAME is automatically fetched by 'gcloud config get-value project' in the script
    ```
3.  Run the deployment script:
    ```bash
    # To deploy all configurations from the default 'kubernetes/config/' directory
    ./kubernetes/kube-deploy.sh apply

    # To deploy a specific configuration file
    ./kubernetes/kube-deploy.sh kubernetes/config/3-service/product-svc.yaml apply

    # To run interactively (prompts for each file)
    ./kubernetes/kube-deploy.sh apply -i
    ```

**To delete the deployment:**
```bash
./kubernetes/kube-deploy.sh delete
```

## Configuration Management

*   **Environment Variables:** The `kube-deploy.sh` script uses `REGISTRY_HOST` and `REVISION_ID`. Individual services within Kubernetes might also be configured via environment variables defined in their deployment YAML files (often sourced from ConfigMaps or Secrets).
*   **Kubernetes Manifests:** The `kubernetes/config/` directory contains all Kubernetes resources:
    *   `0-env/`: Namespaces, ConfigMaps, Secrets.
    *   `1-data/`: StatefulSets/Deployments for databases (Postgres, Cassandra, Redis) and messaging (RabbitMQ).
    *   `2-monitor/`: Deployments for monitoring components (Elasticsearch, Fluentd, Jaeger, Prometheus).
    *   `3-service/`: Deployments and Services for backend microservices.
    *   `4-ui/`: Deployments and Services for frontend applications.
    *   `5-scaler/`: HorizontalPodAutoscaler definitions.

Carefully review and customize these manifests, especially ConfigMaps and Secrets, to match your target environment.

## Important Notes

*   **GCP Costs:** Be mindful of costs associated with GCP services (Compute Engine, Container Registry, Kubernetes Engine, network traffic). Stop or delete resources when not in use.
*   **Permissions:** Ensure the GCP user/service account used has appropriate IAM permissions for all required actions.
*   **`build.sh` (Central Build Script):** The effectiveness of the `bin/gcp-build-images.sh` script heavily depends on the main `./build.sh` script (not provided in this documentation snippet but present in the repository's root on the build VM). This script should contain the logic to build all necessary Docker images.
*   **Customization:** This deployment process is specific to this project's setup. Adapt scripts and configurations as needed for your environment.
