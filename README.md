
# Group Buying Engine (Core)

A production-ready backend engine for a group-buy marketplace where users collaborate to unlock discounted prices based on participation thresholds.

This project demonstrates how to coordinate multiple users within a shared transactional flow, including threshold activation, participation tracking, lifecycle management, and aggregated order generation.

---

## Features

- Deal creation with participation thresholds
- Group buy campaign management
- User participation tracking
- Threshold-based activation
- Group lifecycle state transitions
- Aggregated order generation
- Inventory reservation support
- Payment intent tracking
- Idempotent participation requests
- Audit logs and outbox events
- Background jobs for expiry and finalization

---

## Use Cases

- Bulk community purchases
- Discount unlocking campaigns
- Cooperative buying platforms
- Time-boxed threshold commerce
- Marketplace-backed pooled orders

---

## Tech Stack

- PHP 8.2+
- Custom MVC architecture
- MySQL 8+
- PDO
- Composer PSR-4 autoloading
- Redis (optional but recommended)
- Dotenv for environment configuration

---

## Core Domain Concepts

### Deal
A purchasable offer with a threshold-based discount.

### Group Buy
A live campaign instance of a deal that accepts user participation within a defined time window.

### Participation
A user’s commitment to join a group buy.

### Threshold Activation
The process of marking a group as successful once the required participant count is reached.

### Aggregated Order
A single consolidated order generated from all successful participations in an activated group.

---

## Group Buy Lifecycle

- `pending`
- `open`
- `threshold_met`
- `activated`
- `expired`
- `cancelled`
- `fulfilled`
- `closed`

---

## Participation Lifecycle

- `pending`
- `reserved`
- `confirmed`
- `cancelled`
- `expired`
- `refunded`

---

## Project Structure

```txt
group-buy-engine/
├── app/
│   ├── Config/
│   ├── Controllers/
│   ├── Core/
│   ├── DTOs/
│   ├── Enums/
│   ├── Events/
│   ├── Exceptions/
│   ├── Jobs/
│   ├── Middleware/
│   ├── Models/
│   ├── Policies/
│   ├── Repositories/
│   ├── Routes/
│   ├── Services/
│   ├── Support/
│   └── Validators/
├── bootstrap/
├── database/
│   ├── migrations/
│   ├── seeders/
│   └── factories/
├── public/
├── storage/
├── tests/
├── .env.example
├── composer.json
└── README.md
````

---

## Example API Endpoints

### Auth

* `POST /api/v1/auth/register`
* `POST /api/v1/auth/login`
* `GET /api/v1/auth/me`

### Deals

* `GET /api/v1/deals`
* `POST /api/v1/deals`
* `GET /api/v1/deals/{dealId}`
* `PUT /api/v1/deals/{dealId}`

### Group Buys

* `GET /api/v1/group-buys`
* `POST /api/v1/group-buys`
* `GET /api/v1/group-buys/{groupId}`
* `GET /api/v1/group-buys/{groupId}/progress`

### Participations

* `POST /api/v1/group-buys/{groupId}/join`
* `GET /api/v1/participations`
* `GET /api/v1/participations/{participationId}`
* `PATCH /api/v1/participations/{participationId}/cancel`

### Aggregated Orders

* `GET /api/v1/aggregated-orders`
* `GET /api/v1/aggregated-orders/{orderId}`

### Payments

* `POST /api/v1/participations/{participationId}/payment-intents`
* `POST /api/v1/payments/webhook`

---

## Join Flow

1. User sends a join request with an idempotency key
2. System locks the target group-buy row
3. System validates:

   * group is open
   * deadline not passed
   * user has not already joined
   * quantity is allowed
4. Participation is created
5. Group counters are updated
6. If threshold is reached, activation workflow starts
7. Aggregated order is generated asynchronously

---

## Concurrency Protection

This project is designed to be safe under concurrent traffic.

Key protections:

* database transactions
* row locking with `SELECT ... FOR UPDATE`
* unique constraint on `(group_buy_id, user_id)`
* idempotency keys for join requests
* background finalization jobs
* optional Redis locking

---

## Setup

### 1. Clone the repository

```bash
git clone https://github.com/your-org/group-buy-engine.git
cd group-buy-engine
```

### 2. Install dependencies

```bash
composer install
```

### 3. Copy environment file

```bash
cp .env.example .env
```

### 4. Configure environment

Set your database, cache, queue, and app values in `.env`.

Example:

```env
APP_NAME=BundlyCore
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost:8000

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=group_buy_engine
DB_USERNAME=root
DB_PASSWORD=

REDIS_HOST=127.0.0.1
REDIS_PORT=6379
```

### 5. Run migrations

```bash
php artisan-like-cli.php migrate
```

### 6. Seed sample data

```bash
php artisan-like-cli.php seed
```

### 7. Start local server

```bash
php -S localhost:8000 -t public
```

---

## Recommended Jobs / Scheduler

Run these periodically:

* `ExpireGroupBuysJob`
* `FinalizeThresholdActivationJob`
* `GenerateAggregatedOrderJob`
* `SendNotificationJob`

---

## Testing

```bash
vendor/bin/phpunit
```

Test coverage should include:

* threshold activation
* duplicate participation prevention
* expiry handling
* order aggregation correctness
* webhook/payment intent flows
* concurrent join safety

---

## Future Enhancements

* coupon support
* wallet integration
* split fulfillment
* vendor settlement
* shipping coordination
* multi-tenant marketplace mode
* analytics dashboard
* real-time participation counters

---

