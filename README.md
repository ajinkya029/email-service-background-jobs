# 📧 Email Service with Background Jobs

A production-style **asynchronous email delivery backend** built with **Node.js, Express.js, Redis, BullMQ, MongoDB, and Nodemailer**.

Instead of making an API request wait for an SMTP server to send an email, this service places the email into a **Redis-backed background job queue** and immediately returns a job ID. A dedicated worker processes the job, sends the email through SMTP, retries failed deliveries automatically, and stores delivery status in MongoDB.

---

## 🛠️ Technologies Used

![Node.js](https://img.shields.io/badge/Node.js-20%2B-339933?style=for-the-badge&logo=node.js&logoColor=white)
![Express.js](https://img.shields.io/badge/Express.js-5.x-000000?style=for-the-badge&logo=express&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-7.x-DC382D?style=for-the-badge&logo=redis&logoColor=white)
![BullMQ](https://img.shields.io/badge/BullMQ-Background%20Jobs-E63946?style=for-the-badge)
![MongoDB](https://img.shields.io/badge/MongoDB-8.x-47A248?style=for-the-badge&logo=mongodb&logoColor=white)
![Mongoose](https://img.shields.io/badge/Mongoose-8.x-880000?style=for-the-badge&logo=mongoose&logoColor=white)
![Nodemailer](https://img.shields.io/badge/Nodemailer-SMTP-22B573?style=for-the-badge)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Zod](https://img.shields.io/badge/Zod-Validation-3E67B1?style=for-the-badge)
![REST API](https://img.shields.io/badge/API-REST-02569B?style=for-the-badge)

---

## ✨ Features

- 📬 Asynchronous email delivery
- ⚡ Redis-backed background job processing
- 🔄 BullMQ job queue
- 👷 Dedicated email worker
- 🔁 Automatic retry with exponential backoff
- 📊 MongoDB email delivery tracking
- 🔍 Email job status API
- 📤 SMTP email delivery using Nodemailer
- 🛡️ API-key authentication
- 🔐 Helmet security middleware
- 🚦 API rate limiting
- 🌐 CORS support
- ✅ Request validation using Zod
- 🐳 Docker Compose development environment
- 🧪 Automated validation tests
- 📁 Clean and scalable backend architecture

---

# 🏗️ Architecture

```text
                         ┌──────────────────┐
                         │      Client      │
                         └────────┬─────────┘
                                  │
                                  │ POST /api/emails
                                  ▼
                         ┌──────────────────┐
                         │   Express API    │
                         │                  │
                         │ Validation       │
                         │ Authentication   │
                         │ Rate Limiting    │
                         └────────┬─────────┘
                                  │
                                  │ Add Job
                                  ▼
                         ┌──────────────────┐
                         │ Redis + BullMQ   │
                         │                  │
                         │ Email Queue      │
                         │ Retry Handling   │
                         └────────┬─────────┘
                                  │
                                  │ Consume Job
                                  ▼
                         ┌──────────────────┐
                         │   Email Worker   │
                         │                  │
                         │ Concurrency: 5   │
                         └───────┬───┬──────┘
                                 │   │
                     ┌───────────┘   └────────────┐
                     ▼                            ▼
             ┌──────────────┐              ┌──────────────┐
             │  Nodemailer  │              │   MongoDB    │
             │     SMTP     │              │ Email Status │
             └──────┬───────┘              └──────────────┘
                    │
                    ▼
             ┌──────────────┐
             │ Email Inbox  │
             └──────────────┘
```

---

# 🔄 How It Works

### 1. Client sends an email request

```http
POST /api/emails
```

The API validates the request.

### 2. Email is added to the queue

The API adds a job to the **BullMQ email queue** stored in Redis.

### 3. API immediately responds

The client receives a job ID without waiting for SMTP delivery.

```json
{
  "success": true,
  "message": "Email queued successfully",
  "data": {
    "jobId": "1",
    "emailId": "68c...",
    "status": "queued"
  }
}
```

### 4. Worker processes the job

A separate worker retrieves the job from Redis and sends the email through Nodemailer.

### 5. Failed jobs are retried

If SMTP delivery fails, BullMQ automatically retries the job using **exponential backoff**.

```text
Attempt 1
   ↓
Failure
   ↓
Wait
   ↓
Attempt 2
   ↓
Failure
   ↓
Wait longer
   ↓
Attempt 3
   ↓
...
```

### 6. MongoDB stores the final status

The email record is updated as:

```text
queued
   ↓
processing
   ↓
sent
```

or:

```text
queued
   ↓
processing
   ↓
failed
```

---

# 📂 Project Structure

```text
email-service-background-jobs/
│
├── src/
│   ├── config/
│   │   ├── env.js
│   │   ├── mongo.js
│   │   └── redis.js
│   │
│   ├── controllers/
│   │   └── email.controller.js
│   │
│   ├── middleware/
│   │   ├── auth.js
│   │   └── error.js
│   │
│   ├── models/
│   │   └── email.model.js
│   │
│   ├── queue/
│   │   └── email.queue.js
│   │
│   ├── routes/
│   │   └── email.routes.js
│   │
│   ├── services/
│   │   └── mailer.service.js
│   │
│   ├── validators/
│   │   └── email.validator.js
│   │
│   ├── workers/
│   │   └── email.worker.js
│   │
│   ├── app.js
│   └── server.js
│
├── tests/
│   └── validator.test.js
│
├── docs/
│   └── api-examples.json
│
├── .env.example
├── .gitignore
├── docker-compose.yml
├── package.json
├── LICENSE
└── README.md
```

---

# 🚀 Getting Started

## Prerequisites

Make sure you have installed:

- [Node.js](https://nodejs.org/) 20+
- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- An SMTP email provider

You can use:

- Gmail SMTP
- SendGrid SMTP
- Mailgun SMTP
- Amazon SES
- Other SMTP providers

---

## 📥 Installation

Clone the repository:

```bash
git clone https://github.com/ajinkya029/email-service-background-jobs.git
```

Navigate into the project:

```bash
cd email-service-background-jobs
```

Install dependencies:

```bash
npm install
```

---

# 🐳 Start MongoDB and Redis

The project includes Docker Compose configuration for Redis and MongoDB.

```bash
docker compose up -d
```

Check running containers:

```bash
docker ps
```

You should see:

```text
email-service-redis
email-service-mongo
```

---

# ⚙️ Environment Variables

Create your `.env` file:

```bash
cp .env.example .env
```

Configure it:

```env
NODE_ENV=development

PORT=5000

MONGO_URI=mongodb://localhost:27017/email_service

REDIS_URL=redis://localhost:6379

SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_SECURE=false

SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-password

EMAIL_FROM="Email Service <your-email@gmail.com>"

API_KEY=change-me

EMAIL_MAX_ATTEMPTS=5
EMAIL_BACKOFF_DELAY=5000
```

> ⚠️ Never commit your `.env` file or SMTP credentials to GitHub.

---

# 📧 Gmail SMTP Configuration

If you are using Gmail, create a Google **App Password** and use it as:

```env
SMTP_PASS=your-app-password
```

Do **not** use your normal Gmail password.

---

# ▶️ Run the Application

The project contains two processes:

### API Server

```bash
npm run dev:api
```

### Background Worker

```bash
npm run dev:worker
```

Or run both simultaneously:

```bash
npm run dev
```

The API will be available at:

```text
http://localhost:5000
```

---

# ❤️ Health Check

### Request

```http
GET /health
```

### Response

```json
{
  "success": true,
  "service": "email-service",
  "status": "healthy",
  "timestamp": "2026-10-07T12:00:00.000Z"
}
```

---

# 📮 API Documentation

## Queue an Email

### Endpoint

```http
POST /api/emails
```

### Headers

```http
Content-Type: application/json
x-api-key: change-me
```

### Request Body

```json
{
  "to": "recipient@example.com",
  "subject": "Welcome to our platform",
  "html": "<h1>Welcome!</h1><p>Your account is ready.</p>",
  "text": "Welcome! Your account is ready."
}
```

### Multiple Recipients

```json
{
  "to": [
    "user1@example.com",
    "user2@example.com"
  ],
  "subject": "Team Update",
  "html": "<h2>Weekly Update</h2>",
  "text": "Weekly Update"
}
```

### CC / BCC

```json
{
  "to": "recipient@example.com",
  "cc": [
    "manager@example.com"
  ],
  "bcc": [
    "admin@example.com"
  ],
  "subject": "Project Update",
  "html": "<p>Project update...</p>"
}
```

---

## Successful Response

```json
{
  "success": true,
  "message": "Email queued successfully",
  "data": {
    "jobId": "1",
    "emailId": "68c123456789",
    "status": "queued"
  }
}
```

The API responds with **HTTP 202 Accepted** because the email has been accepted for background processing rather than synchronously delivered.

---

# 🔎 Check Email Status

### Endpoint

```http
GET /api/emails/:jobId
```

Example:

```http
GET /api/emails/1
```

### Response

```json
{
  "success": true,
  "data": {
    "jobId": "1",
    "to": [
      "recipient@example.com"
    ],
    "subject": "Welcome",
    "status": "sent",
    "attempts": 1,
    "messageId": "<some-message-id>",
    "sentAt": "2026-10-07T12:00:00.000Z"
  }
}
```

---

# 🧪 Testing with cURL

Queue an email:

```bash
curl -X POST http://localhost:5000/api/emails \
  -H "Content-Type: application/json" \
  -H "x-api-key: change-me" \
  -d '{
    "to": "your-email@example.com",
    "subject": "Background Job Test",
    "html": "<h2>Email delivered by a background worker</h2>",
    "text": "Email delivered by a background worker"
  }'
```

The response will contain:

```json
{
  "data": {
    "jobId": "1"
  }
}
```

Then check the job:

```bash
curl http://localhost:5000/api/emails/1 \
  -H "x-api-key: change-me"
```

---

# 🧪 Run Tests

Run the automated tests:

```bash
npm test
```

The tests currently cover:

- Valid email payloads
- Invalid email payloads
- Required email content
- Multiple recipients

---

# 🔁 Retry Strategy

The service uses BullMQ's retry mechanism.

Default configuration:

```env
EMAIL_MAX_ATTEMPTS=5
EMAIL_BACKOFF_DELAY=5000
```

The worker uses exponential backoff:

```text
Attempt 1
   ↓
5 seconds
   ↓
Attempt 2
   ↓
10 seconds
   ↓
Attempt 3
   ↓
20 seconds
   ↓
Attempt 4
   ↓
40 seconds
   ↓
Attempt 5
```

This prevents temporary SMTP failures from immediately causing permanent delivery failures.

---

# ⚡ Worker Concurrency

The email worker is configured to process multiple jobs concurrently:

```javascript
{
  concurrency: 5
}
```

This allows the service to process multiple email jobs without blocking one email behind another.

The worker can also be horizontally scaled by running additional worker processes.

---

# 🗄️ MongoDB Email Tracking

Each queued email is stored in MongoDB.

Example document:

```json
{
  "jobId": "1",
  "to": [
    "recipient@example.com"
  ],
  "subject": "Welcome",
  "status": "sent",
  "attempts": 1,
  "messageId": "<abc@example.com>",
  "sentAt": "2026-10-07T12:00:00.000Z"
}
```

Supported states:

```text
queued
processing
sent
failed
```

---

# 🔐 Security

The API includes several basic security measures.

### Helmet

Adds secure HTTP headers.

### Rate Limiting

Protects the API from excessive requests.

### API Key

Requests can optionally require:

```http
x-api-key: your-api-key
```

### Environment Variables

SMTP credentials and secrets are stored outside the source code.

---

# 📈 Production Improvements

For a production deployment, the following enhancements can be added:

- JWT authentication
- User/tenant-based email quotas
- Idempotency keys
- Email templates
- Template variables
- Scheduled emails
- Dead-letter queues
- Email attachments
- Provider failover
- SendGrid/Amazon SES integration
- Redis persistence and high availability
- MongoDB Atlas
- Structured logging
- Winston/Pino
- OpenTelemetry
- Prometheus metrics
- Grafana dashboards
- BullMQ monitoring dashboard
- Dockerized API and worker containers
- Kubernetes deployment
- Horizontal worker autoscaling
- SPF configuration
- DKIM configuration
- DMARC configuration

---

# 💡 Why Background Jobs?

A synchronous email API might look like:

```text
Client
  ↓
API
  ↓
SMTP Server
  ↓
Email
  ↓
Response
```

The client has to wait for SMTP processing.

With background jobs:

```text
Client
  ↓
API
  ↓
Redis Queue
  ↓
Immediate Response
```

Then independently:

```text
Redis Queue
  ↓
Worker
  ↓
SMTP
  ↓
Email
```

This architecture provides:

- ⚡ Faster API responses
- 🔄 Automatic retries
- 📈 Better scalability
- 🧱 Separation of responsibilities
- 🛡️ Better failure handling
- 👷 Independent worker scaling

---

# 🎯 Learning Outcomes

This project demonstrates practical backend concepts including:

- REST API development
- Express.js architecture
- Redis
- Message queues
- Background processing
- BullMQ
- Worker processes
- Job retries
- Exponential backoff
- SMTP
- Nodemailer
- MongoDB
- Mongoose
- Request validation
- Middleware
- API security
- Rate limiting
- Docker
- Environment configuration
- Error handling
- Scalable backend architecture

---

# 👨‍💻 Author

**Ajinkya Dhatrak**

Full-Stack MERN Developer

[![GitHub](https://img.shields.io/badge/GitHub-Ajinkya%20Dhatrak-181717?style=for-the-badge&logo=github)](https://github.com/ajinkya029)

---

# ⭐ Support

If you found this project useful, consider giving it a ⭐ on GitHub.
