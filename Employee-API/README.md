<img width="800" height="536" alt="image" src="https://github.com/user-attachments/assets/4377c0cf-d997-412e-8af4-925545eadc3e" />

# Employee API – Proof of Concept (POC)

---

## Table of Contents
- [Introduction](#introduction)
-  [Document Metadata](#document-metadata)
-  [Purpose](#purpose)   
-  [Scope](#scope)  
-  [Prerequisites](#prerequisites)  
-  [Architecture / Design](#architecture--design)  
-  [Implementation Steps](#implementation-steps)  
-  [Success Criteria](#success-criteria)  
-  [Result & Validation](#result--validation)  
-  [Troubleshooting Steps](#troubleshooting-steps)  
-  [Deliverables](#deliverables)  
-  [Contact Information](#contact-information)  
-  [Conclusion](#conclusion)  
-  [References](#references)

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

<img width="1691" height="674" alt="image" src="https://github.com/user-attachments/assets/bcd690fd-6d8d-4aaa-b788-0fb88bcfeaf4" />



- **API** written in GoLang.
- **PostgreSQL** for relational storage.
- **Redis** for caching.
- **ScyllaDB** as a NoSQL data backend.
- **Prometheus** used for monitoring metrics.

---

## Implementation Steps

### 1. Install Git (If Not Already Installed)

To clone the project from GitHub:

```bash
sudo apt update
sudo apt install git -y
git --version
```
<img width="1438" height="700" alt="image" src="https://github.com/user-attachments/assets/9c8c2a08-d27e-480c-b3e1-670b03fff89a" />

---

### 2. Clone the Repository

```bash
git clone https://github.com/OT-MICROSERVICES/employee-api
cd employee-api/
````
<img width="1166" height="229" alt="image" src="https://github.com/user-attachments/assets/5c36498e-6253-4c43-85dd-9bf0c095fcd5" />

---

### 3. Configure Go Environment

```bash
echo "export PATH=$PATH:/usr/local/go/bin" >> ~/.bashrc
source ~/.bashrc
go version
```

>  Ensure Go is installed in `/usr/local/go`. If not, install it manually.
```bash
sudo apt  install golang-go
```
<img width="1166" height="123" alt="image" src="https://github.com/user-attachments/assets/9608f28b-7c1d-42f0-b36d-a80d428730c8" />

---

### 4. Install Redis

```bash
sudo apt update
sudo apt install redis -y
sudo systemctl start redis
sudo systemctl enable redis-server --now
```
<img width="1166" height="250" alt="Screenshot from 2025-07-29 23-14-53" src="https://github.com/user-attachments/assets/1b4205cd-8c14-4334-920a-f2df240ec325" />

Verify Redis:

```bash
redis-server --version
sudo systemctl status redis-server
```
<img width="1166" height="430" alt="image" src="https://github.com/user-attachments/assets/2efe4054-5c54-462a-878a-2c492c1af9c1" />

---

### 5. Install PostgreSQL

```bash
sudo apt install postgresql postgresql-contrib -y
sudo systemctl enable postgresql --now
```

Verify PostgreSQL:

```bash
psql --version
sudo systemctl status postgresql
```
<img width="1166" height="344" alt="image" src="https://github.com/user-attachments/assets/ce0fbd63-d0a1-4dde-a8fd-a1909bc49cd1" />

---

### 6. Install ScyllaDB (Unified) – Requires 20 GB+ RAM

```bash
# 1. Add GPG key (for secure repo access)
sudo mkdir -p /etc/apt/keyrings
sudo gpg --homedir /tmp --no-default-keyring --keyring /etc/apt/keyrings/scylladb.gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 491c93b9de7496a7

# 2. Add ScyllaDB Repository (Debian/Ubuntu based)
sudo wget -O /etc/apt/sources.list.d/scylla.list http://downloads.scylladb.com/deb/debian/scylla-6.1.list

# 3. Update apt cache
sudo apt update

# 4. Install Java & Scylla
sudo apt install openjdk-11-jdk -y
sudo apt-get install scylla

# 5. Run I/O tuning (recommended before first start)
sudo scylla_io_setup

# 6. Enable developer mode (needed for low-RAM VMs like t2.micro)
sudo /opt/scylladb/scripts/scylla_dev_mode_setup --developer-mode 1

# 7. Start and enable Scylla service
sudo systemctl start scylla-server
sudo systemctl enable scylla-server
```

Verify ScyllaDB:

```bash
sudo systemctl status scylla-server
ss -ltnp | grep 9042
```
<img width="1166" height="705" alt="image" src="https://github.com/user-attachments/assets/804893e7-c194-40d4-a543-3cf20b5d6d34" />

>  **Note**: ScyllaDB recommends at least **20 GB RAM**. Low-memory instances may crash or fail to start.

---

#### 7. Create Keyspace and Table in ScyllaDB
```bash
cqlsh -u scylla -p password
```
Then inside cqlsh:

```bash
CREATE KEYSPACE employee_db
WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1};

USE employee_db;

CREATE TABLE employee (
    id UUID PRIMARY KEY,
    name TEXT,
    designation TEXT,
    location TEXT
);

exit;
```
<img width="1332" height="494" alt="Screenshot from 2025-07-30 01-03-38" src="https://github.com/user-attachments/assets/8a0459d8-dadf-4a1b-bade-31d206f3e1d8" />


### 8. Configure the Application

Edit the following files to configure DB connections and initial data:

```bash
nano config.yaml 
```
- ##### Add DB credentials and service endpoints
```bash
host: ["localhost:9042"]
  username: scylladb
  password: password
  keyspace: employee_db

redis:
  enabled: false
  host: localhost:6379
  password: password
  database: 0
```
```bash
nano migration.json      
```
- #### Add initial seed data
```bash
{
  "database": "cassandra://localhost:9042/employee_db?username=scylladb&password=password"
}
```

```bash
nano main.go     
```
- #### Add public IP

```bash
package main

import (
	docs "employee-api/docs"
	"employee-api/middleware"
	"employee-api/routes"
	"github.com/gin-gonic/gin"
	"github.com/penglongli/gin-metrics/ginmetrics"
	"github.com/sirupsen/logrus"
	swaggerfiles "github.com/swaggo/files"
	ginSwagger "github.com/swaggo/gin-swagger"
)

var router = gin.New()

func init() {
	logrus.SetLevel(logrus.InfoLevel)
	logrus.SetFormatter(&logrus.JSONFormatter{}) // NEW
}

// @title Employee API
// @version 1.0
// @description The REST API documentation for employee webserver
// @termsOfService http://swagger.io/terms/

// @contact.name Opstree Solutions
// @contact.url https://opstree.com
// @contact.email opensource@opstree.com

// @license.name Apache 2.0
// @license.url http://www.apache.org/licenses/LICENSE-2.0.html
// @BasePath /api/v1
// @schemes http
func main() {
	monitor := ginmetrics.GetMonitor()
	monitor.SetMetricPath("/metrics")
	monitor.SetSlowTime(1)
	monitor.SetDuration([]float64{0.1, 0.3, 1.2, 5, 10})
	monitor.Use(router)
	router.Use(gin.Recovery())                  // NEW
	router.Use(middlewares.LoggingMiddleware()) // NEW
	v1 := router.Group("/api/v1")
	docs.SwaggerInfo.BasePath = "/api/v1/employee"
	routes.CreateRouterForEmployee(v1)
	url := ginSwagger.URL("http://54.160.111.7:8080/swagger/doc.json")
	router.GET("/swagger/*any", ginSwagger.WrapHandler(swaggerfiles.Handler, url))
	router.Run(":8080")
}

```
---

### 9. Install `make` if not already installed:

```bash
sudo apt install make -y
```
<img width="1332" height="168" alt="image" src="https://github.com/user-attachments/assets/28a659e2-ab96-4c77-8d44-a227b512737a" />



### 10. Build and Run the Employee API

```bash
make build
./employee-api
```
<img width="1855" height="740" alt="image" src="https://github.com/user-attachments/assets/917493fd-786d-4738-8e48-678f0f608ed4" />

---

### 11. Verify via Swagger UI

Open in browser:

```
http://<your-public-ip>:8080/swagger/index.html
```
<img width="1855" height="1007" alt="image" src="https://github.com/user-attachments/assets/41551863-39f8-4f1e-92f3-28ce7c7a8395" />

You should see the interactive API documentation and be able to test endpoints :

| **Endpoint**                        | **Method** | **Description**                                                                                   |
|-------------------------------------|------------|---------------------------------------------------------------------------------------------------|
| /metrics                            | GET        | Application healthcheck and performance metrics are available on this endpoint                    |
| /api/v1/employee/health             | GET        | Endpoint for providing shallow healthcheck information about application health and readiness     |
| /api/v1/employee/health/detail      | GET        | Endpoint for providing detailed health check about dependencies as well like - ScyllaDB and Redis |
| /api/v1/employee/create             | POST       | Data creation endpoint which accepts certain JSON body to add employee information in database    |
| /api/v1/employee/search             | GET        | Endpoint for searching data information using the params in the URL                               |
| /api/v1/employee/search/all         | GET        | Endpoint for searching all information across the system                                          |
| /api/v1/employee/search/location    | GET        | Application endpoint for getting the count and information of location                            |
| /api/v1/employee/search/designation | GET        | Application endpoint for getting the count and information of designation                         |
| /swagger/index.html                 | GET        | Swagger endpoint for the application API documentation and their data models                      |


- #### For Health Check

```
http://54.160.111.7:8080/api/v1/employee/health
```
<img width="1332" height="221" alt="image" src="https://github.com/user-attachments/assets/24dc2516-ba10-4256-855c-5d12faa9c534" />

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
* [ScyllaDB Documentation](https://www.scylladb.com/doc/)

