# Technology-Agnostic Architectural Reasoning Partner for Cursor

> Transform vague human specifications into targeted technical intents for complex multi-component systems—regardless of technology stack.

## What is This?

The **Architectural Reasoning Partner** is a set of Cursor rules and skills that help developers navigate and make changes in large, complex systems. It works with **any technology stack**—Java, Angular, Node.js, Python, React, microservices, monoliths, or hybrid architectures.

### The Problem It Solves

In complex systems, developers face a recurring challenge:

```
Business Analyst says: "When a high-value transaction is detected, show a warning 
                       in the merchant portal and notify the risk team."

Developer thinks:      "Which of our 50+ components handles transaction detection?
                        Is the merchant portal Angular or React?
                        How does the risk team get notified?
                        What events already exist?"
```

This partner **reasons about business capabilities** rather than specific technologies, making it:

- **Technology-Agnostic:** Works with Java, Angular, Node.js, Python, React, or any stack
- **Business-Focused:** Reasons about what components DO, not how they're built
- **Scalable:** Handles 10 or 100 components with the same framework
- **Adaptable:** New technologies can be added without framework changes

---

## Quick Start

### 1. Analyze a Specification

Use the `/analyze-architecture` command with any business requirement:

```
/analyze-architecture
When a high-value transaction is detected, display a warning in the merchant 
portal, notify the risk team via their dashboard, and create an investigation case.
```

The partner will:
1. Parse and tag business entities (transaction, merchant portal, risk team, case)
2. Map to responsible business domains (Transaction, Frontend, Integration)
3. Generate an impact matrix with confidence levels
4. Produce component-level hypotheses
5. Ask clarifying questions

### 2. Discover Existing Components

When you need to verify assumptions about the codebase:

```
/discover-component transaction detection high value
/discover-component merchant portal dashboard
/discover-component risk team notification
```

The partner searches across all technologies to find relevant components.

### 3. Refine the Design

Interactive refinement through dialogue:

```
/refine-hld
```

The partner asks targeted questions:
- "Is the merchant portal Angular or React?"
- "Should notification use sync API calls or async events?"

### 4. Generate Final HLD

Once refined:

```
/generate-final-hld High-Value Transaction Alert System
```

Produces a complete High-Level Design document with changes per component, regardless of technology.

---

## How It Works

### Business Domain Framework

At the core is a **domain-based reasoning framework** that focuses on business capabilities:

```
┌─────────────────────────────────────────────────────────────────────────┐
│               THINK IN BUSINESS CAPABILITIES, NOT TECHNOLOGIES          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  [Order Management]       [Customer & Identity]     [Transaction]       │
│  ├── order-service        ├── customer-service      ├── payment-service │
│  ├── fulfillment-service  ├── auth-service          ├── monitoring-svc  │
│  └── returns-service      └── tier-service          └── refund-service  │
│                                                                         │
│  [Frontend Experience]    [Integration & Events]    [Analytics]         │
│  ├── web-app (Angular)    ├── notification-service  ├── analytics-svc   │
│  ├── admin-portal (React) ├── event-bus             └── reporting-svc   │
│  └── mobile-app           └── case-management                           │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Reasoning Rules

The framework applies these rules to determine which components to modify:

| Rule | Question | Example |
|------|----------|---------|
| **Primary Data Owner** | Which domain OWNS the entity? | "Create order" → `order-service` owns orders |
| **User Interface** | Does the user see/interact with this? | "Display warning" → frontend app involved |
| **Side Effect** | Is there a reaction/notification? | "Notify risk team" → `notification-service` |
| **Lookup/Dependency** | Do you need data from elsewhere? | "Check customer tier" → calls `tier-service` |

### Confidence Scoring

Every hypothesis includes confidence levels:

| Level | Meaning | Action |
|-------|---------|--------|
| **HIGH** | Direct ownership + clear action | Proceed with hypothesis |
| **MEDIUM** | Cross-domain or ambiguous | Ask question or discover |
| **LOW** | Speculative involvement | Must clarify before proceeding |

---

## Project Structure

```
your-project/
├── .cursor/
│   ├── rules/
│   │   ├── 00_arch_responsibility_framework.mdc   # Domain ownership rules
│   │   ├── 10_tech_agnostic_discovery.mdc         # Discovery commands
│   │   └── components/                            # Registered component rules
│   │       ├── _registry.mdc                      # Component registry
│   │       ├── _guardians.mdc                     # Guardian registry (who owns what)
│   │       ├── _template.mdc                      # Template for new components
│   │       └── {component-name}.mdc               # Individual component rules
│   └── skills/
│       ├── architecture/
│       │   ├── tech_agnostic_analysis.md          # Spec analysis workflow
│       │   └── agnostic_hld_refinement.md         # Interactive refinement
│       ├── frontend/
│       │   ├── angular_patterns.md                # Angular implementation
│       │   └── react_patterns.md                  # React implementation
│       ├── backend/
│       │   └── service_patterns.md                # Backend patterns (multi-lang)
│       └── component-management/
│           ├── generate_component_descriptor.md   # PORTABLE: Copy to any project
│           └── add_component.md                   # Add components to this project
├── docs/
│   └── architecture/
│       ├── component-responsibilities.md          # Component ownership
│       └── integration-patterns.md                # Event & API contracts
├── README.md                                      # This file
└── start.md                                       # Original design document
```

---

## Complete Workflow Example

### Scenario

**Business Specification:**
> "When a high-value transaction is detected, display a warning flag in the merchant portal, notify the risk analysis team via their dashboard, and create a case in the investigation system."

### Step 1: Initial Analysis

```
/analyze-architecture
When a high-value transaction is detected, display a warning flag in the 
merchant portal, notify the risk team via their dashboard, and create a 
case in the investigation system.
```

**Output:**

```markdown
## ARCHITECTURAL ANALYSIS

### Impact Matrix
| Domain | Components | Impact | Confidence |
|--------|------------|--------|------------|
| Transaction & Financial | transaction-monitoring-service | Core Change | HIGH |
| Frontend Experience | merchant-portal, risk-dashboard | UI Change | HIGH |
| Integration & Event | notification-service, case-management | Side Effect | HIGH |
| Customer & Identity | customer-service | Dependency | MEDIUM |

### Questions
1. Which component currently handles transaction monitoring?
2. Are merchant-portal and risk-dashboard separate applications?
3. What technology stack are the frontends built with?
4. Is there an existing case management system?

### Suggested Discovery
- `/discover-component transaction monitoring high value`
- `/discover-component merchant portal dashboard`
```

### Step 2: Discovery

```
/discover-component transaction monitoring high value
```

**Output:**

```markdown
## DISCOVERY: transaction monitoring high value

### Found Components
Backend Services:
├── transaction-monitoring-service/ (Java/Spring)
│   └── src/.../HighValueDetector.java
│   └── src/.../AlertController.java
│       API: GET /api/alerts/high-value

Frontend Applications:
├── merchant-portal/ (Angular)
│   └── src/app/transactions/
├── risk-team-dashboard/ (React)
│   └── src/pages/AlertFeed/

### Technology Notes
- Backend: Java/Spring Boot
- Merchant Portal: Angular 16
- Risk Dashboard: React 18
```

### Step 3: Refinement

```
/refine-hld
```

**Dialogue:**

```markdown
AI: "The spec mentions two frontends. I found:
     - merchant-portal (Angular)
     - risk-team-dashboard (React)
     
     Both need UI changes. Should they receive real-time updates?"
     
User: "Yes, both connect to our WebSocket service for real-time alerts."

AI: "For creating investigation cases, should this be:
     a) Automatic (monitoring service creates case directly)
     b) Manual (risk team clicks button in dashboard)
     c) Hybrid (auto-create for critical, manual for others)"
     
User: "Option (a) - automatic for all high-value alerts."

AI: "All questions resolved. Ready to generate HLD."
```

### Step 4: Final HLD

```
/generate-final-hld High-Value Transaction Alert System
```

**Output:**

```markdown
# HLD: High-Value Transaction Alert System

## 1. transaction-monitoring-service (Java/Spring)
**Changes:**
- Enhance HighValueDetector to publish HighValueTransactionAlertEvent
- Call case-management-service API to create case
- Enrich alert with customer context

**Integration:**
- Publishes: HighValueTransactionAlertEvent to Kafka
- Calls: customer-service API for customer data
- Calls: case-management-service API to create case

## 2. merchant-portal (Angular)
**Changes:**
- New: HighValueAlertFlagComponent
- Modified: TransactionListComponent (integrates flag)
- Service: AlertWebSocketService (real-time updates)

**Implementation Notes:**
- Use existing design system for flag styling
- Subscribe to WebSocket channel 'alerts.high-value'

## 3. risk-team-dashboard (React)
**Changes:**
- New: AlertFeedWidget component
- New: useAlertStream custom hook
- Modified: DashboardLayout (adds widget)

**Implementation Notes:**
- Use existing React Query setup for data fetching
- Real-time updates via same WebSocket service

## 4. notification-service
**Changes:**
- New consumer for HighValueTransactionAlertEvent
- Send Slack notification to #risk-alerts channel
- Send email to risk team distribution list

## 5. case-management-service
**Changes:**
- New API: POST /api/cases/from-alert
- Auto-assign based on alert severity
- Link case to original transaction

## Communication Flow
transaction-monitoring-service → Kafka → [frontends, notification-service]
transaction-monitoring-service → API → case-management-service
```

---

## Technology-Specific Implementation

After generating the HLD, use technology-specific skills for implementation details:

### For Angular Components
The `angular_patterns.md` skill provides:
- Smart vs. presentational component patterns
- Service patterns for API integration
- WebSocket subscription with RxJS
- NgRx patterns (if applicable)

### For React Components
The `react_patterns.md` skill provides:
- Container vs. presentational patterns
- Custom hooks for data fetching
- Context + Reducer state management
- React Query patterns

### For Backend Services
The `service_patterns.md` skill provides:
- Service layer patterns (Java, Node.js, Python examples)
- Event publishing patterns
- Cross-service communication
- Database migration patterns

---

## Commands Reference

| Command | Purpose | Example |
|---------|---------|---------|
| `/analyze-architecture [spec]` | Analyze business specification | `/analyze-architecture When order ships...` |
| `/discover-component [context]` | Search codebase for components | `/discover-component payment processing` |
| `/refine-hld` | Start interactive refinement | `/refine-hld` |
| `/generate-final-hld [name]` | Generate final HLD document | `/generate-final-hld Order Alert System` |
| `/generate-component-descriptor` | Generate component description (in target project) | See below |
| `/add-component` | Register a component in the Partner | See below |

---

## Registering Components Incrementally

You can register components one at a time as needed. This is useful for:
- Large systems where documenting everything upfront isn't practical
- New components being added to the architecture
- Mixed-technology components (monorepos with backend + frontend)

### Step 1: Generate Component Descriptor (In Target Project)

1. **Copy the portable skill** from this project:
   ```
   .cursor/skills/component-management/generate_component_descriptor.md
   ```
   
2. **Paste it** into the target component's project:
   ```
   {component-project}/.cursor/skills/generate_component_descriptor.md
   ```

3. **Run the command** in the component's project:
   ```
   /generate-component-descriptor
   ```

4. **Output:** A YAML descriptor with all component details

### Step 2: Add to Architectural Partner (In This Project)

1. **Copy the YAML output** from Step 1

2. **Run the command** in this project:
   ```
   /add-component
   [paste YAML here]
   ```

3. **Result:** 
   - Creates `.cursor/rules/components/{component-name}.mdc`
   - Updates the component registry

### Example: Registering a Mixed-Technology Component

**Scenario:** You have an `admin-dashboard` monorepo with Java backend and Angular frontend.

**Step 1:** In the admin-dashboard project, run `/generate-component-descriptor`:

```yaml
component:
  name: "admin-dashboard"
  description: "Full-stack admin application for system configuration"
  domain: "Frontend Experience"
  type: "monorepo"

technologies:
  primary: "Angular + Java/Spring"
  stack_details:
    backend: "Java/Spring"
    frontend: "Angular"
    database: "PostgreSQL"

# Backend portion
backend:
  path: "backend/"
  apis:
    rest:
      - method: "GET"
        path: "/api/admin/config"
        description: "Get system configuration"

# Frontend portion
frontend:
  path: "frontend/"
  framework: "Angular 17"
  pages:
    - name: "ConfigPage"
      route: "/admin/config"

trigger_phrases:
  - "admin dashboard"
  - "system configuration"
  - "admin panel"
```

**Step 2:** In this project, run `/add-component` and paste the YAML.

**Result:** The component is now registered and will be included in architectural analysis.

### Viewing Registered Components

Check the registry:
```
.cursor/rules/components/_registry.mdc
```

View a specific component:
```
.cursor/rules/components/{component-name}.mdc
```

---

## Component Guardians

Every component must have **guardians**—people responsible for the component.

### Guardian Types

| Type | Required | Purpose |
|------|----------|---------|
| **Main Guardian** | 1+ required | Primary owners/maintainers. Contact for architectural decisions and approvals. |
| **Secondary Guardian** | 0+ optional | Backup contacts and contributors. Contact when main guardians unavailable. |

### Specifying Guardians

When generating a component descriptor, include guardians:

```yaml
guardians:
  main:
    - name: "Sarah Chen"
      email: "sarah.chen@company.com"
      role: "Tech Lead"
    - name: "Marcus Johnson"
      email: "marcus.johnson@company.com"
      role: "Senior Developer"
  secondary:
    - name: "Alex Rivera"
      email: "alex.rivera@company.com"
      role: "Developer"
```

### Viewing Guardians

**All guardians by component:**
```
.cursor/rules/components/_guardians.mdc
```

**Find who owns a specific component:**
- Check the component's rule file: `.cursor/rules/components/{name}.mdc`
- Or search the guardians registry

**Find what components a person owns:**
- Check the "Guardians by Person" section in `_guardians.mdc`

### Guardian Responsibilities

**Main Guardians:**
- Review and approve architectural changes
- Be notified of HLD changes affecting their component
- Maintain component documentation
- Coordinate with dependent component guardians

**Secondary Guardians:**
- Available as backup when main guardians unavailable
- Stay informed about component changes
- Assist with code reviews and implementation

---

## Customizing for Your System

### 1. Update the Domain Map

Edit `.cursor/rules/00_arch_responsibility_framework.mdc`:

```markdown
### [Your Domain Name]
- **Business Responsibility:** What this domain handles
- **Typical Components:** List your components
- **Owns Data Models:** What data this domain owns
- **Trigger Phrases:** Common terms for this domain
```

### 2. Document Your Components

Update `docs/architecture/component-responsibilities.md`:
- Component name and type (backend, frontend, library)
- Technology stack
- APIs exposed
- Events published/consumed

### 3. Add Technology-Specific Skills

Create skills for your specific technologies:
- `.cursor/skills/frontend/vue_patterns.md`
- `.cursor/skills/backend/python_patterns.md`
- `.cursor/skills/mobile/flutter_patterns.md`

---

## Best Practices

### For Architects
1. **Define stable domains:** Business capabilities change slower than technology
2. **Document component ownership:** Clear ownership prevents confusion
3. **Standardize integration patterns:** Consistent event/API patterns across teams

### For Developers
1. **Start with analysis:** Always run `/analyze-architecture` first
2. **Verify with discovery:** Use `/discover-component` for MEDIUM confidence items
3. **Ask about technology:** The partner will ask when it needs tech-specific details

### For Teams
1. **Share HLDs:** Generated HLDs become team documentation
2. **Update framework:** When predictions are wrong, fix the domain rules
3. **Add patterns:** Share successful implementation patterns as skills

---

## Mixed-Technology Components

The framework fully supports components that use multiple technologies (monorepos, full-stack apps):

### How It Works

1. **Detection:** The `/generate-component-descriptor` skill detects all technologies present
2. **Mapping:** Backend and frontend portions are documented separately
3. **Analysis:** When analyzing specs, both portions are considered
4. **HLD Output:** Changes are grouped by technology portion

### Example: Admin Dashboard (Java + Angular)

```
admin-dashboard/
├── backend/              ← Java/Spring API
│   └── src/
└── frontend/             ← Angular UI
    └── src/
```

When a spec mentions "admin configuration page":
- **Frontend portion** gets UI changes (Angular components)
- **Backend portion** gets API changes (if new endpoints needed)

The HLD will clearly separate:
```markdown
## admin-dashboard (Mixed Technology)

### Backend Changes (Java/Spring)
- Add endpoint: GET /api/admin/new-feature

### Frontend Changes (Angular)
- New component: NewFeatureComponent
- Modified: AdminModule to include new route
```

---

## Benefits

| Benefit | Description |
|---------|-------------|
| **Technology Independence** | Same reasoning works for Java, Angular, Node, Python, React |
| **Mixed-Tech Support** | Handles monorepos with multiple technologies |
| **Incremental Registration** | Add components one at a time as needed |
| **Business Alignment** | Focus on capabilities, not implementation details |
| **Reduced Cognitive Load** | Don't need to know every component upfront |
| **Faster Onboarding** | New devs understand architecture through business domains |
| **Future-Proof** | Add new technologies without rewriting the framework |

---

## Troubleshooting

### "Analysis is too generic"
- Add more specific trigger phrases to your domain definitions
- Run `/discover-component` to find actual component names
- Update component-responsibilities.md with real component details

### "Can't determine technology"
- The framework stays agnostic until you tell it
- Run discovery to find actual tech stack
- Add technology info to component documentation

### "Too many components suggested"
- Be more specific in the business specification
- Add qualifiers (which customer type? which transaction type?)
- Use discovery to narrow down

---

## License

MIT License - Use and adapt freely for your projects.

---

## Acknowledgments

This system is inspired by:
- Domain-Driven Design (Eric Evans)
- Clean Architecture (Robert C. Martin)
- Enterprise Integration Patterns (Hohpe & Woolf)
- Technology-agnostic design principles
