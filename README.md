# Digital Banking System — Microservices

A containerized digital banking backend built with **Spring Boot microservices**, **Apache Kafka**, **Redis**, **MySQL**, and **Spring Cloud Gateway**.

The system supports account management, balance tracking, money transfers, transaction history, payment integration, fraud detection, and notifications using an event-driven microservices architecture.

---

## 🚀 Features

- Account creation and management
- Balance inquiry
- Money transfers between accounts
- Transaction history
- Event-driven transaction processing using Apache Kafka
- Real-time fraud detection using Redis
- API Gateway with rate limiting
- Payment processing integration with Razorpay
- Notification service for transaction/fraud events
- MySQL persistence
- Redis-based fraud detection and rate limiting
- Dockerized microservices
- Environment-based configuration for sensitive credentials

---

## 🏗️ Architecture

```text
                         ┌─────────────────────┐
                         │       Client        │
                         │  Postman / Frontend │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │    API Gateway      │
                         │       :8080         │
                         │  Rate Limiting      │
                         └──────────┬──────────┘
                                    │
              ┌─────────────────────┼─────────────────────┐
              │                     │                     │
              ▼                     ▼                     ▼
     ┌────────────────┐   ┌─────────────────┐   ┌─────────────────┐
     │ Account Service│   │Transaction      │   │ Payment Service │
     │     :8081      │   │Service :8082   │   │     :8083       │
     └───────┬────────┘   └────────┬────────┘   └────────┬────────┘
             │                     │                     │
             │                     ▼                     │
             │             ┌───────────────┐             │
             │             │ Apache Kafka  │◄────────────┘
             │             └───────┬───────┘
             │                     │
             │          ┌──────────┴──────────┐
             │          │                     │
             ▼          ▼                     ▼
     ┌─────────────┐  ┌────────────────┐  ┌──────────────────┐
     │    MySQL    │  │ Fraud Detection │  │  Notification    │
     │             │  │     :8084       │  │     :8085        │
     │ account_db  │  │     Redis       │  │                  │
     │ payment_db  │  │                 │  │                  │
     │transaction_db│  └────────────────┘  └──────────────────┘
     └─────────────┘
🧩 Microservices

| Service                 |   Port | Responsibility                                 |
| ----------------------- | -----: | ---------------------------------------------- |
| API Gateway             | `8080` | Single entry point, routing and rate limiting  |
| Account Service         | `8081` | Account creation, account details and balances |
| Transaction Service     | `8082` | Money transfers and transaction management     |
| Payment Service         | `8083` | Payment processing and Razorpay integration    |
| Fraud Detection Service | `8084` | Transaction fraud analysis using Redis         |
| Notification Service    | `8085` | Processes transaction and fraud notifications  |

🛠️ Technology Stack
Backend
Java 17
Spring Boot
Spring Cloud Gateway
Spring Data JPA
Spring Validation
Spring Web
Maven
Database & Caching
MySQL 8.4
Redis
Messaging
Apache Kafka
Apache Zookeeper
DevOps / Tools
Docker
Docker Compose
Git
GitHub
Postman
DBeaver
Payment
Razorpay API
🔄 Transaction Flow

A typical money transfer follows this flow:

Client
   │
   ▼
API Gateway
   │
   ▼
Transaction Service
   │
   │  transaction.initiated
   ▼
Apache Kafka
   │
   ▼
Fraud Detection Service
   │
   │  fraud.check.result
   ▼
Apache Kafka
   │
   ▼
Transaction Service
   │
   ├──────────────► Account Service
   │                   │
   │                   ▼
   │              Update Balance
   │
   └──────────────► Notification Service
                       │
                       ▼
                  Alert User

The transaction initially enters a processing state. Fraud detection evaluates the transaction and the result is communicated through Kafka before the transaction is completed.

📡 Kafka Events

| Topic                   | Producer                | Consumer                              |
| ----------------------- | ----------------------- | ------------------------------------- |
| `transaction.initiated` | Transaction Service     | Fraud Detection Service               |
| `fraud.check.result`    | Fraud Detection Service | Transaction Service                   |
| `transaction.completed` | Transaction Service     | Account Service, Notification Service |
| `fraud.detected`        | Fraud Detection Service | Account Service, Notification Service |
| `payment.completed`     | Payment Service         | Notification Service                  |

🗄️ Databases

The application uses separate databases for different services:
MySQL
│
├── account_db
├── transaction_db
└── payment_db

This keeps service-specific persistence separated within the banking system.

Redis is used by the fraud detection service for transaction-related checks and by the API Gateway for rate limiting.

🐳 Running the Project
Prerequisites

Install:

Docker Desktop
Git
Java 17
Maven 3.9+
Postman (optional, for API testing)
1. Clone the repository
git clone git@github.com:NikhilNexus790/digital-banking-system-microservices.git

cd digital-banking-system-microservices

2. Configure environment variables

Create a .env file in the project root.

Example:
MYSQL_ROOT_PASSWORD=your_mysql_root_password

RAZORPAY_KEY_ID=your_razorpay_test_key_id
RAZORPAY_KEY_SECRET=your_razorpay_test_key_secret
RAZORPAY_WEBHOOK_SECRET=your_razorpay_webhook_secret
Never commit .env to GitHub.

A safe template is available in:
.env.example

3. Build and start the complete system

From the project root:
docker compose up --build -d

This builds the six Spring Boot services and starts the complete infrastructure.

4. Check running containers
docker compose ps
You should see services for:
zookeeper
kafka
mysql
redis
account-service
transaction-service
payment-service
fraud-detection-service
notification-service
api-gateway

5. View logs

For all services:
docker compose logs -f

For a specific service:
docker compose logs -f transaction-service

6. Stop the system
docker compose down

To stop and remove volumes as well:

docker compose down -v

Use down -v carefully because it removes Docker volumes and therefore persistent database data stored in those volumes.

🔌 API Gateway

Client requests are routed through:

http://localhost:8080

Example routes:

/api/v1/accounts/**
/api/v1/transactions/**
/api/v1/payments/**

The API Gateway also provides request rate limiting using Redis.

🧪 API Testing

The APIs were tested using Postman.

Example banking flow:
1. Create Sender Account
          ↓
2. Create Receiver Account
          ↓
3. Check Sender Balance
          ↓
4. Transfer Money
          ↓
5. Fraud Detection
          ↓
6. Transaction Completion
          ↓
7. Account Balance Update
          ↓
8. Transaction History

The transaction workflow was tested successfully with account creation, balance verification, money transfer, transaction completion, and transaction history retrieval.

🔐 Security & Configuration

Sensitive configuration is externalized using environment variables.

The repository does not store:

MySQL passwords
Razorpay secret keys
Razorpay webhook secrets
API credentials

The following files are excluded from Git:
.env
.idea/
**/target/

📁 Project Structure
digital-banking-system-microservices/
│
├── account-service/
├── transaction-service/
├── payment-service/
├── fraud-detection-service/
├── notification-service/
├── api-gateway/
│
├── docker-compose.yml
├── .env.example
├── .gitignore
└── README.md

📈 Future Improvements
Add Spring Security with JWT authentication
Add centralized configuration using Spring Cloud Config
Add distributed tracing
Add Prometheus and Grafana monitoring
Add automated unit and integration tests
Add CI/CD pipeline using GitHub Actions
Deploy the system to AWS
Improve notification delivery with email/SMS providers
Add comprehensive API documentation with Swagger/OpenAPI

👨‍💻 Author

Nikhil Prasad

B.Tech — Electronics & Communication Engineering

GitHub:
https://github.com/NikhilNexus790

📄 License

This project is intended for educational and portfolio purposes.