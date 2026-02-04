name: agnostic-hld-refinement
description: Interactive skill to refine architectural hypotheses into concrete HLD documents for any technology stack.
version: 2.0.0

---

# TECHNOLOGY-AGNOSTIC HLD REFINEMENT

## Overview

This skill transforms initial architectural hypotheses into finalized High-Level Design documents through collaborative dialogue. It works regardless of whether your system uses Java, Angular, Node.js, Python, or any other technology.

---

## REFINEMENT DIALOGUE PATTERNS

### Pattern A: Clarifying Component Boundaries

**When to Use:** Multiple components could own a business capability.

**Dialogue Template:**

```markdown
AI: "You mentioned '[business entity]'. I found references in multiple places:

     1. `[component-1]` ([type], [what it does])
     2. `[component-2]` ([type], [what it does])
     3. `[component-3]` ([type], [what it does])
     
     Which component should own the [creation/modification/deletion] logic?"
     
User: "[Component-1] handles the business logic, [component-2] defines 
       the shared model, [component-3] displays it."
     
AI: "Updated assignments:
     - `[component-1]` → Core Change (owns business logic)
     - `[component-2]` → Model Change (shared definitions)
     - `[component-3]` → UI Change (consumption only)
     
     [Proceeds with refined analysis]"
```

**Real Example:**

```markdown
AI: "You mentioned 'support ticket'. I found references in:

     1. `support-service` (backend, handles ticket lifecycle, has database)
     2. `shared-models` (library, defines TicketDTO interface)
     3. `agent-portal` (Angular frontend, displays tickets in dashboard)
     
     Which component should own the ticket creation logic?"
     
User: "support-service handles creation and business rules. shared-models 
       defines the contract. agent-portal just displays."
     
AI: "Updated:
     - `support-service` → Core Change (creation logic, event publishing)
     - `shared-models` → Model Change (extend TicketDTO if needed)
     - `agent-portal` → UI Change (display updates only)
     
     This means premium status check goes in support-service, not frontend."
```

---

### Pattern B: Identifying Integration Patterns

**When to Use:** Components need to communicate across boundaries.

**Dialogue Template:**

```markdown
AI: "For '[action]', components need to communicate.

     Options:
     1. **Synchronous API call:** [caller] directly calls [callee] API
     2. **Asynchronous event:** [publisher] emits event, [consumer] reacts
     3. **Shared database:** Both access same data store (usually not recommended)
     
     What integration pattern fits your architecture?"
     
User: "[Pattern selection and context]"
     
AI: "Updated integration design:
     [Publisher] → [Event/API] → [Consumer]
     
     Implementation notes: [specific guidance]"
```

**Real Example:**

```markdown
AI: "For 'notify team lead when premium ticket created', options are:

     1. **Sync:** support-service calls notification-service API directly
     2. **Async:** support-service publishes PremiumTicketCreatedEvent, 
        notification-service consumes it
     3. **Hybrid:** support-service publishes event, notification-service 
        calls customer-service to enrich data
     
     What pattern does your system use?"
     
User: "We use async events via Kafka. notification-service listens to 
       events and has its own templates for different channels."
     
AI: "Updated design:
     support-service → PremiumTicketCreatedEvent → Kafka → notification-service
     
     Event payload should include:
     - ticketId, customerId, isPremium
     - teamLeadId (for routing)
     - Enough context for notification template
     
     notification-service handles channel selection (email, Slack, etc.)"
```

---

### Pattern C: Resolving Frontend Ambiguity

**When to Use:** UI requirements span multiple applications or unclear technology.

**Dialogue Template:**

```markdown
AI: "The spec mentions '[UI element]' on '[surface]'. I need to understand:

     1. Is this a single application or multiple?
     2. What frontend technology/framework is used?
     3. Should this be a new component or modification to existing?"
     
User: "[Details about frontend architecture]"
     
AI: "Updated frontend design:
     - Application: [app-name] ([framework])
     - Component: [new/modified] [component-name]
     - Integration: [how it gets data]"
```

**Real Example:**

```markdown
AI: "The spec mentions displaying alerts in both 'merchant portal' and 
     'risk team dashboard'. Questions:

     1. Are these separate applications or modules in one app?
     2. What framework(s) are they built with?
     3. How do they receive real-time updates?"
     
User: "Separate apps. Merchant portal is Angular, risk dashboard is React. 
       Both connect to our WebSocket service for real-time updates."
     
AI: "Updated frontend design:

     merchant-portal (Angular):
     - New: HighValueAlertFlagComponent
     - Modified: TransactionListComponent (integrates flag)
     - Service: AlertWebSocketService (connects to WS)
     
     risk-team-dashboard (React):
     - New: AlertFeedWidget component
     - New: useAlertStream custom hook
     - Modified: DashboardLayout (adds widget)
     
     Both consume same WebSocket endpoint, different UI implementations."
```

---

### Pattern D: Confirming Data Model Changes

**When to Use:** Changes affect data models across components.

**Dialogue Template:**

```markdown
AI: "The proposed changes require data model updates:

     **[component-1]:** [changes to model]
     **[component-2]:** [changes to shared types]
     **Database:** [schema changes if any]
     
     Questions:
     1. Is there a shared model library these should use?
     2. Are there API contracts that need updating?
     3. Any backward compatibility concerns?"
```

**Real Example:**

```markdown
AI: "To support premium ticket flagging, data model changes:

     **support-service:**
     - Add `isPremiumCustomer: boolean` to SupportTicket entity
     - Add `assignedTeamLeadId: string` to SupportTicket
     
     **shared-models:**
     - Update SupportTicketDTO interface with new fields
     
     **Database (support-service):**
     - Migration to add columns to support_tickets table
     
     Questions:
     1. Is shared-models used by all consumers of ticket data?
     2. Should old API consumers receive null for new fields (backward compat)?"
     
User: "Yes, shared-models is used everywhere. Old consumers should get 
       null for new fields - we handle that gracefully."
     
AI: "Updated:
     - shared-models gets new optional fields (isPremiumCustomer?, teamLeadId?)
     - API maintains backward compatibility
     - Migration is additive only (no breaking changes)
     - Frontend apps update when ready"
```

---

### Pattern E: Event Schema Design

**When to Use:** New events need to be designed.

**Dialogue Template:**

```markdown
AI: "Proposed event for [trigger]:

     **[EventName]**
     ```
     {
       [field]: [type]  // [purpose]
       ...
     }
     ```
     
     Questions:
     1. Are there existing event patterns I should follow?
     2. What consumers need what data?
     3. Should any data be looked up vs. included?"
```

---

## COMMAND: /refine-hld

**Usage:**
```
/refine-hld
```

Initiates interactive refinement based on current hypothesis.

**Flow:**
1. Present current hypothesis summary
2. Identify unresolved ambiguities
3. Ask clarifying questions (one topic at a time)
4. Update hypothesis based on answers
5. Repeat until resolved
6. Offer to generate final HLD

---

## COMMAND: /generate-final-hld

**Usage:**
```
/generate-final-hld [feature-name]
```

Generates final HLD document after refinement is complete.

---

## FINAL HLD TEMPLATE (Technology-Agnostic)

```markdown
# HIGH-LEVEL DESIGN: [Feature Name]

## 1. Overview
**Specification:** [Original spec text]
**Date:** [Generated date]
**Status:** Ready for Implementation

## 2. Architecture Summary

### Business Domains Impacted
| Domain | Components | Impact Level |
|--------|------------|--------------|
| [Domain] | [component-1, component-2] | [Core/Supporting] |

### Communication Flow
```
[component-1] → [mechanism] → [component-2]
[component-2] → [mechanism] → [component-3]
```

## 3. Component Changes

### 3.1 [component-name] ([Domain])

**Type:** [Backend Service / Frontend App / Shared Library / Integration]
**Technology:** [Java/Node/Angular/React/etc. - if known]

**Role in Feature:** [Brief description]

#### Data Model Changes
- [ ] Add `[field]: [type]` to `[Model]`
- [ ] Create new `[Model]` with fields: [list]

#### API Changes
- [ ] New endpoint: `[METHOD] /path` - [description]
- [ ] Modified endpoint: `[METHOD] /path` - [what changed]

#### Business Logic Changes
- [ ] [Description of logic change]
- [ ] [Another logic change]

#### Integration Changes
- [ ] Publishes: `[EventName]` when [condition]
- [ ] Consumes: `[EventName]` from [source]
- [ ] Calls: `[API]` on [other-component]

#### UI Changes (if frontend)
- [ ] New component: `[ComponentName]` - [purpose]
- [ ] Modified component: `[ComponentName]` - [what changed]

### 3.2 [next-component]
[Repeat structure...]

## 4. Event/Message Schemas

### [EventName]
```
{
  id: string           // Unique event ID
  timestamp: datetime  // When event occurred
  [field]: [type]      // [Purpose]
  ...
}
```
**Publishers:** [component-list]
**Consumers:** [component-list]

## 5. Integration Architecture

### Synchronous Dependencies
| Caller | Callee | Interface | Purpose |
|--------|--------|-----------|---------|
| [comp] | [comp] | [API/method] | [why] |

### Asynchronous Communication
| Publisher | Event/Message | Consumers |
|-----------|---------------|-----------|
| [comp] | [Event] | [comp-list] |

## 6. Data Migrations

| Component | Migration | Description |
|-----------|-----------|-------------|
| [comp] | [name/version] | [what it does] |

## 7. Implementation Notes

### Technology Considerations
- [Notes about specific tech stack implications]

### Parallel Development Opportunities
- [What can be built in parallel]
- [Dependencies that block progress]

### Backward Compatibility
- [Notes on compatibility with existing consumers]

## 8. Testing Strategy

### Unit Tests
- [ ] [component]: Test [functionality]

### Integration Tests
- [ ] [component] ↔ [component]: Test [interaction]

### End-to-End Tests
- [ ] [Scenario description]

## 9. Open Questions
[Any remaining items to resolve during implementation]
```

---

## REFINEMENT SESSION EXAMPLE

```markdown
AI: "Starting HLD refinement for 'High-Value Transaction Alert System'.
     
     Current hypothesis has 3 unresolved items:
     
     **1. Component Ownership** (HIGH priority)
     Which component handles transaction monitoring?
     a) `payment-service` (processes payments)
     b) `transaction-monitoring-service` (dedicated monitoring)
     c) `fraud-detection-service` (if fraud-focused)
     d) Other"

User: "We have a dedicated `transaction-monitoring-service` that watches 
       all transactions and applies rules."

AI: "Confirmed: transaction-monitoring-service owns detection.
     
     **2. Frontend Architecture** (HIGH priority)
     The spec mentions 'merchant portal' and 'risk dashboard'. Are these:
     a) Same application, different pages
     b) Separate applications
     c) Modules in a monorepo"

User: "Separate apps. Merchant portal is Angular for external merchants. 
       Risk dashboard is React for internal risk team."

AI: "Confirmed: Two separate frontend apps with different tech stacks.
     
     **3. Notification Method** (MEDIUM priority)
     How should risk team be notified?
     a) Email only
     b) In-app notification on their dashboard
     c) Slack/Teams integration
     d) Multiple channels"

User: "Both in-app on dashboard and Slack. Email is optional."

AI: "All questions resolved. Summary:
     
     - Detection: transaction-monitoring-service
     - Merchant UI: merchant-portal (Angular)
     - Risk UI: risk-team-dashboard (React)  
     - Notifications: In-app + Slack (email optional)
     
     Run `/generate-final-hld High-Value Transaction Alerts` to create document."
```

---

## BEST PRACTICES

1. **One Question at a Time:** Don't overwhelm with multiple questions
2. **Provide Options:** When possible, offer choices rather than open questions
3. **Confirm Understanding:** Restate answers to ensure alignment
4. **Stay Technology-Flexible:** Don't assume tech until confirmed
5. **Track Progress:** Show what's resolved vs. remaining
6. **Be Specific:** Use actual component names discovered, not generic terms
7. **Consider All Consumers:** Think about all components affected by changes
