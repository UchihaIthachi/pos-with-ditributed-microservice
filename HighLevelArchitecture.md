# High-Level Architecture

This document provides a high-level overview of the application's architecture.

## Overview

The application is designed using a **microservices architecture**. This approach breaks down the system into smaller, independent, and deployable services. These services communicate with each other over the network, typically via lightweight protocols like HTTP/REST or messaging queues.

This architecture promotes:
*   **Scalability:** Individual services can be scaled independently.
*   **Flexibility:** Different technologies can be used for different services.
*   **Resilience:** Failure in one service is less likely to affect others.
*   **Maintainability:** Smaller codebases are easier to understand and manage.

## Key Components

The system comprises several key components:

1.  **Frontend Applications:**
    *   **Single Page Application (SPA):** A modern user interface likely built with a JavaScript framework (e.g., React, based on `spa/README.md`). Provides a rich user experience.
    *   **Web Application:** A server-side rendered web application (Python/Django based on `web/` directory). May serve traditional web pages or admin interfaces.

2.  **API Gateway (`gateway-svc`):**
    *   Acts as a single entry point for all client requests.
    *   Routes requests to the appropriate backend microservices.
    *   Can handle concerns like authentication, rate limiting, and request/response transformation.

3.  **Backend Microservices (`services/` directory):**
    *   A collection of specialized services, each responsible for a specific business capability. Based on the directory structure and configurations, these include:
        *   `admin-svc`: Handles administrative functionalities.
        *   `auth-svc`: Manages user authentication and authorization.
        *   `product-svc`: Manages product catalog and information.
        *   `order-svc`: Handles order creation and management.
        *   `inventory-svc`: Manages stock and inventory levels.
    *   These are primarily Java/Spring Boot applications.

4.  **Data Stores:**
    *   **PostgreSQL (`postgres`):** A relational database likely used for transactional data requiring ACID properties (e.g., user accounts, orders).
    *   **Cassandra (`cassandra`):** A distributed NoSQL database optimized for high availability and scalability. Likely used for data that requires high write throughput or has large volume (e.g., activity logs, product recommendations, time-series data). A 3-node cluster is common for resilience.
    *   **Redis (`redis`):** An in-memory data store, often used for caching, session management, or as a fast message broker.

5.  **Messaging Queue (`rabbitmq`):**
    *   Used for asynchronous communication between services.
    *   Helps decouple services and improve resilience by queuing messages if a consumer service is temporarily unavailable.
    *   Example use cases: processing orders, sending notifications.

6.  **Service Discovery (`eureka`):**
    *   Allows services to find and communicate with each other dynamically without hardcoding hostnames and ports.
    *   Each microservice registers itself with the discovery server (Eureka) and can then look up other services.

7.  **Monitoring and Observability Stack:**
    *   **Fluentd:** A data collector for logs.
    *   **Elasticsearch:** A search and analytics engine, often used to store and search logs collected by Fluentd.
    *   **Kibana:** A visualization tool for Elasticsearch data, allowing users to explore logs and create dashboards.
    *   **Prometheus:** A time-series database and monitoring system for collecting metrics from services.
    *   **Jaeger:** A distributed tracing system that helps monitor and troubleshoot transactions across multiple microservices.
    *   Various exporters (`es-exporter`, `pg-exporter`, `redis-exporter`) are used to expose metrics from different components to Prometheus.

8.  **Load Balancers (`lb-web`, `lb-services`):**
    *   Distribute incoming traffic across multiple instances of frontend and backend services to improve performance, scalability, and availability.

## Architectural Diagram (Textual Representation)

```
+---------------------+      +-------------------------+      +-----------------------+
|      User           |----->|   Frontend Applications |----->|     API Gateway       |
| (Browser/Mobile)    |      | (SPA - React, Web - Django)|      |    (gateway-svc)      |
+---------------------+      +-------------------------+      +-----------------------+
                                     ^      |                            |
                                     |      |                            v
                                     |      |      +------------------------------------------+
                                     |      |      |          Backend Microservices             |
                                     |      +----->| (admin, auth, product, order, inventory) |
                                     |             +------------------------------------------+
                                     |                            |   ^   |   ^
                                     |                            |   |   |   | (Service Discovery - Eureka)
                                     |                            v   |   v   |
       +----------------------------------------------------------+   |   +--------------------------------------------------------+
       |                        Data Stores                        |   |   |                            Messaging Queue             |
       | (PostgreSQL, Cassandra Cluster, Redis)                    |<--+   | (RabbitMQ)                                             |
       +----------------------------------------------------------+       +--------------------------------------------------------+

+-----------------------------------------------------------------------------------------------------------------+
|                                          Orchestration & Monitoring                                             |
|                                                                                                                 |
| Kubernetes: Manages deployment, scaling, and operation of containerized applications.                           |
| Monitoring: Prometheus (metrics), Elasticsearch/Kibana (logs), Jaeger (tracing)                                 |
+-----------------------------------------------------------------------------------------------------------------+
```

## Containerization and Orchestration

*   **Docker:** All major components (services, databases, UI applications) are containerized using Docker. This ensures consistency across environments and simplifies deployment.
*   **Kubernetes (`kubernetes/` directory):** The application is designed to be deployed and managed by Kubernetes.
    *   Kubernetes YAML configuration files define deployments, services, persistent volumes, config maps, secrets, and auto-scalers.
    *   It handles service discovery (internal to Kubernetes), load balancing (via Services and Ingress), self-healing, and automated rollouts/rollbacks.

## Technology Stack Summary

*   **Backend Microservices:** Java (Spring Boot)
*   **Frontend SPA:** JavaScript (likely React, based on Create React App structure)
*   **Web Application:** Python (Django)
*   **Databases:** PostgreSQL, Cassandra, Redis
*   **Messaging:** RabbitMQ
*   **Service Discovery:** Netflix Eureka
*   **API Gateway:** Spring Cloud Gateway (inferred from typical Spring Boot microservice patterns) or a custom gateway solution.
*   **Containerization:** Docker
*   **Orchestration:** Kubernetes
*   **Monitoring:** Prometheus, Elasticsearch, Kibana, Jaeger, Fluentd

This architecture provides a robust foundation for a scalable, resilient, and maintainable application.
