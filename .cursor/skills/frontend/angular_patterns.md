name: angular-patterns
description: Implementation patterns for Angular applications in the architecture. Use when Frontend Experience domain changes involve Angular apps.
version: 1.0.0

---

# ANGULAR IMPLEMENTATION PATTERNS

## Overview

This skill provides Angular-specific guidance when implementing changes identified by the technology-agnostic analysis. Use this after the HLD identifies an Angular frontend application.

---

## COMPONENT ARCHITECTURE

### Smart vs. Presentational Components

When the HLD specifies a new UI component, decide the pattern:

**Smart Components (Containers):**
- Fetch data from services
- Handle user interactions
- Manage local state
- Connect to store (if using NgRx)

```typescript
// Example: Smart component for alert dashboard
@Component({
  selector: 'app-alert-dashboard',
  template: `
    <app-alert-list 
      [alerts]="alerts$ | async"
      (alertSelected)="onAlertSelect($event)">
    </app-alert-list>
  `
})
export class AlertDashboardComponent implements OnInit {
  alerts$ = this.alertService.getAlerts();
  
  constructor(private alertService: AlertService) {}
  
  onAlertSelect(alert: Alert) {
    this.alertService.markAsRead(alert.id);
  }
}
```

**Presentational Components (Dumb):**
- Receive data via @Input
- Emit events via @Output
- No service injection
- Pure display logic

```typescript
// Example: Presentational alert flag component
@Component({
  selector: 'app-alert-flag',
  template: `
    <span class="alert-flag" [class.high-priority]="isPriority">
      <mat-icon>warning</mat-icon>
      {{ label }}
    </span>
  `
})
export class AlertFlagComponent {
  @Input() isPriority = false;
  @Input() label = 'Alert';
}
```

---

## SERVICE PATTERNS

### API Integration Service

When the HLD specifies "component calls [backend] API":

```typescript
@Injectable({ providedIn: 'root' })
export class TransactionAlertService {
  private readonly apiUrl = '/api/alerts';
  
  constructor(private http: HttpClient) {}
  
  // GET list with query params
  getHighValueAlerts(params?: AlertQueryParams): Observable<Alert[]> {
    return this.http.get<Alert[]>(`${this.apiUrl}/high-value`, { params });
  }
  
  // GET single item
  getAlert(id: string): Observable<Alert> {
    return this.http.get<Alert>(`${this.apiUrl}/${id}`);
  }
  
  // POST create
  acknowledgeAlert(id: string): Observable<void> {
    return this.http.post<void>(`${this.apiUrl}/${id}/acknowledge`, {});
  }
}
```

### Real-Time Event Stream Service

When the HLD specifies "real-time updates" or "subscribe to events":

```typescript
@Injectable({ providedIn: 'root' })
export class AlertStreamService {
  private socket: WebSocket | null = null;
  private alerts$ = new BehaviorSubject<Alert[]>([]);
  
  constructor() {}
  
  connect(): Observable<Alert[]> {
    if (!this.socket) {
      this.socket = new WebSocket('wss://api.example.com/alerts/stream');
      
      this.socket.onmessage = (event) => {
        const alert = JSON.parse(event.data) as Alert;
        const current = this.alerts$.value;
        this.alerts$.next([alert, ...current]);
      };
      
      this.socket.onerror = (error) => {
        console.error('WebSocket error:', error);
      };
    }
    return this.alerts$.asObservable();
  }
  
  disconnect(): void {
    this.socket?.close();
    this.socket = null;
  }
}
```

**Alternative using SSE (Server-Sent Events):**

```typescript
@Injectable({ providedIn: 'root' })
export class AlertSSEService {
  getAlertStream(): Observable<Alert> {
    return new Observable(observer => {
      const eventSource = new EventSource('/api/alerts/stream');
      
      eventSource.onmessage = (event) => {
        observer.next(JSON.parse(event.data));
      };
      
      eventSource.onerror = (error) => {
        observer.error(error);
      };
      
      return () => eventSource.close();
    });
  }
}
```

---

## STATE MANAGEMENT

### Simple State: Service + BehaviorSubject

For components that need shared state without NgRx:

```typescript
@Injectable({ providedIn: 'root' })
export class AlertStateService {
  private alertsSubject = new BehaviorSubject<Alert[]>([]);
  private selectedAlertSubject = new BehaviorSubject<Alert | null>(null);
  
  alerts$ = this.alertsSubject.asObservable();
  selectedAlert$ = this.selectedAlertSubject.asObservable();
  
  setAlerts(alerts: Alert[]): void {
    this.alertsSubject.next(alerts);
  }
  
  selectAlert(alert: Alert): void {
    this.selectedAlertSubject.next(alert);
  }
  
  addAlert(alert: Alert): void {
    const current = this.alertsSubject.value;
    this.alertsSubject.next([alert, ...current]);
  }
}
```

### NgRx Pattern (If Already in Use)

```typescript
// actions
export const loadAlerts = createAction('[Alerts] Load');
export const loadAlertsSuccess = createAction(
  '[Alerts] Load Success',
  props<{ alerts: Alert[] }>()
);
export const alertReceived = createAction(
  '[Alerts] Real-time Alert Received',
  props<{ alert: Alert }>()
);

// reducer
export const alertsReducer = createReducer(
  initialState,
  on(loadAlertsSuccess, (state, { alerts }) => ({
    ...state,
    alerts,
    loading: false
  })),
  on(alertReceived, (state, { alert }) => ({
    ...state,
    alerts: [alert, ...state.alerts]
  }))
);

// effects
@Injectable()
export class AlertEffects {
  loadAlerts$ = createEffect(() =>
    this.actions$.pipe(
      ofType(loadAlerts),
      switchMap(() =>
        this.alertService.getHighValueAlerts().pipe(
          map(alerts => loadAlertsSuccess({ alerts }))
        )
      )
    )
  );
  
  constructor(
    private actions$: Actions,
    private alertService: TransactionAlertService
  ) {}
}
```

---

## MODULE ORGANIZATION

When HLD specifies a new feature area:

```typescript
// alerts.module.ts
@NgModule({
  declarations: [
    AlertDashboardComponent,
    AlertListComponent,
    AlertFlagComponent,
    AlertDetailModalComponent
  ],
  imports: [
    CommonModule,
    AlertsRoutingModule,
    SharedModule,
    MatIconModule,
    MatDialogModule
  ],
  exports: [
    AlertFlagComponent  // Export if used by other modules
  ]
})
export class AlertsModule {}

// alerts-routing.module.ts
const routes: Routes = [
  {
    path: '',
    component: AlertDashboardComponent,
    children: [
      { path: ':id', component: AlertDetailComponent }
    ]
  }
];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule]
})
export class AlertsRoutingModule {}
```

---

## SHARED MODEL INTEGRATION

When using shared-models library from the HLD:

```typescript
// Import from shared library
import { Alert, AlertPriority, AlertStatus } from '@company/shared-models';

// Extend with UI-specific properties if needed
interface AlertViewModel extends Alert {
  isNew: boolean;  // UI-only flag
  displayTime: string;  // Formatted for display
}

// Transform in service or component
function toViewModel(alert: Alert): AlertViewModel {
  return {
    ...alert,
    isNew: Date.now() - new Date(alert.timestamp).getTime() < 60000,
    displayTime: formatDistanceToNow(new Date(alert.timestamp))
  };
}
```

---

## INTEGRATION WITH BACKEND EVENTS

When HLD specifies "frontend subscribes to [EventName]":

```typescript
// In component or service
@Component({...})
export class AlertDashboardComponent implements OnInit, OnDestroy {
  private destroy$ = new Subject<void>();
  
  ngOnInit() {
    // Initial load
    this.alertService.getHighValueAlerts()
      .pipe(takeUntil(this.destroy$))
      .subscribe(alerts => this.alerts = alerts);
    
    // Real-time updates
    this.alertStreamService.connect()
      .pipe(takeUntil(this.destroy$))
      .subscribe(newAlert => {
        this.alerts = [newAlert, ...this.alerts];
        this.showToast('New high-value alert detected');
      });
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
    this.alertStreamService.disconnect();
  }
}
```

---

## TESTING PATTERNS

### Component Testing

```typescript
describe('AlertFlagComponent', () => {
  let component: AlertFlagComponent;
  let fixture: ComponentFixture<AlertFlagComponent>;
  
  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [AlertFlagComponent],
      imports: [MatIconModule]
    }).compileComponents();
    
    fixture = TestBed.createComponent(AlertFlagComponent);
    component = fixture.componentInstance;
  });
  
  it('should show high-priority class when isPriority is true', () => {
    component.isPriority = true;
    fixture.detectChanges();
    
    const element = fixture.nativeElement.querySelector('.alert-flag');
    expect(element.classList).toContain('high-priority');
  });
});
```

### Service Testing with HttpClientTestingModule

```typescript
describe('TransactionAlertService', () => {
  let service: TransactionAlertService;
  let httpMock: HttpTestingController;
  
  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [TransactionAlertService]
    });
    
    service = TestBed.inject(TransactionAlertService);
    httpMock = TestBed.inject(HttpTestingController);
  });
  
  it('should fetch high-value alerts', () => {
    const mockAlerts: Alert[] = [{ id: '1', priority: 'HIGH' }];
    
    service.getHighValueAlerts().subscribe(alerts => {
      expect(alerts).toEqual(mockAlerts);
    });
    
    const req = httpMock.expectOne('/api/alerts/high-value');
    expect(req.request.method).toBe('GET');
    req.flush(mockAlerts);
  });
});
```

---

## CHECKLIST: Angular Implementation

When implementing an HLD component in Angular:

- [ ] Decide smart vs. presentational component pattern
- [ ] Create service for API integration
- [ ] Set up real-time connection if needed
- [ ] Choose state management approach
- [ ] Create feature module with routing
- [ ] Import shared models from library
- [ ] Add error handling and loading states
- [ ] Write unit tests for components and services
- [ ] Follow existing project conventions for styling
