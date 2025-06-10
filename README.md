# Order Management System (OMS) - README

This document provides a starting guide for the Order Management System. For a comprehensive technical overview, including system architecture, Kubernetes deployment, database schemas, and detailed service configurations, please see the [System Overview Document](docs/SYSTEM_OVERVIEW.md).

## What is this repository for?

This repository contains the source code for the Order Management System (OMS), a microservices-based application designed to handle product catalog management, inventory control, order processing, and user authentication.

- **Version:** (Specify Project Version, e.g., 1.0.0)
- **Detailed Documentation:** [docs/SYSTEM_OVERVIEW.md](docs/SYSTEM_OVERVIEW.md)

## How do I get set up?

This section needs to be populated with detailed setup instructions. Key areas to cover:

- **Prerequisites:**
  - Java Development Kit (JDK) version (e.g., JDK 11 or 17)
  - Apache Maven
  - Node.js and npm (for the SPA)
  - Docker and Docker Compose (for local containerized instances)
  - `kubectl` (for Kubernetes interaction)
  - (Any other essential tools)
- **Configuration:**
  - Cloning the repository.
  - Setting up environment variables (if any required for local development beyond service defaults).
  - Local database setup (PostgreSQL, Cassandra) - consider providing Docker Compose scripts for these.
  - Applying database schemas (see `bin/create-schema.sql` and `docker/cassandra/image/create-schema.cql`).
- **Building the Code:**
  - Building Java services (e.g., `mvn clean install` in the `services` directory).
  - Building the SPA (e.g., `npm install && npm run build` in the `spa` directory).
  - Building Docker images (scripts are available in `docker/` and `kubernetes/gcp-build-images.sh`).
- **Running the Application Locally:**
  - Instructions for running services (Spring Boot apps).
  - Running the SPA (`npm start` from `spa` directory).
  - Using Docker Compose for a local multi-container setup (e.g., `docker-compose up` from `docker` directory after configuring `.env`).
- **How to run tests:**
  - Running backend unit/integration tests (e.g., `mvn test`).
  - Running SPA tests (e.g., `npm test` from `spa` directory).
  - End-to-end testing strategy (if any, e.g., using tools mentioned in `tests/`).
- **Deployment instructions:**
  - Overview of deployment to Kubernetes (refer to `kubernetes/` directory and `docs/SYSTEM_OVERVIEW.md`).
  - Scripts for deployment (e.g., `kubernetes/kube-deploy.sh`).

## Contribution guidelines

This section should detail how to contribute to the OMS project.

- **Writing tests:** Expectations for unit, integration, and end-to-end tests.
- **Code review process:** Branching strategy, pull request requirements, code style.
- **Issue tracking:** How issues are reported and managed.
- (Other guidelines)

## Who do I talk to?

- **Repo Owner/Admin:** (Specify Name/Team and Contact Email)
- **Development Team Contact:** (Specify Name/Team and Contact Email)
- **Community/Support Channel:** (Specify if any, e.g., Slack channel, mailing list)

---

_This README was updated to replace the generic template. Further details for setup, build, and contribution guidelines need to be filled in by the project team._
