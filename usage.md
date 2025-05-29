# Usage Guide

This guide explains how to run the system locally using Docker Compose and access its various components.

## Running the System Locally (Docker Compose)

The primary way to run the entire system for development or demonstration purposes is by using the Docker Compose configuration. The most comprehensive setup is provided in `docker/demos/11-distributed-db/`.

1.  **Clone the Repository:**
    ```bash
    git clone <repository_url>
    cd <repository_directory>
    ```

2.  **Navigate to Demo Directory:**
    ```bash
    cd docker/demos/11-distributed-db/
    ```

3.  **Prerequisites & Configuration:**
    *   Ensure Docker and Docker Compose are installed (see `Deploy.md`).
    *   A `.env` file should be present in this directory (`docker/demos/11-distributed-db/.env`). This file contains crucial environment variables for configuring the services. Review it and customize if necessary.
    *   **Important Note on Missing Eureka:** As noted in `Deploy.md`, the `docker/eureka` service directory and its Dockerfile appear to be missing. This will prevent the `docker-compose up` command from completing successfully if the `eureka` service is included in the `docker-compose.yml`.
        *   **Workaround:** To run the other services, you might need to:
            1.  Comment out or remove the `eureka` service definition from `docker/demos/11-distributed-db/docker-compose.yml`.
            2.  Comment out or remove dependencies on `eureka` from other services in the same file.
            This will allow the rest of the stack to start, but service discovery will not function as originally intended.

4.  **Build and Start Services:**
    *   To build any images that are defined with a `build` context (if not using pre-built images):
        ```bash
        docker-compose build
        ```
    *   To start all services in detached mode:
        ```bash
        docker-compose up -d --remove-orphans
        ```

5.  **Stopping Services:**
    *   To stop all running services:
        ```bash
        docker-compose down
        ```
    *   To stop and remove volumes (deletes data):
        ```bash
        docker-compose down -v
        ```

## Accessing Services and UIs

Once the system is running (and assuming necessary workarounds for missing components), you can access the various parts via your web browser or API tools. Default ports are listed below (these might be different if you've changed configurations):

*   **Main Web Application (Python UI):**
    *   URL: `http://localhost/` (or `http://<docker_host_ip>/`)
    *   Port: `80` (exposed via `lb-web`)
    *   Note: `lb-web` forwards to the `web` service on port `8000`.

*   **Single Page Application (SPA - React UI):**
    *   URL: `http://localhost:3000/` (or `http://<docker_host_ip>:3000/`)
    *   Port: `3000` (exposed directly by the `spa` service)

*   **API Gateway (`gateway-svc`):**
    *   URL: `http://localhost:81/` (or `http://<docker_host_ip>:81/`)
    *   Port: `81` (exposed via `lb-services`)
    *   Note: `lb-services` forwards to `gateway-svc` on port `8080`. Use this base URL to interact with the backend APIs (e.g., `/api/products`, `/api/orders`).

*   **Service Discovery (Eureka):**
    *   URL: `http://localhost:8761/`
    *   Port: `8761`
    *   Note: This service will not be available if the Eureka component is missing.

*   **Kibana (Logging UI):**
    *   URL: `http://localhost:5601/`
    *   Port: `5601`

*   **Prometheus (Metrics UI):**
    *   URL: `http://localhost:9090/`
    *   Port: `9090`

*   **Jaeger UI (Distributed Tracing):**
    *   URL: `http://localhost:16686/` (Jaeger Query UI)
    *   Port: `16686`

*   **RabbitMQ Management Console:**
    *   URL: `http://localhost:15672/`
    *   Port: `15672`
    *   Default Credentials (usually): `guest` / `guest`

*   **PostgreSQL Database:**
    *   Port: `54321` (exposed on host, maps to `5432` in container)
    *   Access with a PostgreSQL client. Connection details (user, password, database name) are in the `.env` file.

*   **Cassandra Database:**
    *   Ports: `9042` (cassandra-1), `9043` (cassandra-2), `9044` (cassandra-3) exposed on host.
    *   Access with a Cassandra client (cqlsh).

*   **Redis:**
    *   Port: `6379` (exposed on host)
    *   Access with a Redis client.

## Interacting with APIs

Use tools like `curl`, Postman, or Insomnia to interact with the backend APIs through the API Gateway (`http://localhost:81/`). Refer to API documentation (if available separately) for specific endpoints and request/response formats.

Example (conceptual):
```bash
# Get products (assuming an endpoint like /api/products exists)
curl http://localhost:81/api/products

# Post an order (conceptual)
curl -X POST -H "Content-Type: application/json" -d '{"productId": "123", "quantity": 1}' http://localhost:81/api/orders
```

This guide should help you get started with running and using the system. For deployment to production environments, refer to `Deploy.md`.
