# JobFlow

**JobFlow** is a production-oriented job scheduling and background-processing platform built with Java and Spring Boot.

The project is being developed incrementally to understand how real-world software systems **schedule, execute, monitor, retry, recover, and scale background jobs**.

The initial version focuses on building a reliable single-node job execution engine. Future versions will introduce distributed workers, Kafka, retries, dead-letter queues, observability, Docker, Kubernetes, and other production-grade capabilities.

---

## 🎯 Project Goal

It is to understand the engineering behind systems that perform work asynchronously or periodically.

JobFlow aims to answer questions such as:

* How does a system know **when** a job should run?
* How is a job execution tracked?
* What happens when a job fails?
* How are failed jobs retried?
* How do we prevent duplicate execution?
* How can multiple workers execute jobs concurrently?
* How do we recover from worker/server failures?
* How do we make background processing scalable?
* How do we monitor thousands or millions of job executions?
* How do we safely process the same message more than once?
* How do schedulers and workers coordinate in a distributed environment?

---

## 🏗️ Current Architecture — v0.1

The first version intentionally uses a simple architecture.

```text
                         ┌─────────────────────┐
                         │     REST API        │
                         │    Spring Boot      │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │     Job Manager     │
                         └──────────┬──────────┘
                                    │
                     ┌──────────────┴──────────────┐
                     │                             │
                     ▼                             ▼
              ┌─────────────┐              ┌─────────────┐
              │  Scheduler  │              │  PostgreSQL  │
              └──────┬──────┘              └─────────────┘
                     │
                     ▼
              ┌─────────────┐
              │Job Executor │
              └──────┬──────┘
                     │
                     ▼
              ┌─────────────┐
              │ Job Handler │
              └──────┬──────┘
                     │
                     ▼
              ┌─────────────────┐
              │ Customer Sync   │
              │     Job         │
              └─────────────────┘
```

The scheduler is responsible for deciding **when** a job should run.

The execution engine is responsible for managing **how** the execution happens.

The job handler contains the actual **business logic**.

---

# 🚀 v0.1

The first version implements a basic job execution platform.

### Features

* Job configuration
* Cron-based scheduling
* Manual job execution
* Job execution tracking
* Execution status management
* Basic failure handling
* Job handler abstraction
* Database-backed job configuration
* Customer data synchronization example
* REST APIs
* PostgreSQL persistence
* Flyway database migrations
* Application health monitoring

---

## 🔄 Job Lifecycle

A job execution follows a controlled lifecycle.

```text
                 ┌───────────┐
                 │ SCHEDULED │
                 └─────┬─────┘
                       │
                       ▼
                 ┌───────────┐
                 │  RUNNING  │
                 └─────┬─────┘
                       │
                 ┌─────┴─────┐
                 │           │
                 ▼           ▼
            ┌─────────┐  ┌────────┐
            │ SUCCESS │  │ FAILED │
            └─────────┘  └────────┘
```

Every execution receives a unique execution ID.

Example:

```text
Job:
CUSTOMER_SYNC

Execution:
1001

Status:
SUCCESS

Records Processed:
1500

Started:
08:00:00

Completed:
08:00:03
```

---

# 🧩 Example Job

The first real JobFlow job is a **Customer Data Synchronization Job**.

It simulates a common enterprise integration scenario where data must be transferred from one system to another.

```text
SOURCE_CUSTOMER
       │
       │
       │  processed = false
       ▼
 Customer Sync Job
       │
       ├── Read records
       ├── Validate records
       ├── Transform records
       ├── Write target records
       └── Mark records processed
       │
       ▼
TARGET_CUSTOMER
```

Example:

```text
SOURCE_CUSTOMER

ID    NAME       EMAIL              PROCESSED
------------------------------------------------
1     Rahul      rahul@gmail.com    false
2     Amit       amit@gmail.com     false
3     John       john@gmail.com     true
```

After execution:

```text
TARGET_CUSTOMER

ID    SOURCE_ID    NAME       EMAIL
-------------------------------------------
1     1            Rahul      rahul@gmail.com
2     2            Amit       amit@gmail.com
```

---

# 🗄️ Database

The v0.1 database contains four primary tables.

```text
JOB
 │
 │ 1:N
 ▼
JOB_EXECUTION


SOURCE_CUSTOMER
 │
 │
 ▼
TARGET_CUSTOMER
```

### `job`

Stores job configuration.

```text
id
name
description
job_type
cron_expression
enabled
created_at
updated_at
```

### `job_execution`

Stores every execution of a job.

```text
id
job_id
status
trigger_type
started_at
completed_at
records_processed
error_message
created_at
```

### `source_customer`

Represents the source data used by the example synchronization job.

### `target_customer`

Represents the destination data.

---

# 🛠️ Technology Stack

| Technology           | Purpose                     |
| -------------------- | --------------------------- |
| Java 21              | Programming language        |
| Spring Boot          | Backend framework           |
| Spring Web           | REST APIs                   |
| Spring Data JPA      | Persistence                 |
| PostgreSQL           | Relational database         |
| Flyway               | Database migrations         |
| Maven                | Build/dependency management |
| JUnit 5              | Unit testing                |
| Mockito              | Mocking                     |
| Testcontainers       | Integration testing         |
| Spring Boot Actuator | Health/monitoring           |
| Git/GitHub           | Version control             |

Additional technologies will be introduced in later versions when the architecture requires them.

---

# 📁 Project Structure

```text
jobflow/
│
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/
│   │   │       └── aditya/
│   │   │           └── jobflow/
│   │   │
│   │   │               ├── job/
│   │   │               │   ├── controller/
│   │   │               ├── service/
│   │   │               ├── repository/
│   │   │               ├── entity/
│   │   │               └── dto/
│   │   │
│   │   │               ├── execution/
│   │   │               ├── scheduler/
│   │   │               ├── handler/
│   │   │               ├── customer/
│   │   │               └── common/
│   │   │
│   │   └── resources/
│   │       ├── application.yml
│   │       └── db/
│   │           └── migration/
│   │
│   └── test/
│
├── pom.xml
└── README.md
```

The structure is intentionally modular so that the application can evolve into a distributed architecture without completely restructuring the codebase.

---

# 🔌 API

The v0.1 API will provide endpoints such as:

### Jobs

```http
POST   /api/v1/jobs
GET    /api/v1/jobs
GET    /api/v1/jobs/{id}
PUT    /api/v1/jobs/{id}
POST   /api/v1/jobs/{id}/enable
POST   /api/v1/jobs/{id}/disable
POST   /api/v1/jobs/{id}/run
```

### Executions

```http
GET /api/v1/executions
GET /api/v1/executions/{id}
```

API documentation will be added as the REST layer is implemented.

---

# ⚙️ Local Setup

## Prerequisites

Install:

* Java 21
* Maven
* PostgreSQL
* Git

Verify:

```bash
java -version
mvn -version
psql --version
```

---

## Clone the repository

```bash
git clone <repository-url>
cd jobflow
```

---

## Configure PostgreSQL

Create a database:

```sql
CREATE DATABASE jobflow;
```

Configure the application:

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/jobflow
    username: <username>
    password: <password>

  jpa:
    hibernate:
      ddl-auto: validate
```

Flyway will manage the database schema.

The application should **not** use Hibernate's automatic schema creation for production-style database management.

---

## Run the application

```bash
./mvnw spring-boot:run
```

On Windows:

```bash
mvnw.cmd spring-boot:run
```

---

# 🧪 Testing

The project will contain multiple levels of testing.

### Unit Tests

Test individual components:

```text
JobService
ExecutionService
CustomerSyncJobHandler
Scheduler
```

### Integration Tests

Verify:

```text
Spring Boot
     ↓
PostgreSQL
     ↓
Repositories
     ↓
Job Execution
```

Testcontainers will eventually be used to provide an isolated PostgreSQL instance during integration tests.

Run tests:

```bash
./mvnw test
```

---

# 📈 Roadmap

JobFlow will be developed incrementally.

## v0.1 — Single Node Job Engine

* [x] Database schema
* [ ] JPA entities
* [ ] Repositories
* [ ] Job Handler abstraction
* [ ] Customer Sync Job
* [ ] Execution Engine
* [ ] Scheduler
* [ ] Manual execution
* [ ] REST APIs
* [ ] Unit tests
* [ ] Integration tests

---

## v0.2 — Reliability

Introduce the first real production problems.

* [ ] Distributed locking
* [ ] Prevent duplicate executions
* [ ] Retry mechanism
* [ ] Exponential backoff
* [ ] Job timeout
* [ ] Execution cancellation
* [ ] Better failure handling
* [ ] Idempotency

---

## v0.3 — Distributed Workers

Move from:

```text
Scheduler
    ↓
Job Executor
```

to:

```text
Scheduler
    ↓
Message Queue
    ↓
Worker 1
Worker 2
Worker 3
```

Planned technologies:

* [ ] Apache Kafka
* [ ] Kafka producers
* [ ] Kafka consumers
* [ ] Worker service
* [ ] Consumer groups
* [ ] Partitioning
* [ ] Horizontal scaling

---

## v0.4 — Retry & Failure Infrastructure

* [ ] Retry topics/queues
* [ ] Dead Letter Queue
* [ ] Poison message handling
* [ ] Idempotent processing
* [ ] Backoff policies
* [ ] Failure classification
* [ ] Manual retry from DLQ

---

## v0.5 — Observability

* [ ] Structured logging
* [ ] Correlation IDs
* [ ] Micrometer metrics
* [ ] Prometheus
* [ ] Grafana
* [ ] OpenTelemetry
* [ ] Distributed tracing
* [ ] Alerting
* [ ] Job dashboards

---

## v0.6 — Production Operations

* [ ] Docker
* [ ] Docker Compose
* [ ] CI/CD
* [ ] GitHub Actions
* [ ] Code quality checks
* [ ] Security scanning
* [ ] Configuration management
* [ ] Secrets management

---

## v0.7 — Kubernetes

* [ ] Kubernetes deployment
* [ ] Multiple scheduler instances
* [ ] Multiple workers
* [ ] Horizontal Pod Autoscaling
* [ ] Readiness probes
* [ ] Liveness probes
* [ ] ConfigMaps
* [ ] Kubernetes Secrets

---

## v0.8 — Advanced Scheduling

* [ ] Job dependencies
* [ ] DAG-based workflows
* [ ] Job priorities
* [ ] Concurrency limits
* [ ] Rate limiting
* [ ] Dependency failure handling
* [ ] Scheduled job recovery

Example:

```text
IMPORT
   │
   ▼
VALIDATE
   │
   ├───────────┐
   ▼           ▼
RECONCILE   CLEANUP
   │
   ▼
REPORT
   │
   ▼
NOTIFY
```

---

## v0.9 — Scale & Performance

* [ ] Load testing
* [ ] Performance benchmarks
* [ ] Database indexing
* [ ] Query optimization
* [ ] Partitioning
* [ ] Execution history archival
* [ ] Kafka throughput testing
* [ ] Worker autoscaling
* [ ] Backpressure

---

## v1.0 — Production-Grade Job Platform

Target architecture:

```text
                         ┌──────────────┐
                         │   JobFlow UI │
                         └───────┬──────┘
                                 │
                                 ▼
                         ┌──────────────┐
                         │ API Gateway  │
                         └───────┬──────┘
                                 │
                    ┌────────────┴────────────┐
                    │                         │
                    ▼                         ▼
              Job Manager                Scheduler
                    │                         │
                    └────────────┬────────────┘
                                 │
                                 ▼
                            ┌─────────┐
                            │  Kafka  │
                            └────┬────┘
                                 │
              ┌──────────────────┼──────────────────┐
              ▼                  ▼                  ▼
           Worker 1           Worker 2           Worker N
              │                  │                  │
              └──────────────────┼──────────────────┘
                                 │
              ┌──────────────────┼──────────────────┐
              ▼                  ▼                  ▼
          PostgreSQL          External APIs       Storage

       ┌──────────────────────────────────────────────┐
       │              Observability                  │
       │ Logs | Metrics | Traces | Alerts | Health   │
       └──────────────────────────────────────────────┘
```

---

# 🧠 Engineering Concepts

One of the main purposes of this project is to understand the engineering decisions behind distributed systems.

Throughout the project, we will explore:

* Scheduling
* Background processing
* Batch processing
* Transactions
* Concurrency
* Distributed locking
* Idempotency
* Retry strategies
* Exponential backoff
* At-least-once delivery
* Message queues
* Kafka
* Dead Letter Queues
* Worker pools
* Horizontal scaling
* Backpressure
* Rate limiting
* Fault tolerance
* Leader election
* Database indexing
* Database partitioning
* Observability
* Distributed tracing
* Health checks
* CI/CD
* Containerization
* Kubernetes

The goal is to understand **why** these mechanisms exist, not just how to configure them.

---

# 🧪 Failure Scenarios

JobFlow will intentionally test failure scenarios such as:

```text
Worker crashes
Database becomes unavailable
Kafka becomes unavailable
Job exceeds timeout
External API returns 500
Network failure
Duplicate message
Duplicate execution
Partial batch failure
Scheduler instance crashes
```

The system should progressively evolve to recover from these failures automatically.

---

# 📚 Learning Philosophy

JobFlow follows a problem-first development approach.

Instead of adding infrastructure because it is popular, each technology will be introduced when a real architectural problem appears.

For example:

```text
Duplicate execution
        ↓
Distributed Lock

Too much work for one process
        ↓
Kafka + Workers

Repeated temporary failures
        ↓
Retry + Backoff

Messages repeatedly failing
        ↓
Dead Letter Queue

Duplicate message delivery
        ↓
Idempotency

Too many workers
        ↓
Autoscaling

Difficult production debugging
        ↓
Observability
```

This allows the project to evolve in the same way real systems evolve: **a problem appears → constraints are identified → a solution is designed → the solution is tested under failure and scale.**

---

# 📌 Current Status

**Version:** `v0.1`
**Stage:** Database setup / Core job engine development
**Status:** 🚧 In Development

The initial database schema has been created and the next milestone is implementing the Java persistence layer and core execution engine.

---

## License

This project is intended primarily as a learning and systems-engineering project.

License information will be added when the project reaches its first stable release.
