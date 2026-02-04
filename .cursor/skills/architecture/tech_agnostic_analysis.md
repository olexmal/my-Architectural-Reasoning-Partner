name: tech-agnostic-analysis-engine
description: Primary reasoning engine that analyzes specifications against business domains without technology assumptions.
version: 2.0.0

---

# TECHNOLOGY-AGNOSTIC ANALYSIS ENGINE

## Overview

This skill analyzes business specifications and maps them to architectural components regardless of underlying technology. It focuses on **what components do** rather than **how they're built**.

---

## ANALYSIS WORKFLOW

### Phase 1: Parse & Tag Specification

**Goal:** Extract business entities, actions, and qualifiers from natural language.

**Input Example:**
```
"When a premium customer submits a support ticket, show their priority status 
on the agent dashboard and notify the assigned team lead."
```

**Tagging Process:**

```yaml
Business Entities:
  - premium customer (qualified customer type)
  - support ticket (business object)
  - agent dashboard (UI surface)
  - team lead (user role)
  - priority status (derived attribute)

Business Actions:
  - submits (user action, creates entity)
  - show (display, UI requirement)
  - notify (communication, integration)

Business Qualifiers:
  - premium (tier/status filter)
  - assigned (relationship qualifier)
  - priority (importance indicator)

Relationships:
  - customer → submits → ticket
  - ticket → displayed on → dashboard
  - team lead → receives → notification
```

**Output:** Structured tag map for domain analysis.

---

### Phase 2: Domain Mapping

**Goal:** Map tagged elements to responsible business domains.

**Mapping Logic:**

```javascript
// Conceptual mapping (not actual code)
const businessToDomainMap = {
  // Entities
  'customer': ['Customer & Identity'],
  'premium tier': ['Customer & Identity'],
  'support ticket': ['Customer & Identity', 'Order Management'], // Ambiguous
  'team lead': ['Customer & Identity'],
  
  // UI Elements
  'dashboard': ['Frontend Experience'],
  'portal': ['Frontend Experience'],
  'display': ['Frontend Experience'],
  
  // Actions
  'notify': ['Integration & Event'],
  'send email': ['Integration & Event'],
  'show status': ['Frontend Experience'],
  'submit form': ['Frontend Experience', 'Backend Processing'],
  
  // Qualifiers
  'track': ['Analytics & Reporting'],
  'metrics': ['Analytics & Reporting'],
};
```

**Ambiguity Flags:**
- `support ticket` could be in Customer domain or its own Support domain
- `dashboard` could involve Analytics for data aggregation

---

### Phase 3: Generate Impact Matrix

**Goal:** Create structured view of impacted domains with confidence.

**Impact Matrix Template:**

| Domain | Impact Type | Confidence | Reasoning |
|--------|-------------|------------|-----------|
| **Customer & Identity** | Core Change | HIGH | Owns customer data, premium status, agent/team relationships |
| **Frontend Experience** | UI Change | HIGH | "show on dashboard" requires UI component changes |
| **Integration & Event** | Side Effect | HIGH | "notify" action requires messaging |
| **Analytics & Reporting** | Possible | MEDIUM | Dashboards often involve analytics backends |

**Impact Types:**
- **Core Change:** Direct modification to domain's owned entities/logic
- **UI Change:** Frontend presentation or interaction changes
- **API Change:** New or modified service interfaces
- **Side Effect:** Reaction triggered by another domain's event
- **Dependency:** Needs to read/call another domain

---

### Phase 4: Propose Component-Level Hypothesis

**Goal:** Narrow from domains to specific components.

**Hypothesis Structure:**

```markdown
## ARCHITECTURAL HYPOTHESIS

### 1. Customer & Identity Domain
**Likely Components:** `customer-service`, `support-service`, `user-management`

**Probable Changes:**
- `support-service`:
  - Enhance ticket creation to include premium status check
  - Add team lead assignment logic
  - Publish `PremiumTicketCreatedEvent`
  
- `customer-service`:
  - Expose API for premium status lookup (if not exists)
  - No data model changes expected

**Questions to Confirm:**
1. Which component currently manages support tickets?
2. Where is "premium customer" status stored?
3. How is team lead assignment determined?

---

### 2. Frontend Experience Domain
**Likely Components:** `agent-portal`, `admin-dashboard`, or unified `web-app`

**Probable Changes:**
- Add "customer priority" visual indicator to ticket display
- Create/modify dashboard view for team leads
- Add real-time notification display

**Questions to Confirm:**
1. Is there a dedicated agent-facing frontend application?
2. What UI framework is in use (Angular, React, Vue)?
3. Should this be a new component or enhancement to existing?

---

### 3. Integration & Event Domain
**Likely Components:** `notification-service`, `event-bus`, `message-router`

**Probable Changes:**
- Create/extend event: `PremiumTicketCreatedEvent`
- Add notification routing to team leads
- Configure notification channels (email, in-app, Slack)

**Questions to Confirm:**
1. Is there an existing event for ticket creation?
2. What notification channels are available?
3. Is there a notification preferences system?
```

---

### Phase 5: Interactive Refinement Prompts

**Goal:** Generate questions to resolve ambiguities.

**Question Categories:**

#### A. Component Ownership Questions
```
"I need to confirm component ownership:
1. Which service currently owns the support ticket lifecycle?
2. Does customer premium status live in `customer-service` or a separate `tier-service`?"
```

#### B. Technology Context Questions
```
"To understand the implementation context:
1. What frontend framework is the agent dashboard built with?
2. Is there a shared model library used across services?"
```

#### C. Integration Pattern Questions
```
"For the notification flow:
1. Do services communicate via events (async) or direct API calls (sync)?
2. Is there an existing notification service I should integrate with?"
```

#### D. Existing Component Questions
```
"Before proposing new components:
1. Is there an existing `TicketCreatedEvent` I should extend?
2. Does the dashboard already have a notification area?"
```

---

## COMMAND: /analyze-architecture

**Usage:**
```
/analyze-architecture [specification text]
```

**Example:**
```
/analyze-architecture
When a high-value transaction is detected, display a warning flag in the 
merchant portal, notify the risk analysis team, and create an investigation case.
```

**Output Format:**

```markdown
# ARCHITECTURAL ANALYSIS

## Specification Summary
[Restated spec with business entities highlighted]

## Business Domain Impact
[Phase 3 impact matrix]

## Component-Level Hypothesis
[Phase 4 detailed hypothesis per domain]

## Clarifying Questions
[Phase 5 prioritized questions]

## Suggested Discovery Commands
- `/discover-component transaction detection`
- `/discover-component merchant portal`
- `/discover-component risk team notification`

## Next Steps
1. Answer clarifying questions
2. Run suggested discoveries
3. Refine hypothesis
4. Generate final HLD
```

---

## WORKED EXAMPLE

### Input Specification
```
"When a high-value transaction is detected, display a warning flag in the 
merchant portal, notify the risk analysis team via their dashboard, and 
create a case in the investigation system. Also, ensure the customer's 
recent transactions are reviewed."
```

### Phase 1: Tags
```yaml
Business Entities:
  - high-value transaction (qualified event)
  - merchant portal (UI application)
  - warning flag (UI element)
  - risk analysis team (user group)
  - risk dashboard (UI surface)
  - investigation case (business object)
  - customer transactions (data for review)

Business Actions:
  - detected (system trigger)
  - display (UI presentation)
  - notify (communication)
  - create case (entity creation)
  - review (business process)

Business Qualifiers:
  - high-value (threshold filter)
  - recent (time qualifier)
```

### Phase 2: Domain Mapping
```
high-value transaction → Transaction/Financial
merchant portal, risk dashboard → Frontend Experience
warning flag, display → Frontend Experience
notify → Integration & Event
investigation case → Integration & Event (or dedicated domain)
customer transactions → Transaction/Financial, Customer
review → Analytics or Manual Process
```

### Phase 3: Impact Matrix

| Domain | Impact Type | Confidence | Reasoning |
|--------|-------------|------------|-----------|
| Transaction & Financial | Core Change | HIGH | Owns transaction detection |
| Frontend Experience | UI Change | HIGH | Two UIs: merchant portal + risk dashboard |
| Integration & Event | Core Change | HIGH | Notification + case creation |
| Customer & Identity | Dependency | MEDIUM | Customer transaction history lookup |
| Analytics | Possible | LOW | "Review" might involve analytics |

### Phase 4: Component Hypothesis

**Transaction Domain:**
- `transaction-monitoring-service`: Detect high-value, publish alert event
- `transaction-service`: Provide transaction history API

**Frontend Domain:**
- `merchant-portal` (unknown tech): Add warning flag component
- `risk-dashboard` (unknown tech): Add alert feed widget

**Integration Domain:**
- `case-management-service`: Create investigation case
- `notification-service`: Route alerts to risk team

**Customer Domain:**
- `customer-service`: Provide customer context for transactions

### Phase 5: Questions
1. Which service handles transaction monitoring/detection?
2. Are merchant-portal and risk-dashboard separate applications?
3. Is there an existing case management system?
4. What notification channels does risk team use?
5. Should transaction review be automatic or manual?

---

## ERROR HANDLING

### Unrecognized Business Terms
```
"I couldn't map '[term]' to a known domain. 
Could you clarify what '[term]' refers to in your system?"
```

### Ambiguous Component Ownership
```
"'[entity]' could belong to multiple domains:
- Option A: [Domain 1] - because [reason]
- Option B: [Domain 2] - because [reason]
Which is correct in your architecture?"
```

### Missing Technology Context
```
"To provide specific implementation guidance, I need to know:
- What frontend framework is [app-name] built with?
- What backend technology does [service-name] use?"
```

---

## BEST PRACTICES

1. **Stay Technology-Agnostic** until discovery confirms tech stack
2. **Ask About Existing Components** before proposing new ones
3. **Consider All UI Surfaces** mentioned in the spec
4. **Always Include Integration** if notifications/events are mentioned
5. **Flag Ambiguities Early** rather than making assumptions
6. **Group Changes by Domain** for clear ownership
