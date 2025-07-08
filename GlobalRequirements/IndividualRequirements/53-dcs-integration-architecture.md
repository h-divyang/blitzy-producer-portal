# 53.0 DCS Integration Architecture

## Overview
Data Capture Solutions (DCS) provides comprehensive driver, vehicle, and criminal background verification APIs for the insurance industry. This requirement defines the complete integration architecture, authentication patterns, configuration management, and implementation standards for all DCS APIs.

## DCS API Endpoints

### DCS Household Drivers API v2.7
- **Endpoint**: `https://ws.dcsinfosys.com:442/apidevV2.7/DcsSearchApi/HouseholdDrivers`
- **Entity Type**: `DCS_HOUSEHOLD_DRIVERS`
- **Purpose**: Driver verification, license validation, household member discovery
- **Request Format**: XML
- **Response Format**: XML
- **Capabilities**:
  - License validation by number and state
  - Household member discovery
  - Address verification
  - Driver history analysis
  - Alias name discovery

### DCS Household Vehicles API v2.3
- **Endpoint**: `https://ws.dcsinfosys.com:442/apidevV2.3/DcsSearchApi/HouseholdVehicles`
- **Entity Type**: `DCS_HOUSEHOLD_VEHICLES`
- **Purpose**: Vehicle verification, VIN decoding, registration data
- **Request Format**: XML
- **Response Format**: XML
- **Capabilities**:
  - VIN validation and decoding
  - Registration status verification
  - Vehicle history by address
  - Ownership verification
  - Prior registration tracking

### DCS Criminal Background API v1.0
- **Endpoint**: `https://ws.dcsinfosys.com:442/apidevV2.8/DcsSearchApi/Criminal`
- **Entity Type**: `DCS_CRIMINAL`
- **Purpose**: Criminal history search, background verification
- **Request Format**: XML
- **Response Format**: XML
- **Capabilities**:
  - Criminal history by name/DOB
  - Offense details and court records
  - Background verification
  - Alias name criminal search
  - Risk assessment data

## Authentication & Security

### HTTP Basic Authentication
- **Format**: `Account:User:Password` (Base64 encoded)
- **Header**: `Authorization: Basic {base64_encoded_credentials}`
- **Account Structure**: Environment-specific (DEV/UAT/PROD)

### Credential Management
```php
// Vault-based credential retrieval
class DcsCredentialManager
{
    public function getDcsCredentials($programId)
    {
        $program = Program::findOrFail($programId);
        
        // Get credentials from HashiCorp Vault
        $vaultPath = "secret/dcs/{$program->environment}/credentials";
        
        return VaultService::getSecret($vaultPath, [
            'account_number',
            'user_id', 
            'password'
        ]);
    }
}
```

### Component-Based Security
```php
// Security permissions for DCS access
class DcsSecurityComponent
{
    public function canAccessDcsDriverApi($user, $program)
    {
        return SecurityGroup::hasPermission($user, [
            'component' => 'DCS_INTEGRATION',
            'action' => 'DRIVER_VERIFICATION',
            'scope' => 'program',
            'scope_id' => $program->id
        ]);
    }
}
```

## Configuration Management

### Configuration Hierarchy
Configuration follows simple three-level hierarchy with most specific taking precedence:
1. **System Level**: Default settings for all environments
2. **Program Level**: Environment-specific overrides
3. **Entity Level**: API-specific configurations

### System-Level Configuration
```json
{
  "base_url": "https://ws.dcsinfosys.com:442",
  "timeout_ms": 30000,
  "retry_attempts": 3,
  "circuit_breaker_threshold": 5,
  "circuit_breaker_timeout_ms": 60000,
  "cache_ttl_seconds": 300,
  "rate_limit_per_minute": 100
}
```

### Program-Level Overrides
```json
{
  "account_number": "PROD12345",
  "environment": "production",
  "enable_criminal_check": true,
  "enable_vehicle_lookup": false,
  "custom_timeout_ms": 25000
}
```

### Entity-Specific Configuration
```json
{
  "endpoint": "/apidevV2.7/DcsSearchApi/HouseholdDrivers",
  "api_version": "v2.7",
  "capabilities": [
    "driver_verification",
    "household_lookup",
    "criminal_history"
  ],
  "custom_fields": {
    "include_household_members": true,
    "include_address_history": false
  }
}
```

### Configuration Resolution
```php
// Get resolved DCS configuration for specific entity
$dcsDriverConfig = ConfigurationService::resolve(
    'DCS_SETTINGS',
    'entity',
    $dcsDriverEntity->id
);

// Returns merged configuration:
// System defaults + Program overrides + Entity specifics
```

## Entity Type Definitions

### DCS Entity Creation Pattern
```php
// Create DCS Driver entity
$dcsDriverEntity = Entity::create([
    'entity_type_id' => EntityType::where('code', 'DCS_HOUSEHOLD_DRIVERS')->first()->id,
    'code' => 'DCS_PROD_DRIVERS',
    'name' => 'DCS Production Drivers API',
    'metadata' => [
        'provider' => 'Data Capture Solutions',
        'base_url' => 'https://ws.dcsinfosys.com:442',
        'api_version' => 'v2.7',
        'endpoint' => '/apidevV2.7/DcsSearchApi/HouseholdDrivers',
        'auth_type' => 'basic',
        'capabilities' => ['driver_verification', 'household_lookup', 'criminal_history'],
        'timeout_ms' => 30000,
        'retry_attempts' => 3,
        'circuit_breaker_threshold' => 5
    ]
]);
```

### Entity Type JSON Schemas
```sql
-- DCS Household Drivers entity type with complete schema
INSERT INTO entity_type (code, name, category_id, metadata_schema) VALUES 
('DCS_HOUSEHOLD_DRIVERS', 'DCS Household Drivers API', 1, 
'{
  "type": "object",
  "properties": {
    "provider": {"type": "string", "const": "Data Capture Solutions"},
    "base_url": {"type": "string", "format": "uri"},
    "api_version": {"type": "string"},
    "endpoint": {"type": "string"},
    "auth_type": {"type": "string", "enum": ["basic", "oauth2", "api_key"]},
    "capabilities": {"type": "array", "items": {"type": "string"}},
    "timeout_ms": {"type": "number", "default": 30000},
    "retry_attempts": {"type": "number", "default": 3},
    "circuit_breaker_threshold": {"type": "number", "default": 5}
  },
  "required": ["provider", "base_url", "api_version", "endpoint", "auth_type", "capabilities"]
}');
```

## Circuit Breaker & Error Handling

### Circuit Breaker Pattern
```php
class DcsCircuitBreaker
{
    private $redis;
    private $entityCode;
    private $config;
    
    public function __construct($entityCode, $config)
    {
        $this->entityCode = $entityCode;
        $this->config = $config;
        $this->redis = Redis::connection();
    }
    
    public function isOpen()
    {
        $key = "circuit_breaker:dcs:{$this->entityCode}";
        $failures = $this->redis->get($key) ?? 0;
        
        return $failures >= $this->config['circuit_breaker_threshold'];
    }
    
    public function recordFailure()
    {
        $key = "circuit_breaker:dcs:{$this->entityCode}";
        $this->redis->incr($key);
        $this->redis->expire($key, $this->config['circuit_breaker_timeout_ms'] / 1000);
    }
    
    public function recordSuccess()
    {
        $key = "circuit_breaker:dcs:{$this->entityCode}";
        $this->redis->del($key);
    }
}
```

### Error Handling Strategy
- **Graceful Degradation**: Allow manual entry if DCS unavailable
- **Retry Logic**: Exponential backoff with 3 attempts
- **Circuit Breaker**: Opens after 5 consecutive failures
- **Timeout Management**: 30 seconds default, configurable per API
- **Error Classification**: Transient vs permanent failures

## Performance Requirements

### API Response Times (95th percentile)
- **Driver API**: < 5 seconds
- **Vehicle API**: < 3 seconds
- **Criminal API**: < 10 seconds
- **Multi-API workflow**: < 15 seconds total

### Rate Limiting
- **Per Entity**: 100 requests per minute
- **Per Program**: 1000 requests per minute
- **Burst Allowance**: 150% for 30 seconds

### Caching Strategy
```php
class DcsResultCache
{
    public function getCachedDriverData($licenseNumber, $state)
    {
        $cacheKey = "dcs:driver:{$state}:{$licenseNumber}";
        
        return Cache::remember($cacheKey, 300, function() use ($licenseNumber, $state) {
            // Only cache successful, complete responses
            return null; // Force fresh API call for quote creation
        });
    }
    
    public function cacheDriverData($licenseNumber, $state, $data, $ttl = 300)
    {
        $cacheKey = "dcs:driver:{$state}:{$licenseNumber}";
        
        // Only cache if data is complete and valid
        if ($this->isCompleteDriverData($data)) {
            Cache::put($cacheKey, $data, $ttl);
        }
    }
}
```

## Multi-API Workflow Patterns

### Complete Driver Verification Workflow
```php
class DcsDriverVerificationWorkflow
{
    public function verifyDriver($driverId, $includeVehicles = true, $includeCriminal = true)
    {
        $correlationId = 'driver-verification-' . $driverId . '-' . time();
        
        // Step 1: Driver and household lookup
        $driverRequest = $this->buildDriverRequest($driverId);
        $driverResult = $this->callDcsApi('DCS_HOUSEHOLD_DRIVERS', $driverRequest, $correlationId);
        
        $results = ['driver' => $driverResult];
        
        // Step 2: Vehicle information (if requested and driver found)
        if ($includeVehicles && $driverResult->isSuccess()) {
            $vehicleRequest = $this->buildVehicleRequest($driverResult->getAddress());
            $vehicleResult = $this->callDcsApi('DCS_HOUSEHOLD_VEHICLES', $vehicleRequest, $correlationId);
            $results['vehicles'] = $vehicleResult;
        }
        
        // Step 3: Criminal background check (if requested and driver found)
        if ($includeCriminal && $driverResult->isSuccess()) {
            $criminalRequest = $this->buildCriminalRequest($driverResult->getName(), $driverResult->getDob());
            $criminalResult = $this->callDcsApi('DCS_CRIMINAL', $criminalRequest, $correlationId);
            $results['criminal'] = $criminalResult;
        }
        
        return new DcsVerificationResult($results);
    }
}
```

### Workflow Patterns

#### Driver Quote Verification
1. Initial driver check with license number
2. Household discovery at address
3. Vehicle association lookup
4. Criminal screening if enabled
5. Data correlation and risk assessment

#### VIN Verification
1. VIN validation and decoding
2. Registration status check
3. Owner verification cross-reference
4. Historical data retrieval

#### Background Check
1. Primary search by name/DOB
2. Alias name searches
3. Address history checks
4. Risk assessment analysis

## Communication Tracking

### Communication Record Creation
```php
// Create communication record for DCS API call
$communication = Communication::create([
    'source_type' => 'quote',
    'source_id' => $quoteId,
    'target_type' => 'entity',
    'target_id' => $dcsEntity->id,
    'correlation_id' => $correlationId,
    'communication_type_id' => CommunicationType::where('code', 'API_REQUEST')->first()->id,
    'request_data' => [
        'license_number' => $licenseNumber,
        'state' => $state,
        'request_type' => 'driver_verification'
    ],
    'status_id' => Status::where('code', 'PENDING')->first()->id
]);
```

### Correlation ID Management
- **Format**: `{workflow}-{entity_id}-{timestamp}`
- **Purpose**: Link multi-API calls in same workflow
- **Retention**: 7 years for compliance
- **Usage**: Distributed tracing and debugging

## Compliance & Audit

### Data Retention
- **API Responses**: 7 years (insurance regulatory requirement)
- **Audit Logs**: 7 years
- **Communication Records**: 7 years
- **Cache Data**: 5 minutes (configurable)

### Privacy Protection
```php
class DcsAuditLogger
{
    public function logDriverVerification($quoteId, $licenseNumber, $state, $result, $correlationId)
    {
        AuditEvent::create([
            'event_type' => 'DCS_DRIVER_VERIFICATION',
            'source_type' => 'quote',
            'source_id' => $quoteId,
            'correlation_id' => $correlationId,
            'event_data' => [
                'license_number_masked' => $this->maskLicenseNumber($licenseNumber),
                'state' => $state,
                'verification_result' => $result->getStatus(),
                'api_response_size' => strlen(json_encode($result->getRawData())),
                'processing_time_ms' => $result->getProcessingTime()
            ],
            'user_id' => auth()->id(),
            'created_at' => now()
        ]);
    }
    
    private function maskLicenseNumber($licenseNumber)
    {
        // Mask all but last 4 characters for PII protection
        return substr($licenseNumber, 0, 2) . str_repeat('*', strlen($licenseNumber) - 6) . substr($licenseNumber, -4);
    }
}
```

### Compliance Requirements
- **NAIC Model Law**: Full compliance with data retention
- **State Regulations**: Meet all state-specific requirements
- **CCPA/GDPR**: Support right to deletion requests
- **PII Protection**: Automatic masking in logs
- **Encryption**: At-rest encryption for all DCS data

## Testing Patterns

### Integration Testing
```php
class DcsDriverVerificationIntegrationTest extends TestCase
{
    use DcsTestHelpers;
    
    public function test_driver_verification_success_flow()
    {
        // Mock DCS response
        $this->mockDcsResponse('DCS_HOUSEHOLD_DRIVERS', [
            'success' => true,
            'driver_found' => true,
            'driver_data' => $this->validDriverData(),
            'household_data' => $this->validHouseholdData()
        ]);
        
        // Execute verification
        $service = new PrimaryInsuredVerificationService();
        $result = $service->verifyDriverWithDcs('TX12345678', 'TX');
        
        // Verify results
        $this->assertTrue($result->isSuccess());
        $this->assertNotNull($result->getDriverData());
        
        // Verify communication tracking
        $this->assertDatabaseHas('communication', [
            'source_type' => 'quote',
            'target_type' => 'entity',
            'correlation_id' => $result->getCorrelationId()
        ]);
    }
}
```

### Circuit Breaker Testing
```php
public function test_circuit_breaker_prevents_cascade_failures()
{
    // Simulate multiple failures
    for ($i = 0; $i < 5; $i++) {
        $this->mockDcsFailure('DCS_HOUSEHOLD_DRIVERS');
        
        try {
            $service->verifyDriverWithDcs('TX12345678', 'TX');
        } catch (DcsApiException $e) {
            // Expected failures
        }
    }
    
    // Next call should fail fast
    $this->expectException(DcsCircuitOpenException::class);
    $service->verifyDriverWithDcs('TX12345678', 'TX');
}
```

## Monitoring & Alerting

### Performance Metrics
```php
class DcsPerformanceMonitor
{
    public function recordApiCall($entityCode, $duration, $success, $correlationId)
    {
        // Record in time-series database
        Metrics::timing("dcs.api.{$entityCode}.duration", $duration, [
            'success' => $success ? 'true' : 'false',
            'correlation_id' => $correlationId
        ]);
        
        // Update circuit breaker state
        if (!$success) {
            CircuitBreaker::recordFailure($entityCode);
        } else {
            CircuitBreaker::recordSuccess($entityCode);
        }
        
        // Alert on performance degradation
        if ($duration > 5000) { // 5 seconds
            Alert::send("DCS API slow response: {$duration}ms for {$entityCode}");
        }
    }
}
```

### Monitoring Dashboards
- **API Response Times**: Real-time performance tracking
- **Error Rates**: Success/failure ratios by API
- **Circuit Breaker Status**: Open/closed states
- **Rate Limit Usage**: Current consumption levels
- **Cache Hit Rates**: Effectiveness metrics

## Implementation Checklist

### Pre-Implementation
- [ ] Vault credentials configured for environment
- [ ] Entity types created with JSON schemas
- [ ] Security components and permissions configured
- [ ] Circuit breaker thresholds set
- [ ] Cache strategy defined

### Implementation
- [ ] Communication tracking implemented
- [ ] Correlation ID generation in place
- [ ] Error handling with graceful degradation
- [ ] Performance monitoring active
- [ ] Audit logging with PII masking

### Post-Implementation
- [ ] Integration tests passing
- [ ] Circuit breaker tests validated
- [ ] Performance benchmarks met
- [ ] Compliance requirements verified
- [ ] Monitoring dashboards operational

## References
- Universal Entity Management → See GR-52
- Communication Architecture → See GR-44
- External Integrations Catalog → See GR-48
- Data Services & Caching → See GR-33
- Authentication & Security → See GR-36