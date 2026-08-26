# 🛡️ BCP Messenger

### Business Continuity Plan Messenger

> **An enterprise-grade event-driven microservices platform for crisis management, emergency employee safety surveys, and instant incident notifications.**

BCP Messenger is a **Java-based microservices application** designed to help organizations communicate with employees during emergencies such as floods, cyclones, fire emergencies, power outages, internet outages, and other incidents.

The platform allows administrators to create emergency campaigns, approvers to authorize them, employees to confirm their safety through surveys, and management to monitor real-time response analytics.

---

## 📌 Project Overview

During an emergency, organizations need to:

* Notify employees immediately.
* Confirm employee safety.
* Identify employees who need assistance.
* Collect emergency survey responses.
* Track organizational response status.
* Send emergency notifications asynchronously.
* Analyze employee responses.

BCP Messenger solves these requirements using an **Event-Driven Microservices Architecture**.

### Core Technologies

* ☕ Java 17
* 🌱 Spring Boot 3.4.2
* ☁️ Spring Cloud 2024.0.0
* 🚪 Spring Cloud Gateway
* 🔎 Netflix Eureka
* ⚙️ Spring Cloud Config Server
* 📨 Apache Kafka
* 🗄️ MySQL 8
* 🔄 Flyway
* 🔐 Spring Security + JWT
* 🔒 BCrypt
* ⚛️ React 18
* ⚡ Vite
* 📡 Axios
* 🧪 Postman
* 🏗️ Maven

---

# 🏗️ Architecture

```text
                         ┌─────────────────────────┐
                         │     React Frontend      │
                         │       Port 5173         │
                         └────────────┬────────────┘
                                      │
                              REST / JSON + JWT
                                      │
                                      ▼
                         ┌─────────────────────────┐
                         │   Spring Cloud Gateway  │
                         │       Port 8080         │
                         └────────────┬────────────┘
                                      │
                ┌─────────────────────┼─────────────────────┐
                │                     │                     │
                ▼                     ▼                     ▼
       ┌────────────────┐    ┌────────────────┐    ┌──────────────────┐
       │  User Service  │    │  BCP Service   │    │ Notification Svc │
       │    :8081       │    │     :8082      │    │      :8083       │
       └───────┬────────┘    └───────┬────────┘    └────────┬─────────┘
               │                     │                      │
               ▼                     │                      ▼
       ┌────────────────┐            │             ┌──────────────────┐
       │ bcp_user_db    │            │             │bcp_notification_db│
       └────────────────┘            │             └──────────────────┘
                                     │
                              CampaignCreatedEvent
                                     │
                                     ▼
                            ┌──────────────────┐
                            │  Apache Kafka    │
                            │ Port 9092        │
                            │                  │
                            │ bcp.campaign.    │
                            │ created          │
                            └────────┬─────────┘
                                     │
                                     ▼
                            Notification Service

                         ┌─────────────────────────┐
                         │       bcp_db            │
                         │ Campaigns / Surveys     │
                         └─────────────────────────┘


        Supporting Infrastructure
        ──────────────────────────
        Eureka Server       → Port 8761
        Config Server       → Port 8888
```

The architecture uses **Eureka for service discovery** and **Spring Cloud Config Server for externalized configuration**.

---

# 🔧 Microservices

| Service                | Port | Database              | Responsibility                         |
| ---------------------- | ---: | --------------------- | -------------------------------------- |
| `eureka-server`        | 8761 | None                  | Service discovery                      |
| `config-server`        | 8888 | None                  | Centralized configuration              |
| `api-gateway`          | 8080 | None                  | Single entry point and routing         |
| `user-service`         | 8081 | `bcp_user_db`         | Authentication and employee management |
| `bcp-service`          | 8082 | `bcp_db`              | Campaigns, surveys and analytics       |
| `notification-service` | 8083 | `bcp_notification_db` | Kafka consumer and email notifications |
| `bcp-frontend`         | 5173 | Browser               | React user interface                   |

The backend consists of six Spring Boot services, with the React frontend acting as the client application.

---

# 🔐 Authentication

The project uses:

* Spring Security
* JWT authentication
* BCrypt password hashing
* Role-based authorization

### Supported Roles

| Role       | Responsibility                           |
| ---------- | ---------------------------------------- |
| `ADMIN`    | Create campaigns and view analytics      |
| `APPROVER` | Review and approve emergency campaigns   |
| `EMPLOYEE` | Receive alerts and submit safety surveys |

JWT tokens contain information such as the user's email, role, and employee ID.

---

# 🚪 API Gateway

The React frontend communicates only with the **API Gateway on port 8080**.

```text
React
  ↓
API Gateway :8080
  ↓
Eureka
  ↓
Required Microservice
```

### Why API Gateway?

* Provides a single entry point.
* Hides internal service ports.
* Provides centralized routing.
* Supports dynamic service discovery.
* Handles common cross-cutting concerns.
* Prevents frontend code from directly depending on individual microservice ports.

The gateway uses Spring Cloud Gateway with WebFlux and routes services through logical discovery addresses such as `lb://USER-SERVICE` and `lb://BCP-SERVICE`.

---

# 🔎 Eureka Service Discovery

Eureka acts as the central service registry.

When a microservice starts:

```text
Microservice
     ↓
Registers with Eureka
     ↓
Eureka Registry
     ↓
Gateway discovers service
```

Instead of hardcoding service IP addresses and ports, services are discovered dynamically through Eureka.

This makes the architecture more suitable for environments where service instances may change or scale.

---

# ⚙️ Config Server

Spring Cloud Config Server provides centralized external configuration.

Configuration includes items such as:

* Kafka topics
* JWT configuration
* SMTP settings
* Service-specific properties

The project uses the **native Config Server profile** for local development.

---

# 📨 Apache Kafka

Kafka is used for **asynchronous notification processing**.

When an approver approves an emergency campaign:

```text
Approver
   ↓
BCP Service
   ↓
CampaignCreatedEvent
   ↓
Kafka Topic
bcp.campaign.created
   ↓
Notification Service
   ↓
Email Notification
```

### Why Kafka?

If the BCP Service directly sent hundreds or thousands of emails synchronously, the request could become slow and eventually timeout.

Kafka provides:

* Asynchronous processing
* Decoupling between services
* Persistent event storage
* Consumer groups
* Fault tolerance
* Scalable notification processing

The Notification Service consumes events using the consumer group:

```text
bcp-notification-group
```

---

# 🗄️ Database Architecture

The project follows the **Database-per-Service** pattern.

```text
User Service
     ↓
bcp_user_db

BCP Service
     ↓
bcp_db

Notification Service
     ↓
bcp_notification_db
```

### Database Tables

#### `bcp_user_db`

```text
users
```

Stores:

* User ID
* Employee ID
* Name
* Email
* Password
* Department
* Role
* Active status

#### `bcp_db`

```text
campaigns
survey_questions
survey_responses
survey_answers
```

#### `bcp_notification_db`

```text
notification_logs
```

This separation prevents services from directly depending on another service's database schema.

---

# 🔄 Flyway Database Migration

Flyway is used for database version management.

Instead of allowing Hibernate to automatically modify the database schema, the project uses:

```properties
spring.jpa.hibernate.ddl-auto=validate
```

Database changes are managed through versioned SQL migration files such as:

```text
V1__init_*.sql
V2__*.sql
```

This makes database changes:

* Version controlled
* Reproducible
* Predictable
* Suitable for CI/CD environments

---

# 🔗 Inter-Service Communication

The project uses two communication mechanisms.

## 1. Synchronous REST

Used when an immediate response is required.

```text
React
  ↓
API Gateway
  ↓
User Service / BCP Service
```

Examples:

* Login
* Fetch survey questions
* Submit surveys
* View analytics

## 2. Asynchronous Kafka

Used for background processing.

```text
BCP Service
     ↓
Kafka
     ↓
Notification Service
```

Used for:

* Emergency notification dispatch
* Campaign-created events

---

# 🔄 Complete Application Flow

## Step 1 — User Login

```text
React Login Form
       ↓
POST /api/auth/login
       ↓
API Gateway :8080
       ↓
User Service :8081
       ↓
BCrypt Password Validation
       ↓
JWT Token
       ↓
React LocalStorage
```

---

## Step 2 — Create Emergency Campaign

Admin selects an incident type:

```text
FLOOD
CYCLONE
POWER_OUTAGE
INTERNET_OUTAGE
FIRE_EMERGENCY
OTHER
```

The frontend retrieves the corresponding questions from the BCP Service.

The administrator then creates the emergency campaign.

Initial status:

```text
PENDING_APPROVAL
```

---

## Step 3 — Approve Campaign

The Approver reviews the campaign and selects:

```text
Approve & Send
```

The BCP Service:

1. Updates campaign status.
2. Creates a `CampaignCreatedEvent`.
3. Publishes the event to Kafka.

---

## Step 4 — Notification Processing

Notification Service consumes the Kafka event.

It:

1. Receives the campaign event.
2. Generates the emergency email.
3. Adds the employee survey URL.
4. Sends the email or simulates it in offline mode.
5. Stores notification history.

---

## Step 5 — Employee Survey

Employee opens the survey URL:

```text
/survey/{surveyToken}
```

The employee answers the emergency questions.

Example:

```text
Are you safe?                     YES
Do you need assistance?           NO
Can you work from home?           YES
Need accommodation?               NO
Need transportation?              NO
```

The BCP Service stores the response in MySQL.

If an assistance-related question is answered **YES**, the system marks the employee as requiring assistance.

---

## Step 6 — Analytics

Administrators and Approvers can view:

* Safety percentage
* YES vs NO responses
* Employee response statistics
* Employees requiring urgent assistance

The analytics endpoint is:

```http
GET /api/bcp/campaigns/{id}/analytics
```

The complete request lifecycle is documented in the project source.

---

# 🚨 Supported Incident Types

### 🌊 Flood

Safety, assistance, work-from-home, accommodation and transportation questions.

### 🌀 Cyclone

Employee and family safety, emergency assistance and work-from-home questions.

### ⚡ Power Outage

Electricity availability and alternate workplace questions.

### 🌐 Internet Outage

Connectivity and alternate location questions.

### 🔥 Fire Emergency

Evacuation, medical assistance and emergency shelter questions.

### ⚠️ Other

General safety, assistance and work capability questions.

---

# 📋 Sample Users

| Email                   | Password       | Role     |
| ----------------------- | -------------- | -------- |
| `admin@company.com`     | `Admin@123`    | ADMIN    |
| `approver@company.com`  | `Approver@123` | APPROVER |
| `employee1@company.com` | `Employee@123` | EMPLOYEE |
| `employee2@company.com` | `Employee@123` | EMPLOYEE |
| `employee3@company.com` | `Employee@123` | EMPLOYEE |

These accounts are pre-seeded through Flyway.

> ⚠️ These credentials are for local/demo purposes only. Do not use them in production.

---

# 🛠️ Prerequisites

Install the following:

```text
Java JDK 17
Node.js 18+
MySQL 8.x
Apache Kafka 3.x
Maven 3.8+
```

Required ports:

| Component            | Port |
| -------------------- | ---: |
| Eureka               | 8761 |
| Config Server        | 8888 |
| API Gateway          | 8080 |
| User Service         | 8081 |
| BCP Service          | 8082 |
| Notification Service | 8083 |
| Kafka                | 9092 |
| React Frontend       | 5173 |
| MySQL                | 3306 |

---

# 🗄️ MySQL Setup

Run the database setup script:

```bash
mysql -u root -p < scripts/setup-databases.sql
```

Or create the databases manually:

```sql
CREATE DATABASE IF NOT EXISTS bcp_user_db
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;

CREATE DATABASE IF NOT EXISTS bcp_db
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;

CREATE DATABASE IF NOT EXISTS bcp_notification_db
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;
```

---

# 📨 Kafka Setup

The project uses Kafka in **KRaft mode**, so ZooKeeper is not required.

Example Windows setup:

```powershell
cd C:\kafka

$CLUSTER_ID = .\bin\windows\kafka-storage.bat random-uuid

.\bin\windows\kafka-storage.bat format `
-t $CLUSTER_ID `
-c .\config\kraft\server.properties

.\bin\windows\kafka-server-start.bat `
.\config\kraft\server.properties
```

Kafka should be available on:

```text
localhost:9092
```

---

# ▶️ Running the Backend

Start services in this order.

### 1. Eureka Server

```cmd
cd backend\eureka-server
mvn spring-boot:run
```

Open:

```text
http://localhost:8761
```

### 2. Config Server

```cmd
cd backend\config-server
mvn spring-boot:run
```

### 3. User Service

```cmd
cd backend\user-service
mvn spring-boot:run
```

### 4. BCP Service

```cmd
cd backend\bcp-service
mvn spring-boot:run
```

### 5. Notification Service

```cmd
cd backend\notification-service
mvn spring-boot:run
```

### 6. API Gateway

```cmd
cd backend\api-gateway
mvn spring-boot:run
```

Alternatively, on Windows:

```cmd
scripts\start-all-services.bat
```

---

# ⚛️ Running the Frontend

```bash
cd frontend

npm install

npm run dev
```

Open:

```text
http://localhost:5173
```

---

# 🧪 Postman API Testing

Import:

```text
postman/BCP_Messenger_API_Collection.json
```

### Login

```http
POST http://localhost:8080/api/auth/login
```

```json
{
  "email": "admin@company.com",
  "password": "Admin@123"
}
```

### Get Questions

```http
GET http://localhost:8080/api/bcp/questions/FLOOD
```

### Create Campaign

```http
POST http://localhost:8080/api/bcp/campaigns
```

```json
{
  "title": "Heavy Flooding in Chennai",
  "message": "Due to continuous torrential rain, please confirm your safety status.",
  "incidentType": "FLOOD"
}
```

### Approve Campaign

```http
PUT http://localhost:8080/api/bcp/campaigns/1/approve
```

### Submit Survey

```http
POST http://localhost:8080/api/bcp/surveys/{surveyToken}/submit
```

### View Analytics

```http
GET http://localhost:8080/api/bcp/campaigns/1/analytics
```

---

# 📧 Email Configuration

By default, the application runs in **offline email mode**.

```properties
app.email.enabled=false
```

In offline mode:

* Emails are not actually sent.
* Formatted emails are logged to the console.
* Notification records are stored as `SIMULATED`.

To enable SMTP:

```text
APP_EMAIL_ENABLED=true
MAIL_USERNAME=your-email@gmail.com
MAIL_PASSWORD=your-app-password
```

---

# 🔐 Security Features

The project implements:

* JWT-based authentication
* BCrypt password hashing
* Role-based authorization
* Stateless authentication
* Database isolation
* Externalized configuration
* Service discovery
* Asynchronous event processing

---

# 🧩 Key Design Decisions

### Why Microservices?

Allows independent scaling, fault isolation and separate deployment of business capabilities.

### Why API Gateway?

Provides one external entry point and hides internal microservice topology.

### Why Eureka?

Allows dynamic service discovery without hardcoded service locations.

### Why Kafka?

Provides asynchronous, durable and scalable notification processing.

### Why Database-per-Service?

Maintains strong service boundaries and prevents direct database coupling.

### Why JWT?

Provides stateless authentication suitable for distributed microservices.

### Why BCrypt?

Provides salted and adaptive password hashing.

### Why Flyway?

Provides version-controlled and reproducible database migrations.

---

# 🔄 Communication Architecture

```text
                 SYNCHRONOUS
React ───────► API Gateway
                 │
                 ├────► User Service
                 │
                 └────► BCP Service


                 ASYNCHRONOUS
BCP Service
     │
     │ CampaignCreatedEvent
     ▼
  Kafka
     │
     ▼
Notification Service
     │
     ▼
Email / Notification
```

---

# 📁 Project Structure

```text
BCP-Messenger/
│
├── backend/
│   │
│   ├── eureka-server/
│   │
│   ├── config-server/
│   │
│   ├── api-gateway/
│   │
│   ├── user-service/
│   │
│   ├── bcp-service/
│   │
│   └── notification-service/
│
├── frontend/
│
├── scripts/
│   ├── setup-databases.sql
│   ├── start-kafka-kraft.bat
│   ├── start-all-services.bat
│   └── stop-all-services.bat
│
├── postman/
│   └── BCP_Messenger_API_Collection.json
│
└── README.md
```

---

# 🧯 Troubleshooting

### Port already in use

Run:

```cmd
scripts\stop-all-services.bat
```

### Kafka connection refused

Make sure Kafka is running:

```text
localhost:9092
```

before starting:

```text
bcp-service
notification-service
```

### Database access denied

Verify:

```text
MySQL → localhost:3306
Username → root
Password → configured password
```

---

# 🎓 Architecture Highlights

This project demonstrates practical implementation of:

* Microservices Architecture
* Event-Driven Architecture
* Service Discovery
* API Gateway Pattern
* Database-per-Service Pattern
* Asynchronous Messaging
* JWT Authentication
* Role-Based Authorization
* Database Migration
* Reactive Gateway
* Fault Isolation
* Decoupled Services
* REST API Communication
* Kafka Producer/Consumer
* React Frontend Integration

---

# 📊 Technology Architecture

```text
Frontend
   │
   └── React + Vite + Axios
             │
             ▼
API Gateway
   │
   ├── Spring Cloud Gateway
   └── WebFlux
             │
             ▼
Service Discovery
   │
   └── Eureka
             │
      ┌──────┴─────────┐
      ▼                ▼
User Service       BCP Service
      │                │
      ▼                ▼
MySQL DB             MySQL DB
                       │
                       ▼
                     Kafka
                       │
                       ▼
              Notification Service
                       │
                       ▼
                    MySQL DB
```

---

# 🚀 Future Enhancements

Potential production enhancements include:

* SMS and WhatsApp notification channels
* Push notifications
* Docker/Kubernetes deployment
* Cloud deployment
* Distributed tracing
* Centralized logging
* Prometheus/Grafana monitoring
* API rate limiting
* Advanced disaster recovery
* Multi-region deployment
* Automated CI/CD pipelines

---

# 👨‍💻 Project Status

```text
✅ Microservices Architecture
✅ API Gateway
✅ Eureka Service Discovery
✅ Config Server
✅ JWT Authentication
✅ BCrypt Password Security
✅ Apache Kafka Producer/Consumer
✅ Database-per-Service
✅ Flyway Migrations
✅ Emergency Campaign Management
✅ Employee Safety Surveys
✅ Notification Processing
✅ Analytics Dashboard
✅ React Frontend
✅ Postman API Collection
✅ Windows Startup Scripts
```

---

# 📜 Conclusion

**BCP Messenger** demonstrates how modern microservices and event-driven architecture can be applied to a real-world business continuity and emergency communication problem.

The system separates authentication, campaign management, notification processing, and frontend responsibilities into independent components while using **REST for immediate communication** and **Kafka for asynchronous event processing**.

This architecture provides better scalability, fault isolation, maintainability, and service independence compared with a traditional monolithic application.
