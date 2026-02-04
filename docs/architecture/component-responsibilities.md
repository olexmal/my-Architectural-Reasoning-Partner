# Component Responsibilities Reference

## Overview

This document defines the canonical ownership of business capabilities across components. Use this as a reference when determining which component should own changes for a given capability.

**Principle:** Each business capability should have ONE owning component responsible for its core logic. Other components may consume data via APIs or events but should not duplicate the business logic.

---

## Domain: Order Management

### order-service
**Type:** Backend Service  
**Technology:** [Your tech stack]

**Owns:**
- Order entity lifecycle (create, update, cancel)
- Order validation rules
- Order status management

**Data Models:**
- Order, OrderLine, OrderStatus

**APIs Exposed:**
```
POST   /api/orders              - Create order
GET    /api/orders/{id}         - Get order
PUT    /api/orders/{id}         - Update order
DELETE /api/orders/{id}         - Cancel order
GET    /api/orders?customerId=  - List customer orders
```

**Events Published:**
- `OrderCreatedEvent`
- `OrderUpdatedEvent`
- `OrderCancelledEvent`

**Dependencies:**
- Calls `customer-service` for customer validation
- Calls `inventory-service` for stock reservation

---

### fulfillment-service
**Type:** Backend Service

**Owns:**
- Shipment creation and tracking
- Fulfillment workflow
- Carrier integration

**Data Models:**
- Shipment, FulfillmentTask, Carrier

**APIs Exposed:**
```
POST /api/shipments              - Create shipment
GET  /api/shipments/{id}         - Get shipment
GET  /api/shipments/{id}/track   - Get tracking info
PUT  /api/shipments/{id}/status  - Update status
```

**Events Published:**
- `ShipmentCreatedEvent`
- `ShipmentShippedEvent`
- `ShipmentDeliveredEvent`

**Events Consumed:**
- `OrderCreatedEvent` → Triggers fulfillment task creation

---

### returns-service
**Type:** Backend Service

**Owns:**
- Return request lifecycle
- Return policy enforcement
- Refund triggering

**Data Models:**
- ReturnRequest, ReturnReason, ReturnStatus

**APIs Exposed:**
```
POST /api/returns                - Create return request
GET  /api/returns/{id}           - Get return
PUT  /api/returns/{id}/approve   - Approve return
PUT  /api/returns/{id}/reject    - Reject return
```

**Events Published:**
- `ReturnRequestedEvent`
- `ReturnApprovedEvent`
- `ReturnRejectedEvent`

---

## Domain: Customer & Identity

### customer-service
**Type:** Backend Service

**Owns:**
- Customer profile management
- Address management
- Contact preferences

**Data Models:**
- Customer, Address, ContactPreference

**APIs Exposed:**
```
POST /api/customers              - Create customer
GET  /api/customers/{id}         - Get customer
PUT  /api/customers/{id}         - Update customer
GET  /api/customers/{id}/addresses - Get addresses
```

**Events Published:**
- `CustomerCreatedEvent`
- `CustomerUpdatedEvent`

---

### auth-service
**Type:** Backend Service

**Owns:**
- Authentication (login, logout)
- Token management
- Password management

**Data Models:**
- User, Credential, Session

**APIs Exposed:**
```
POST /api/auth/login             - Authenticate
POST /api/auth/logout            - End session
POST /api/auth/refresh           - Refresh token
POST /api/auth/password/reset    - Password reset
```

**Events Published:**
- `UserLoggedInEvent`
- `UserLoggedOutEvent`
- `PasswordChangedEvent`

---

### tier-service
**Type:** Backend Service

**Owns:**
- Customer tier definitions (GOLD, SILVER, BRONZE)
- Tier assignment logic
- Tier benefits

**Data Models:**
- Tier, TierAssignment, TierBenefit

**APIs Exposed:**
```
GET /api/tiers                   - List all tiers
GET /api/customers/{id}/tier     - Get customer's tier
PUT /api/customers/{id}/tier     - Update tier
GET /api/tiers/{name}/benefits   - Get tier benefits
```

**Events Published:**
- `TierChangedEvent`

---

## Domain: Frontend Experience

### web-app
**Type:** Frontend Application  
**Technology:** Angular/React/Vue

**Owns:**
- Customer-facing UI
- Shopping experience
- Account management UI

**Pages/Features:**
- Home, Product catalog, Cart, Checkout
- Order history, Profile settings
- Returns initiation

**Consumes APIs From:**
- `order-service`, `customer-service`, `product-service`

**Real-time Subscriptions:**
- Order status updates
- Notification stream

---

### admin-portal
**Type:** Frontend Application  
**Technology:** Angular/React

**Owns:**
- Administrative UI
- Back-office operations
- Configuration management

**Pages/Features:**
- Order management dashboard
- Customer lookup
- Reports and analytics views

**Consumes APIs From:**
- `order-service`, `customer-service`, `reporting-service`

---

### merchant-portal
**Type:** Frontend Application  
**Technology:** [Framework]

**Owns:**
- Merchant-facing UI
- Transaction views
- Alert management

**Pages/Features:**
- Transaction dashboard
- Alert notifications
- Settlement reports

---

### risk-dashboard
**Type:** Frontend Application  
**Technology:** [Framework]

**Owns:**
- Risk team UI
- Fraud monitoring views
- Investigation tools

**Pages/Features:**
- Real-time alert feed
- Case management UI
- Risk metrics dashboard

---

## Domain: Integration & Events

### notification-service
**Type:** Backend Service

**Owns:**
- Notification delivery (email, SMS, push)
- Template management
- Delivery tracking

**Data Models:**
- NotificationTemplate, NotificationHistory

**APIs Exposed:**
```
POST /api/notifications/send     - Send notification
GET  /api/notifications/history  - Get history
```

**Events Consumed:**
- Listens to various domain events for notification triggers

**Integrations:**
- Email provider (SendGrid, SES)
- SMS provider (Twilio)
- Push notification service

---

### event-bus
**Type:** Infrastructure Component

**Owns:**
- Event routing
- Topic management
- Message persistence

**Technologies:**
- Kafka, RabbitMQ, AWS EventBridge, etc.

**Topics:**
```
orders.*              - Order domain events
customers.*           - Customer domain events
transactions.*        - Transaction domain events
notifications.*       - Notification events
```

---

### case-management-service
**Type:** Backend Service

**Owns:**
- Investigation case lifecycle
- Case assignment
- Case workflow

**Data Models:**
- Case, CaseNote, CaseStatus

**APIs Exposed:**
```
POST /api/cases                  - Create case
GET  /api/cases/{id}             - Get case
PUT  /api/cases/{id}/assign      - Assign case
PUT  /api/cases/{id}/resolve     - Resolve case
```

**Events Published:**
- `CaseCreatedEvent`
- `CaseResolvedEvent`

---

## Domain: Transaction & Financial

### payment-service
**Type:** Backend Service

**Owns:**
- Payment processing
- Payment method management
- Transaction records

**Data Models:**
- Payment, PaymentMethod, Transaction

**APIs Exposed:**
```
POST /api/payments               - Process payment
GET  /api/payments/{id}          - Get payment
POST /api/payments/{id}/refund   - Refund payment
```

**Events Published:**
- `PaymentCompletedEvent`
- `PaymentFailedEvent`
- `RefundCompletedEvent`

---

### transaction-monitoring-service
**Type:** Backend Service

**Owns:**
- Transaction monitoring rules
- High-value detection
- Alert generation

**Data Models:**
- TransactionAlert, MonitoringRule

**APIs Exposed:**
```
GET  /api/alerts                 - List alerts
GET  /api/alerts/high-value      - High-value alerts
POST /api/alerts/{id}/acknowledge - Acknowledge alert
```

**Events Published:**
- `HighValueTransactionAlertEvent`
- `SuspiciousActivityDetectedEvent`

---

## Domain: Analytics & Reporting

### analytics-service
**Type:** Backend Service

**Owns:**
- Event tracking
- Metric aggregation
- Success metrics

**Data Models:**
- Metric, MetricSnapshot, CustomerMetrics

**APIs Exposed:**
```
POST /api/analytics/track        - Track event
GET  /api/analytics/metrics      - Get metrics
GET  /api/analytics/dashboard    - Dashboard data
```

**Events Consumed:**
- Listens to business events for metric updates

---

### reporting-service
**Type:** Backend Service

**Owns:**
- Report generation
- Scheduled reports
- Data export

**Data Models:**
- Report, ReportTemplate, Schedule

**APIs Exposed:**
```
POST /api/reports/generate       - Generate report
GET  /api/reports/{id}           - Get report
GET  /api/reports/{id}/download  - Download report
```

---

## Shared Libraries

### shared-models
**Type:** Library (npm package / Maven artifact)

**Contains:**
- Common DTOs and interfaces
- Event schemas
- Shared constants

**Used By:**
- All backend services
- Frontend applications

**Examples:**
```typescript
// @company/shared-models
export interface OrderDTO { ... }
export interface CustomerDTO { ... }
export interface HighValueTransactionAlertEvent { ... }
```

### common-utils
**Type:** Library

**Contains:**
- Date/time utilities
- Validation helpers
- Error handling utilities

---

## Quick Lookup Table

| Capability | Owning Component | Type |
|------------|-----------------|------|
| Order lifecycle | order-service | Backend |
| Shipment tracking | fulfillment-service | Backend |
| Return processing | returns-service | Backend |
| Customer profile | customer-service | Backend |
| Authentication | auth-service | Backend |
| Customer tier | tier-service | Backend |
| Customer UI | web-app | Frontend |
| Admin UI | admin-portal | Frontend |
| Notifications | notification-service | Backend |
| Payment processing | payment-service | Backend |
| Transaction alerts | transaction-monitoring-service | Backend |
| Analytics | analytics-service | Backend |
| Reports | reporting-service | Backend |

---

## Updating This Document

When adding new components:
1. Identify the responsible domain
2. Document owned capabilities
3. Define APIs/interfaces exposed
4. List events published/consumed
5. Document dependencies
6. Update the quick lookup table
