# IP269 - DCS Integration Specification

## DCS Driver Verification Integration

### Overview
The Primary Insured step leverages the DCS Household Drivers API for real-time driver verification during quote creation. This integration follows the universal entity management pattern (GR 52) and communication architecture (GR 44) established in the comprehensive implementation plan.

### DCS API Integration Points

#### DCS Household Drivers API v2.7
- **Endpoint**: `https://ws.dcsinfosys.com:442/apidevV2.7/DcsSearchApi/HouseholdDrivers`
- **Entity Type**: `DCS_HOUSEHOLD_DRIVERS`
- **Authentication**: HTTP Basic (Account:User:Password)
- **Timeout**: 30 seconds with 3 retry attempts
- **Circuit Breaker**: 5 failures trigger open state

#### Integration Workflow
```php
// Driver verification during search
class PrimaryInsuredVerificationService
{
    public function verifyDriverWithDcs($licenseNumber, $state, $correlationId = null)
    {
        $correlationId = $correlationId ?: 'primary-insured-' . uniqid();
        
        // Get DCS entity configuration
        $dcsEntity = Entity::where('code', 'DCS_PROD_DRIVERS')->firstOrFail();
        
        // Create communication record for tracking
        $communication = Communication::create([
            'source_type' => 'quote',
            'source_id' => $this->quoteId,
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
        
        try {
            // Call DCS API
            $dcsClient = DcsApiClientFactory::createForEntity('DCS_PROD_DRIVERS');
            $response = $dcsClient->verifyDriver([
                'license_number' => $licenseNumber,
                'state' => $state
            ]);
            
            // Update communication with response
            $communication->update([
                'response_data' => $response->toArray(),
                'status_id' => Status::where('code', 'COMPLETED')->first()->id,
                'completed_at' => now()
            ]);
            
            return $this->processDcsResponse($response, $communication);
            
        } catch (DcsApiException $e) {
            // Update communication with error
            $communication->update([
                'response_data' => ['error' => $e->getMessage()],
                'status_id' => Status::where('code', 'FAILED')->first()->id,
                'completed_at' => now()
            ]);
            
            // Log for monitoring
            Log::error('DCS API verification failed', [
                'correlation_id' => $correlationId,
                'license_number' => $licenseNumber,
                'state' => $state,
                'error' => $e->getMessage()
            ]);
            
            // Return graceful degradation
            return DcsVerificationResult::failed($e->getMessage());
        }
    }
    
    private function processDcsResponse($response, $communication)
    {
        if ($response->isSuccessful() && $response->hasDriverData()) {
            // Store verification data for quote
            $verificationData = [
                'dcs_verified' => true,
                'verification_date' => now(),
                'dcs_driver_id' => $response->getDriverId(),
                'household_data' => $response->getHouseholdData(),
                'communication_id' => $communication->id
            ];
            
            return DcsVerificationResult::success($verificationData);
        }
        
        return DcsVerificationResult::notFound();
    }
}
```

### Configuration Management (GR 52)

#### System-Level DCS Configuration
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

#### Program-Level Overrides
```json
{
  "account_number": "PROD12345",
  "environment": "production",
  "enable_criminal_check": true,
  "enable_vehicle_lookup": false,
  "custom_timeout_ms": 25000
}
```

#### Entity-Specific Configuration
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

### Security Implementation (GR 36)

#### Component-Based Security
```php
// Security component for DCS access
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
    
    public function canViewDcsResults($user, $quote)
    {
        return SecurityGroup::hasPermission($user, [
            'component' => 'QUOTE_MANAGEMENT',
            'action' => 'VIEW_VERIFICATION_DATA',
            'scope' => 'quote',
            'scope_id' => $quote->id
        ]);
    }
}
```

#### Credential Management
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

### Performance Monitoring (GR 33)

#### DCS API Performance Metrics
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

#### Caching Strategy
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

### Error Handling & Circuit Breaker

#### Circuit Breaker Implementation
```php
class DcsCircuitBreaker
{
    public function callWithCircuitBreaker($entityCode, callable $apiCall)
    {
        $circuitKey = "circuit_breaker:dcs:{$entityCode}";
        
        // Check if circuit is open
        if ($this->isCircuitOpen($circuitKey)) {
            throw new DcsCircuitOpenException("Circuit breaker is open for {$entityCode}");
        }
        
        try {
            $result = $apiCall();
            $this->recordSuccess($circuitKey);
            return $result;
            
        } catch (Exception $e) {
            $this->recordFailure($circuitKey);
            throw $e;
        }
    }
    
    private function isCircuitOpen($circuitKey)
    {
        $failures = Redis::get($circuitKey) ?? 0;
        return $failures >= 5; // Circuit breaker threshold
    }
    
    private function recordFailure($circuitKey)
    {
        Redis::incr($circuitKey);
        Redis::expire($circuitKey, 300); // 5 minute timeout
    }
    
    private function recordSuccess($circuitKey)
    {
        Redis::del($circuitKey);
    }
}
```

### Audit & Compliance (GR 44)

#### Complete Audit Trail
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

### Testing Patterns

#### Integration Test Example
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
            'correlation_id' => $result->getCorrelationId(),
            'status_id' => Status::where('code', 'COMPLETED')->first()->id
        ]);
        
        // Verify audit logging
        $this->assertDatabaseHas('audit_event', [
            'event_type' => 'DCS_DRIVER_VERIFICATION',
            'correlation_id' => $result->getCorrelationId()
        ]);
    }
    
    public function test_circuit_breaker_prevents_cascade_failures()
    {
        // Simulate multiple failures to trigger circuit breaker
        for ($i = 0; $i < 5; $i++) {
            $this->mockDcsFailure('DCS_HOUSEHOLD_DRIVERS');
            
            $service = new PrimaryInsuredVerificationService();
            
            try {
                $service->verifyDriverWithDcs('TX12345678', 'TX');
            } catch (DcsApiException $e) {
                // Expected failures
            }
        }
        
        // Next call should fail fast with circuit breaker
        $this->expectException(DcsCircuitOpenException::class);
        
        $service = new PrimaryInsuredVerificationService();
        $service->verifyDriverWithDcs('TX12345678', 'TX');
    }
}
```

This integration specification ensures the Primary Insured step fully leverages the DCS integration capabilities while maintaining complete compliance with all global requirements and ProducerPortal-specific patterns.