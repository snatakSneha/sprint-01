<img width="800" height="536" alt="image" src="https://github.com/user-attachments/assets/4377c0cf-d997-412e-8af4-925545eadc3e" />

# Employee API – Proof of Concept (POC)

---

## Table of Contents
1. [Introduction](#introduction) 
2. [Document Metadata](#document-metadata)  
3. [Purpose](#purpose)   
4. [Scope](#scope)  
5. [Prerequisites](#prerequisites)  
6. [Architecture / Design](#architecture--design)  
7. [Implementation Steps](#implementation-steps)  
8. [Success Criteria](#success-criteria)  
9. [Result & Validation](#result--validation)  
10. [Troubleshooting Steps](#troubleshooting-steps)  
11. [Deliverables](#deliverables)  
12. [Contact Information](#contact-information)  
13. [Conclusion](#conclusion)  
14. [References](#references)

---

## Introduction

The Employee API is a GoLang-based microservice that provides a RESTful interface to manage employee records. It uses PostgreSQL, Redis, and ScyllaDB as its backend data stores and integrates with Prometheus for metrics and observability. This POC aims to demonstrate its deployment on a Linux-based environment and ensure that all its endpoints are working as expected.

---

## Document Metadata

| Created     | Version | Author        | Modified    | Comment         | Reviewer         |
|-------------|---------|---------------|-------------|-----------------|------------------|
| 28-07-2025  | V1      | Sneha Joshi   | 28-07-2025  | Internal Review | Siddharth Pawar  |


---

## Purpose

The purpose of this POC is to deploy and validate the functionality of the **Employee API** microservice available at [https://github.com/OT-MICROSERVICES/employee-api](https://github.com/OT-MICROSERVICES/employee-api). This includes environment setup, dependency installation, database and service integration, and successful application execution.

---



## Scope

### In-Scope
- Environment setup for Employee API
- Installation and configuration of PostgreSQL, Redis, Prometheus, ScyllaDB
- Building and running the Employee API
- Validating API functionality

### Out-of-Scope
- Containerization using Docker
- Deployment on Kubernetes
- CI/CD pipeline automation

---

## Prerequisites

### System Requirements

| Component        | Requirement                                         |
|------------------|-----------------------------------------------------|
| Operating System | Ubuntu 22.04 LTS                                    |
| User Access      | Sudo privileges required                            |
| Network          | Open ports: 8080 (API), 5432 (PostgreSQL), 6379 (Redis), 9042 (ScyllaDB) |
| System Memory    | **20 GB+ RAM** recommended (for ScyllaDB compatibility) |



### Software Dependencies

| Software       | Required Version or Info                        |
|----------------|--------------------------------------------------|
| GoLang         | v1.20+ (Installed in `/usr/local/go/`)          |
| Redis          | Latest stable (Installed via APT)               |
| PostgreSQL     | v14+                                            |
| ScyllaDB       | Running and accessible on port 9042             |
| Prometheus     | v2.51.2 (Binary from official GitHub release)   |
| Make Utility   | Required to build and run migrations            |
| Git            | Installed for cloning the repo                  |
| `cqlsh`        | Required for accessing ScyllaDB via CLI         |

---

## Architecture / Design

### High-Level Architecture

```

+-------------------+
\|    Frontend App   |   <- Optional
+-------------------+
|
v
+-------------------------+
\|      Employee API       |
\|      (GoLang App)       |
+-------------------------+
\|      |        |
v      v        v
PostgreSQL Redis  ScyllaDB
\|                   |
v                   v
User Data        NoSQL/Log Storage

````

- **API** written in GoLang.
- **PostgreSQL** for relational storage.
- **Redis** for caching.
- **ScyllaDB** as a NoSQL data backend.
- **Prometheus** used for monitoring metrics.

---

## Implementation Steps


### 1. Clone the Repository

```bash
git clone https://github.com/OT-MICROSERVICES/employee-api
cd employee-api/
````

---

### 2. Configure Go Environment

```bash
echo "export PATH=$PATH:/usr/local/go/bin" >> ~/.bashrc
source ~/.bashrc
go version
```

>  Ensure Go is installed in `/usr/local/go`. If not, install it manually.

---

### 3. Install Redis

```bash
sudo apt update
sudo apt install redis -y
sudo systemctl enable redis --now
```

Verify Redis:

```bash
redis-server --version
sudo systemctl status redis-server
```

---

### 4. Install PostgreSQL

```bash
sudo apt install postgresql postgresql-contrib -y
sudo systemctl enable postgresql --now
```

Verify PostgreSQL:

```bash
psql --version
sudo systemctl status postgresql
```

---

### 5. Install ScyllaDB (Unified) – Requires 20 GB+ RAM

```bash
# Add GPG key
sudo mkdir -p /etc/apt/keyrings
sudo gpg --homedir /tmp --no-default-keyring --keyring /etc/apt/keyrings/scylladb.gpg \
  --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 491C93B9DE7496A7

# Add ScyllaDB repository
echo "deb [signed-by=/etc/apt/keyrings/scylladb.gpg] http://downloads.scylladb.com/deb/ubuntu/scylla-5.4-ubuntu22.04-x86_64/latest stable main" | \
  sudo tee /etc/apt/sources.list.d/scylla.list

# Update and install
sudo apt update
sudo apt install scylla -y

# Enable and start the service
sudo systemctl enable scylla-server --now
```

Verify ScyllaDB:

```bash
sudo systemctl status scylla-server
ss -ltnp | grep 9042
```

>  **Note**: ScyllaDB recommends at least **20 GB RAM**. Low-memory instances may crash or fail to start.

---

### 6. Test ScyllaDB Access

```bash
cqlsh -u scylladb -p password
```

> If `cqlsh` is not installed:

```bash
sudo apt install python3-pip -y
pip install cqlsh
```

---

### 7. Install Prometheus

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.51.2/prometheus-2.51.2.linux-amd64.tar.gz
tar -xvzf prometheus-2.51.2.linux-amd64.tar.gz
cd prometheus-2.51.2.linux-amd64
./prometheus
```

> Keep Prometheus running in background or another terminal.

---

### 8. Configure the Application

Edit the following files to configure DB connections and initial data:

```bash
nano config.yaml         # Add DB credentials and service endpoints
nano migration.json      # Add initial seed data
```

---

### 9. Run Migration with Makefile

Install `make` if not already installed:

```bash
sudo apt install make -y
```

Run the migration:

```bash
make run-migration
```

---

### 10. Build and Run the Employee API

```bash
make build
./employee-api
```

---

### 11. Verify via Swagger UI

Open in browser:

```
http://<your-public-ip>:8080/swagger/index.html
```

You should see the interactive API documentation and be able to test endpoints like:

* `GET /api/v1/employees`
* `POST /api/v1/employee`
* `PUT /api/v1/employee/{id}`
* `DELETE /api/v1/employee/{id}`

---


## Success Criteria

| Criterion                  | Expected Outcome                            |
| -------------------------- | ------------------------------------------- |
| API Build & Execution      | Successful execution of `./employee-api`    |
| API Access via Swagger UI  | Swagger loads at `:8080/swagger/index.html` |
| Redis/Postgres Integration | Logs show successful DB connection          |
| Prometheus Metrics Exposed | Prometheus metrics available on `/metrics`  |
|ScyllaDB CQL Connectivity   | Successful login using cqlsh                |

---

## Result & Validation


| Test Case                         | Status   |
| --------------------------------- | -------- |
| Swagger UI loaded                 |  Success |
| PostgreSQL connection established |  Success |
| Redis server integrated           |  Success |
| Prometheus started and exposed    |  Success |
| ScyllaDB accessible via CQLSH     |  Success |
| Migration executed via Makefile   |  Success |

---

## Troubleshooting Steps


| Issue                               | Resolution                                   |             |
| ----------------------------------- | -------------------------------------------- | ----------- |
| `make` not found                    | `sudo apt install make`                      |             |
| Go binary not in PATH               | `echo "export PATH=$PATH:/usr/local/go/bin"` |             |
| Redis not running                   | `sudo systemctl start redis-server`          |             |
| PostgreSQL not accepting connection | `sudo systemctl restart postgresql`          |             |
| ScyllaDB not reachable (port 9042)  | \`ss -ltnp                                   | grep 9042\` |
| Can't access ScyllaDB               | Use `cqlsh -u scylladb -p password`          |             |
| Swagger not loading                 | Check if `./employee-api` is running         |             |
| Prometheus not starting             | Ensure path and binary permissions           |             |

---

## Deliverables


| Deliverable          | Description                                                     |
| -------------------- | --------------------------------------------------------------- |
| Source Code          | [GitHub Repo](https://github.com/OT-MICROSERVICES/employee-api) |
| Build Binary         | `employee-api` Go binary                                        |
| Config Files         | `config.yaml`, `migration.json`                                 |
| Documentation        | This POC document (README.md format)                            |
| Architecture Diagram | `employee_api_architecture_final.png`                           |

---

## Contact Information

| Name         | Email Address                                 |
|--------------|-----------------------------------------------|
| Sneha Joshi  | sneha.joshi.snaatak@mygurukulam.co            |

---

## Conclusion

This Proof of Concept successfully demonstrates that the **Employee API** microservice can be deployed and executed in a standard Linux environment with all dependencies properly configured. The API is functional, scalable, and ready for further enhancements or containerization for production-level deployment.

---

## References

* [Employee API GitHub Repo](https://github.com/OT-MICROSERVICES/employee-api)
* [Redis Documentation](https://redis.io/docs/)
* [PostgreSQL Documentation](https://www.postgresql.org/docs/)
* [GoLang Documentation](https://golang.org/doc/)
* [Prometheus Documentation](https://prometheus.io/docs/)
* [ScyllaDB Documentation](https://www.scylladb.com/doc/)

