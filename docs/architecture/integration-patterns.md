# Integration Patterns & Event Contracts

## Overview

This document defines the communication patterns between components. It covers both synchronous (API calls) and asynchronous (events/messages) integration.

---

## Communication Principles

### 1. Prefer Async for Side Effects
When a component needs to trigger actions in other components (notifications, analytics, etc.), use events rather than direct API calls.

### 2. Use Sync for Data Queries
When a component needs data from another component to complete its operation, use direct API calls.

### 3. Include Sufficient Context in Events
Events should contain enough information for consumers to act without additional API calls when possible.

### 4. Use Shared Models
Define event schemas in a shared library to ensure consistency across publishers and consumers.

---

## Event Schema Standards

### Base Event Structure

All events MUST include these fields:

```typescript
interface BaseEvent {
  // Unique identifier for this event instance
  eventId: string;
  
  // When the event occurred
  timestamp: string;  // ISO 8601 format
  
  // Event type for routing/filtering
  eventType: string;
  
  // Correlation ID for distributed tracing
  correlationId: string;
  
  // Source component that published the event
  source: string;
  
  // Schema version for evolution
  schemaVersion: number;
}
```

### Event Naming Convention

- **Format:** `{Entity}{Action}Event`
- **Examples:**
  - `OrderCreatedEvent`
  - `ShipmentShippedEvent`
  - `TransactionAlertDetectedEvent`
  - `CustomerTierChangedEvent`

---

## Event Schemas by Domain

### Order Domain Events

#### OrderCreatedEvent
```typescript
interface OrderCreatedEvent extends BaseEvent {
  eventType: 'OrderCreated';
  
  // Core data
  orderId: string;
  customerId: string;
  
  // Order details
  orderTotal: number;
  currency: string;
  orderSource: 'WEB' | 'MOBILE' | 'API';
  
  // Line items (summary)
  itemCount: number;
  
  // Timestamps
  orderDate: string;
}
```

**Publishers:** `order-service`  
**Consumers:** `fulfillment-service`, `notification-service`, `analytics-service`

---

#### OrderCancelledEvent
```typescript
interface OrderCancelledEvent extends BaseEvent {
  eventType: 'OrderCancelled';
  
  orderId: string;
  customerId: string;
  cancellationReason: string;
  cancelledBy: 'CUSTOMER' | 'SYSTEM' | 'ADMIN';
  refundAmount?: number;
}
```

**Publishers:** `order-service`  
**Consumers:** `inventory-service`, `payment-service`, `notification-service`

---

#### ShipmentShippedEvent
```typescript
interface ShipmentShippedEvent extends BaseEvent {
  eventType: 'ShipmentShipped';
  
  shipmentId: string;
  orderId: string;
  customerId: string;
  
  // Tracking info
  trackingNumber: string;
  carrier: string;
  estimatedDelivery: string;
  
  // For notifications
  customerEmail: string;
}
```

**Publishers:** `fulfillment-service`  
**Consumers:** `notification-service`, `analytics-service`

---

### Customer Domain Events

#### CustomerCreatedEvent
```typescript
interface CustomerCreatedEvent extends BaseEvent {
  eventType: 'CustomerCreated';
  
  customerId: string;
  email: string;
  registrationSource: 'WEB' | 'MOBILE' | 'IMPORT';
  initialTier: string;
}
```

**Publishers:** `customer-service`  
**Consumers:** `notification-service`, `analytics-service`

---

#### TierChangedEvent
```typescript
interface TierChangedEvent extends BaseEvent {
  eventType: 'TierChanged';
  
  customerId: string;
  previousTier: string;
  newTier: string;
  changeReason: 'UPGRADE' | 'DOWNGRADE' | 'MANUAL';
  effectiveDate: string;
}
```

**Publishers:** `tier-service`  
**Consumers:** `notification-service`, `analytics-service`

---

### Transaction Domain Events

#### HighValueTransactionAlertEvent
```typescript
interface HighValueTransactionAlertEvent extends BaseEvent {
  eventType: 'HighValueTransactionAlert';
  
  // Alert info
  alertId: string;
  transactionId: string;
  
  // Transaction details
  amount: number;
  currency: string;
  riskLevel: 'LOW' | 'MEDIUM' | 'HIGH' | 'CRITICAL';
  
  // Customer context
  customerId: string;
  customerEmail?: string;
  
  // For case creation
  suggestedAction: 'MONITOR' | 'REVIEW' | 'BLOCK';
}
```

**Publishers:** `transaction-monitoring-service`  
**Consumers:** `notification-service`, `case-management-service`, `analytics-service`

---

#### PaymentCompletedEvent
```typescript
interface PaymentCompletedEvent extends BaseEvent {
  eventType: 'PaymentCompleted';
  
  paymentId: string;
  orderId: string;
  customerId: string;
  
  amount: number;
  currency: string;
  paymentMethod: 'CREDIT_CARD' | 'PAYPAL' | 'BANK_TRANSFER';
  transactionReference: string;
}
```

**Publishers:** `payment-service`  
**Consumers:** `order-service`, `notification-service`, `analytics-service`

---

### Integration Domain Events

#### NotificationSentEvent
```typescript
interface NotificationSentEvent extends BaseEvent {
  eventType: 'NotificationSent';
  
  notificationId: string;
  recipientId: string;
  channel: 'EMAIL' | 'SMS' | 'PUSH' | 'SLACK';
  templateName: string;
  deliveryStatus: 'SENT' | 'DELIVERED' | 'FAILED';
}
```

**Publishers:** `notification-service`  
**Consumers:** `analytics-service`

---

## API Contract Standards

### RESTful API Conventions

#### URL Patterns
```
GET    /api/{resource}              - List resources
POST   /api/{resource}              - Create resource
GET    /api/{resource}/{id}         - Get single resource
PUT    /api/{resource}/{id}         - Update resource
DELETE /api/{resource}/{id}         - Delete resource
POST   /api/{resource}/{id}/{action} - Perform action on resource
```

#### Response Formats

**Success Response:**
```json
{
  "data": { ... },
  "meta": {
    "requestId": "req-123",
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

**Error Response:**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [
      { "field": "email", "message": "Invalid email format" }
    ]
  },
  "meta": {
    "requestId": "req-123",
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

#### Pagination
```
GET /api/orders?page=0&size=20&sort=createdAt,desc
```

Response includes:
```json
{
  "data": [...],
  "pagination": {
    "page": 0,
    "size": 20,
    "totalElements": 150,
    "totalPages": 8
  }
}
```

---

## Consumer Responsibilities Matrix

### notification-service

| Event | Action |
|-------|--------|
| OrderCreatedEvent | Send order confirmation email |
| ShipmentShippedEvent | Send shipping notification |
| HighValueTransactionAlertEvent | Alert risk team (email + Slack) |
| TierChangedEvent | Send tier change notification |
| PaymentCompletedEvent | Send payment receipt |

### analytics-service

| Event | Metric Updated |
|-------|---------------|
| OrderCreatedEvent | orders_placed, revenue |
| ShipmentShippedEvent | shipments_sent |
| HighValueTransactionAlertEvent | alerts_generated, risk_score |
| CustomerCreatedEvent | new_customers |
| TierChangedEvent | tier_distribution |

### case-management-service

| Event | Action |
|-------|--------|
| HighValueTransactionAlertEvent | Create investigation case |
| SuspiciousActivityDetectedEvent | Create fraud review case |

---

## Real-Time Communication

### WebSocket Patterns

For real-time UI updates, use WebSocket connections:

**Connection URL:** `wss://api.example.com/ws`

**Subscription Message:**
```json
{
  "action": "subscribe",
  "channel": "alerts.high-value",
  "filters": {
    "riskLevel": ["HIGH", "CRITICAL"]
  }
}
```

**Event Message:**
```json
{
  "channel": "alerts.high-value",
  "event": "new-alert",
  "data": {
    "alertId": "alert-123",
    "transactionId": "txn-456",
    "amount": 15000,
    "riskLevel": "HIGH"
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

### Server-Sent Events (SSE)

Alternative for one-way real-time updates:

**Endpoint:** `GET /api/alerts/stream`

**Response:** Event stream
```
event: high-value-alert
data: {"alertId":"alert-123","riskLevel":"HIGH"}

event: high-value-alert
data: {"alertId":"alert-124","riskLevel":"CRITICAL"}
```

---

## Error Handling

### Retry Policy for Event Processing
- Initial retry: 1 second delay
- Max retries: 3
- Backoff multiplier: 2x
- Max delay: 30 seconds

### Dead Letter Handling
Failed messages after max retries go to dead letter queue:
- Topic pattern: `dlq.{original-topic}`
- Include original message + error details
- Alert operations team

### Circuit Breaker for API Calls
When calling other services:
- Failure threshold: 5 failures in 60 seconds
- Open state duration: 30 seconds
- Half-open: Allow 1 request to test

---

## Schema Evolution Guidelines

### Adding Fields
- New optional fields can be added without version bump
- Consumers MUST ignore unknown fields
- Document new fields in this contract

### Removing Fields
- Mark field as deprecated first
- Keep deprecated field for 2 release cycles minimum
- Then remove and bump schemaVersion

### Changing Field Types
- NEVER change type of existing field
- Add new field with new type
- Deprecate old field
- Bump schemaVersion

### Version Handling
Consumers should check schemaVersion:
```typescript
function handleEvent(event: BaseEvent) {
  if (event.schemaVersion < 2) {
    // Handle legacy schema
    return handleLegacyEvent(event);
  }
  // Handle current schema
  return handleCurrentEvent(event);
}
```

---

## Testing Integration

### Event Testing
- Use test topic prefix: `test.{topic-name}`
- Verify event schema compliance
- Test consumer idempotency

### API Contract Testing
- Use consumer-driven contract testing (Pact, Spring Cloud Contract)
- Verify request/response schemas
- Test error scenarios

### Integration Test Scenarios
1. Happy path: Event published → Consumed → Action taken
2. Retry: Consumer fails → Retries → Succeeds
3. Dead letter: Consumer fails permanently → Goes to DLQ
4. Circuit breaker: Service down → Circuit opens → Fallback

---

## Monitoring

### Event Metrics
- `events.published.count` - Events published by type
- `events.consumed.count` - Events consumed by type
- `events.processing.duration` - Processing time
- `events.failed.count` - Failed processing attempts

### API Metrics
- `api.requests.count` - Requests by endpoint
- `api.requests.duration` - Response time
- `api.errors.count` - Errors by type

### Alerts
- Event processing lag > 5 minutes
- Dead letter queue size > 100
- API error rate > 5%
- Circuit breaker open
