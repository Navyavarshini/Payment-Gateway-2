
# Quick Start Guide – Production Payment Gateway

## Prerequisites

Ensure the following are installed before proceeding:

* Docker and Docker Compose
* Node.js 18+ (for local development only)
* PostgreSQL 15 (provided via Docker)
* Redis 7 (provided via Docker)

---

## Setup and Launch

### 1. Start All Services

```bash
cd payment-gateway
docker-compose up --build
```

Services start in a dependency-safe order using health checks:

1. PostgreSQL (5432)
2. Redis (6379)
3. API Server (8000)
4. Worker Service (background jobs)
5. Dashboard Frontend (3000)
6. Checkout Page (3001)

---

### 2. Verify Service Health

```bash
# API health check
curl http://localhost:8000/health

# Job queue status
curl http://localhost:8000/api/v1/test/jobs/status

# Redis connectivity
redis-cli -p 6379 ping
```

All commands should return a successful response.

---

### 3. Retrieve Test Merchant Credentials

```bash
curl http://localhost:8000/api/v1/test/merchant
```

Example response:

```json
{
  "id": "merchant-uuid",
  "api_key": "key_test_abc123",
  "api_secret": "secret_test_xyz789",
  "webhook_url": null,
  "webhook_secret": "whsec_test_abc123"
}
```

These credentials are used for all local testing.

---

## End-to-End Payment Flow Testing

### Step 1: Access the Dashboard

1. Open [http://localhost:3000](http://localhost:3000)
2. Log in using the test credentials
3. Confirm dashboard access

---

### Step 2: Create a Test Order

1. Click **Create Test Order**
2. You are redirected to the checkout page
3. Complete the payment using the simulated processor

---

### Step 3: Monitor Background Processing

1. Open:

   ```
   http://localhost:8000/api/v1/test/jobs/status
   ```
2. Observe job states transitioning from:

   ```
   pending → processing → completed
   ```
3. Status refreshes every two seconds

---

### Step 4: Validate Webhook Delivery

#### Start Test Webhook Receiver

```bash
cd test-merchant
npm install
node webhook.js
```

#### Configure Webhook in Dashboard

* **Mac / Windows**

  ```
  http://host.docker.internal:4000/webhook
  ```
* **Linux**

  ```
  http://172.17.0.1:4000/webhook
  ```

Create a new payment and verify webhook payloads are received and validated.

---

### Step 5: Test Refund Flow

```bash
curl -X POST http://localhost:8000/api/v1/payments/{payment_id}/refunds \
  -H "X-Api-Key: key_test_abc123" \
  -H "X-Api-Secret: secret_test_xyz789" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 50000,
    "reason": "Customer requested"
  }'
```

Confirm refund processing and webhook notification.

---

## API Usage Examples

### Create Order

```bash
curl -X POST http://localhost:8000/api/v1/orders \
  -H "X-Api-Key: key_test_abc123" \
  -H "X-Api-Secret: secret_test_xyz789" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 50000,
    "currency": "INR",
    "receipt": "receipt_123"
  }'
```

---

### Create Payment (Idempotent)

```bash
curl -X POST http://localhost:8000/api/v1/payments \
  -H "X-Api-Key: key_test_abc123" \
  -H "X-Api-Secret: secret_test_xyz789" \
  -H "Idempotency-Key: unique-request-123" \
  -H "Content-Type: application/json" \
  -d '{
    "order_id": "order_abc123",
    "method": "upi",
    "vpa": "user@paytm"
  }'
```

---

### Get Payment Status

```bash
curl http://localhost:8000/api/v1/payments/{payment_id}/public
```

---

### List Webhook Logs

```bash
curl http://localhost:8000/api/v1/webhooks \
  -H "X-Api-Key: key_test_abc123" \
  -H "X-Api-Secret: secret_test_xyz789"
```

---

### Retry Failed Webhook

```bash
curl -X POST http://localhost:8000/api/v1/webhooks/{webhook_id}/retry \
  -H "X-Api-Key: key_test_abc123" \
  -H "X-Api-Secret: secret_test_xyz789"
```

---

## SDK Integration

### Embed Checkout on Your Website

```html
<script src="http://localhost:3001/checkout.js"></script>
<button id="pay-button">Pay Now</button>

<script>
document.getElementById('pay-button').addEventListener('click', () => {
  const checkout = new PaymentGateway({
    key: 'key_test_abc123',
    orderId: 'order_xyz',
    onSuccess: (response) => {
      console.log('Payment successful:', response.paymentId);
    },
    onFailure: (error) => {
      console.log('Payment failed:', error);
    },
    onClose: () => {
      console.log('Checkout closed');
    }
  });
  checkout.open();
});
</script>
```

---

## Testing Modes

### Deterministic Payment Outcomes

Configure backend `.env`:

```env
TEST_MODE=true
TEST_PAYMENT_SUCCESS=true
TEST_PROCESSING_DELAY=1000
```

To force failures:

```env
TEST_PAYMENT_SUCCESS=false
TEST_PROCESSING_DELAY=500
```

---

### Accelerated Webhook Retry Testing

```env
WEBHOOK_RETRY_INTERVALS_TEST=true
```

Retry schedule:

* Immediate
* 5 seconds
* 10 seconds
* 15 seconds
* 20 seconds

---

## Database Debugging

### Connect to PostgreSQL

```bash
psql postgresql://gateway_user:gateway_pass@localhost:5432/payment_gateway
```

### Common Queries

```sql
SELECT id, status, created_at FROM payments ORDER BY created_at DESC LIMIT 10;
SELECT id, event, status, attempts FROM webhook_logs ORDER BY created_at DESC;
SELECT id, payment_id, status FROM refunds ORDER BY created_at DESC;
SELECT key, merchant_id, expires_at FROM idempotency_keys;
```

---

## Redis Debugging

```bash
redis-cli -p 6379
```

```bash
LLEN bull:payments:wait
LLEN bull:webhooks:wait
LLEN bull:refunds:wait
HGETALL bull:payments:stats
MONITOR
```

---

## Logs and Troubleshooting

### View Logs

```bash
docker-compose logs -f
docker-compose logs -f api
docker-compose logs -f worker
```

---

### Common Issues

**Webhooks not delivered**

* Webhook URL not configured
* Receiver not reachable
* Signature mismatch
* Network issues

**Jobs not processing**

* Redis not running
* Worker container down
* Queue backlog

**Payment delays**

* Worker concurrency too low
* Processing delay too high
* Redis under load

**Signature verification failure**

* Secret mismatch
* Payload mutation
* Encoding issues

---

## Performance Tuning

### Increase Worker Concurrency

```js
concurrency: 10
```

### Redis Optimization

```js
enableOfflineQueue: true
```

### Database Pooling

```js
max: 20,
min: 5
```

---

## Stopping Services

```bash
docker-compose down
docker-compose down -v
docker-compose restart
```

---

## Production Deployment Checklist

### Before Going Live

* Disable test mode
* Enforce HTTPS
* Rotate secrets
* Enable rate limiting
* Use managed PostgreSQL and Redis
* Enable monitoring and alerting
* Scale workers horizontally

---

## Access Points

* Dashboard: [http://localhost:3000/dashboard](http://localhost:3000/dashboard)
* API Docs: [http://localhost:3000/dashboard/docs](http://localhost:3000/dashboard/docs)
* Webhooks: [http://localhost:3000/dashboard/webhooks](http://localhost:3000/dashboard/webhooks)
* Implementation Guide: `IMPLEMENTATION_GUIDE.md`

