# Deployment Guide

This document provides instructions for deploying the system using Docker Compose and Kubernetes.

## Prerequisites

Before deploying, ensure you have the following tools installed:

*   **Docker:** For building and running containers. (Refer to [Docker documentation](https://docs.docker.com/get-docker/))
*   **Docker Compose:** For orchestrating multi-container Docker applications. (Refer to [Docker Compose documentation](https://docs.docker.com/compose/install/))
*   **kubectl:** For interacting with Kubernetes clusters. (Refer to [Kubernetes documentation](https://kubernetes.io/docs/tasks/tools/install-kubectl/))
*   **Git:** For cloning the repository.
*   **(Optional) Google Cloud SDK (`gcloud`):** If deploying to Google Kubernetes Engine (GKE) using the provided GCP scripts (e.g., `kubernetes/gcp-create-cluster.sh`).

## Building the Components

The system consists of multiple services that need to be built into Docker images.

*   **General Build System:** The `build-system.sh` script in the root directory appears to be a top-level script for managing builds.
*   **Java Services:** The `services/build.sh` script is likely used to compile and package the Java-based microservices (typically into WAR or JAR files, which are then used in their respective Docker builds). The `services/Dockerfile` uses a build argument `SERVICE` to specify which service WAR to package.
*   **Web Application (Python):** The `docker/web/Dockerfile` builds the Python web application image. It expects a `PyUI.tar.gz` in its `image` directory.
*   **SPA (JavaScript):** The `docker/spa/Dockerfile` builds the Nginx image to serve the compiled SPA. It expects a `reactapp.tar.gz` in its `image` directory. The `spa/build.sh` script is likely responsible for building the SPA and creating this archive.

Before deploying, you might need to run these build scripts or ensure the necessary artifacts (WAR files, tar.gz archives) are in place for the Docker builds.

## Deployment Options

### 1. Docker Compose

Docker Compose is suitable for local development, testing, and running demo environments.

*   **Demo Configurations:** The `docker/demos/` directory contains various `docker-compose.yml` files for different setups. The most comprehensive one appears to be `docker/demos/11-distributed-db/docker-compose.yml`.
*   **Environment Variables:** Many services are configured using environment variables. An `.env` file (e.g., `docker/demos/11-distributed-db/.env`) is typically used to manage these settings. Ensure this file is present and correctly configured.
*   **Running with Docker Compose:**
    1.  Navigate to the chosen demo directory (e.g., `cd docker/demos/11-distributed-db/`).
    2.  Ensure any necessary `.env` files are present and configured.
    3.  Run `docker-compose pull` to pull any pre-built images if available.
    4.  Run `docker-compose build` to build images for services defined with a `build` context in the compose file.
    5.  Run `docker-compose up -d` to start the services in detached mode.

*   **Important Note:** As of the last check, the `docker/eureka` directory and its Dockerfile were missing. This will prevent Docker Compose setups that include the `eureka` service (like `docker/demos/11-distributed-db/docker-compose.yml`) from starting correctly. You may need to provide the Eureka service implementation or remove it from the Docker Compose file for the rest of the services to run.

### 2. Kubernetes

Kubernetes is recommended for staging and production environments, providing robust orchestration, scaling, and management.

*   **Configuration Files:** Kubernetes manifests (YAML files) are located in the `kubernetes/config/` directory. These define namespaces, ConfigMaps, Secrets, deployments, services, volumes, etc.
*   **Deployment Scripts:**
    *   `kubernetes/kube-deploy.sh`: This script likely automates the process of applying the Kubernetes configurations to a cluster.
    *   `kubernetes/gcp-*.sh` scripts: These scripts (`gcp-create-cluster.sh`, `gcp-create-network.sh`, `gcp-push-images.sh`) are provided for managing infrastructure and deployments on Google Kubernetes Engine (GKE).
*   **General Steps for Kubernetes Deployment (Conceptual):**
    1.  **Set up a Kubernetes Cluster:** Use a managed service (GKE, EKS, AKS) or set up your own. The `gcp-create-cluster.sh` can be used for GKE.
    2.  **Configure `kubectl`:** Ensure `kubectl` is configured to communicate with your cluster.
    3.  **Push Docker Images:** Build all necessary Docker images and push them to a container registry accessible by your Kubernetes cluster (e.g., Google Container Registry (GCR), Docker Hub, etc.). The `gcp-push-images.sh` script might assist with this for GCR.
    4.  **Persistent Volumes:** If using persistent data (for databases like PostgreSQL, Cassandra), you might need to provision Persistent Volumes (PVs) and Persistent Volume Claims (PVCs). See `kubernetes/volume/persistent-volumes.yaml`. The `kubernetes/kube-volumes.sh` script might be relevant.
    5.  **Apply Configurations:** Use `kubectl apply -f <directory_or_file>` with the manifests in `kubernetes/config/`. The `kube-deploy.sh` script likely automates this. It's common to apply them in order (e.g., namespace, configmaps/secrets, storage, services, deployments).
    6.  **Verify Deployment:** Use `kubectl get pods -n <namespace>`, `kubectl get services -n <namespace>`, etc., to check the status of your deployed applications.

*   **Testing on Kubernetes:** The `kubernetes/kube-test.sh` script and files in `kubernetes/test/` (e.g., `jmeter.yaml`) suggest means for running tests against a Kubernetes deployment.

## Post-Deployment

*   Once deployed, refer to `usage.md` for information on accessing the various services and UIs.
*   Monitor logs and metrics using the observability stack (Kibana, Prometheus, Jaeger).
