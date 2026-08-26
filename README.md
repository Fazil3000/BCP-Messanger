# 🛡️ BCP Messenger

## Business Continuity Plan Messenger

> **Enterprise-grade Microservices Platform for Crisis Management, Emergency Employee Safety Surveys, and Instant Incident Notifications.**

BCP Messenger is an **event-driven microservices application** designed to help organizations communicate with employees during emergencies such as floods, cyclones, fire emergencies, power outages, internet outages, and other incidents.

The platform allows administrators to create emergency campaigns, approvers to authorize campaigns, employees to confirm their safety through surveys, and management to monitor response analytics.

The system is built using **Java, Spring Boot, Spring Cloud, Apache Kafka, MySQL, React, JWT, and Docker**.

---

# 📋 Table of Contents

1. [Project Overview](#-project-overview)
2. [Key Features](#-key-features)
3. [Technology Stack](#-technology-stack)
4. [Microservices Architecture](#-microservices-architecture)
5. [Services and Responsibilities](#-services-and-responsibilities)
6. [Architecture Flow](#-architecture-flow)
7. [Why Microservices?](#-why-microservices)
8. [API Gateway](#-api-gateway)
9. [Eureka Service Discovery](#-eureka-service-discovery)
10. [Config Server](#-config-server)
11. [Apache Kafka](#-apache-kafka)
12. [Database Architecture](#-database-architecture)
13. [Inter-Service Communication](#-inter-service-communication)
14. [Authentication and Security](#-authentication-and-security)
15. [Complete Application Flow](#-complete-application-flow)
16. [Incident Types](#-incident-types)
17. [Prerequisites](#-prerequisites)
18. [MySQL Setup](#-mysql-setup)
19. [Kafka with Docker](#-kafka-with-docker)
20. [Running the Microservices](#-running-the-microservices)
21. [Running the React Frontend](#-running-the-react-frontend)
22. [Application URLs](#-application-urls)
23. [Sample Users](#-sample-users)
24. [Postman API Testing](#-postman-api-testing)
25. [Email Configuration](#-email-configuration)
26. [Project Structure](#-project-structure)
27. [Troubleshooting](#-troubleshooting)
28. [Key Architectural Decisions](#-key-architectural-decisions)
29. [Future Enhancements](#-future-enhancements)

---

# 📌 Project Overview

During an emergency, organizations need to:

* Immediately notify employees.
* Confirm employee safety.
* Identify employees requiring assistance.
* Collect emergency survey responses.
* Track employee response status.
* Send emergency notifications.
* Analyze safety responses.

BCP Messenger solves these requirements using an **Event-Driven Microservices Architecture**.

The application separates authentication, campaign management, notification processing, configuration, service discovery, and API routing into independent services.

---

# ✨ Key Features

* 🚨 Emergency campaign management
* 👨‍💼 Admin and Approver workflow
* 👥 Employee management
* 🔐 JWT authentication
* 🔒 BCrypt password hashing
* 🎭 Role-based authorization
* 🌊 Multiple emergency incident types
* 📋 Pre-stored emergency survey questions
* 📨 Kafka-based asynchronous notifications
* 📧 Email notification support
* 📊 Emergency response analytics
* 🗄️ Database-per-service architecture
* 🔄 Flyway database migrations
* 🔎 Eureka service discovery
* ⚙️ Spring Cloud Config Server
* 🚪 Spring Cloud API Gateway
* ⚛️ React frontend
* 🐳 Docker-based Kafka setup
* 🧪 Postman API collection
* 🪟 Windows startup scripts

---

# 🛠️ Technology Stack

| Technology            | Purpose                             |
| --------------------- | ----------------------------------- |
| Java 17               | Backend development                 |
| Spring Boot 3.4.2     | Microservices framework             |
| Spring Cloud 2024.0.0 | Cloud-native microservices features |
| Spring Cloud Gateway  | API Gateway                         |
| Netflix Eureka        | Service Discovery                   |
| Spring Cloud Config   | Centralized configuration           |
| Spring Security       | Authentication and authorization    |
| JWT                   | Stateless authentication            |
| BCrypt                | Password hashing                    |
| Apache Kafka          | Event-driven communication          |
| MySQL 8               | Database                            |
| Flyway                | Database migration                  |
| React 18              | Frontend                            |
| Vite                  | Frontend build tool                 |
| Axios                 | REST API communication              |
| Maven                 | Backend build tool                  |
| Docker                | Kafka containerization              |
| Postman               | API testing                         |
| IntelliJ IDEA         | Development environment             |

---

# 🏗️ Microservices Architecture

BCP Messenger follows a **Microservices Architecture**.

Instead of creating one large monolithic application, the system is divided into independent services.

```text
                         ┌─────────────────────┐
                         │   React Frontend    │
                         │      :5173          │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │    API Gateway      │
                         │      :8080          │
                         └──────────┬──────────┘
                                    │
                    ┌───────────────┼────────────────┐
                    │               │                │
                    ▼               ▼                ▼
             ┌─────────────┐ ┌─────────────┐ ┌─────────────────┐
             │User Service │ │ BCP Service │ │Notification Svc │
             │    :8081    │ │    :8082    │ │      :8083      │
             └──────┬──────┘ └──────┬──────┘ └────────┬────────┘
                    │               │                  │
                    ▼               │                  ▼
             ┌─────────────┐        │          ┌─────────────────┐
             │bcp_user_db  │        │          │bcp_notification │
             └─────────────┘        │          │      _db        │
                                    │          └─────────────────┘
                                    ▼
                              ┌─────────────┐
                              │    Kafka    │
                              │    :9092    │
                              └──────┬──────┘
                                     │
                                     ▼
                              Notification Service


       Supporting Infrastructure
       ──────────────────────────
       Eureka Server      :8761
       Config Server      :8888
```

---

# 🔧 Services and Responsibilities

## 1. Eureka Server

**Port:** `8761`

Responsible for service discovery.

All microservices register themselves with Eureka.

```text
Microservice
     ↓
Eureka Registry
     ↓
Service Discovery
```

---

## 2. Config Server

**Port:** `8888`

Provides centralized configuration for the microservices.

Configuration can include:

* Kafka configuration
* JWT configuration
* Database properties
* Email configuration
* Environment-specific settings

---

## 3. API Gateway

**Port:** `8080`

Acts as the single entry point for the frontend.

```text
React
  ↓
API Gateway
  ↓
Required Microservice
```

Responsibilities:

* Request routing
* Service discovery
* CORS management
* Centralized gateway configuration
* Forwarding requests to backend services

The frontend does not directly call ports `8081`, `8082`, or `8083`.

---

## 4. User Service

**Port:** `8081`

Responsible for:

* User management
* Employee management
* Login
* JWT authentication
* BCrypt password hashing
* Role-based access

Database:

```text
bcp_user_db
```

---

## 5. BCP Service

**Port:** `8082`

This is the core business service.

Responsible for:

* Emergency campaign creation
* Campaign approval
* Incident management
* Survey questions
* Survey submission
* Survey responses
* Safety analytics
* Publishing Kafka events

Database:

```text
bcp_db
```

---

## 6. Notification Service

**Port:** `8083`

Responsible for:

* Consuming Kafka events
* Emergency email generation
* Sending/simulating emails
* Survey URL generation
* Notification history
* Delivery logs

Database:

```text
bcp_notification_db
```

---

# 🔄 Architecture Flow

```text
React Frontend
      │
      ▼
API Gateway
      │
      ▼
Eureka Service Discovery
      │
      ├──────────────► User Service
      │
      └──────────────► BCP Service
                              │
                              │ CampaignCreatedEvent
                              ▼
                         Apache Kafka
                              │
                              ▼
                     Notification Service
                              │
                              ▼
                       Email Notification
```

---

# 🤔 Why Microservices?

Microservices were selected because different parts of the application have different responsibilities and workload requirements.

### Independent Scaling

During an emergency, notification traffic can increase significantly.

The Notification Service can be scaled independently.

### Fault Isolation

If the email service has an issue, the User Service and BCP Service can continue operating.

### Independent Deployment

Each service can be built, tested, deployed, and maintained independently.

### Loose Coupling

Services communicate using REST APIs and Kafka events rather than directly depending on each other's internal implementation.

### Clear Responsibilities

```text
User Service          → Authentication & Users

BCP Service           → Campaigns & Surveys

Notification Service  → Notifications

API Gateway           → Routing

Eureka                → Service Discovery

Config Server         → Configuration
```

---

# 🚪 API Gateway

The API Gateway provides a single entry point.

Without the gateway:

```text
React
 ├── User Service :8081
 ├── BCP Service :8082
 └── Notification Service :8083
```

With the gateway:

```text
React
  ↓
API Gateway :8080
  ↓
Required Service
```

Benefits:

* Single entry point
* Hides internal service ports
* Centralized routing
* Dynamic service discovery
* Centralized CORS
* Easier frontend integration

---

# 🔎 Eureka Service Discovery

Eureka acts as the service registry.

When a service starts:

```text
User Service
     ↓
Registers with Eureka
     ↓
Eureka Registry
```

The Gateway can then discover the service dynamically.

Example:

```text
lb://USER-SERVICE
lb://BCP-SERVICE
lb://NOTIFICATION-SERVICE
```

This avoids hardcoding service IP addresses.

---

# ⚙️ Config Server

Spring Cloud Config Server provides externalized configuration.

The project uses the **native configuration profile** for local development.

This allows configuration to be managed separately from application code.

---

# 📨 Apache Kafka

Kafka provides asynchronous communication between the BCP Service and Notification Service.

## Kafka Flow

```text
BCP Service
     │
     │ Publish CampaignCreatedEvent
     ▼
Kafka Topic
bcp.campaign.created
     │
     ▼
Notification Service
     │
     ▼
Email Notification
```

## Why Kafka?

* Asynchronous processing
* Decoupling
* Fault tolerance
* Persistent events
* Consumer groups
* Scalable notification processing

The Kafka consumer group is:

```text
bcp-notification-group
```

---

# 🗄️ Database Architecture

The application follows the **Database-per-Service** pattern.

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

Each service owns its database.

### User Database

```text
users
```

### BCP Database

```text
campaigns
survey_questions
survey_responses
survey_answers
```

### Notification Database

```text
notification_logs
```

This provides database isolation and prevents direct database coupling between services.

---

# 🔗 Inter-Service Communication

The project uses two communication mechanisms.

## Synchronous REST

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
* Fetch questions
* Submit survey
* View analytics

## Asynchronous Kafka

Used for background processing.

```text
BCP Service
    ↓
Kafka
    ↓
Notification Service
```

Used for emergency notification dispatch.

---

# 🔐 Authentication and Security

The project uses:

### JWT

JWT provides stateless authentication.

```text
Login
  ↓
User Service
  ↓
JWT Token
  ↓
Frontend
```

### BCrypt

Passwords are stored using BCrypt hashing instead of plain text.

### Roles

```text
ADMIN
APPROVER
EMPLOYEE
```

---

# 🔄 Complete Application Flow

## 1. Login

```text
React
 ↓
POST /api/auth/login
 ↓
API Gateway :8080
 ↓
User Service :8081
 ↓
Validate BCrypt Password
 ↓
Generate JWT
 ↓
React stores JWT
```

---

## 2. Create Emergency Campaign

Admin selects an incident type.

Example:

```text
FLOOD
```

Frontend requests the corresponding questions:

```http
GET /api/bcp/questions/FLOOD
```

Admin enters the campaign title and message.

Campaign is created with:

```text
PENDING_APPROVAL
```

---

## 3. Approve Campaign

Approver reviews the campaign.

```http
PUT /api/bcp/campaigns/{id}/approve
```

The BCP Service:

1. Approves the campaign.
2. Updates the campaign status.
3. Publishes `CampaignCreatedEvent`.
4. Sends the event to Kafka.

---

## 4. Kafka Notification Processing

```text
BCP Service
      ↓
CampaignCreatedEvent
      ↓
Kafka
      ↓
Notification Service
```

Notification Service consumes the event and generates the emergency email.

---

## 5. Employee Survey

Employee receives the notification and opens:

```text
/survey/{surveyToken}
```

Employee answers the questions.

Example:

```text
Are you safe?                     YES
Do you need assistance?           NO
Can you work from home?           YES
Need temporary accommodation?     NO
Need transportation assistance?  NO
```

The response is stored in MySQL.

If an assistance-related question is answered `YES`, the system flags the employee as requiring assistance.

---

## 6. Analytics

Admin/Approver opens the analytics dashboard.

The system displays:

* Safety percentage
* YES/NO response statistics
* Employee responses
* Employees requiring assistance

---

# 🚨 Supported Incident Types

The system contains pre-stored questions for:

### 🌊 FLOOD

Safety, assistance, work-from-home, accommodation and transportation.

### 🌐 INTERNET_OUTAGE

Internet connectivity and alternate work location.

### ⚡ POWER_OUTAGE

Electricity availability and alternate workplace.

### 🌀 CYCLONE

Employee/family safety, assistance and work-from-home.

### 🔥 FIRE_EMERGENCY

Evacuation, medical assistance and emergency shelter.

### ⚠️ OTHER

General safety, assistance and work capability.

---

# 💻 Prerequisites

Install the following software before running the project.

| Software       | Version                      |
| -------------- | ---------------------------- |
| Java JDK       | 17                           |
| Maven          | 3.8+                         |
| Node.js        | 18+                          |
| npm            | Included with Node.js        |
| MySQL          | 8.x                          |
| Docker Desktop | Latest                       |
| Docker Compose | Included with Docker Desktop |
| Git            | Latest                       |
| IntelliJ IDEA  | Recommended                  |
| Postman        | Recommended                  |

---

# 🐳 Docker Desktop

Docker Desktop is required because **Apache Kafka is run using Docker**.

Verify Docker:

```bash
docker --version
```

Verify Docker Compose:

```bash
docker compose version
```

Make sure Docker Desktop is running before starting Kafka.

---

# 🗄️ MySQL Setup

Make sure MySQL Server is running on:

```text
localhost:3306
```

Create the databases using:

```bash
mysql -u root -p < scripts/setup-databases.sql
```

The following databases are required:

```text
bcp_user_db
bcp_db
bcp_notification_db
```

Flyway will create and migrate the required tables when the services start.

---

# 🐳 Kafka Setup Using Docker

Kafka is run through Docker instead of installing Kafka directly on Windows.

From the project root:

```bash
docker compose up -d
```

Check the running containers:

```bash
docker ps
```

Kafka should be available on:

```text
localhost:9092
```

View Kafka logs:

```bash
docker compose logs -f kafka
```

Stop Kafka:

```bash
docker compose stop
```

Start Kafka again:

```bash
docker compose start
```

Stop and remove containers:

```bash
docker compose down
```

> Make sure Kafka is running before starting the BCP Service and Notification Service.

---

# ▶️ Running the Microservices

Start the services in the following order.

## 1. Eureka Server

```bash
cd backend/eureka-server
mvn spring-boot:run
```

Port:

```text
8761
```

Verify:

```text
http://localhost:8761
```

---

## 2. Config Server

```bash
cd backend/config-server
mvn spring-boot:run
```

Port:

```text
8888
```

---

## 3. User Service

```bash
cd backend/user-service
mvn spring-boot:run
```

Port:

```text
8081
```

---

## 4. BCP Service

```bash
cd backend/bcp-service
mvn spring-boot:run
```

Port:

```text
8082
```

---

## 5. Notification Service

```bash
cd backend/notification-service
mvn spring-boot:run
```

Port:

```text
8083
```

This service connects to Kafka as a consumer.

---

## 6. API Gateway

```bash
cd backend/api-gateway
mvn spring-boot:run
```

Port:

```text
8080
```

---

# ⚛️ Running the React Frontend

Open a terminal inside the frontend directory:

```bash
cd frontend
```

Install dependencies:

```bash
npm install
```

Start the application:

```bash
npm run dev
```

Open:

```text
http://localhost:5173
```

---

# 🚀 Complete Startup Order

The recommended startup sequence is:

```text
1. Docker Desktop
        ↓
2. Kafka Container
        ↓
3. MySQL
        ↓
4. Eureka Server :8761
        ↓
5. Config Server :8888
        ↓
6. User Service :8081
        ↓
7. BCP Service :8082
        ↓
8. Notification Service :8083
        ↓
9. API Gateway :8080
        ↓
10. React Frontend :5173
```

---

# 🌐 Application URLs

| Component            | URL                   |
| -------------------- | --------------------- |
| React Frontend       | http://localhost:5173 |
| API Gateway          | http://localhost:8080 |
| Eureka Dashboard     | http://localhost:8761 |
| Config Server        | http://localhost:8888 |
| User Service         | http://localhost:8081 |
| BCP Service          | http://localhost:8082 |
| Notification Service | http://localhost:8083 |
| Kafka                | localhost:9092        |
| MySQL                | localhost:3306        |

---

# 👥 Sample Users

The project contains pre-seeded test users.

| Email                   | Password       | Role     | Employee ID |
| ----------------------- | -------------- | -------- | ----------- |
| `admin@company.com`     | `Admin@123`    | ADMIN    | EMP001      |
| `approver@company.com`  | `Approver@123` | APPROVER | EMP002      |
| `employee1@company.com` | `Employee@123` | EMPLOYEE | EMP003      |
| `employee2@company.com` | `Employee@123` | EMPLOYEE | EMP004      |
| `employee3@company.com` | `Employee@123` | EMPLOYEE | EMP005      |

> ⚠️ These credentials are for local development/testing only.

---

# 🧪 Postman API Testing

Import the Postman collection:

```text
postman/BCP_Messenger_API_Collection.json
```

## Login

```http
POST http://localhost:8080/api/auth/login
```

```json
{
  "email": "admin@company.com",
  "password": "Admin@123"
}
```

---

## Get Incident Questions

```http
GET http://localhost:8080/api/bcp/questions/FLOOD
```

---

## Create Campaign

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

---

## Approve Campaign

```http
PUT http://localhost:8080/api/bcp/campaigns/1/approve
```

This approval triggers the Kafka notification flow.

---

## Submit Survey

```http
POST http://localhost:8080/api/bcp/surveys/{surveyToken}/submit
```

Example:

```json
{
  "employeeEmail": "employee1@company.com",
  "employeeName": "John Doe",
  "employeeId": "EMP003",
  "comments": "Safe at home, no assistance needed.",
  "answers": [
    {
      "questionId": 1,
      "answer": "YES"
    },
    {
      "questionId": 2,
      "answer": "NO"
    }
  ]
}
```

---

## View Analytics

```http
GET http://localhost:8080/api/bcp/campaigns/1/analytics
```

---

# 📧 Email Configuration

The Notification Service supports offline mode.

By default:

```properties
app.email.enabled=false
```

In offline mode:

* Emails are not actually sent.
* Email content is printed in the console.
* Notification records are stored as `SIMULATED`.

For actual SMTP delivery:

```text
APP_EMAIL_ENABLED=true
MAIL_USERNAME=your-email@gmail.com
MAIL_PASSWORD=your-app-password
```

> Never commit real email passwords, API keys, JWT secrets, or other credentials to GitHub.

---

# 🔄 Project Request Flow

```text
                    USER LOGIN
                       │
                       ▼
                 React Frontend
                       │
                       ▼
                API Gateway :8080
                       │
                       ▼
                User Service :8081
                       │
                       ▼
                    JWT Token
                       │
                       ▼
                  User Logged In


                 CREATE CAMPAIGN
                       │
                       ▼
                BCP Service :8082
                       │
                       ▼
                PENDING_APPROVAL
                       │
                       ▼
                 Approver Approval
                       │
                       ▼
                CampaignCreatedEvent
                       │
                       ▼
                  Apache Kafka
                       │
                       ▼
            Notification Service :8083
                       │
                       ▼
                 Email / Simulation
                       │
                       ▼
                  Employee Survey
                       │
                       ▼
                BCP Service :8082
                       │
                       ▼
                   MySQL
                       │
                       ▼
                   Analytics
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
│   ├── start-all-services.bat
│   ├── stop-all-services.bat
│   └── ...
│
├── postman/
│   └── BCP_Messenger_API_Collection.json
│
├── docker-compose.yml
│
└── README.md
```

---

# 🧯 Troubleshooting

## Port Already in Use

If a service shows:

```text
Address already in use: bind
```

Stop the running services or use:

```cmd
scripts\stop-all-services.bat
```

---

## Kafka Connection Refused

If you see:

```text
Connection to node -1 could not be established
```

Check:

```bash
docker ps
```

Make sure the Kafka container is running.

Also verify:

```text
localhost:9092
```

---

## MySQL Connection Error

Make sure:

```text
MySQL Server → Running
Host → localhost
Port → 3306
```

Also verify the configured username and password.

---

## Service Not Found Through Gateway

Check the Eureka dashboard:

```text
http://localhost:8761
```

Verify that:

```text
USER-SERVICE
BCP-SERVICE
NOTIFICATION-SERVICE
```

are registered.

---

# 🧠 Key Architectural Decisions

| Decision             | Reason                                             |
| -------------------- | -------------------------------------------------- |
| Microservices        | Independent scaling and fault isolation            |
| API Gateway          | Single entry point and centralized routing         |
| Eureka               | Dynamic service discovery                          |
| Config Server        | Centralized external configuration                 |
| Kafka                | Asynchronous notification processing               |
| REST                 | Immediate request/response operations              |
| Database-per-Service | Data isolation and loose coupling                  |
| JWT                  | Stateless authentication                           |
| BCrypt               | Secure password hashing                            |
| Flyway               | Version-controlled database migrations             |
| Docker               | Simplified Kafka setup and environment consistency |
| React                | Responsive frontend application                    |

---

# 🎯 Why REST and Kafka Both?

REST and Kafka solve different problems.

### REST

Used when the caller needs an immediate response.

```text
Login
Fetch Questions
Submit Survey
View Analytics
```

### Kafka

Used for asynchronous background processing.

```text
Campaign Approved
       ↓
Kafka Event
       ↓
Notification Processing
```

This combination keeps the application responsive while allowing notification processing to happen independently.

---

# 📈 Scalability

The architecture allows individual services to scale according to workload.

For example:

```text
Normal Situation

User Service       → 1 instance
BCP Service        → 1 instance
Notification       → 1 instance
```

During a major emergency:

```text
User Service       → 1 instance
BCP Service        → 2 instances
Notification       → Multiple instances
```

Additional Notification Service instances can consume Kafka messages using the same consumer group.

---

# 🔒 Security Considerations

The project implements:

* JWT authentication
* BCrypt password hashing
* Role-based access
* Stateless authentication
* Separate databases
* Externalized configuration

For production deployment:

* Store secrets in environment variables or a secrets manager.
* Do not commit passwords to Git.
* Use HTTPS.
* Rotate JWT secrets.
* Use secure database credentials.
* Restrict database/network access.

---

# 🚀 Future Enhancements

Possible future improvements include:

* 📱 SMS notifications
* 💬 WhatsApp notifications
* 🔔 Push notifications
* 🐳 Full Dockerization of all microservices
* ☸️ Kubernetes deployment
* ☁️ AWS/cloud deployment
* 📊 Prometheus and Grafana monitoring
* 🔍 Distributed tracing
* 📝 Centralized logging
* 🔄 CI/CD pipeline
* 🌍 Multi-region deployment
* 🛡️ Advanced disaster recovery

---

# ✅ Project Highlights

```text
✅ Java 17
✅ Spring Boot Microservices
✅ Spring Cloud
✅ API Gateway
✅ Eureka Service Discovery
✅ Config Server
✅ Apache Kafka
✅ Kafka Producer & Consumer
✅ Docker-based Kafka
✅ MySQL
✅ Database-per-Service
✅ Flyway
✅ JWT Authentication
✅ BCrypt Password Security
✅ Role-Based Authorization
✅ React + Vite
✅ Emergency Campaign Management
✅ Employee Safety Surveys
✅ Notification Service
✅ Analytics Dashboard
✅ Postman API Testing
```

---

# 🏁 Conclusion

BCP Messenger demonstrates a real-world implementation of **Microservices Architecture and Event-Driven Architecture** for emergency business continuity management.

The system separates authentication, campaign management, notification processing, configuration, service discovery, and API routing into independent services.

It combines:

```text
Microservices
      +
API Gateway
      +
Eureka Service Discovery
      +
Config Server
      +
REST APIs
      +
Apache Kafka
      +
Database-per-Service
      +
JWT Security
      +
React Frontend
      +
Docker
```

The result is a **scalable, loosely coupled, maintainable, and event-driven emergency communication platform** suitable for demonstrating enterprise microservices concepts.
