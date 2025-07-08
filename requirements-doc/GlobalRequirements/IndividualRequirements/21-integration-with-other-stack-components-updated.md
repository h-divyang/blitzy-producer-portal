# 21.0 Integration with Other Stack Components - Updated

## Integration Architecture Overview

### Service Integration Layers
1. **API Gateway Layer**: Kong manages external API traffic and authentication
2. **Service Mesh Layer**: Istio handles internal service-to-service communication
3. **Application Layer**: Laravel services integrate with infrastructure components
4. **Data Layer**: Database, cache, and message queue integrations
5. **External Layer**: Third-party service integrations via Apache Camel

## Kubernetes & Container Integration

### EKS Integration
```yaml
# Laravel Deployment Configuration
apiVersion: apps/v1
kind: Deployment
metadata:
  name: policy-service
  namespace: tenant-a
spec:
  replicas: 3
  selector:
    matchLabels:
      app: policy-service
  template:
    metadata:
      labels:
        app: policy-service
        version: v1
      annotations:
        sidecar.istio.io/inject: "true"
    spec:
      containers:
      - name: laravel
        image: ecr.amazonaws.com/insurance/policy-service:latest
        ports:
        - containerPort: 8080
        env:
        - name: DB_CONNECTION
          value: mysql
        - name: DB_HOST
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: host
        - name: REDIS_HOST
          value: redis-service.tenant-a.svc.cluster.local
```

### Service Discovery
- **Kubernetes DNS**: Internal service discovery using cluster DNS
- **Service Names**: Predictable naming convention for services
- **Namespace Isolation**: Tenant-specific service discovery
- **Health Checks**: Readiness and liveness probes for all services

## Database Integration

### AWS RDS MariaDB Connection
```php
// config/database.php
'connections' => [
    'tenant' => [
        'driver' => 'mysql',
        'host' => env('DB_HOST'),
        'port' => env('DB_PORT', '3306'),
        'database' => env('DB_DATABASE'),
        'username' => env('DB_USERNAME'),
        'password' => env('DB_PASSWORD'),
        'charset' => 'utf8mb4',
        'collation' => 'utf8mb4_unicode_ci',
        'options' => [
            PDO::ATTR_SSL_CA => env('MYSQL_ATTR_SSL_CA'),
            PDO::ATTR_SSL_VERIFY_SERVER_CERT => false,
        ],
    ],
],
```

### Database Migration Management
```php
// Multi-tenant migration command
class TenantMigrateCommand extends Command
{
    public function handle()
    {
        $tenants = Tenant::active()->get();
        
        foreach ($tenants as $tenant) {
            $this->info("Migrating tenant: {$tenant->name}");
            
            // Switch to tenant database
            config(['database.connections.tenant.database' => $tenant->database]);
            DB::purge('tenant');
            
            // Run migrations
            $this->call('migrate', [
                '--database' => 'tenant',
                '--path' => 'database/migrations/tenant',
            ]);
        }
    }
}
```

## Redis Caching Integration

### ElastiCache Redis Configuration
```php
// config/cache.php
'redis' => [
    'client' => env('REDIS_CLIENT', 'predis'),
    'cluster' => true,
    'options' => [
        'cluster' => env('REDIS_CLUSTER', 'redis'),
        'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
    ],
    'default' => [
        'url' => env('REDIS_URL'),
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', '6379'),
        'database' => env('REDIS_DB', '0'),
    ],
],
```

### Cache Invalidation Strategy
```php
class PolicyObserver
{
    public function updated(Policy $policy): void
    {
        // Invalidate specific cache entries
        Cache::tags(['policies', "tenant:{$policy->tenant_id}"])
            ->forget("policy:{$policy->id}");
        
        // Invalidate related caches
        Cache::tags(['quotes', "tenant:{$policy->tenant_id}"])
            ->forget("quote:{$policy->quote_id}");
            
        // Update search index
        dispatch(new UpdateSearchIndex($policy));
    }
}
```

## Message Queue Integration

### Laravel Queue with Redis
```php
// config/queue.php
'connections' => [
    'redis' => [
        'driver' => 'redis',
        'connection' => 'queue',
        'queue' => env('REDIS_QUEUE', '{tenant}_default'),
        'retry_after' => 90,
        'block_for' => null,
        'after_commit' => false,
    ],
    'high-priority' => [
        'driver' => 'redis',
        'connection' => 'queue',
        'queue' => '{tenant}_high',
        'retry_after' => 60,
    ],
],
```

### Event Broadcasting
```php
// app/Events/PolicyBound.php
class PolicyBound implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;
    
    public function broadcastOn()
    {
        return [
            new PrivateChannel("tenant.{$this->policy->tenant_id}"),
            new PrivateChannel("policy.{$this->policy->id}"),
        ];
    }
    
    public function broadcastWith()
    {
        return [
            'policy_id' => $this->policy->id,
            'policy_number' => $this->policy->number,
            'status' => 'bound',
            'timestamp' => now()->toIso8601String(),
        ];
    }
}
```

### Future Kafka Integration Preparation
```php
// app/Services/EventPublisher.php
interface EventPublisher
{
    public function publish(string $topic, array $event): void;
}

class KafkaEventPublisher implements EventPublisher
{
    public function publish(string $topic, array $event): void
    {
        // Future Kafka implementation
        $this->producer->send([
            'topic' => $topic,
            'value' => json_encode($event),
            'key' => $event['aggregate_id'] ?? null,
            'headers' => [
                'tenant_id' => $event['tenant_id'],
                'event_type' => $event['type'],
                'timestamp' => time(),
            ],
        ]);
    }
}
```

## API Gateway Integration

### Kong Gateway Integration
```php
// app/Http/Middleware/KongAuthentication.php
class KongAuthentication
{
    public function handle(Request $request, Closure $next)
    {
        // Kong adds these headers after authentication
        $consumerId = $request->header('X-Consumer-ID');
        $consumerName = $request->header('X-Consumer-Username');
        $authenticated = $request->header('X-Authenticated-Userid');
        
        if (!$consumerId || !$authenticated) {
            return response()->json(['error' => 'Unauthorized'], 401);
        }
        
        // Set authenticated user context
        $request->attributes->set('kong_consumer_id', $consumerId);
        $request->attributes->set('authenticated_user', $authenticated);
        
        return $next($request);
    }
}
```

### Rate Limiting Integration
```php
// Kong handles rate limiting, but we can add application-level checks
class ApiRateLimiter
{
    public function handle(Request $request, Closure $next)
    {
        $remaining = $request->header('X-RateLimit-Remaining');
        
        if ($remaining !== null && (int)$remaining <= 10) {
            // Log warning for approaching rate limit
            Log::warning('API rate limit approaching', [
                'consumer' => $request->header('X-Consumer-Username'),
                'remaining' => $remaining,
                'limit' => $request->header('X-RateLimit-Limit'),
            ]);
        }
        
        return $next($request);
    }
}
```

## Service Mesh Integration

### Istio Service Communication
```yaml
# VirtualService for canary deployment
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: policy-service
  namespace: tenant-a
spec:
  hosts:
  - policy-service
  http:
  - match:
    - headers:
        canary:
          exact: "true"
    route:
    - destination:
        host: policy-service
        subset: v2
      weight: 100
  - route:
    - destination:
        host: policy-service
        subset: v1
      weight: 90
    - destination:
        host: policy-service
        subset: v2
      weight: 10
```

### Circuit Breaker Configuration
```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: rating-service
spec:
  host: rating-service
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 100
        http2MaxRequests: 100
    outlierDetection:
      consecutiveErrors: 5
      interval: 30s
      baseEjectionTime: 30s
      maxEjectionPercent: 50
```

## Observability Integration

### Logging with Loki
```php
// config/logging.php
'channels' => [
    'loki' => [
        'driver' => 'custom',
        'via' => App\Logging\LokiLoggerFactory::class,
        'url' => env('LOKI_URL', 'http://loki:3100'),
        'labels' => [
            'app' => env('APP_NAME'),
            'env' => env('APP_ENV'),
            'tenant' => '{tenant_id}',
            'namespace' => env('K8S_NAMESPACE'),
        ],
    ],
],
```

### Metrics with Prometheus
```php
// app/Providers/MetricsServiceProvider.php
class MetricsServiceProvider extends ServiceProvider
{
    public function boot()
    {
        $this->app['prometheus']->registerCollectors([
            new PolicyMetricsCollector(),
            new ClaimMetricsCollector(),
            new PaymentMetricsCollector(),
        ]);
    }
}

class PolicyMetricsCollector implements CollectorInterface
{
    public function collect(): array
    {
        return [
            'policies_created_total' => Policy::whereDate('created_at', today())->count(),
            'policies_bound_total' => Policy::whereDate('bound_at', today())->count(),
            'policies_by_status' => Policy::groupBy('status')->selectRaw('status, count(*) as count')->get(),
        ];
    }
}
```

### Distributed Tracing with Tempo
```php
// app/Http/Middleware/DistributedTracing.php
class DistributedTracing
{
    public function handle(Request $request, Closure $next)
    {
        // Extract trace context from headers
        $traceId = $request->header('X-B3-TraceId') ?? Str::uuid()->toString();
        $spanId = $request->header('X-B3-SpanId') ?? Str::random(16);
        $parentSpanId = $request->header('X-B3-ParentSpanId');
        
        // Set trace context
        app()->instance('trace.context', [
            'trace_id' => $traceId,
            'span_id' => $spanId,
            'parent_span_id' => $parentSpanId,
        ]);
        
        // Add trace headers to response
        $response = $next($request);
        $response->headers->set('X-B3-TraceId', $traceId);
        $response->headers->set('X-B3-SpanId', $spanId);
        
        return $response;
    }
}
```

## Storage Integration

### AWS S3 Integration
```php
// config/filesystems.php
'disks' => [
    's3' => [
        'driver' => 's3',
        'key' => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
        'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
        'bucket' => env('AWS_BUCKET'),
        'url' => env('AWS_URL'),
        'endpoint' => env('AWS_ENDPOINT'),
        'visibility' => 'private',
        'options' => [
            'ServerSideEncryption' => 'AES256',
        ],
    ],
    's3-public' => [
        'driver' => 's3',
        'bucket' => env('AWS_PUBLIC_BUCKET'),
        'visibility' => 'public',
    ],
],
```

### S3 Lifecycle Integration
```php
class DocumentService
{
    public function storeDocument(Document $document, UploadedFile $file): string
    {
        // Determine storage class based on document type
        $storageClass = match($document->type) {
            'active_policy' => 'STANDARD',
            'claim_photo' => 'STANDARD_IA',
            'archived_policy' => 'GLACIER',
            default => 'STANDARD',
        };
        
        // Store with appropriate storage class
        $path = Storage::disk('s3')->putFileAs(
            "tenant/{$document->tenant_id}/documents",
            $file,
            $document->id . '.' . $file->extension(),
            [
                'StorageClass' => $storageClass,
                'Metadata' => [
                    'document-id' => $document->id,
                    'document-type' => $document->type,
                    'tenant-id' => $document->tenant_id,
                ],
            ]
        );
        
        return $path;
    }
}
```

## External Service Integration

### Apache Camel Integration
```php
// app/Services/CamelIntegrationService.php
class CamelIntegrationService
{
    public function sendToPartner(string $route, array $data): mixed
    {
        $response = Http::post("http://camel-service:8080/camel/{$route}", [
            'data' => $data,
            'format' => 'json',
            'tenant_id' => tenant()->id,
        ]);
        
        if ($response->failed()) {
            // Trigger circuit breaker
            throw new ExternalServiceException("Camel route {$route} failed");
        }
        
        return $response->json();
    }
}
```

## Security Integration

### HashiCorp Vault Integration
```php
// app/Services/VaultService.php
class VaultService
{
    private VaultClient $client;
    
    public function getSecret(string $path): array
    {
        $response = $this->client->get("/v1/secret/data/{$path}");
        
        return $response['data']['data'] ?? [];
    }
    
    public function getDatabaseCredentials(string $tenant): array
    {
        // Dynamic database credentials
        $response = $this->client->post("/v1/database/creds/tenant-{$tenant}");
        
        return [
            'username' => $response['data']['username'],
            'password' => $response['data']['password'],
            'ttl' => $response['lease_duration'],
        ];
    }
}
```

### AWS Secrets Manager Integration
```php
// app/Providers/SecretsServiceProvider.php
class SecretsServiceProvider extends ServiceProvider
{
    public function boot()
    {
        // Load secrets at application boot
        $secretsManager = app(SecretsManagerClient::class);
        
        $result = $secretsManager->getSecretValue([
            'SecretId' => 'insurance-app/production',
        ]);
        
        $secrets = json_decode($result['SecretString'], true);
        
        foreach ($secrets as $key => $value) {
            config([$key => $value]);
        }
    }
}
```

## Performance Optimization Integration

### CDN Integration
```php
// app/Services/AssetService.php
class AssetService
{
    public function getAssetUrl(string $path): string
    {
        if (config('app.env') === 'production') {
            // Use CloudFront CDN in production
            return "https://cdn.example.com/{$path}";
        }
        
        return asset($path);
    }
}
```

### Database Query Optimization
```php
// app/Repositories/PolicyRepository.php
class PolicyRepository
{
    public function getActivePoliciessWithRelations(int $tenantId): Collection
    {
        return Policy::query()
            ->where('tenant_id', $tenantId)
            ->where('status', 'active')
            ->with(['insured', 'coverages', 'lastPayment'])
            ->withCount('claims')
            ->remember(3600) // Cache for 1 hour
            ->get();
    }
}
```