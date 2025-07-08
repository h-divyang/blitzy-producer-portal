# 9.0 State Management - Updated

## State Management Architecture

### State Categories
1. **Local Component State**: UI-specific state that doesn't need sharing
2. **Shared Application State**: Cross-component state (auth, theme, notifications)
3. **Server State**: Data fetched from backend APIs
4. **Form State**: Temporary form data during user input
5. **Navigation State**: Routing and navigation history

## Local Component State

### React Hooks for Local State
```typescript
// Simple state with useState
const [isOpen, setIsOpen] = useState(false);
const [formData, setFormData] = useState<PolicyData>(initialData);

// Complex state with useReducer
const [state, dispatch] = useReducer(policyReducer, initialState);

// Example reducer for complex state logic
const policyReducer = (state: PolicyState, action: PolicyAction) => {
  switch (action.type) {
    case 'UPDATE_COVERAGE':
      return { ...state, coverage: action.payload };
    case 'CALCULATE_PREMIUM':
      return { ...state, premium: calculatePremium(state) };
    default:
      return state;
  }
};
```

### When to Use Local State
- Toggle states (modals, dropdowns, accordions)
- Form inputs before submission
- UI-only states (hover, focus, animation)
- Component-specific loading states
- Temporary data that doesn't persist

## Shared Application State with React Context

### Context Architecture
```typescript
// contexts/AuthContext.tsx
interface AuthContextType {
  user: User | null;
  permissions: Permission[];
  login: (credentials: LoginCredentials) => Promise<void>;
  logout: () => void;
  isAuthenticated: boolean;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);

export const AuthProvider: React.FC<{ children: ReactNode }> = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);
  
  const login = async (credentials: LoginCredentials) => {
    const response = await api.login(credentials);
    setUser(response.user);
    localStorage.setItem('token', response.token);
  };
  
  return (
    <AuthContext.Provider value={{ user, login, logout, isAuthenticated }}>
      {children}
    </AuthContext.Provider>
  );
};

// Custom hook for using auth context
export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
};
```

### Core Application Contexts

#### Authentication Context
- **Purpose**: User authentication state and methods
- **State**: Current user, permissions, auth tokens
- **Methods**: Login, logout, token refresh
- **Persistence**: Token in localStorage/secure storage

#### Theme Context
```typescript
interface ThemeContextType {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
  clientTheme: ClientTheme;
}
```
- **Purpose**: Application theming and client branding
- **State**: Current theme, client-specific overrides
- **Methods**: Theme toggle, theme selection
- **Persistence**: User preference in localStorage

#### Notification Context
```typescript
interface NotificationContextType {
  notifications: Notification[];
  addNotification: (notification: Notification) => void;
  removeNotification: (id: string) => void;
  clearAll: () => void;
}
```
- **Purpose**: Global notification management
- **State**: Active notifications queue
- **Methods**: Add, remove, clear notifications
- **Features**: Auto-dismiss, priority levels

#### Multi-Tenant Context
```typescript
interface TenantContextType {
  currentTenant: Tenant;
  tenants: Tenant[];
  switchTenant: (tenantId: string) => void;
  tenantConfig: TenantConfiguration;
}
```
- **Purpose**: Multi-tenant client management
- **State**: Current client, available clients
- **Methods**: Client switching, configuration loading
- **Features**: Client-specific settings and permissions

## Server State Management with React Query/TanStack Query

### Query Configuration
```typescript
// queryClient.ts
export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 minutes
      cacheTime: 10 * 60 * 1000, // 10 minutes
      retry: 3,
      refetchOnWindowFocus: false,
    },
  },
});

// App.tsx
<QueryClientProvider client={queryClient}>
  <App />
</QueryClientProvider>
```

### Data Fetching Patterns
```typescript
// hooks/usePolicies.ts
export const usePolicies = (filters?: PolicyFilters) => {
  return useQuery({
    queryKey: ['policies', filters],
    queryFn: () => api.getPolicies(filters),
    select: (data) => data.policies,
  });
};

// Infinite scrolling
export const useInfinitePolicies = () => {
  return useInfiniteQuery({
    queryKey: ['policies', 'infinite'],
    queryFn: ({ pageParam = 0 }) => api.getPolicies({ page: pageParam }),
    getNextPageParam: (lastPage) => lastPage.nextPage,
  });
};
```

### Mutations and Optimistic Updates
```typescript
// hooks/usePolicyMutations.ts
export const useCreatePolicy = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: api.createPolicy,
    onMutate: async (newPolicy) => {
      // Cancel in-flight queries
      await queryClient.cancelQueries(['policies']);
      
      // Snapshot previous value
      const previousPolicies = queryClient.getQueryData(['policies']);
      
      // Optimistically update
      queryClient.setQueryData(['policies'], (old) => [...old, newPolicy]);
      
      return { previousPolicies };
    },
    onError: (err, newPolicy, context) => {
      // Rollback on error
      queryClient.setQueryData(['policies'], context.previousPolicies);
    },
    onSettled: () => {
      // Refetch after mutation
      queryClient.invalidateQueries(['policies']);
    },
  });
};
```

### Cache Management
```typescript
// Prefetching
const prefetchPolicy = async (policyId: string) => {
  await queryClient.prefetchQuery({
    queryKey: ['policy', policyId],
    queryFn: () => api.getPolicy(policyId),
  });
};

// Cache invalidation
const invalidatePolicyData = () => {
  queryClient.invalidateQueries(['policies']);
  queryClient.invalidateQueries(['policy']);
};

// Direct cache updates
const updatePolicyInCache = (policyId: string, updates: Partial<Policy>) => {
  queryClient.setQueryData(['policy', policyId], (old) => ({
    ...old,
    ...updates,
  }));
};
```

### Policy Reinstatement State Management

#### Reinstatement Hooks
```typescript
// Custom hooks for policy reinstatement workflow (GR-64)

// Hook for checking reinstatement eligibility
const useReinstatementEligibility = (policyId: string) => {
  return useQuery({
    queryKey: ['policy', policyId, 'reinstatement-eligibility'],
    queryFn: () => api.checkReinstatementEligibility(policyId),
    refetchInterval: 60000, // Check every minute for time-sensitive eligibility
    enabled: !!policyId,
  });
};

// Hook for reinstatement calculation
const useReinstatementCalculation = (policyId: string, reinstatementDate?: Date) => {
  return useQuery({
    queryKey: ['policy', policyId, 'reinstatement-calculation', reinstatementDate],
    queryFn: () => api.calculateReinstatementAmount(policyId, reinstatementDate),
    enabled: !!policyId && !!reinstatementDate,
    staleTime: 5 * 60 * 1000, // 5 minutes - calculations change with time
  });
};

// Hook for processing reinstatement
const useReinstatementProcess = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: ({ policyId, paymentData }: { policyId: string; paymentData: PaymentData }) =>
      api.processReinstatement(policyId, paymentData),
    onSuccess: (data, variables) => {
      // Update policy status in cache
      queryClient.setQueryData(['policy', variables.policyId], (old: Policy) => ({
        ...old,
        status: 'ACTIVE',
        effective_date: data.new_effective_date,
        reinstatement_date: data.reinstatement_date,
      }));
      
      // Invalidate related queries
      queryClient.invalidateQueries(['policy', variables.policyId, 'reinstatement-eligibility']);
      queryClient.invalidateQueries(['policies']);
    },
    onError: (error) => {
      // Handle reinstatement processing errors
      console.error('Reinstatement processing failed:', error);
    },
  });
};

// Hook for reinstatement eligibility countdown
const useReinstatementCountdown = (policy: Policy) => {
  const [timeRemaining, setTimeRemaining] = useState<{
    days: number;
    hours: number;
    minutes: number;
    expired: boolean;
  }>({ days: 0, hours: 0, minutes: 0, expired: true });

  useEffect(() => {
    if (!policy?.cancelled_at || !policy?.program?.reinstatement_window_days) {
      return;
    }

    const updateCountdown = () => {
      const cancelledAt = new Date(policy.cancelled_at);
      const expirationDate = new Date(cancelledAt);
      expirationDate.setDate(expirationDate.getDate() + policy.program.reinstatement_window_days);
      
      const now = new Date();
      const diff = expirationDate.getTime() - now.getTime();
      
      if (diff <= 0) {
        setTimeRemaining({ days: 0, hours: 0, minutes: 0, expired: true });
        return;
      }
      
      const days = Math.floor(diff / (1000 * 60 * 60 * 24));
      const hours = Math.floor((diff % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60));
      const minutes = Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60));
      
      setTimeRemaining({ days, hours, minutes, expired: false });
    };

    updateCountdown();
    const interval = setInterval(updateCountdown, 60000); // Update every minute

    return () => clearInterval(interval);
  }, [policy?.cancelled_at, policy?.program?.reinstatement_window_days]);

  return timeRemaining;
};
```

#### Reinstatement Form State
```typescript
// Form data for reinstatement process
interface ReinstatementFormData {
  policyId: string;
  reinstatementDate: Date;
  paymentMethod: PaymentMethod;
  paymentAmount: number;
  acknowledgements: {
    noBackdating: boolean;
    feesAccepted: boolean;
    scheduleUpdated: boolean;
  };
}

// Reinstatement form component with state management
const ReinstatementForm: React.FC<{ policy: Policy }> = ({ policy }) => {
  const { register, handleSubmit, watch, setValue, formState: { errors } } = useForm<ReinstatementFormData>({
    defaultValues: {
      policyId: policy.id,
      reinstatementDate: new Date(),
      acknowledgements: {
        noBackdating: false,
        feesAccepted: false,
        scheduleUpdated: false,
      },
    },
    resolver: yupResolver(reinstatementSchema),
  });

  const reinstatementDate = watch('reinstatementDate');
  
  // Get real-time calculation based on selected date
  const { data: calculation, isLoading: calculationLoading } = useReinstatementCalculation(
    policy.id,
    reinstatementDate
  );
  
  // Update payment amount when calculation changes
  useEffect(() => {
    if (calculation?.total_due) {
      setValue('paymentAmount', calculation.total_due);
    }
  }, [calculation, setValue]);

  const reinstatementMutation = useReinstatementProcess();

  const onSubmit = async (data: ReinstatementFormData) => {
    await reinstatementMutation.mutateAsync({
      policyId: data.policyId,
      paymentData: {
        amount: data.paymentAmount,
        method: data.paymentMethod,
        date: data.reinstatementDate,
      },
    });
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* Form implementation */}
    </form>
  );
};
```

## Form State Management

### React Hook Form Integration
```typescript
// components/PolicyForm.tsx
interface PolicyFormData {
  policyNumber: string;
  effectiveDate: Date;
  coverage: Coverage[];
  premium: number;
}

const PolicyForm: React.FC = () => {
  const { register, handleSubmit, watch, formState: { errors } } = useForm<PolicyFormData>({
    defaultValues: {
      effectiveDate: new Date(),
      coverage: [],
    },
    resolver: yupResolver(policySchema),
  });
  
  // Watch specific fields for dependent calculations
  const coverage = watch('coverage');
  
  useEffect(() => {
    // Recalculate premium when coverage changes
    const premium = calculatePremium(coverage);
    setValue('premium', premium);
  }, [coverage]);
  
  const onSubmit = async (data: PolicyFormData) => {
    await createPolicy(data);
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* Form fields */}
    </form>
  );
};
```

### Form State Features
- **Validation**: Schema-based validation with Yup/Zod
- **Field Arrays**: Dynamic form fields for coverages
- **Dependent Fields**: Calculated fields based on other inputs
- **Error Handling**: Field-level and form-level errors
- **Dirty Checking**: Track unsaved changes

## State Persistence

### Local Storage Strategy
```typescript
// utils/storage.ts
const STORAGE_KEYS = {
  AUTH_TOKEN: 'auth_token',
  USER_PREFERENCES: 'user_preferences',
  FORM_DRAFTS: 'form_drafts',
} as const;

export const storage = {
  get: <T>(key: string): T | null => {
    try {
      const item = localStorage.getItem(key);
      return item ? JSON.parse(item) : null;
    } catch {
      return null;
    }
  },
  
  set: <T>(key: string, value: T): void => {
    localStorage.setItem(key, JSON.stringify(value));
  },
  
  remove: (key: string): void => {
    localStorage.removeItem(key);
  },
};
```

### Session Storage for Temporary Data
```typescript
// Temporary form data that shouldn't persist
const saveFormDraft = (formId: string, data: any) => {
  sessionStorage.setItem(`draft_${formId}`, JSON.stringify(data));
};

const loadFormDraft = (formId: string) => {
  const draft = sessionStorage.getItem(`draft_${formId}`);
  return draft ? JSON.parse(draft) : null;
};
```

## Performance Optimization

### Context Optimization
```typescript
// Split contexts to prevent unnecessary renders
const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [permissions, setPermissions] = useState([]);
  
  // Memoize context value
  const authValue = useMemo(
    () => ({ user, permissions, login, logout }),
    [user, permissions]
  );
  
  return (
    <AuthContext.Provider value={authValue}>
      {children}
    </AuthContext.Provider>
  );
};
```

### Selective Re-rendering
```typescript
// Use React.memo for expensive components
const PolicyList = React.memo(({ policies }) => {
  return policies.map(policy => <PolicyCard key={policy.id} policy={policy} />);
}, (prevProps, nextProps) => {
  // Custom comparison logic
  return prevProps.policies.length === nextProps.policies.length;
});
```

## State DevTools

### React Query DevTools
```typescript
// Development only
if (process.env.NODE_ENV === 'development') {
  import('react-query/devtools').then(({ ReactQueryDevtools }) => {
    // Add devtools to app
  });
}
```

### Context DevTools
```typescript
// Custom hook for debugging context
export const useDebugContext = (context: any, name: string) => {
  useEffect(() => {
    console.log(`${name} Context Updated:`, context);
  }, [context, name]);
  
  return context;
};
```

## Best Practices

### State Management Guidelines
1. **Keep state as local as possible**: Only lift state when necessary
2. **Normalize complex state**: Use normalized data structures
3. **Avoid prop drilling**: Use context for deeply nested components
4. **Separate concerns**: UI state vs. server state vs. form state
5. **Optimize renders**: Memoize expensive computations and components

### Anti-Patterns to Avoid
- Storing derived state (calculate on render instead)
- Duplicating server state in local state
- Over-using context for all state
- Not handling loading and error states
- Mutating state directly

## Testing State Management

### Testing Contexts
```typescript
// Test utilities
const renderWithProviders = (ui: ReactElement, options = {}) => {
  const AllProviders = ({ children }) => (
    <QueryClientProvider client={testQueryClient}>
      <AuthProvider>
        <ThemeProvider>
          {children}
        </ThemeProvider>
      </AuthProvider>
    </QueryClientProvider>
  );
  
  return render(ui, { wrapper: AllProviders, ...options });
};
```

### Testing Async State
```typescript
// Testing React Query hooks
test('fetches policies successfully', async () => {
  const { result } = renderHook(() => usePolicies(), {
    wrapper: createWrapper(),
  });
  
  await waitFor(() => {
    expect(result.current.isSuccess).toBe(true);
  });
  
  expect(result.current.data).toHaveLength(10);
});
```

## Cross-References

### Related Global Requirements
- **GR-64**: Policy Reinstatement with Lapse Process - Frontend state management patterns for reinstatement workflow
- **GR-18**: Workflow Requirements - Integration with workflow state transitions
- **GR-20**: Business Logic Standards - API integration for state management hooks
- **GR-37**: Action Tracking - State updates tracking for audit compliance