name: react-patterns
description: Implementation patterns for React applications in the architecture. Use when Frontend Experience domain changes involve React apps.
version: 1.0.0

---

# REACT IMPLEMENTATION PATTERNS

## Overview

This skill provides React-specific guidance when implementing changes identified by the technology-agnostic analysis. Use this after the HLD identifies a React frontend application.

---

## COMPONENT ARCHITECTURE

### Container vs. Presentational Components

When the HLD specifies a new UI component, decide the pattern:

**Container Components:**
- Fetch and manage data
- Handle business logic
- Connect to global state
- Pass data to presentational components

```tsx
// Example: Container component for alert dashboard
const AlertDashboardContainer: React.FC = () => {
  const { alerts, isLoading, error } = useAlerts();
  const { markAsRead } = useAlertActions();
  
  if (isLoading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;
  
  return (
    <AlertDashboard
      alerts={alerts}
      onAlertSelect={(alert) => markAsRead(alert.id)}
    />
  );
};
```

**Presentational Components:**
- Receive data via props
- Emit events via callbacks
- Focus on rendering
- Highly reusable

```tsx
// Example: Presentational alert flag component
interface AlertFlagProps {
  isPriority: boolean;
  label?: string;
}

const AlertFlag: React.FC<AlertFlagProps> = ({ 
  isPriority, 
  label = 'Alert' 
}) => {
  return (
    <span className={`alert-flag ${isPriority ? 'high-priority' : ''}`}>
      <WarningIcon />
      {label}
    </span>
  );
};
```

---

## CUSTOM HOOKS

### API Integration Hook

When the HLD specifies "component calls [backend] API":

```tsx
// useAlerts.ts
interface UseAlertsResult {
  alerts: Alert[];
  isLoading: boolean;
  error: Error | null;
  refetch: () => void;
}

export function useAlerts(params?: AlertQueryParams): UseAlertsResult {
  const [alerts, setAlerts] = useState<Alert[]>([]);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);
  
  const fetchAlerts = useCallback(async () => {
    setIsLoading(true);
    setError(null);
    try {
      const response = await fetch('/api/alerts/high-value');
      if (!response.ok) throw new Error('Failed to fetch alerts');
      const data = await response.json();
      setAlerts(data);
    } catch (err) {
      setError(err as Error);
    } finally {
      setIsLoading(false);
    }
  }, [params]);
  
  useEffect(() => {
    fetchAlerts();
  }, [fetchAlerts]);
  
  return { alerts, isLoading, error, refetch: fetchAlerts };
}
```

**With React Query (if available):**

```tsx
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

export function useAlerts(params?: AlertQueryParams) {
  return useQuery({
    queryKey: ['alerts', 'high-value', params],
    queryFn: () => alertApi.getHighValueAlerts(params),
  });
}

export function useAcknowledgeAlert() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: (alertId: string) => alertApi.acknowledge(alertId),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['alerts'] });
    },
  });
}
```

### Real-Time Event Stream Hook

When the HLD specifies "real-time updates" or "subscribe to events":

```tsx
// useAlertStream.ts
export function useAlertStream(): Alert[] {
  const [alerts, setAlerts] = useState<Alert[]>([]);
  
  useEffect(() => {
    const socket = new WebSocket('wss://api.example.com/alerts/stream');
    
    socket.onmessage = (event) => {
      const alert = JSON.parse(event.data) as Alert;
      setAlerts(prev => [alert, ...prev]);
    };
    
    socket.onerror = (error) => {
      console.error('WebSocket error:', error);
    };
    
    return () => socket.close();
  }, []);
  
  return alerts;
}
```

**With Server-Sent Events:**

```tsx
export function useAlertSSE(): { alerts: Alert[]; connectionStatus: string } {
  const [alerts, setAlerts] = useState<Alert[]>([]);
  const [status, setStatus] = useState('connecting');
  
  useEffect(() => {
    const eventSource = new EventSource('/api/alerts/stream');
    
    eventSource.onopen = () => setStatus('connected');
    
    eventSource.onmessage = (event) => {
      const alert = JSON.parse(event.data);
      setAlerts(prev => [alert, ...prev]);
    };
    
    eventSource.onerror = () => setStatus('error');
    
    return () => eventSource.close();
  }, []);
  
  return { alerts, connectionStatus: status };
}
```

---

## STATE MANAGEMENT

### Context + Reducer Pattern

For feature-specific state without Redux:

```tsx
// AlertContext.tsx
interface AlertState {
  alerts: Alert[];
  selectedAlert: Alert | null;
  filter: AlertFilter;
}

type AlertAction =
  | { type: 'SET_ALERTS'; payload: Alert[] }
  | { type: 'ADD_ALERT'; payload: Alert }
  | { type: 'SELECT_ALERT'; payload: Alert | null }
  | { type: 'SET_FILTER'; payload: AlertFilter };

const AlertContext = createContext<{
  state: AlertState;
  dispatch: React.Dispatch<AlertAction>;
} | null>(null);

function alertReducer(state: AlertState, action: AlertAction): AlertState {
  switch (action.type) {
    case 'SET_ALERTS':
      return { ...state, alerts: action.payload };
    case 'ADD_ALERT':
      return { ...state, alerts: [action.payload, ...state.alerts] };
    case 'SELECT_ALERT':
      return { ...state, selectedAlert: action.payload };
    case 'SET_FILTER':
      return { ...state, filter: action.payload };
    default:
      return state;
  }
}

export const AlertProvider: React.FC<{ children: React.ReactNode }> = ({ 
  children 
}) => {
  const [state, dispatch] = useReducer(alertReducer, initialState);
  
  return (
    <AlertContext.Provider value={{ state, dispatch }}>
      {children}
    </AlertContext.Provider>
  );
};

export function useAlertContext() {
  const context = useContext(AlertContext);
  if (!context) {
    throw new Error('useAlertContext must be used within AlertProvider');
  }
  return context;
}
```

### Redux Toolkit Pattern (If Already in Use)

```tsx
// alertSlice.ts
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';

export const fetchAlerts = createAsyncThunk(
  'alerts/fetchAlerts',
  async (params: AlertQueryParams) => {
    const response = await alertApi.getHighValueAlerts(params);
    return response;
  }
);

const alertSlice = createSlice({
  name: 'alerts',
  initialState: {
    items: [] as Alert[],
    loading: false,
    error: null as string | null,
  },
  reducers: {
    alertReceived: (state, action: PayloadAction<Alert>) => {
      state.items.unshift(action.payload);
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchAlerts.pending, (state) => {
        state.loading = true;
      })
      .addCase(fetchAlerts.fulfilled, (state, action) => {
        state.loading = false;
        state.items = action.payload;
      })
      .addCase(fetchAlerts.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message ?? 'Failed to fetch';
      });
  },
});

export const { alertReceived } = alertSlice.actions;
export default alertSlice.reducer;
```

---

## COMPONENT ORGANIZATION

When HLD specifies a new feature area:

```
src/
├── features/
│   └── alerts/
│       ├── components/
│       │   ├── AlertDashboard.tsx
│       │   ├── AlertList.tsx
│       │   ├── AlertFlag.tsx
│       │   └── AlertDetailModal.tsx
│       ├── hooks/
│       │   ├── useAlerts.ts
│       │   └── useAlertStream.ts
│       ├── context/
│       │   └── AlertContext.tsx
│       ├── api/
│       │   └── alertApi.ts
│       ├── types/
│       │   └── index.ts
│       └── index.ts  // Public exports
```

**Feature Barrel Export:**

```tsx
// features/alerts/index.ts
export { AlertDashboard } from './components/AlertDashboard';
export { AlertFlag } from './components/AlertFlag';
export { useAlerts } from './hooks/useAlerts';
export { AlertProvider, useAlertContext } from './context/AlertContext';
export type { Alert, AlertPriority } from './types';
```

---

## SHARED MODEL INTEGRATION

When using shared-models library from the HLD:

```tsx
// Import from shared library
import { Alert, AlertPriority, AlertStatus } from '@company/shared-models';

// Extend with UI-specific properties if needed
interface AlertViewModel extends Alert {
  isNew: boolean;  // UI-only flag
  displayTime: string;  // Formatted for display
}

// Transform function
function toViewModel(alert: Alert): AlertViewModel {
  return {
    ...alert,
    isNew: Date.now() - new Date(alert.timestamp).getTime() < 60000,
    displayTime: formatDistanceToNow(new Date(alert.timestamp)),
  };
}

// Use in component
const AlertList: React.FC<{ alerts: Alert[] }> = ({ alerts }) => {
  const viewModels = useMemo(
    () => alerts.map(toViewModel),
    [alerts]
  );
  
  return (
    <ul>
      {viewModels.map(alert => (
        <AlertItem key={alert.id} alert={alert} />
      ))}
    </ul>
  );
};
```

---

## INTEGRATION WITH BACKEND EVENTS

When HLD specifies "frontend subscribes to [EventName]":

```tsx
const AlertDashboard: React.FC = () => {
  // Initial data
  const { data: initialAlerts, isLoading } = useAlerts();
  
  // Real-time updates
  const streamAlerts = useAlertStream();
  
  // Combine initial + streamed
  const allAlerts = useMemo(() => {
    if (!initialAlerts) return streamAlerts;
    
    // Merge, avoiding duplicates
    const alertMap = new Map<string, Alert>();
    [...initialAlerts, ...streamAlerts].forEach(alert => {
      alertMap.set(alert.id, alert);
    });
    
    return Array.from(alertMap.values())
      .sort((a, b) => new Date(b.timestamp).getTime() - new Date(a.timestamp).getTime());
  }, [initialAlerts, streamAlerts]);
  
  // Show toast for new alerts
  useEffect(() => {
    if (streamAlerts.length > 0) {
      const latest = streamAlerts[0];
      toast.info(`New alert: ${latest.title}`);
    }
  }, [streamAlerts.length]);
  
  if (isLoading) return <LoadingSpinner />;
  
  return <AlertList alerts={allAlerts} />;
};
```

---

## TESTING PATTERNS

### Component Testing with React Testing Library

```tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { AlertFlag } from './AlertFlag';

describe('AlertFlag', () => {
  it('should display high-priority styling when isPriority is true', () => {
    render(<AlertFlag isPriority={true} label="High Value" />);
    
    const flag = screen.getByText('High Value').closest('span');
    expect(flag).toHaveClass('high-priority');
  });
  
  it('should display default label when not provided', () => {
    render(<AlertFlag isPriority={false} />);
    
    expect(screen.getByText('Alert')).toBeInTheDocument();
  });
});
```

### Hook Testing

```tsx
import { renderHook, waitFor } from '@testing-library/react';
import { useAlerts } from './useAlerts';

// Mock fetch
global.fetch = jest.fn();

describe('useAlerts', () => {
  it('should fetch and return alerts', async () => {
    const mockAlerts = [{ id: '1', priority: 'HIGH' }];
    (fetch as jest.Mock).mockResolvedValueOnce({
      ok: true,
      json: async () => mockAlerts,
    });
    
    const { result } = renderHook(() => useAlerts());
    
    expect(result.current.isLoading).toBe(true);
    
    await waitFor(() => {
      expect(result.current.isLoading).toBe(false);
    });
    
    expect(result.current.alerts).toEqual(mockAlerts);
  });
});
```

### Integration Testing with MSW

```tsx
import { rest } from 'msw';
import { setupServer } from 'msw/node';

const server = setupServer(
  rest.get('/api/alerts/high-value', (req, res, ctx) => {
    return res(ctx.json([
      { id: '1', title: 'Test Alert', priority: 'HIGH' }
    ]));
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

test('AlertDashboard displays fetched alerts', async () => {
  render(<AlertDashboard />);
  
  await waitFor(() => {
    expect(screen.getByText('Test Alert')).toBeInTheDocument();
  });
});
```

---

## CHECKLIST: React Implementation

When implementing an HLD component in React:

- [ ] Decide container vs. presentational pattern
- [ ] Create custom hooks for data fetching
- [ ] Set up real-time connection hook if needed
- [ ] Choose state management (local, context, Redux)
- [ ] Organize in feature folder structure
- [ ] Import shared models from library
- [ ] Add loading and error states
- [ ] Memoize expensive computations
- [ ] Write unit tests for components and hooks
- [ ] Follow existing project conventions for styling
