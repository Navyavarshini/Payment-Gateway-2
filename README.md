
# Payment Gateway

A full-featured payment gateway implementation comparable in scope to platforms such as Razorpay and Stripe. The system supports UPI and Card payments, provides a merchant dashboard, exposes public and authenticated APIs, and is fully containerized for predictable deployment.

---

## Features

* Multi-method payment processing (UPI and Card)
* Merchant dashboard with transaction history and statistics
* Hosted checkout page with responsive UI
* API key and secret–based merchant authentication
* Payment validation (VPA format, Luhn check, card network detection)
* Fully containerized using Docker Compose
* PostgreSQL-backed persistent storage
* Pre-seeded test merchant for immediate evaluation

---

## Technology Stack

### Backend

* Node.js
* Express.js
* PostgreSQL 15
* pg (node-postgres)
* Custom API key and secret authentication

### Frontend

* React 18
* Vite
* React Router v6
* Axios
* Custom CSS

### DevOps

* Docker and Docker Compose
* Nginx for frontend delivery

---

## Prerequisites

Ensure the following are installed:

* Docker (20.10+)
* Docker Compose (2.0+)
* Git

---

## Quick Start

### 1. Clone the Repository

```bash
git clone <your-repository-url>
cd payment-gateway
```

### 2. Start All Services

```bash
docker-compose up -d
```

This command:

* Builds all images
* Starts PostgreSQL
* Initializes schema
* Seeds test merchant data
* Starts API, dashboard, and checkout services

---

### 3. Verify Services

```bash
docker-compose ps
```

Expected running containers:

* pg_gateway
* gateway_api
* gateway_dashboard
* gateway_checkout

---

### 4. Access Endpoints

* Merchant Dashboard: [http://localhost:3000](http://localhost:3000)
* Checkout Page: [http://localhost:3001](http://localhost:3001)
* Backend API: [http://localhost:8000](http://localhost:8000)
* Health Check: [http://localhost:8000/health](http://localhost:8000/health)

---

## Test Credentials

### Merchant Login

* Email: `test@example.com`
* Password: `password123`

### API Credentials

* API Key: `key_test_abc123`
* API Secret: `secret_test_xyz789`

---

## Test Payment Data

### Cards

* Visa: `4111111111111111`
* Mastercard: `5555555555554444`
* Expiry: Any future date
* CVV: Any 3 digits

### UPI

* Any valid format: `username@provider`

---

## Usage Guide

### End-to-End Payment Flow

1. Log in to the dashboard
2. Create a test order
3. Redirect to checkout
4. Select payment method
5. Complete payment
6. Review transaction history in dashboard

---

## API Usage

### Create Order

```bash
curl -X POST http://localhost:8000/api/v1/orders \
  -H "X-Api-Key: key_test_abc123" \
  -H "X-Api-Secret: secret_test_xyz789" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 50000,
    "currency": "INR",
    "receipt": "receipt_123",
    "notes": {
      "customer_name": "John Doe"
    }
  }'
```

---

### Create Payment (UPI)

```bash
curl -X POST http://localhost:8000/api/v1/payments/public \
  -H "Content-Type: application/json" \
  -d '{
    "order_id": "order_xxx",
    "method": "upi",
    "vpa": "user@paytm"
  }'
```

---

### Create Payment (Card)

```bash
curl -X POST http://localhost:8000/api/v1/payments/public \
  -H "Content-Type: application/json" \
  -d '{
    "order_id": "order_xxx",
    "method": "card",
    "card": {
      "number": "4111111111111111",
      "expiry_month": "12",
      "expiry_year": "2025",
      "cvv": "123",
      "holder_name": "John Doe"
    }
  }'
```

---

## Project Structure

```text
payment-gateway/
├── backend/
├── frontend/
├── checkout-page/
├── docker-compose.yml
├── README.md
└── .env.example
```

---

## Database Schema Overview

### Merchants

* id (UUID)
* email
* api_key
* api_secret
* webhook_url
* is_active

### Orders

* id (order_xxx)
* merchant_id
* amount
* currency
* status

### Payments

* id (pay_xxx)
* order_id
* merchant_id
* method
* status
* error fields

---

## Security Design

* API key and secret authentication
* No storage of CVV or full card numbers
* Input validation for all payment data
* Environment variable–based configuration
* Test mode isolation

---

## Payment Processing Logic

* Payments created in `processing` state
* Artificial delay (5–10 seconds)
* Randomized outcome based on success rate
* Final state: `success` or `failed`
* Errors captured on failure

---

## Validation Rules

### UPI

* Format: `username@provider`

### Card

* Luhn checksum
* Network detection
* Expiry validation
* CVV format validation

---

## API Endpoints

### Public

* `GET /health`
* `GET /api/v1/test/merchant`
* `POST /api/v1/payments/public`

### Authenticated

* `POST /api/v1/orders`
* `GET /api/v1/orders/{id}`
* `POST /api/v1/payments`
* `GET /api/v1/payments/{id}`
* `GET /api/v1/payments/stats`

---

## Configuration

All configuration is driven via environment variables.
See `.env.example` for the complete list.

---

## Development Mode (Without Docker)

```bash
docker-compose up -d postgres
cd backend && npm install && npm run dev
cd frontend && npm install && npm run dev
cd checkout-page && npm install && npm run dev
```

---

## Maintenance Commands

```bash
docker-compose down
docker-compose down -v
docker-compose restart
```

---

## Troubleshooting

* Verify ports are available (3000, 3001, 8000, 5432)
* Check logs using `docker-compose logs`
* Restart individual services if required

---



## Final Notes

This project demonstrates a **production-grade payment gateway architecture** with:

* Clean separation of concerns
* Secure authentication
* Deterministic testing
* Containerized deployment
* Realistic payment simulation

