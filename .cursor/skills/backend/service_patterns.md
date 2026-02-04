name: backend-service-patterns
description: Implementation patterns for backend services in the architecture. Technology-agnostic patterns with examples in multiple languages.
version: 1.0.0

---

# BACKEND SERVICE IMPLEMENTATION PATTERNS

## Overview

This skill provides backend-specific guidance when implementing changes identified by the technology-agnostic analysis. Patterns are shown in multiple languages; adapt to your specific tech stack.

---

## SERVICE ARCHITECTURE

### Core Service Pattern

When the HLD specifies business logic changes:

**Java/Spring:**
```java
@Service
public class TransactionMonitoringService {
    
    private final TransactionRepository transactionRepository;
    private final EventPublisher eventPublisher;
    private final CustomerServiceClient customerService;
    
    public TransactionMonitoringService(
            TransactionRepository transactionRepository,
            EventPublisher eventPublisher,
            CustomerServiceClient customerService) {
        this.transactionRepository = transactionRepository;
        this.eventPublisher = eventPublisher;
        this.customerService = customerService;
    }
    
    public void processTransaction(Transaction transaction) {
        // Business logic
        if (isHighValue(transaction)) {
            var customerContext = customerService.getCustomer(transaction.getCustomerId());
            var alert = createAlert(transaction, customerContext);
            
            transactionRepository.saveAlert(alert);
            eventPublisher.publish(new HighValueTransactionAlertEvent(alert));
        }
    }
    
    private boolean isHighValue(Transaction transaction) {
        return transaction.getAmount().compareTo(HIGH_VALUE_THRESHOLD) > 0;
    }
}
```

**Node.js/TypeScript:**
```typescript
export class TransactionMonitoringService {
  constructor(
    private readonly transactionRepo: TransactionRepository,
    private readonly eventPublisher: EventPublisher,
    private readonly customerService: CustomerServiceClient
  ) {}
  
  async processTransaction(transaction: Transaction): Promise<void> {
    if (this.isHighValue(transaction)) {
      const customerContext = await this.customerService.getCustomer(
        transaction.customerId
      );
      const alert = this.createAlert(transaction, customerContext);
      
      await this.transactionRepo.saveAlert(alert);
      await this.eventPublisher.publish(
        new HighValueTransactionAlertEvent(alert)
      );
    }
  }
  
  private isHighValue(transaction: Transaction): boolean {
    return transaction.amount > HIGH_VALUE_THRESHOLD;
  }
}
```

**Python/FastAPI:**
```python
class TransactionMonitoringService:
    def __init__(
        self,
        transaction_repo: TransactionRepository,
        event_publisher: EventPublisher,
        customer_service: CustomerServiceClient
    ):
        self.transaction_repo = transaction_repo
        self.event_publisher = event_publisher
        self.customer_service = customer_service
    
    async def process_transaction(self, transaction: Transaction) -> None:
        if self._is_high_value(transaction):
            customer_context = await self.customer_service.get_customer(
                transaction.customer_id
            )
            alert = self._create_alert(transaction, customer_context)
            
            await self.transaction_repo.save_alert(alert)
            await self.event_publisher.publish(
                HighValueTransactionAlertEvent(alert)
            )
    
    def _is_high_value(self, transaction: Transaction) -> bool:
        return transaction.amount > HIGH_VALUE_THRESHOLD
```

---

## API ENDPOINT PATTERNS

### REST Controller

When the HLD specifies "new API endpoint":

**Java/Spring:**
```java
@RestController
@RequestMapping("/api/alerts")
public class AlertController {
    
    private final AlertService alertService;
    
    @GetMapping("/high-value")
    public ResponseEntity<List<AlertDTO>> getHighValueAlerts(
            @RequestParam(required = false) String status,
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size) {
        
        var alerts = alertService.getHighValueAlerts(status, page, size);
        return ResponseEntity.ok(alerts);
    }
    
    @PostMapping("/{id}/acknowledge")
    public ResponseEntity<Void> acknowledgeAlert(@PathVariable String id) {
        alertService.acknowledge(id);
        return ResponseEntity.ok().build();
    }
    
    @PostMapping("/from-transaction")
    public ResponseEntity<AlertDTO> createFromTransaction(
            @RequestBody @Valid CreateAlertFromTransactionRequest request) {
        
        var alert = alertService.createFromTransaction(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(alert);
    }
}
```

**Node.js/Express:**
```typescript
const router = express.Router();

router.get('/high-value', async (req, res, next) => {
  try {
    const { status, page = 0, size = 20 } = req.query;
    const alerts = await alertService.getHighValueAlerts(
      status as string,
      Number(page),
      Number(size)
    );
    res.json(alerts);
  } catch (error) {
    next(error);
  }
});

router.post('/:id/acknowledge', async (req, res, next) => {
  try {
    await alertService.acknowledge(req.params.id);
    res.status(200).send();
  } catch (error) {
    next(error);
  }
});

router.post('/from-transaction', async (req, res, next) => {
  try {
    const alert = await alertService.createFromTransaction(req.body);
    res.status(201).json(alert);
  } catch (error) {
    next(error);
  }
});

export default router;
```

**Python/FastAPI:**
```python
router = APIRouter(prefix="/api/alerts", tags=["alerts"])

@router.get("/high-value", response_model=List[AlertDTO])
async def get_high_value_alerts(
    status: Optional[str] = None,
    page: int = 0,
    size: int = 20,
    alert_service: AlertService = Depends(get_alert_service)
):
    return await alert_service.get_high_value_alerts(status, page, size)

@router.post("/{alert_id}/acknowledge", status_code=200)
async def acknowledge_alert(
    alert_id: str,
    alert_service: AlertService = Depends(get_alert_service)
):
    await alert_service.acknowledge(alert_id)
    return {"status": "acknowledged"}

@router.post("/from-transaction", response_model=AlertDTO, status_code=201)
async def create_from_transaction(
    request: CreateAlertFromTransactionRequest,
    alert_service: AlertService = Depends(get_alert_service)
):
    return await alert_service.create_from_transaction(request)
```

---

## EVENT PUBLISHING PATTERNS

When the HLD specifies "publishes [EventName]":

### Event Definition

**Java:**
```java
public class HighValueTransactionAlertEvent {
    private final String eventId;
    private final Instant timestamp;
    private final String alertId;
    private final String transactionId;
    private final String customerId;
    private final BigDecimal amount;
    private final String riskLevel;
    
    public HighValueTransactionAlertEvent(Alert alert) {
        this.eventId = UUID.randomUUID().toString();
        this.timestamp = Instant.now();
        this.alertId = alert.getId();
        this.transactionId = alert.getTransactionId();
        this.customerId = alert.getCustomerId();
        this.amount = alert.getAmount();
        this.riskLevel = alert.getRiskLevel();
    }
    
    // Getters...
}
```

**TypeScript:**
```typescript
export interface HighValueTransactionAlertEvent {
  eventId: string;
  timestamp: string;
  alertId: string;
  transactionId: string;
  customerId: string;
  amount: number;
  riskLevel: 'LOW' | 'MEDIUM' | 'HIGH' | 'CRITICAL';
}

export function createAlertEvent(alert: Alert): HighValueTransactionAlertEvent {
  return {
    eventId: uuidv4(),
    timestamp: new Date().toISOString(),
    alertId: alert.id,
    transactionId: alert.transactionId,
    customerId: alert.customerId,
    amount: alert.amount,
    riskLevel: alert.riskLevel,
  };
}
```

### Event Publisher

**Java/Spring with Kafka:**
```java
@Component
public class KafkaEventPublisher implements EventPublisher {
    
    private final KafkaTemplate<String, Object> kafkaTemplate;
    private final ObjectMapper objectMapper;
    
    @Override
    public void publish(Object event) {
        String topic = determineTopicFor(event);
        String key = extractKeyFrom(event);
        
        kafkaTemplate.send(topic, key, event)
            .addCallback(
                result -> log.info("Event published: {}", event),
                error -> log.error("Failed to publish event", error)
            );
    }
    
    private String determineTopicFor(Object event) {
        if (event instanceof HighValueTransactionAlertEvent) {
            return "transactions.alerts.high-value";
        }
        throw new IllegalArgumentException("Unknown event type");
    }
}
```

**Node.js with Kafka:**
```typescript
export class KafkaEventPublisher implements EventPublisher {
  constructor(private readonly producer: Producer) {}
  
  async publish(event: DomainEvent): Promise<void> {
    const topic = this.determineTopicFor(event);
    
    await this.producer.send({
      topic,
      messages: [{
        key: event.aggregateId,
        value: JSON.stringify(event),
        headers: {
          'event-type': event.constructor.name,
          'timestamp': new Date().toISOString(),
        },
      }],
    });
    
    console.log(`Event published to ${topic}:`, event.eventId);
  }
  
  private determineTopicFor(event: DomainEvent): string {
    if (event instanceof HighValueTransactionAlertEvent) {
      return 'transactions.alerts.high-value';
    }
    throw new Error(`Unknown event type: ${event.constructor.name}`);
  }
}
```

---

## EVENT CONSUMING PATTERNS

When the HLD specifies "consumes [EventName]":

**Java/Spring with Kafka:**
```java
@Component
public class AlertNotificationHandler {
    
    private final NotificationService notificationService;
    private final TeamService teamService;
    
    @KafkaListener(topics = "transactions.alerts.high-value")
    public void handleHighValueAlert(HighValueTransactionAlertEvent event) {
        log.info("Received high-value alert: {}", event.getAlertId());
        
        // Get risk team members to notify
        var teamMembers = teamService.getRiskTeamMembers();
        
        // Send notifications
        for (var member : teamMembers) {
            notificationService.sendAlert(member, event);
        }
        
        // Also send to Slack channel
        notificationService.sendToSlackChannel("#risk-alerts", event);
    }
}
```

**Node.js with Kafka:**
```typescript
export class AlertNotificationHandler {
  constructor(
    private readonly notificationService: NotificationService,
    private readonly teamService: TeamService
  ) {}
  
  async handleHighValueAlert(event: HighValueTransactionAlertEvent): Promise<void> {
    console.log(`Received high-value alert: ${event.alertId}`);
    
    // Get risk team members to notify
    const teamMembers = await this.teamService.getRiskTeamMembers();
    
    // Send notifications in parallel
    await Promise.all([
      ...teamMembers.map(member => 
        this.notificationService.sendAlert(member, event)
      ),
      this.notificationService.sendToSlackChannel('#risk-alerts', event),
    ]);
  }
}

// Registration with Kafka consumer
consumer.subscribe({ topic: 'transactions.alerts.high-value' });

consumer.run({
  eachMessage: async ({ message }) => {
    const event = JSON.parse(message.value.toString());
    await alertNotificationHandler.handleHighValueAlert(event);
  },
});
```

---

## CROSS-SERVICE COMMUNICATION

### HTTP Client Pattern

When the HLD specifies "calls [other-service] API":

**Java/Spring:**
```java
@Component
public class CustomerServiceClient {
    
    private final RestTemplate restTemplate;
    private final String baseUrl;
    
    public CustomerServiceClient(
            RestTemplate restTemplate,
            @Value("${services.customer.url}") String baseUrl) {
        this.restTemplate = restTemplate;
        this.baseUrl = baseUrl;
    }
    
    public CustomerDTO getCustomer(String customerId) {
        String url = baseUrl + "/api/customers/" + customerId;
        
        try {
            return restTemplate.getForObject(url, CustomerDTO.class);
        } catch (HttpClientErrorException.NotFound e) {
            throw new CustomerNotFoundException(customerId);
        }
    }
    
    public CustomerTierDTO getCustomerTier(String customerId) {
        String url = baseUrl + "/api/customers/" + customerId + "/tier";
        return restTemplate.getForObject(url, CustomerTierDTO.class);
    }
}
```

**Node.js/TypeScript:**
```typescript
export class CustomerServiceClient {
  constructor(
    private readonly httpClient: AxiosInstance,
    private readonly baseUrl: string
  ) {}
  
  async getCustomer(customerId: string): Promise<CustomerDTO> {
    try {
      const response = await this.httpClient.get<CustomerDTO>(
        `${this.baseUrl}/api/customers/${customerId}`
      );
      return response.data;
    } catch (error) {
      if (axios.isAxiosError(error) && error.response?.status === 404) {
        throw new CustomerNotFoundException(customerId);
      }
      throw error;
    }
  }
  
  async getCustomerTier(customerId: string): Promise<CustomerTierDTO> {
    const response = await this.httpClient.get<CustomerTierDTO>(
      `${this.baseUrl}/api/customers/${customerId}/tier`
    );
    return response.data;
  }
}
```

---

## DATA MODEL PATTERNS

### Entity Definition

When the HLD specifies "add [field] to [entity]":

**Java/JPA:**
```java
@Entity
@Table(name = "transaction_alerts")
public class TransactionAlert {
    
    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    private String id;
    
    @Column(name = "transaction_id", nullable = false)
    private String transactionId;
    
    @Column(name = "customer_id", nullable = false)
    private String customerId;
    
    @Column(nullable = false)
    private BigDecimal amount;
    
    @Enumerated(EnumType.STRING)
    @Column(name = "risk_level", nullable = false)
    private RiskLevel riskLevel;
    
    @Column(name = "is_acknowledged")
    private boolean acknowledged = false;
    
    @Column(name = "created_at", nullable = false)
    private Instant createdAt;
    
    @PrePersist
    void prePersist() {
        this.createdAt = Instant.now();
    }
    
    // Getters, setters, builder...
}
```

**TypeScript/TypeORM:**
```typescript
@Entity('transaction_alerts')
export class TransactionAlert {
  @PrimaryGeneratedColumn('uuid')
  id: string;
  
  @Column({ name: 'transaction_id' })
  transactionId: string;
  
  @Column({ name: 'customer_id' })
  customerId: string;
  
  @Column('decimal', { precision: 10, scale: 2 })
  amount: number;
  
  @Column({ type: 'enum', enum: RiskLevel })
  riskLevel: RiskLevel;
  
  @Column({ name: 'is_acknowledged', default: false })
  acknowledged: boolean;
  
  @CreateDateColumn({ name: 'created_at' })
  createdAt: Date;
}
```

---

## DATABASE MIGRATION PATTERNS

When the HLD specifies schema changes:

**Flyway (Java):**
```sql
-- V2024_01_15__add_transaction_alerts_table.sql

CREATE TABLE transaction_alerts (
    id VARCHAR(36) PRIMARY KEY,
    transaction_id VARCHAR(36) NOT NULL,
    customer_id VARCHAR(36) NOT NULL,
    amount DECIMAL(10, 2) NOT NULL,
    risk_level VARCHAR(20) NOT NULL,
    is_acknowledged BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    
    INDEX idx_alerts_customer (customer_id),
    INDEX idx_alerts_created (created_at),
    INDEX idx_alerts_risk (risk_level, is_acknowledged)
);
```

**TypeORM Migration:**
```typescript
export class AddTransactionAlertsTable1705312000000 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.createTable(new Table({
      name: 'transaction_alerts',
      columns: [
        { name: 'id', type: 'uuid', isPrimary: true, default: 'uuid_generate_v4()' },
        { name: 'transaction_id', type: 'uuid', isNullable: false },
        { name: 'customer_id', type: 'uuid', isNullable: false },
        { name: 'amount', type: 'decimal', precision: 10, scale: 2 },
        { name: 'risk_level', type: 'varchar', length: '20' },
        { name: 'is_acknowledged', type: 'boolean', default: false },
        { name: 'created_at', type: 'timestamp', default: 'CURRENT_TIMESTAMP' },
      ],
      indices: [
        { columnNames: ['customer_id'] },
        { columnNames: ['created_at'] },
        { columnNames: ['risk_level', 'is_acknowledged'] },
      ],
    }));
  }
  
  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropTable('transaction_alerts');
  }
}
```

---

## TESTING PATTERNS

### Unit Testing Services

**Java/JUnit:**
```java
@ExtendWith(MockitoExtension.class)
class TransactionMonitoringServiceTest {
    
    @Mock TransactionRepository transactionRepository;
    @Mock EventPublisher eventPublisher;
    @Mock CustomerServiceClient customerService;
    
    @InjectMocks TransactionMonitoringService service;
    
    @Test
    void shouldPublishAlertForHighValueTransaction() {
        // Given
        var transaction = Transaction.builder()
            .id("txn-1")
            .customerId("cust-1")
            .amount(new BigDecimal("15000"))
            .build();
        
        when(customerService.getCustomer("cust-1"))
            .thenReturn(new CustomerDTO("cust-1", "John Doe"));
        
        // When
        service.processTransaction(transaction);
        
        // Then
        verify(transactionRepository).saveAlert(any(Alert.class));
        verify(eventPublisher).publish(any(HighValueTransactionAlertEvent.class));
    }
    
    @Test
    void shouldNotPublishAlertForLowValueTransaction() {
        // Given
        var transaction = Transaction.builder()
            .amount(new BigDecimal("100"))
            .build();
        
        // When
        service.processTransaction(transaction);
        
        // Then
        verifyNoInteractions(eventPublisher);
    }
}
```

---

## CHECKLIST: Backend Implementation

When implementing an HLD service change:

- [ ] Define/update service class with dependencies
- [ ] Create/update API endpoints
- [ ] Define event schemas (if publishing)
- [ ] Create event handlers (if consuming)
- [ ] Add cross-service clients (if calling other services)
- [ ] Update data models and entities
- [ ] Create database migrations
- [ ] Add configuration for thresholds, URLs, topics
- [ ] Write unit tests for business logic
- [ ] Write integration tests for APIs
- [ ] Add logging and metrics
- [ ] Document API changes in OpenAPI/Swagger
