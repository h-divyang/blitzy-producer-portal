# 08-27.0 Comprehensive Performance & Scalability Requirements - Updated

## Performance Strategy Overview

### Performance-First Architecture
- **Multi-Tier Caching**: Application, database, CDN, and browser caching strategies
- **Auto-Scaling**: Kubernetes HPA/VPA with custom metrics for tenant-aware scaling
- **Performance Monitoring**: Real-time monitoring with LGTM stack and performance budgets
- **Resource Optimization**: Efficient resource utilization across compute, memory, and storage
- **Tenant-Aware Performance**: Per-tenant performance isolation and optimization

## Backend Performance Optimization (Laravel)

### Application-Level Caching
```php
// CacheService.php - Comprehensive caching strategy
class CacheService
{
    const CACHE_TTL_SHORT = 300;    // 5 minutes
    const CACHE_TTL_MEDIUM = 3600;  // 1 hour
    const CACHE_TTL_LONG = 86400;   // 24 hours

    /**
     * Multi-layer cache with Redis and application cache
     */
    public function getCachedData(string $key, callable $callback, int $ttl = self::CACHE_TTL_MEDIUM): mixed
    {
        // Try L1 cache (application memory)
        $l1Key = "l1:{$key}";
        if (Cache::tags(['l1'])->has($l1Key)) {
            return Cache::tags(['l1'])->get($l1Key);
        }

        // Try L2 cache (Redis)
        $l2Key = "l2:{$key}";
        if (Cache::tags(['l2'])->has($l2Key)) {
            $data = Cache::tags(['l2'])->get($l2Key);
            
            // Populate L1 cache with shorter TTL
            Cache::tags(['l1'])->put($l1Key, $data, min($ttl, 300));
            
            return $data;
        }

        // Cache miss - fetch data and populate both layers
        $data = $callback();
        
        // Store in L2 (Redis) with full TTL
        Cache::tags(['l2'])->put($l2Key, $data, $ttl);
        
        // Store in L1 (memory) with shorter TTL
        Cache::tags(['l1'])->put($l1Key, $data, min($ttl, 300));

        return $data;
    }

    /**
     * Tenant-aware cache key generation
     */
    public function getTenantCacheKey(string $key, ?int $tenantId = null): string
    {
        $tenantId = $tenantId ?: auth()->user()?->tenant_id;
        return "tenant:{$tenantId}:{$key}";
    }

    /**
     * Smart cache invalidation with dependencies
     */
    public function invalidateWithDependencies(string $key, array $dependencies = []): void
    {
        // Invalidate main key
        Cache::forget($key);
        Cache::tags(['l1'])->forget("l1:{$key}");
        Cache::tags(['l2'])->forget("l2:{$key}");

        // Invalidate dependent keys
        foreach ($dependencies as $dependency) {
            Cache::forget($dependency);
            Cache::tags(['l1'])->forget("l1:{$dependency}");
            Cache::tags(['l2'])->forget("l2:{$dependency}");
        }
    }
}

// Usage in PolicyService
class PolicyService
{
    private CacheService $cache;

    public function getPolicyWithCoverage(int $policyId): Policy
    {
        $cacheKey = $this->cache->getTenantCacheKey("policy:{$policyId}:with_coverage");
        
        return $this->cache->getCachedData(
            $cacheKey,
            fn() => Policy::with(['coverages', 'tenant', 'creator'])
                         ->findOrFail($policyId),
            CacheService::CACHE_TTL_MEDIUM
        );
    }

    public function updatePolicy(Policy $policy, array $data): Policy
    {
        DB::transaction(function() use ($policy, $data) {
            $policy->update($data);
            
            // Invalidate related caches
            $this->cache->invalidateWithDependencies(
                $this->cache->getTenantCacheKey("policy:{$policy->id}:with_coverage"),
                [
                    $this->cache->getTenantCacheKey("policies:list"),
                    $this->cache->getTenantCacheKey("policies:count"),
                    $this->cache->getTenantCacheKey("dashboard:summary")
                ]
            );
        });

        return $policy;
    }
}
```

### Database Performance Optimization
```php
// Database optimization strategies
class DatabaseOptimizationService
{
    /**
     * Query optimization with proper indexing
     */
    public function getOptimizedPolicyQuery(): Builder
    {
        return Policy::query()
            // Use covering indexes for common queries
            ->select(['id', 'policy_number', 'insured_name', 'status', 'premium', 'effective_date'])
            // Optimize joins with proper indexes
            ->with(['tenant:id,name', 'creator:id,name'])
            // Use query hints for complex queries
            ->fromSub(
                DB::table('policies')
                  ->where('tenant_id', auth()->user()->tenant_id)
                  ->orderBy('created_at', 'desc'),
                'optimized_policies'
            );
    }

    /**
     * Connection pooling and read/write splitting
     */
    public function configureConnectionPooling(): void
    {
        // Read replicas for reporting queries
        config([
            'database.connections.mysql_read' => [
                'driver' => 'mysql',
                'host' => env('DB_READ_HOST', env('DB_HOST')),
                'port' => env('DB_READ_PORT', env('DB_PORT')),
                'database' => env('DB_DATABASE'),
                'username' => env('DB_READ_USERNAME', env('DB_USERNAME')),
                'password' => env('DB_READ_PASSWORD', env('DB_PASSWORD')),
                'options' => [
                    PDO::MYSQL_ATTR_USE_BUFFERED_QUERY => true,
                    PDO::MYSQL_ATTR_INIT_COMMAND => 'SET SESSION sql_mode="STRICT_TRANS_TABLES,NO_ZERO_DATE,NO_ZERO_IN_DATE,ERROR_FOR_DIVISION_BY_ZERO"'
                ]
            ]
        ]);
    }

    /**
     * Query result streaming for large datasets
     */
    public function streamLargeDataset(string $query, array $bindings = []): Generator
    {
        $pdo = DB::connection()->getPdo();
        $statement = $pdo->prepare($query);
        $statement->execute($bindings);

        while ($row = $statement->fetch(PDO::FETCH_ASSOC)) {
            yield $row;
        }
    }
}
```

### Queue Performance Optimization
```php
// Queue optimization for background processing
class QueueOptimizationService
{
    /**
     * Priority-based queue management
     */
    public function dispatchWithPriority(object $job, string $priority = 'normal'): void
    {
        $queue = match($priority) {
            'critical' => 'critical',
            'high' => 'high',
            'normal' => 'default',
            'low' => 'low'
        };

        dispatch($job)->onQueue($queue);
    }

    /**
     * Batch job processing for efficiency
     */
    public function processPolicyBatch(Collection $policies): void
    {
        $chunks = $policies->chunk(100);

        Bus::batch($chunks->map(function ($chunk) {
            return new ProcessPolicyChunk($chunk);
        }))
        ->then(function (Batch $batch) {
            Log::info('Policy batch processed successfully', [
                'batch_id' => $batch->id,
                'processed_jobs' => $batch->processedJobs()
            ]);
        })
        ->catch(function (Batch $batch, Throwable $e) {
            Log::error('Policy batch processing failed', [
                'batch_id' => $batch->id,
                'error' => $e->getMessage()
            ]);
        })
        ->allowFailures()
        ->dispatch();
    }
}
```

## Frontend Performance Optimization (React)

### Code Splitting & Lazy Loading
```typescript
// App.tsx - Optimized routing with code splitting
import { lazy, Suspense } from 'react';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import { ErrorBoundary } from './components/ErrorBoundary';
import { LoadingSpinner } from './components/LoadingSpinner';

// Lazy load components for better performance
const Dashboard = lazy(() => import('./pages/Dashboard'));
const PolicyList = lazy(() => import('./pages/PolicyList'));
const PolicyForm = lazy(() => import('./pages/PolicyForm'));
const ClaimsList = lazy(() => import('./pages/ClaimsList'));
const Reports = lazy(() => import('./pages/Reports'));

// Pre-load critical components
const criticalComponents = [
  import('./pages/Dashboard'),
  import('./pages/PolicyList')
];

function App() {
  return (
    <ErrorBoundary>
      <Router>
        <Suspense fallback={<LoadingSpinner />}>
          <Routes>
            <Route path="/" element={<Dashboard />} />
            <Route path="/policies" element={<PolicyList />} />
            <Route path="/policies/new" element={<PolicyForm />} />
            <Route path="/policies/:id/edit" element={<PolicyForm />} />
            <Route path="/claims" element={<ClaimsList />} />
            <Route path="/reports" element={<Reports />} />
          </Routes>
        </Suspense>
      </Router>
    </ErrorBoundary>
  );
}

export default App;
```

### React Performance Optimization
```typescript
// usePolicyData.ts - Optimized data fetching hook
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { useMemo, useCallback } from 'react';

interface PolicyFilters {
  status?: string;
  effectiveDateFrom?: string;
  effectiveDateTo?: string;
  search?: string;
}

export const usePolicyData = (filters: PolicyFilters = {}) => {
  const queryClient = useQueryClient();

  // Memoize query key to prevent unnecessary refetches
  const queryKey = useMemo(() => 
    ['policies', filters], 
    [filters]
  );

  // Optimized query with stale-while-revalidate
  const {
    data: policies,
    isLoading,
    error,
    fetchNextPage,
    hasNextPage
  } = useQuery({
    queryKey,
    queryFn: ({ pageParam = 1 }) => fetchPolicies({ ...filters, page: pageParam }),
    staleTime: 5 * 60 * 1000, // 5 minutes
    cacheTime: 30 * 60 * 1000, // 30 minutes
    refetchOnWindowFocus: false,
    retry: 3,
    retryDelay: attemptIndex => Math.min(1000 * 2 ** attemptIndex, 30000),
    keepPreviousData: true,
    getNextPageParam: (lastPage) => lastPage.nextPage,
  });

  // Optimistic updates for better UX
  const updatePolicyMutation = useMutation({
    mutationFn: updatePolicy,
    onMutate: async (updatedPolicy) => {
      // Cancel outgoing refetches
      await queryClient.cancelQueries({ queryKey });

      // Snapshot previous value
      const previousPolicies = queryClient.getQueryData(queryKey);

      // Optimistically update cache
      queryClient.setQueryData(queryKey, (old: any) => {
        if (!old?.pages) return old;
        
        return {
          ...old,
          pages: old.pages.map((page: any) => ({
            ...page,
            data: page.data.map((policy: any) =>
              policy.id === updatedPolicy.id ? updatedPolicy : policy
            )
          }))
        };
      });

      return { previousPolicies };
    },
    onError: (err, updatedPolicy, context) => {
      // Rollback on error
      if (context?.previousPolicies) {
        queryClient.setQueryData(queryKey, context.previousPolicies);
      }
    },
    onSettled: () => {
      // Refetch to ensure consistency
      queryClient.invalidateQueries({ queryKey });
    },
  });

  // Prefetch related data
  const prefetchPolicyDetails = useCallback((policyId: number) => {
    queryClient.prefetchQuery({
      queryKey: ['policy', policyId],
      queryFn: () => fetchPolicyDetails(policyId),
      staleTime: 10 * 60 * 1000, // 10 minutes
    });
  }, [queryClient]);

  return {
    policies: policies?.pages?.flatMap(page => page.data) || [],
    isLoading,
    error,
    fetchNextPage,
    hasNextPage,
    updatePolicy: updatePolicyMutation.mutate,
    isUpdating: updatePolicyMutation.isLoading,
    prefetchPolicyDetails,
  };
};
```

### Virtual Scrolling for Large Lists
```typescript
// PolicyVirtualList.tsx - Virtualized list for performance
import { FixedSizeList as List } from 'react-window';
import { memo, useMemo } from 'react';

interface PolicyVirtualListProps {
  policies: Policy[];
  onPolicyClick: (policy: Policy) => void;
}

const PolicyRow = memo(({ index, style, data }: any) => {
  const { policies, onPolicyClick } = data;
  const policy = policies[index];

  return (
    <div style={style} className="policy-row">
      <div className="policy-item" onClick={() => onPolicyClick(policy)}>
        <div className="policy-number">{policy.policyNumber}</div>
        <div className="policy-name">{policy.insuredName}</div>
        <div className="policy-status">{policy.status}</div>
        <div className="policy-premium">${policy.premium.toLocaleString()}</div>
      </div>
    </div>
  );
});

export const PolicyVirtualList: React.FC<PolicyVirtualListProps> = ({
  policies,
  onPolicyClick
}) => {
  // Memoize item data to prevent recreating on every render
  const itemData = useMemo(() => ({
    policies,
    onPolicyClick
  }), [policies, onPolicyClick]);

  return (
    <List
      height={600}
      itemCount={policies.length}
      itemSize={80}
      itemData={itemData}
      overscanCount={5}
    >
      {PolicyRow}
    </List>
  );
};
```

### Image and Asset Optimization
```typescript
// ImageOptimizer.tsx - Optimized image loading
import { useState, useCallback } from 'react';

interface OptimizedImageProps {
  src: string;
  alt: string;
  width?: number;
  height?: number;
  className?: string;
}

export const OptimizedImage: React.FC<OptimizedImageProps> = ({
  src,
  alt,
  width,
  height,
  className
}) => {
  const [isLoaded, setIsLoaded] = useState(false);
  const [error, setError] = useState(false);

  // Generate responsive image URLs
  const generateSrcSet = useCallback((baseSrc: string) => {
    const sizes = [480, 768, 1024, 1200];
    return sizes
      .map(size => `${baseSrc}?w=${size}&q=80 ${size}w`)
      .join(', ');
  }, []);

  const handleLoad = useCallback(() => {
    setIsLoaded(true);
  }, []);

  const handleError = useCallback(() => {
    setError(true);
  }, []);

  if (error) {
    return (
      <div className="image-placeholder error" style={{ width, height }}>
        <span>Failed to load image</span>
      </div>
    );
  }

  return (
    <div className={`image-container ${className}`}>
      {!isLoaded && (
        <div className="image-placeholder loading" style={{ width, height }}>
          <div className="loading-skeleton" />
        </div>
      )}
      <img
        src={src}
        srcSet={generateSrcSet(src)}
        sizes="(max-width: 768px) 100vw, (max-width: 1024px) 50vw, 25vw"
        alt={alt}
        width={width}
        height={height}
        loading="lazy"
        decoding="async"
        onLoad={handleLoad}
        onError={handleError}
        style={{ opacity: isLoaded ? 1 : 0 }}
        className="optimized-image"
      />
    </div>
  );
};
```

## AWS Infrastructure Performance Optimization

### EKS Auto-Scaling Configuration
```yaml
# k8s/hpa.yaml - Horizontal Pod Autoscaler
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: insurance-backend-hpa
  namespace: insurance-production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: insurance-backend
  minReplicas: 3
  maxReplicas: 20
  metrics:
  # CPU-based scaling
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  # Memory-based scaling
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  # Custom metrics for request rate
  - type: Pods
    pods:
      metric:
        name: requests_per_second
      target:
        type: AverageValue
        averageValue: "100"
  # Queue depth for background jobs
  - type: External
    external:
      metric:
        name: redis_queue_depth
        selector:
          matchLabels:
            queue: "default"
      target:
        type: AverageValue
        averageValue: "50"
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 25
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 50
        periodSeconds: 30
      - type: Pods
        value: 2
        periodSeconds: 60
      selectPolicy: Max
---
# Vertical Pod Autoscaler for right-sizing
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: insurance-backend-vpa
  namespace: insurance-production
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: insurance-backend
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
    - containerName: backend
      minAllowed:
        cpu: 100m
        memory: 128Mi
      maxAllowed:
        cpu: 2
        memory: 4Gi
      controlledResources: ["cpu", "memory"]
```

### Tenant-Aware Performance Monitoring
```yaml
# k8s/servicemonitor.yaml - Prometheus monitoring configuration
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: insurance-app-metrics
  namespace: insurance-production
spec:
  selector:
    matchLabels:
      app: insurance-backend
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics
    relabelings:
    - sourceLabels: [__meta_kubernetes_pod_label_tenant]
      targetLabel: tenant_id
    - sourceLabels: [__meta_kubernetes_namespace]
      targetLabel: namespace
---
# PrometheusRule for performance alerts
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: insurance-performance-alerts
  namespace: insurance-production
spec:
  groups:
  - name: performance.rules
    rules:
    # High response time alert
    - alert: HighResponseTime
      expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 2
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "High response time detected"
        description: "95th percentile response time is {{ $value }}s for tenant {{ $labels.tenant_id }}"
    
    # High error rate alert  
    - alert: HighErrorRate
      expr: rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) > 0.05
      for: 2m
      labels:
        severity: critical
      annotations:
        summary: "High error rate detected"
        description: "Error rate is {{ $value | humanizePercentage }} for tenant {{ $labels.tenant_id }}"
    
    # Database connection pool exhaustion
    - alert: DatabaseConnectionPoolHigh
      expr: mysql_connection_pool_active / mysql_connection_pool_max > 0.8
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "Database connection pool utilization high"
        description: "Connection pool is {{ $value | humanizePercentage }} full for tenant {{ $labels.tenant_id }}"
    
    # Redis memory usage
    - alert: RedisMemoryHigh
      expr: redis_memory_used_bytes / redis_memory_max_bytes > 0.85
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "Redis memory usage high"
        description: "Redis memory usage is {{ $value | humanizePercentage }} for tenant {{ $labels.tenant_id }}"
```

## Multi-Tenant Performance Isolation

### Tenant-Aware Resource Management
```php
// TenantPerformanceService.php - Tenant isolation and optimization
class TenantPerformanceService
{
    /**
     * Get tenant-specific performance configuration
     */
    public function getTenantPerformanceConfig(int $tenantId): array
    {
        return Cache::remember(
            "tenant:{$tenantId}:performance_config",
            3600,
            function() use ($tenantId) {
                $tenant = Tenant::find($tenantId);
                
                return [
                    'max_concurrent_requests' => $tenant->subscription_tier === 'enterprise' ? 1000 : 100,
                    'cache_ttl_multiplier' => $tenant->subscription_tier === 'enterprise' ? 2 : 1,
                    'query_timeout' => $tenant->subscription_tier === 'enterprise' ? 30 : 15,
                    'max_export_records' => $tenant->subscription_tier === 'enterprise' ? 100000 : 10000,
                    'priority_queue' => $tenant->subscription_tier === 'enterprise' ? 'high' : 'normal'
                ];
            }
        );
    }

    /**
     * Apply tenant-specific rate limiting
     */
    public function applyTenantRateLimit(Request $request): bool
    {
        $tenantId = auth()->user()->tenant_id;
        $config = $this->getTenantPerformanceConfig($tenantId);
        
        $key = "rate_limit:tenant:{$tenantId}:" . $request->ip();
        $attempts = Cache::increment($key);
        
        if ($attempts === 1) {
            Cache::expire($key, 60); // 1 minute window
        }
        
        return $attempts <= $config['max_concurrent_requests'];
    }

    /**
     * Tenant-aware database connection selection
     */
    public function selectDatabaseConnection(int $tenantId): string
    {
        $tenant = Tenant::find($tenantId);
        
        // Enterprise tenants get dedicated read replicas
        if ($tenant->subscription_tier === 'enterprise') {
            return "mysql_tenant_{$tenantId}_read";
        }
        
        // Standard tenants use shared read replicas
        return 'mysql_read';
    }
}
```

### Kafka Performance Optimization
```yaml
# kafka/performance-config.yaml - Kafka optimization for multi-tenant streaming
apiVersion: v1
kind: ConfigMap
metadata:
  name: kafka-performance-config
  namespace: insurance-production
data:
  server.properties: |
    # Broker performance settings
    num.network.threads=8
    num.io.threads=16
    socket.send.buffer.bytes=102400
    socket.receive.buffer.bytes=102400
    socket.request.max.bytes=104857600
    
    # Log settings for performance
    num.partitions=12
    default.replication.factor=3
    min.insync.replicas=2
    
    # Compression for network efficiency
    compression.type=lz4
    
    # Retention policies
    log.retention.hours=168  # 7 days
    log.retention.bytes=1073741824  # 1GB
    log.segment.bytes=268435456  # 256MB
    
    # Performance tuning
    replica.fetch.max.bytes=1048576
    message.max.bytes=1000000
    replica.socket.timeout.ms=30000
    
    # JVM heap settings for performance
    heap.opts=-Xmx4G -Xms4G
    
    # Tenant isolation through topic naming
    auto.create.topics.enable=false
    delete.topic.enable=true
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: kafka
  namespace: insurance-production
spec:
  serviceName: kafka-headless
  replicas: 3
  selector:
    matchLabels:
      app: kafka
  template:
    metadata:
      labels:
        app: kafka
    spec:
      containers:
      - name: kafka
        image: confluentinc/cp-kafka:7.5.0
        resources:
          requests:
            memory: "4Gi"
            cpu: "1"
          limits:
            memory: "8Gi"
            cpu: "2"
        env:
        - name: KAFKA_HEAP_OPTS
          value: "-Xmx4G -Xms4G"
        - name: KAFKA_JVM_PERFORMANCE_OPTS
          value: "-server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+ExplicitGCInvokesConcurrent -Djava.awt.headless=true"
        volumeMounts:
        - name: kafka-storage
          mountPath: /var/lib/kafka/data
  volumeClaimTemplates:
  - metadata:
      name: kafka-storage
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: gp3-encrypted
      resources:
        requests:
          storage: 500Gi
```

## Performance Monitoring & Analytics

### Real-Time Performance Dashboard
```typescript
// PerformanceDashboard.tsx - Real-time performance monitoring
import { useQuery } from '@tanstack/react-query';
import { Line, Bar } from 'react-chartjs-2';

export const PerformanceDashboard: React.FC = () => {
  // Real-time metrics query
  const { data: metrics } = useQuery({
    queryKey: ['performance-metrics'],
    queryFn: fetchPerformanceMetrics,
    refetchInterval: 5000, // 5 seconds
    staleTime: 1000,
  });

  const responseTimeData = {
    labels: metrics?.timestamps || [],
    datasets: [
      {
        label: 'Average Response Time',
        data: metrics?.responseTime || [],
        borderColor: 'rgb(75, 192, 192)',
        tension: 0.1,
      },
      {
        label: '95th Percentile',
        data: metrics?.responseTime95p || [],
        borderColor: 'rgb(255, 99, 132)',
        tension: 0.1,
      }
    ],
  };

  const throughputData = {
    labels: metrics?.timestamps || [],
    datasets: [
      {
        label: 'Requests/Second',
        data: metrics?.throughput || [],
        backgroundColor: 'rgba(54, 162, 235, 0.2)',
        borderColor: 'rgba(54, 162, 235, 1)',
      }
    ],
  };

  return (
    <div className="performance-dashboard">
      <div className="metrics-grid">
        <div className="metric-card">
          <h3>Response Time</h3>
          <Line 
            data={responseTimeData} 
            options={{
              responsive: true,
              animation: false,
              scales: {
                y: {
                  beginAtZero: true,
                  title: {
                    display: true,
                    text: 'Time (ms)'
                  }
                }
              }
            }}
          />
        </div>
        
        <div className="metric-card">
          <h3>Throughput</h3>
          <Bar 
            data={throughputData}
            options={{
              responsive: true,
              animation: false,
              scales: {
                y: {
                  beginAtZero: true,
                  title: {
                    display: true,
                    text: 'Requests/Second'
                  }
                }
              }
            }}
          />
        </div>
        
        <div className="metric-card">
          <h3>System Health</h3>
          <div className="health-indicators">
            <div className={`indicator ${metrics?.cpu > 80 ? 'critical' : 'healthy'}`}>
              CPU: {metrics?.cpu}%
            </div>
            <div className={`indicator ${metrics?.memory > 85 ? 'critical' : 'healthy'}`}>
              Memory: {metrics?.memory}%
            </div>
            <div className={`indicator ${metrics?.errorRate > 1 ? 'critical' : 'healthy'}`}>
              Error Rate: {metrics?.errorRate}%
            </div>
          </div>
        </div>
      </div>
    </div>
  );
};
```

### Performance Budget Enforcement
```javascript
// performance-budget.js - Performance budget configuration
module.exports = {
  ci: {
    collect: {
      url: ['http://localhost:3000'],
      numberOfRuns: 3,
    },
    assert: {
      assertions: {
        // Performance budgets
        'categories:performance': ['error', { minScore: 0.85 }],
        'categories:accessibility': ['error', { minScore: 0.9 }],
        'categories:best-practices': ['error', { minScore: 0.9 }],
        'categories:seo': ['error', { minScore: 0.8 }],
        
        // Core Web Vitals
        'metrics:largest-contentful-paint': ['error', { maxNumericValue: 2500 }],
        'metrics:first-input-delay': ['error', { maxNumericValue: 100 }],
        'metrics:cumulative-layout-shift': ['error', { maxNumericValue: 0.1 }],
        
        // Other important metrics
        'metrics:first-contentful-paint': ['error', { maxNumericValue: 1800 }],
        'metrics:speed-index': ['error', { maxNumericValue: 3000 }],
        'metrics:interactive': ['error', { maxNumericValue: 3800 }],
        
        // Resource budgets
        'resource-summary:document:size': ['error', { maxNumericValue: 50000 }],
        'resource-summary:script:size': ['error', { maxNumericValue: 500000 }],
        'resource-summary:stylesheet:size': ['error', { maxNumericValue: 100000 }],
        'resource-summary:image:size': ['error', { maxNumericValue: 1000000 }],
        
        // Network requests
        'resource-summary:document:count': ['error', { maxNumericValue: 1 }],
        'resource-summary:script:count': ['error', { maxNumericValue: 10 }],
        'resource-summary:stylesheet:count': ['error', { maxNumericValue: 5 }],
        'resource-summary:third-party:count': ['error', { maxNumericValue: 5 }],
      },
    },
    upload: {
      target: 'lhci',
      serverBaseUrl: 'https://lighthouse-server.insurance.com',
      token: process.env.LHCI_TOKEN,
    },
  },
};
```

This comprehensive performance and scalability framework ensures optimal system performance across all tiers while providing tenant-aware scaling and monitoring capabilities. The implementation supports both current monolithic architecture and future microservices evolution with robust performance monitoring and optimization strategies.