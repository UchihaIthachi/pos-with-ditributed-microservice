# High-Level Architecture

This document outlines the high-level architecture of the system.

The system is designed as a **microservices architecture**, promoting modularity, scalability, and independent deployment of components. It consists of several key parts that work together to deliver its functionalities.

## Core Components

1.  **Frontend Tier:**
    *   **Web Application (`web`):** A Python-based web application (likely Django/Flask) serving a server-side rendered user interface.
    *   **Single Page Application (`spa`):** A client-side JavaScript application (likely React, Vue, or Angular) providing a rich user experience. It interacts with the backend via the API Gateway.

2.  **Backend Tier (Microservices):**
    *   **API Gateway (`gateway-svc`):** The single entry point for all client requests to backend services. It handles request routing, aggregation, and potentially concerns like authentication and rate limiting.
    *   **Service Discovery (`eureka`):** Enables services to register themselves and discover other services dynamically. (Note: The implementation files for Eureka appear to be missing in the current snapshot of the repository).
    *   **Core Microservices:**
        *   `admin-svc`: Handles administrative functionalities.
        *   `auth-svc`: Manages user authentication and authorization.
        *   `product-svc`: Manages product information.
        *   `order-svc`: Handles order processing and management.
        *   `inventory-svc`: Manages inventory and stock.
        These services are primarily Java-based.

3.  **Data Storage Tier:**
    *   **PostgreSQL (`postgres`):** Used as the primary relational database for structured data.
    *   **Apache Cassandra (`cassandra`):** A distributed NoSQL database (configured as a 3-node cluster) used for data requiring high availability and scalability.
    *   **Redis (`redis`):** An in-memory data store used for caching, session management, or fast lookups.

4.  **Messaging & Integration:**
    *   **RabbitMQ (`rabbitmq`):** A message broker used for asynchronous communication between services (e.g., `order-svc` likely uses it for order processing tasks). This helps decouple services and improve resilience.

5.  **Observability Stack (Monitoring, Logging, Tracing):**
    *   **Logging:**
        *   `Fluentd`: Aggregates logs from various services.
        *   `Elasticsearch`: Stores and indexes the collected logs.
        *   `Kibana`: Provides a UI for searching and visualizing logs.
    *   **Distributed Tracing:**
        *   `Jaeger (Agent, Collector, Query)`: Used to trace requests as they flow through multiple services, helping in debugging and performance analysis. Traces are often stored in Elasticsearch.
    *   **Metrics:**
        *   `Prometheus`: Collects and stores time-series metrics from services.
        *   `Exporters (es-exporter, pg-exporter, redis-exporter)`: Provide metrics from various components to Prometheus.
        *   (Grafana is often used with Prometheus for visualization, though not explicitly listed in the Docker Compose, it's a common addition).

6.  **Load Balancing:**
    *   **`lb-web`:** An Nginx-based load balancer for distributing traffic to the `web` application instances.
    *   **`lb-services`:** An Nginx-based load balancer/reverse proxy for distributing traffic to the `gateway-svc` and other backend services.

## Deployment

The system is designed to be deployed using containerization technologies:

*   **Docker:** All services are containerized using Docker. Docker Compose is used for local development and demo environments.
*   **Kubernetes:** Configurations are provided for deploying the system to a Kubernetes cluster, enabling orchestration, scaling, and management in production environments.

## Diagram (Conceptual)

```
[SPA/Web UI] ---> [lb-web] ---> [Web App] ---> [lb-services] ---> [API Gateway] --+
                                                                                  |
                                                                                  |
+---------------------------------------------------------------------------------+
|                                                                                 |
v                                                                                 v
[Auth Service] <--- [Eureka (Service Discovery)] ---> [Product Service]
 ^      ^                                                   ^      ^
 |      |                                                   |      |
 v      v                                                   v      v
[Order Service] ----> [RabbitMQ]                           [Inventory Service]
 ^      ^
 |      |
 v      v
[Admin Service]

Databases: [PostgreSQL] [Cassandra Cluster] [Redis]

Observability: [Fluentd -> Elasticsearch/Kibana] [Jaeger] [Prometheus]
```
(Note: This is a simplified conceptual diagram. Actual interactions can be more complex.)

This architecture allows for flexibility in scaling individual services, updating components independently, and maintaining a resilient and observable system.
