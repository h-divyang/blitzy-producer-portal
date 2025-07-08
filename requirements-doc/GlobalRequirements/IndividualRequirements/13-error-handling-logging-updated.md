# 13.0 Error Handling & Logging - Updated

## Error Handling & Logging Framework

### Comprehensive Logging Strategy
- **Structured Logging**: JSON-formatted logs with consistent schema
- **Multi-Level Logging**: Debug, info, warning, error, critical levels
- **Centralized Collection**: LGTM stack integration (Loki, Grafana, Tempo, Mimir)
- **Real-Time Monitoring**: Instant alerting for critical errors
- **Compliance Logging**: Audit trails for regulatory requirements

## LGTM Stack Integration

### Loki Log Aggregation
```yaml
# Promtail configuration for log collection
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: laravel-logs
    static_configs:
      - targets:
          - localhost
        labels:
          job: laravel-app
          environment: production
          __path__: /var/log/laravel/*.log
    pipeline_stages:
      - json:
          expressions:
            timestamp: timestamp
            level: level
            message: message
            context: context
            extra: extra
      - timestamp:
          source: timestamp
          format: RFC3339
      - labels:
          level:
          user_id:
          tenant_id:
```

### Grafana Observability Dashboard
```json
{
  "dashboard": {
    "title": "Insurance Application Monitoring",
    "panels": [
      {
        "title": "Error Rate by Service",
        "targets": [
          {
            "expr": "rate(laravel_requests_total{status=~\"5..\"}[5m])",
            "legendFormat": "{{service}} - {{status}}"
          }
        ]
      },
      {
        "title": "Critical Errors",
        "targets": [
          {
            "expr": "{job=\"laravel-app\", level=\"critical\"}",
            "queryType": "logs"
          }
        ]
      },
      {
        "title": "Policy Processing Errors",
        "targets": [
          {
            "expr": "increase(policy_processing_errors_total[1h])"
          }
        ]
      }
    ]
  }
}
```

## Laravel Error Handling

### Custom Exception Handling
```php
// App/Exceptions/Handler.php
class Handler extends ExceptionHandler
{
    protected $dontReport = [
        ValidationException::class,
        AuthenticationException::class,
        ThrottleRequestsException::class,
    ];

    public function report(Throwable $exception): void
    {
        // Enhanced error reporting with context
        if ($this->shouldReport($exception)) {
            $context = $this->getErrorContext($exception);
            
            // Log to application logs
            Log::error($exception->getMessage(), $context);
            
            // Send to error tracking service
            if (app()->environment('production')) {
                $this->reportToErrorTracking($exception, $context);
            }
            
            // Create incident for critical errors
            if ($this->isCriticalError($exception)) {
                $this->createSecurityIncident($exception, $context);
            }
        }

        parent::report($exception);
    }

    public function render($request, Throwable $exception): Response
    {
        // API error responses
        if ($request->expectsJson()) {
            return $this->renderApiError($request, $exception);
        }

        // Web error responses
        return $this->renderWebError($request, $exception);
    }

    private function getErrorContext(Throwable $exception): array
    {
        return [
            'exception_class' => get_class($exception),
            'file' => $exception->getFile(),
            'line' => $exception->getLine(),
            'stack_trace' => $exception->getTraceAsString(),
            'user_id' => auth()->id(),
            'tenant_id' => auth()->user()?->tenant_id,
            'request_id' => request()->header('X-Request-ID'),
            'url' => request()->fullUrl(),
            'method' => request()->method(),
            'ip_address' => request()->ip(),
            'user_agent' => request()->userAgent(),
            'timestamp' => now()->toISOString(),
            'memory_usage' => memory_get_peak_usage(true),
            'environment' => app()->environment()
        ];
    }

    private function renderApiError(Request $request, Throwable $exception): JsonResponse
    {
        $statusCode = $this->getHttpStatusCode($exception);
        $errorCode = $this->getApplicationErrorCode($exception);
        
        $response = [
            'error' => [
                'code' => $errorCode,
                'message' => $this->getUserFriendlyMessage($exception),
                'details' => config('app.debug') ? $exception->getMessage() : null,
                'request_id' => $request->header('X-Request-ID'),
                'timestamp' => now()->toISOString()
            ]
        ];

        // Add validation errors if applicable
        if ($exception instanceof ValidationException) {
            $response['error']['validation_errors'] = $exception->errors();
        }

        return response()->json($response, $statusCode);
    }
}
```

### Business Logic Error Handling
```php
// Custom exception classes for business logic
class PolicyProcessingException extends Exception
{
    protected $policyId;
    protected $errorContext;

    public function __construct(
        string $message,
        int $policyId,
        array $context = [],
        int $code = 0,
        Throwable $previous = null
    ) {
        $this->policyId = $policyId;
        $this->errorContext = $context;
        
        parent::__construct($message, $code, $previous);
    }

    public function getPolicyId(): int
    {
        return $this->policyId;
    }

    public function getErrorContext(): array
    {
        return $this->errorContext;
    }
}

// Usage in service classes
class PolicyService
{
    public function bindPolicy(int $policyId, array $bindingData): Policy
    {
        try {
            DB::beginTransaction();
            
            $policy = Policy::findOrFail($policyId);
            
            // Validate policy can be bound
            if (!$policy->canBind()) {
                throw new PolicyProcessingException(
                    "Policy {$policyId} cannot be bound in current status",
                    $policyId,
                    ['current_status' => $policy->status, 'required_status' => 'quoted']
                );
            }
            
            // Process premium calculation
            $premium = $this->calculatePremium($policy, $bindingData);
            
            if ($premium <= 0) {
                throw new PolicyProcessingException(
                    "Invalid premium calculated for policy {$policyId}",
                    $policyId,
                    ['calculated_premium' => $premium, 'binding_data' => $bindingData]
                );
            }
            
            // Update policy
            $policy->update([
                'status' => PolicyStatus::BOUND,
                'premium' => $premium,
                'bound_at' => now(),
                'bound_by' => auth()->id()
            ]);
            
            // Create audit trail
            PolicyAuditTrail::create([
                'policy_id' => $policy->id,
                'action' => 'policy_bound',
                'old_values' => $policy->getOriginal(),
                'new_values' => $policy->getAttributes(),
                'user_id' => auth()->id()
            ]);
            
            // Dispatch events
            PolicyBound::dispatch($policy);
            
            DB::commit();
            
            // Log successful binding
            Log::info('Policy bound successfully', [
                'policy_id' => $policy->id,
                'premium' => $premium,
                'user_id' => auth()->id(),
                'tenant_id' => $policy->tenant_id
            ]);
            
            return $policy;
            
        } catch (PolicyProcessingException $e) {
            DB::rollBack();
            
            // Log business logic error
            Log::warning('Policy binding failed - business rule violation', [
                'policy_id' => $e->getPolicyId(),
                'error' => $e->getMessage(),
                'context' => $e->getErrorContext(),
                'user_id' => auth()->id()
            ]);
            
            throw $e;
            
        } catch (Exception $e) {
            DB::rollBack();
            
            // Log system error
            Log::error('Policy binding failed - system error', [
                'policy_id' => $policyId,
                'error' => $e->getMessage(),
                'stack_trace' => $e->getTraceAsString(),
                'user_id' => auth()->id()
            ]);
            
            throw new PolicyProcessingException(
                "System error occurred while binding policy {$policyId}",
                $policyId,
                ['original_error' => $e->getMessage()],
                0,
                $e
            );
        }
    }
}
```

## Structured Logging Implementation

### Laravel Logging Configuration
```php
// config/logging.php
return [
    'default' => env('LOG_CHANNEL', 'lgtm'),
    
    'channels' => [
        'lgtm' => [
            'driver' => 'custom',
            'via' => App\Logging\LgtmLogger::class,
            'level' => env('LOG_LEVEL', 'debug'),
        ],
        
        'security' => [
            'driver' => 'single',
            'path' => storage_path('logs/security.log'),
            'level' => 'debug',
            'formatter' => App\Logging\SecurityFormatter::class,
        ],
        
        'audit' => [
            'driver' => 'single',
            'path' => storage_path('logs/audit.log'),
            'level' => 'info',
            'formatter' => App\Logging\AuditFormatter::class,
        ],
        
        'performance' => [
            'driver' => 'single',
            'path' => storage_path('logs/performance.log'),
            'level' => 'info',
            'formatter' => App\Logging\PerformanceFormatter::class,
        ]
    ]
];

// Custom LGTM logger
class LgtmLogger
{
    public function __invoke(array $config): Logger
    {
        $logger = new Logger('lgtm');
        
        // Add Loki handler for log aggregation
        $logger->pushHandler(new LokiHandler(
            config('logging.loki.url'),
            [
                'job' => 'laravel-app',
                'environment' => app()->environment(),
                'service' => 'insurance-api'
            ]
        ));
        
        // Add structured formatter
        $logger->pushProcessor(new StructuredProcessor());
        
        return $logger;
    }
}

// Structured log processor
class StructuredProcessor
{
    public function __invoke(array $record): array
    {
        $record['extra'] = array_merge($record['extra'], [
            'service' => 'insurance-api',
            'version' => config('app.version'),
            'environment' => app()->environment(),
            'hostname' => gethostname(),
            'process_id' => getmypid(),
            'memory_usage' => memory_get_usage(true),
            'request_id' => request()->header('X-Request-ID'),
            'user_id' => auth()->id(),
            'tenant_id' => auth()->user()?->tenant_id,
            'correlation_id' => request()->header('X-Correlation-ID')
        ]);
        
        return $record;
    }
}
```

### Performance Logging
```php
// Performance monitoring middleware
class PerformanceLoggerMiddleware
{
    public function handle(Request $request, Closure $next): Response
    {
        $startTime = microtime(true);
        $startMemory = memory_get_usage(true);
        
        $response = $next($request);
        
        $endTime = microtime(true);
        $endMemory = memory_get_usage(true);
        
        $performanceData = [
            'route' => $request->route()?->getName(),
            'method' => $request->method(),
            'url' => $request->url(),
            'status_code' => $response->getStatusCode(),
            'execution_time_ms' => round(($endTime - $startTime) * 1000, 2),
            'memory_usage_mb' => round(($endMemory - $startMemory) / 1024 / 1024, 2),
            'peak_memory_mb' => round(memory_get_peak_usage(true) / 1024 / 1024, 2),
            'database_queries' => DB::getQueryLog(),
            'cache_hits' => Cache::getHits(),
            'cache_misses' => Cache::getMisses()
        ];
        
        // Log performance metrics
        Log::channel('performance')->info('Request performance', $performanceData);
        
        // Alert on slow requests
        if ($performanceData['execution_time_ms'] > 2000) {
            Log::warning('Slow request detected', $performanceData);
        }
        
        return $response;
    }
}
```

## React Error Handling

### Error Boundaries
```typescript
// Global error boundary
interface ErrorBoundaryState {
  hasError: boolean;
  error?: Error;
  errorInfo?: ErrorInfo;
}

class GlobalErrorBoundary extends Component<
  PropsWithChildren<{}>,
  ErrorBoundaryState
> {
  constructor(props: PropsWithChildren<{}>) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return {
      hasError: true,
      error
    };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    // Log error to monitoring service
    this.logErrorToService(error, errorInfo);
    
    // Update state with error details
    this.setState({
      error,
      errorInfo
    });
  }

  private logErrorToService(error: Error, errorInfo: ErrorInfo) {
    const errorData = {
      message: error.message,
      stack: error.stack,
      componentStack: errorInfo.componentStack,
      url: window.location.href,
      userAgent: navigator.userAgent,
      userId: getCurrentUserId(),
      tenantId: getCurrentTenantId(),
      timestamp: new Date().toISOString(),
      sessionId: getSessionId(),
      buildVersion: process.env.REACT_APP_VERSION
    };

    // Send to logging endpoint
    fetch('/api/client-errors', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${getAuthToken()}`
      },
      body: JSON.stringify({
        type: 'react_error_boundary',
        data: errorData
      })
    }).catch(err => {
      console.error('Failed to log error to service:', err);
    });
  }

  render() {
    if (this.state.hasError) {
      return (
        <ErrorFallback 
          error={this.state.error}
          onRetry={() => this.setState({ hasError: false })}
        />
      );
    }

    return this.props.children;
  }
}

// Error fallback component
interface ErrorFallbackProps {
  error?: Error;
  onRetry: () => void;
}

const ErrorFallback: React.FC<ErrorFallbackProps> = ({ error, onRetry }) => {
  const [isReporting, setIsReporting] = useState(false);

  const reportError = async () => {
    setIsReporting(true);
    try {
      await fetch('/api/error-reports', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${getAuthToken()}`
        },
        body: JSON.stringify({
          error_message: error?.message,
          user_description: 'User reported error from fallback UI',
          url: window.location.href,
          timestamp: new Date().toISOString()
        })
      });
      
      alert('Error report sent successfully');
    } catch (err) {
      console.error('Failed to send error report:', err);
      alert('Failed to send error report');
    } finally {
      setIsReporting(false);
    }
  };

  return (
    <div className="error-boundary-fallback">
      <h2>Something went wrong</h2>
      <p>We're sorry, but an unexpected error occurred.</p>
      
      {process.env.NODE_ENV === 'development' && error && (
        <details>
          <summary>Error details (development only)</summary>
          <pre>{error.message}</pre>
          <pre>{error.stack}</pre>
        </details>
      )}
      
      <div className="error-actions">
        <button onClick={onRetry} className="btn-primary">
          Try Again
        </button>
        <button 
          onClick={reportError} 
          disabled={isReporting}
          className="btn-secondary"
        >
          {isReporting ? 'Reporting...' : 'Report Error'}
        </button>
      </div>
    </div>
  );
};
```

### API Error Handling
```typescript
// Centralized API error handling
class ApiClient {
  private baseURL: string;
  private authToken: string | null = null;

  constructor(baseURL: string) {
    this.baseURL = baseURL;
  }

  async request<T>(
    endpoint: string,
    options: RequestOptions = {}
  ): Promise<ApiResponse<T>> {
    const url = `${this.baseURL}${endpoint}`;
    const requestId = generateRequestId();
    
    const config: RequestInit = {
      method: options.method || 'GET',
      headers: {
        'Content-Type': 'application/json',
        'Accept': 'application/json',
        'X-Request-ID': requestId,
        ...options.headers
      },
      body: options.body ? JSON.stringify(options.body) : undefined
    };

    if (this.authToken) {
      config.headers!['Authorization'] = `Bearer ${this.authToken}`;
    }

    try {
      const startTime = performance.now();
      const response = await fetch(url, config);
      const endTime = performance.now();

      // Log request performance
      this.logRequestPerformance({
        url,
        method: config.method!,
        status: response.status,
        duration: endTime - startTime,
        requestId
      });

      if (!response.ok) {
        return this.handleErrorResponse(response, requestId);
      }

      const data = await response.json();
      return {
        success: true,
        data,
        status: response.status,
        requestId
      };

    } catch (error) {
      // Network or parsing error
      this.logNetworkError(error, url, requestId);
      
      return {
        success: false,
        error: {
          code: 'NETWORK_ERROR',
          message: 'Network error occurred',
          details: error instanceof Error ? error.message : 'Unknown error'
        },
        status: 0,
        requestId
      };
    }
  }

  private async handleErrorResponse(
    response: Response,
    requestId: string
  ): Promise<ApiResponse<never>> {
    try {
      const errorData = await response.json();
      
      // Log API error
      this.logApiError({
        status: response.status,
        url: response.url,
        error: errorData,
        requestId
      });

      return {
        success: false,
        error: errorData.error || {
          code: 'API_ERROR',
          message: 'An error occurred'
        },
        status: response.status,
        requestId
      };
    } catch (parseError) {
      // Failed to parse error response
      return {
        success: false,
        error: {
          code: 'PARSE_ERROR',
          message: 'Failed to parse error response'
        },
        status: response.status,
        requestId
      };
    }
  }

  private logRequestPerformance(data: any) {
    console.log('API Request Performance:', data);
    
    // Send to monitoring if slow
    if (data.duration > 1000) {
      this.sendToMonitoring('slow_api_request', data);
    }
  }

  private logApiError(data: any) {
    console.error('API Error:', data);
    this.sendToMonitoring('api_error', data);
  }

  private logNetworkError(error: any, url: string, requestId: string) {
    const errorData = {
      error: error instanceof Error ? error.message : 'Unknown error',
      url,
      requestId,
      timestamp: new Date().toISOString()
    };
    
    console.error('Network Error:', errorData);
    this.sendToMonitoring('network_error', errorData);
  }

  private sendToMonitoring(type: string, data: any) {
    // Send to monitoring endpoint
    fetch('/api/client-monitoring', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ type, data })
    }).catch(err => {
      console.error('Failed to send monitoring data:', err);
    });
  }
}
```

## Database Error Handling

### Connection and Query Error Handling
```php
// Database connection monitoring
class DatabaseConnectionMonitor
{
    public function handle(QueryExecuted $event): void
    {
        // Log slow queries
        if ($event->time > 1000) {
            Log::warning('Slow database query detected', [
                'sql' => $event->sql,
                'bindings' => $event->bindings,
                'time' => $event->time,
                'connection' => $event->connectionName
            ]);
        }
        
        // Monitor for N+1 queries
        if ($this->isNPlusOneQuery($event)) {
            Log::warning('Potential N+1 query detected', [
                'sql' => $event->sql,
                'bindings' => $event->bindings
            ]);
        }
    }
    
    public function handleConnectionFailure(ConnectionFailed $event): void
    {
        Log::critical('Database connection failed', [
            'connection' => $event->connectionName,
            'exception' => $event->exception->getMessage()
        ]);
        
        // Alert operations team
        $this->sendCriticalAlert([
            'type' => 'database_connection_failure',
            'connection' => $event->connectionName,
            'timestamp' => now()
        ]);
    }
}

// Transaction error handling
trait TransactionSafe
{
    protected function executeInTransaction(callable $callback, array $context = [])
    {
        $attempts = 0;
        $maxAttempts = 3;
        
        while ($attempts < $maxAttempts) {
            try {
                DB::beginTransaction();
                
                $result = $callback();
                
                DB::commit();
                
                // Log successful transaction
                Log::info('Transaction completed successfully', [
                    'context' => $context,
                    'attempts' => $attempts + 1
                ]);
                
                return $result;
                
            } catch (DeadlockException $e) {
                DB::rollBack();
                $attempts++;
                
                Log::warning('Database deadlock detected', [
                    'attempt' => $attempts,
                    'max_attempts' => $maxAttempts,
                    'context' => $context,
                    'error' => $e->getMessage()
                ]);
                
                if ($attempts >= $maxAttempts) {
                    Log::error('Transaction failed after maximum attempts', [
                        'context' => $context,
                        'attempts' => $attempts,
                        'error' => $e->getMessage()
                    ]);
                    
                    throw $e;
                }
                
                // Exponential backoff
                usleep(pow(2, $attempts) * 100000); // 0.1s, 0.2s, 0.4s
                
            } catch (Exception $e) {
                DB::rollBack();
                
                Log::error('Transaction failed', [
                    'context' => $context,
                    'error' => $e->getMessage(),
                    'stack_trace' => $e->getTraceAsString()
                ]);
                
                throw $e;
            }
        }
    }
}
```

## Alerting & Incident Management

### Critical Error Alerting
```php
// Real-time alerting service
class AlertingService
{
    public function sendCriticalAlert(array $alertData): void
    {
        $alert = [
            'severity' => 'critical',
            'service' => 'insurance-api',
            'environment' => app()->environment(),
            'timestamp' => now()->toISOString(),
            'data' => $alertData
        ];
        
        // Send to multiple channels
        $this->sendToSlack($alert);
        $this->sendToEmail($alert);
        $this->sendToPagerDuty($alert);
        
        // Create incident record
        Incident::create([
            'severity' => 'critical',
            'title' => $alertData['title'] ?? 'Critical System Error',
            'description' => json_encode($alertData),
            'status' => 'open',
            'created_by' => 'system'
        ]);
    }
    
    private function sendToSlack(array $alert): void
    {
        Http::post(config('alerting.slack.webhook_url'), [
            'text' => 'Critical Alert: Insurance System',
            'attachments' => [
                [
                    'color' => 'danger',
                    'fields' => [
                        [
                            'title' => 'Severity',
                            'value' => $alert['severity'],
                            'short' => true
                        ],
                        [
                            'title' => 'Service',
                            'value' => $alert['service'],
                            'short' => true
                        ],
                        [
                            'title' => 'Environment',
                            'value' => $alert['environment'],
                            'short' => true
                        ],
                        [
                            'title' => 'Details',
                            'value' => json_encode($alert['data'], JSON_PRETTY_PRINT)
                        ]
                    ]
                ]
            ]
        ]);
    }
}

// Error threshold monitoring
class ErrorThresholdMonitor
{
    public function checkErrorRates(): void
    {
        $timeWindow = 5; // minutes
        $errorThreshold = 50; // errors per time window
        
        $errorCount = Log::where('level', 'error')
            ->where('created_at', '>=', now()->subMinutes($timeWindow))
            ->count();
            
        if ($errorCount > $errorThreshold) {
            $this->triggerErrorRateAlert($errorCount, $timeWindow);
        }
        
        // Check specific error types
        $this->checkCriticalErrors();
        $this->checkSecurityErrors();
        $this->checkPerformanceErrors();
    }
    
    private function triggerErrorRateAlert(int $errorCount, int $timeWindow): void
    {
        app(AlertingService::class)->sendCriticalAlert([
            'title' => 'High Error Rate Detected',
            'error_count' => $errorCount,
            'time_window_minutes' => $timeWindow,
            'threshold_exceeded' => true
        ]);
    }
}
```

## Compliance & Audit Logging

### Regulatory Compliance Logging
```php
// SOX compliance logging
class SoxAuditLogger
{
    public function logFinancialTransaction(array $transactionData): void
    {
        $auditEntry = [
            'event_type' => 'financial_transaction',
            'transaction_id' => $transactionData['id'],
            'amount' => $transactionData['amount'],
            'currency' => $transactionData['currency'],
            'user_id' => auth()->id(),
            'tenant_id' => auth()->user()->tenant_id,
            'ip_address' => request()->ip(),
            'timestamp' => now()->toISOString(),
            'hash' => $this->calculateHash($transactionData)
        ];
        
        // Store in immutable audit log
        DB::table('sox_audit_log')->insert($auditEntry);
        
        // Send to external audit system
        $this->sendToExternalAuditor($auditEntry);
    }
    
    private function calculateHash(array $data): string
    {
        return hash('sha256', json_encode($data) . config('app.audit_salt'));
    }
}

// GDPR compliance logging
class GdprAuditLogger
{
    public function logPersonalDataAccess(string $dataType, int $userId): void
    {
        Log::channel('audit')->info('Personal data accessed', [
            'event_type' => 'personal_data_access',
            'data_type' => $dataType,
            'subject_user_id' => $userId,
            'accessing_user_id' => auth()->id(),
            'legal_basis' => 'legitimate_interest',
            'timestamp' => now()->toISOString(),
            'retention_period' => '7_years'
        ]);
    }
    
    public function logDataProcessing(string $purpose, array $dataCategories): void
    {
        Log::channel('audit')->info('Personal data processing', [
            'event_type' => 'data_processing',
            'purpose' => $purpose,
            'data_categories' => $dataCategories,
            'legal_basis' => 'contract_performance',
            'timestamp' => now()->toISOString()
        ]);
    }
}
```

## Performance & Health Monitoring

### System Health Checks
```php
// Health check endpoint
class HealthCheckController extends Controller
{
    public function check(): JsonResponse
    {
        $checks = [
            'database' => $this->checkDatabase(),
            'redis' => $this->checkRedis(),
            'storage' => $this->checkStorage(),
            'external_apis' => $this->checkExternalApis(),
            'queue' => $this->checkQueue()
        ];
        
        $overallHealth = collect($checks)->every(fn($check) => $check['status'] === 'healthy');
        
        return response()->json([
            'status' => $overallHealth ? 'healthy' : 'unhealthy',
            'timestamp' => now()->toISOString(),
            'checks' => $checks,
            'version' => config('app.version'),
            'environment' => app()->environment()
        ], $overallHealth ? 200 : 503);
    }
    
    private function checkDatabase(): array
    {
        try {
            DB::select('SELECT 1');
            return ['status' => 'healthy', 'response_time_ms' => 0];
        } catch (Exception $e) {
            Log::error('Database health check failed', ['error' => $e->getMessage()]);
            return ['status' => 'unhealthy', 'error' => $e->getMessage()];
        }
    }
}
```