# Usage Guide

This document describes how to get the application up and running for development and testing purposes.

## Prerequisites

Before you begin, ensure you have the following installed:

*   **Git:** For cloning the repository.
*   **Docker:** For running the containerized application.
*   **Docker Compose:** For orchestrating the multi-container application.
*   **Node.js and npm:** (Optional) If you plan to work on the frontend SPA development.

## Getting the Code

Clone the repository to your local machine:

```bash
git clone <repository-url>
cd <repository-directory>
```
Replace `<repository-url>` with the actual URL of this repository and `<repository-directory>` with the name of the cloned directory.

## Running the Application with Docker Compose

The project includes several Docker Compose files in the `docker/demos/` directory that demonstrate different configurations of the system. These are the recommended way to run the entire application stack.

For example, to run the demo that includes distributed databases and a full monitoring stack:

1.  Navigate to the demo directory:
    ```bash
    cd docker/demos/11-distributed-db/
    ```

2.  (Optional) Create a `.env` file if one is referenced by the `docker-compose.yml` and not already present. Some demos might require specific environment variables. Refer to the `.env` files within each demo directory for examples.

3.  Start the application stack:
    ```bash
    docker-compose up -d
    ```
    This command will download/build the necessary Docker images and start all the services in detached mode.

4.  To stop the application:
    ```bash
    docker-compose down
    ```

Please explore the other `docker-compose.yml` files in `docker/demos/` for different setups (e.g., basic three-tier, with load balancers, logging, etc.).

## Accessing Application Components

Once the services are running via Docker Compose (e.g., using the `11-distributed-db` demo), you can access the various components through your web browser:

*   **Single Page Application (SPA):** [http://localhost:3000](http://localhost:3000) (if `spa` service is included and running)
*   **Web Application (Python/Django):** [http://localhost:80](http://localhost:80) (exposed via `lb-web` which proxies `web` service on port 8000)
*   **API Gateway:** [http://localhost:81](http://localhost:81) (exposed via `lb-services` which proxies `gateway-svc` on port 8080)
*   **Eureka (Service Discovery):** [http://localhost:8761](http://localhost:8761)
*   **Kibana (Logging Visualization):** [http://localhost:5601](http://localhost:5601)
*   **Jaeger UI (Distributed Tracing):** [http://localhost:16686](http://localhost:16686)
*   **Prometheus (Metrics):** [http://localhost:9090](http://localhost:9090)
*   **RabbitMQ Management:** [http://localhost:15672](http://localhost:15672)

*Note: Port numbers might vary based on the specific `docker-compose.yml` file used and local configurations.*

## Frontend (SPA) Development

If you want to work on the Single Page Application (React) development:

1.  Navigate to the SPA directory:
    ```bash
    cd spa/
    ```

2.  Install dependencies:
    ```bash
    npm install
    ```

3.  Start the development server:
    ```bash
    npm start
    ```
    This will typically open the SPA in your default browser at [http://localhost:3000](http://localhost:3000) and enable hot reloading.

## Backend Services

The backend consists of multiple microservices primarily built with Java (Spring Boot) and a web component built with Python (Django). These are containerized and orchestrated by Docker Compose in the development setups described above. For individual service development or debugging, you might need to refer to their specific build configurations (e.g., `pom.xml` for Java services) and potentially run them outside of Docker, though container-based development is encouraged.
