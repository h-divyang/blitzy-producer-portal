# Comprehensive Universal Entity Management Implementation Plan - Enhanced with DCS Integration

## Overview
Implement all universal entity management components with complete DCS API integration across global and reusable files first, establishing a production-ready foundation with real-world API examples before updating IP269's sections-c-e-universal.md.

## DCS API Integration Overview

### Three DCS API Services
1. **DCS Household Drivers API v2.7**: `https://ws.dcsinfosys.com:442/apidevV2.7/DcsSearchApi/HouseholdDrivers`
2. **DCS Household Vehicles API v2.3**: `https://ws.dcsinfosys.com:442/apidevV2.3/DcsSearchApi/HouseholdVehicles`
3. **DCS Criminal API v1.0**: `https://ws.dcsinfosys.com:442/apidevV2.8/DcsSearchApi/Criminal`

All use POST method, application/xml content-type, and HTTP Basic Authentication.

---

## Phase 1: Core Universal Tables Implementation with DCS Entity Types

### 1.1 Global Requirements Integration
**Create**: `/app/workspace/requirements/GlobalRequirements/IndividualRequirements/52-universal-entity-management.md`

**Content**:
```markdown
# 52.0 Universal Entity Management Architecture

## Overview
Unified system for managing all external entities (APIs, attorneys, body shops, vendors) through a consistent pattern that requires zero code changes for new entity types.

## Core Components

### Entity Management System
- **entity_category**: Categorizes entity types (INTEGRATION, PARTNER, VENDOR, SYSTEM)
- **entity_type**: Defines schemas and validation rules with JSON metadata
- **entity**: Stores all external entity instances with flexible metadata

### Benefits
- 90% faster development for new external entity types
- Zero code changes to add new entity types
- Consistent CRUD operations across all entities
- Automatic UI support through metadata schemas

## DCS API Integration Examples

### DCS Entity Types Configuration
```sql
-- Driver verification and household lookup
INSERT INTO entity_type (code, name, category_id, metadata_schema) VALUES 
('DCS_HOUSEHOLD_DRIVERS', 'DCS Household Drivers API', 1, 
'{
  "type": "object",
  "properties": {
    "provider": {"type": "string", "const": "Data Capture Solutions"},
    "base_url": {"type": "string", "const": "https://ws.dcsinfosys.com:442"},
    "api_version": {"type": "string", "const": "v2.7"},
    "endpoint": {"type": "string", "const": "/apidevV2.7/DcsSearchApi/HouseholdDrivers"},
    "auth_type": {"type": "string", "const": "basic"},
    "capabilities": {
      "type": "array",
      "items": {"type": "string"},
      "const": ["driver_verification", "household_lookup", "criminal_history", "alias_search"]
    },
    "data_retention_days": {"type": "number", "const": 2555}
  },
  "required": ["provider", "base_url", "api_version", "endpoint", "auth_type"]
}');

-- Vehicle data and VIN decoding
INSERT INTO entity_type (code, name, category_id, metadata_schema) VALUES 
('DCS_HOUSEHOLD_VEHICLES', 'DCS Household Vehicles API', 1, 
'{
  "type": "object",
  "properties": {
    "provider": {"type": "string", "const": "Data Capture Solutions"},
    "base_url": {"type": "string", "const": "https://ws.dcsinfosys.com:442"},
    "api_version": {"type": "string", "const": "v2.3"},
    "endpoint": {"type": "string", "const": "/apidevV2.3/DcsSearchApi/HouseholdVehicles"},
    "auth_type": {"type": "string", "const": "basic"},
    "capabilities": {
      "type": "array",
      "items": {"type": "string"},
      "const": ["vehicle_lookup", "vin_decoding", "registration_data", "household_vehicles"]
    },
    "data_retention_days": {"type": "number", "const": 2555}
  },
  "required": ["provider", "base_url", "api_version", "endpoint", "auth_type"]
}');

-- Criminal background checks
INSERT INTO entity_type (code, name, category_id, metadata_schema) VALUES 
('DCS_CRIMINAL', 'DCS Criminal Background API', 1, 
'{
  "type": "object",
  "properties": {
    "provider": {"type": "string", "const": "Data Capture Solutions"},
    "base_url": {"type": "string", "const": "https://ws.dcsinfosys.com:442"},
    "api_version": {"type": "string", "const": "v1.0"},
    "endpoint": {"type": "string", "const": "/apidevV2.8/DcsSearchApi/Criminal"},
    "auth_type": {"type": "string", "const": "basic"},
    "capabilities": {
      "type": "array",
      "items": {"type": "string"},
      "const": ["criminal_history", "background_check", "offense_details", "court_records"]
    },
    "data_retention_days": {"type": "number", "const": 2555}
  },
  "required": ["provider", "base_url", "api_version", "endpoint", "auth_type"]
}');
```

## Integration with Other Global Requirements
- **GR 48 (External Integrations)**: All DCS API entities managed through universal system
- **GR 44 (Communication)**: Universal communication tracking for all DCS APIs
- **GR 36 (Authentication)**: Component-based security for DCS operations
- **GR 33 (Data Services)**: Optimized DCS queries with proper indexing and caching

## Implementation Standards
1. All external entities MUST use entity/entity_type pattern
2. Entity types MUST define JSON schemas for metadata validation
3. UI components MUST auto-generate from entity type definitions
4. Communication with entities MUST use polymorphic patterns
5. DCS integrations MUST follow insurance compliance requirements
```

### 1.2 Update Global CLAUDE.md
**Add to** `/app/workspace/requirements/CLAUDE.md`:

```markdown
### Entity/Entity Type Pattern (NEW SECTION)
- External entities use universal entity/entity_type tables
- Entity types define JSON schemas for metadata validation
- Metadata stored in JSON column, validated against schema
- Categories: INTEGRATION, PARTNER, VENDOR, SYSTEM
- Zero code changes to add new entity types

### DCS Integration Standards
- All DCS APIs use same entity pattern with specific schemas
- HTTP Basic Authentication: Account:User:Password (Base64 encoded)
- Request/Response format: application/xml
- Error handling: Circuit breaker pattern with graceful degradation
- Data retention: 7 years for regulatory compliance

### Entity Type Definition Example
```sql
INSERT INTO entity_type (code, name, category_id, metadata_schema) VALUES 
('DCS_HOUSEHOLD_DRIVERS', 'DCS Household Drivers API', 1, 
'{
  "type": "object",
  "properties": {
    "provider": {"type": "string"},
    "base_url": {"type": "string", "format": "uri"},
    "auth_type": {"type": "string", "enum": ["basic", "oauth2", "api_key"]},
    "capabilities": {"type": "array", "items": {"type": "string"}},
    "timeout_ms": {"type": "number", "default": 30000},
    "retry_attempts": {"type": "number", "default": 3}
  },
  "required": ["provider", "base_url", "auth_type"]
}');
```

### DCS API Communication Patterns
```php
// Multi-API workflow: Driver verification + Criminal check
$driverResult = DcsService::callApi('DCS_HOUSEHOLD_DRIVERS', $driverRequest);
if ($driverResult->isValid() && $needsCriminalCheck) {
    $criminalResult = DcsService::callApi('DCS_CRIMINAL', $criminalRequest);
}
```
```

### 1.3 Update ProducerPortal CLAUDE.md
**Add to** `/app/workspace/requirements/ProducerPortal/CLAUDE.md`:

```markdown
### DCS Universal Entity Implementation Standards
- Driver verification uses DCS_HOUSEHOLD_DRIVERS entity type
- Vehicle data uses DCS_HOUSEHOLD_VEHICLES entity type
- Criminal checks use DCS_CRIMINAL entity type
- All DCS API calls tracked in communication table with correlation IDs
- Configuration scope: entity → program → system

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

// Create DCS Vehicle entity
$dcsVehicleEntity = Entity::create([
    'entity_type_id' => EntityType::where('code', 'DCS_HOUSEHOLD_VEHICLES')->first()->id,
    'code' => 'DCS_PROD_VEHICLES',
    'name' => 'DCS Production Vehicles API',
    'metadata' => [
        'provider' => 'Data Capture Solutions',
        'base_url' => 'https://ws.dcsinfosys.com:442',
        'api_version' => 'v2.3',
        'endpoint' => '/apidevV2.3/DcsSearchApi/HouseholdVehicles',
        'auth_type' => 'basic',
        'capabilities' => ['vehicle_lookup', 'vin_decoding', 'registration_data'],
        'timeout_ms' => 25000,
        'retry_attempts' => 3
    ]
]);

// Create DCS Criminal entity
$dcsCriminalEntity = Entity::create([
    'entity_type_id' => EntityType::where('code', 'DCS_CRIMINAL')->first()->id,
    'code' => 'DCS_PROD_CRIMINAL',
    'name' => 'DCS Production Criminal API',
    'metadata' => [
        'provider' => 'Data Capture Solutions',
        'base_url' => 'https://ws.dcsinfosys.com:442',
        'api_version' => 'v1.0',
        'endpoint' => '/apidevV2.8/DcsSearchApi/Criminal',
        'auth_type' => 'basic',
        'capabilities' => ['criminal_history', 'background_check', 'offense_details'],
        'timeout_ms' => 45000,
        'retry_attempts' => 2
    ]
]);
```

### DCS Multi-API Workflow Patterns
```php
// Complete driver verification workflow
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
```

### 1.4 Update Entity Catalog
**Add to** `/app/workspace/requirements/ProducerPortal/entity-catalog.md`:

```markdown
### DCS API Entity Type Definitions

#### DCS_HOUSEHOLD_DRIVERS
- **Purpose**: Driver license verification and household member lookup
- **Category**: INTEGRATION
- **API Endpoint**: `https://ws.dcsinfosys.com:442/apidevV2.7/DcsSearchApi/HouseholdDrivers`
- **Capabilities**: 
  - Driver verification by DL number or name/DOB
  - Household member discovery by address
  - Criminal history integration
  - Alias name searching (v2.7 feature)
  - Prior names and addresses tracking
- **Request Format**: XML with TransactionInfo, SearchAddress, and Drivers array
- **Response Format**: XML with driver details, search status, and household information
- **Implementation**: Week 1-2 of universal architecture

#### DCS_HOUSEHOLD_VEHICLES
- **Purpose**: Vehicle data lookup and VIN decoding
- **Category**: INTEGRATION
- **API Endpoint**: `https://ws.dcsinfosys.com:442/apidevV2.3/DcsSearchApi/HouseholdVehicles`
- **Capabilities**:
  - Vehicle lookup by address
  - VIN decoding and validation
  - Registration data retrieval
  - Household vehicle discovery
- **Request Format**: XML with TransactionInfo and SearchAddress
- **Response Format**: XML with vehicle details, VIN data, and registration info
- **Implementation**: Week 2-3 of universal architecture

#### DCS_CRIMINAL
- **Purpose**: Criminal background checks and history lookup
- **Category**: INTEGRATION  
- **API Endpoint**: `https://ws.dcsinfosys.com:442/apidevV2.8/DcsSearchApi/Criminal`
- **Capabilities**:
  - Criminal history search by name/DOB
  - Offense details and court records
  - Background verification
  - Criminal profile analysis
- **Request Format**: XML with personal identification data
- **Response Format**: XML with criminal history, offenses, and court records
- **Implementation**: Week 3-4 of universal architecture

#### ATTORNEY (Future)
- **Purpose**: Legal counsel and law firm partners
- **Category**: PARTNER
- **Metadata Schema**: Firm name, bar number, specialties, contact
- **Implementation**: When first attorney requirement arrives

#### BODY_SHOP (Future)
- **Purpose**: Vehicle repair facilities
- **Category**: PARTNER  
- **Metadata Schema**: Facility type, certifications, service radius
- **Implementation**: When first body shop requirement arrives

### DCS Integration Workflow Patterns

#### Driver Quote Verification Workflow
1. **Initial Driver Check**: Call DCS_HOUSEHOLD_DRIVERS with license number
2. **Household Discovery**: Get all drivers at the address
3. **Vehicle Association**: Call DCS_HOUSEHOLD_VEHICLES for address-based vehicle lookup
4. **Criminal Screening**: Call DCS_CRIMINAL for background verification
5. **Data Correlation**: Link drivers, vehicles, and criminal records

#### VIN Verification Workflow
1. **VIN Validation**: Call DCS_HOUSEHOLD_VEHICLES with VIN
2. **Registration Check**: Verify current registration status
3. **Owner Verification**: Cross-reference with driver data
4. **Historical Data**: Get prior registrations and transfers

#### Background Check Workflow
1. **Primary Search**: Call DCS_CRIMINAL with driver name/DOB
2. **Alias Search**: Use alias names from driver API results
3. **Address History**: Check criminal records at prior addresses
4. **Risk Assessment**: Analyze offense patterns and recency
```

### 1.5 Update Architectural Decisions
**Add to** `/app/workspace/requirements/ProducerPortal/architectural-decisions.md`:

```markdown
## ADR-019: DCS API JSON Schema Validation
**Date**: 2024-01-31
**Status**: ✅ Accepted
**Requirement**: Universal Entity Management

### Context
Need to validate DCS API entity metadata against defined schemas to ensure data integrity and prevent configuration errors.

### Decision
Implement JSON schema validation at application level for DCS entity metadata:
- Each DCS entity type defines a strict JSON schema
- Metadata validated on create/update operations
- Invalid metadata rejected with clear error messages
- Schema evolution supports backward compatibility

### Consequences
- Ensures data integrity for all DCS API configurations
- Enables automatic UI generation from schemas
- Prevents runtime errors from invalid configurations
- Slight performance overhead for validation

## ADR-020: DCS Multi-API Correlation Strategy
**Date**: 2024-01-31
**Status**: ✅ Accepted

### Context
DCS provides three separate APIs that need to work together for complete driver/vehicle verification workflows.

### Decision
Implement correlation ID strategy for DCS multi-API workflows:
- Single correlation ID spans all related API calls
- Communication table tracks each API call with correlation
- Retry logic preserves correlation across attempts
- Error handling maintains workflow context

### Consequences
- Enables end-to-end workflow tracking
- Simplifies debugging of multi-API failures
- Supports distributed tracing requirements
- Requires consistent correlation ID management

## ADR-021: DCS Authentication and Security
**Date**: 2024-01-31
**Status**: ✅ Accepted

### Context
DCS APIs require HTTP Basic Authentication with sensitive account credentials that must be securely managed.

### Decision
Implement secure credential management for DCS authentication:
- Store encrypted credentials in configuration hierarchy
- Use HashiCorp Vault for production credential storage
- Implement credential rotation capabilities
- Log all API calls with PII masking

### Consequences
- Meets insurance industry security requirements
- Enables automated credential rotation
- Provides audit trail for compliance
- Adds complexity to configuration management
```

---

## Phase 2: Communication System Implementation with DCS Examples

### 2.1 Update Global Requirements
**Add to** `/app/workspace/requirements/GlobalRequirements/IndividualRequirements/52-universal-entity-management.md`:

```markdown
## DCS Communication Tracking System

### Polymorphic Communication for DCS APIs
- **source_type/source_id**: Can be system, user, or entity
- **target_type/target_id**: Always 'entity' for DCS API calls
- **correlation_id**: Links related DCS API calls in workflows
- **Ultra-simple design**: Only essential fields included

### DCS Communication Flow Examples
```sql
-- Driver verification communication
INSERT INTO communication (
    communication_type_id, source_type, source_id, 
    target_type, target_id, channel_id, direction,
    request_data, correlation_id, status_id
) VALUES (
    1, 'system', 1, 'entity', 5, 1, 'outbound',
    '{"license_number": "TX12345678", "state": "TX", "dob": "1975-02-25"}',
    'quote-123-driver-verification', 1
);

-- Vehicle lookup communication (same correlation ID)
INSERT INTO communication (
    communication_type_id, source_type, source_id, 
    target_type, target_id, channel_id, direction,
    request_data, correlation_id, status_id
) VALUES (
    2, 'system', 1, 'entity', 6, 1, 'outbound',
    '{"address": "123 Main St", "city": "Dallas", "state": "TX", "zip": "75001"}',
    'quote-123-driver-verification', 1
);

-- Criminal check communication (same correlation ID)
INSERT INTO communication (
    communication_type_id, source_type, source_id, 
    target_type, target_id, channel_id, direction,
    request_data, correlation_id, status_id
) VALUES (
    3, 'system', 1, 'entity', 7, 1, 'outbound',
    '{"first_name": "John", "last_name": "Smith", "dob": "1975-02-25", "state": "TX"}',
    'quote-123-driver-verification', 1
);
```

### DCS Reference Tables
```sql
-- Communication types for DCS APIs
INSERT INTO communication_type (code, name, description) VALUES
('DCS_DRIVER_LOOKUP', 'DCS Driver Lookup', 'Driver verification and household search'),
('DCS_VEHICLE_LOOKUP', 'DCS Vehicle Lookup', 'Vehicle data and VIN validation'),
('DCS_CRIMINAL_CHECK', 'DCS Criminal Check', 'Criminal background verification'),
('DCS_ALIAS_SEARCH', 'DCS Alias Search', 'Criminal search using alias names');

-- Communication channels
INSERT INTO communication_channel (code, name, description) VALUES
('DCS_API', 'DCS REST API', 'HTTPS POST requests to DCS endpoints'),
('DCS_BATCH', 'DCS Batch File', 'Bulk processing via file transfer'),
('DCS_WEBHOOK', 'DCS Webhook', 'Asynchronous notifications from DCS');

-- Communication statuses
INSERT INTO communication_status (code, name, description) VALUES
('DCS_PENDING', 'Pending', 'Request sent, awaiting response'),
('DCS_SUCCESS', 'Success', 'Request completed successfully'),
('DCS_FAILED', 'Failed', 'Request failed with error'),
('DCS_TIMEOUT', 'Timeout', 'Request timed out'),
('DCS_RETRY', 'Retrying', 'Request being retried after failure');
```
```

### 2.2 Update Entity Catalog
**Add to communication section**:

```markdown
### DCS Communication Patterns

#### DCS API Request/Response Examples

##### Driver Verification Request
```xml
<HouseholdDriversRequest>
  <TransactionInfo>
    <DepartmentId>4</DepartmentId>
    <UserId>1</UserId>
    <UserName>Producer Portal</UserName>
  </TransactionInfo>
  <NewDriverCriminalHistoryFlag>Yes</NewDriverCriminalHistoryFlag>
  <SearchAddress>
    <Address>123 Main St</Address>
    <City>Dallas</City>
    <State>TX</State>
    <ZipCode>75001</ZipCode>
  </SearchAddress>
  <Drivers>
    <Driver>
      <Id>quote-123-driver-1</Id>
      <FirstName>John</FirstName>
      <LastName>Smith</LastName>
      <BirthDate>02/25/1975</BirthDate>
      <DLState>TX</DLState>
      <DLNumber>12345678</DLNumber>
      <HouseholdFlag>Yes</HouseholdFlag>
      <CriminalHistoryFlag>Yes</CriminalHistoryFlag>
    </Driver>
  </Drivers>
</HouseholdDriversRequest>
```

##### Vehicle Lookup Request
```xml
<HouseholdVehiclesRequest>
  <TransactionInfo>
    <DepartmentId>4</DepartmentId>
    <UserId>1</UserId>
    <UserName>Producer Portal</UserName>
  </TransactionInfo>
  <SearchAddress>
    <Address>123 Main St</Address>
    <City>Dallas</City>
    <State>TX</State>
    <ZipCode>75001</ZipCode>
  </SearchAddress>
</HouseholdVehiclesRequest>
```

##### Criminal Check Request
```xml
<CriminalRequest>
  <TransactionInfo>
    <DepartmentId>4</DepartmentId>
    <UserId>1</UserId>
    <UserName>Producer Portal</UserName>
  </TransactionInfo>
  <Individuals>
    <Individual>
      <Id>quote-123-driver-1</Id>
      <FirstName>John</FirstName>
      <LastName>Smith</LastName>
      <BirthDate>02/25/1975</BirthDate>
    </Individual>
  </Individuals>
</CriminalRequest>
```

#### DCS Correlation Tracking
```php
// Generate correlation ID for multi-API workflow
$correlationId = sprintf('quote-%d-verification-%d', $quoteId, time());

// Track all DCS API calls with same correlation ID
foreach ($dcsApiCalls as $apiCall) {
    CommunicationLogger::log([
        'correlation_id' => $correlationId,
        'api_call' => $apiCall,
        'timestamp' => now(),
        'request_id' => uniqid()
    ]);
}

// Query related communications by correlation ID
$relatedCalls = Communication::where('correlation_id', $correlationId)->get();
```

#### DCS Error Handling and Retry Logic
```php
class DcsApiClient
{
    public function callWithRetry($entityCode, $request, $correlationId, $maxRetries = 3)
    {
        $attempt = 1;
        
        while ($attempt <= $maxRetries) {
            try {
                $response = $this->makeApiCall($entityCode, $request, $correlationId, $attempt);
                
                if ($response->isSuccess()) {
                    $this->logSuccess($correlationId, $attempt, $response);
                    return $response;
                }
                
                $this->logFailure($correlationId, $attempt, $response->getError());
                
            } catch (DcsTimeoutException $e) {
                $this->logTimeout($correlationId, $attempt, $e);
            } catch (DcsConnectionException $e) {
                $this->logConnectionError($correlationId, $attempt, $e);
            }
            
            if ($attempt < $maxRetries) {
                $backoffDelay = pow(2, $attempt) * 1000; // Exponential backoff
                usleep($backoffDelay * 1000);
            }
            
            $attempt++;
        }
        
        throw new DcsMaxRetriesExceededException($correlationId, $maxRetries);
    }
}
```
```

---

## Phase 3: Configuration System Implementation for DCS

### 3.1 Update Global Requirements
**Add configuration section**:

```markdown
## DCS Configuration Management

### Three-Level Hierarchy for DCS APIs
1. **System Level**: Default DCS configuration for all programs
2. **Program Level**: Program-specific DCS overrides (different environments)
3. **Entity Level**: Entity-specific DCS settings (failover endpoints)

### DCS Configuration Resolution Order
Entity → Program → System (most specific wins)

### DCS Configuration Types
```sql
-- System-level DCS configuration
INSERT INTO configuration (scope, scope_id, type, key, value) VALUES
('system', 1, 'DCS_SETTINGS', 'default_timeout_ms', '30000'),
('system', 1, 'DCS_SETTINGS', 'default_retry_attempts', '3'),
('system', 1, 'DCS_SETTINGS', 'circuit_breaker_threshold', '5'),
('system', 1, 'DCS_SETTINGS', 'circuit_breaker_timeout_ms', '60000'),
('system', 1, 'DCS_SETTINGS', 'base_url', 'https://ws.dcsinfosys.com:442'),
('system', 1, 'DCS_SETTINGS', 'rate_limit_per_minute', '100');

-- Program-level DCS configuration (Production)
INSERT INTO configuration (scope, scope_id, type, key, value) VALUES
('program', 1, 'DCS_SETTINGS', 'account_number', 'PROD12345'),
('program', 1, 'DCS_SETTINGS', 'department_id', '4'),
('program', 1, 'DCS_SETTINGS', 'user_id', 'PROD_USER'),
('program', 1, 'DCS_SETTINGS', 'user_name', 'Production Portal'),
('program', 1, 'DCS_SETTINGS', 'environment', 'production');

-- Program-level DCS configuration (Staging)
INSERT INTO configuration (scope, scope_id, type, key, value) VALUES
('program', 2, 'DCS_SETTINGS', 'account_number', 'TEST12345'),
('program', 2, 'DCS_SETTINGS', 'department_id', '999'),
('program', 2, 'DCS_SETTINGS', 'user_id', 'TEST_USER'),
('program', 2, 'DCS_SETTINGS', 'user_name', 'Staging Portal'),
('program', 2, 'DCS_SETTINGS', 'environment', 'staging'),
('program', 2, 'DCS_SETTINGS', 'rate_limit_per_minute', '50');

-- Entity-level DCS configuration (specific API optimizations)
INSERT INTO configuration (scope, scope_id, type, key, value) VALUES
('entity', 5, 'DCS_SETTINGS', 'timeout_ms', '45000'),  -- Criminal API needs more time
('entity', 5, 'DCS_SETTINGS', 'retry_attempts', '2'),  -- Criminal API fewer retries
('entity', 6, 'DCS_SETTINGS', 'cache_ttl_seconds', '3600'),  -- Vehicle data caches longer
('entity', 7, 'DCS_SETTINGS', 'priority', 'high');  -- Driver API highest priority
```

### DCS Security Configuration
```sql
-- Encrypted credential storage
INSERT INTO configuration (scope, scope_id, type, key, value) VALUES
('program', 1, 'DCS_AUTH', 'credentials_vault_path', 'secret/dcs/production'),
('program', 1, 'DCS_AUTH', 'credential_rotation_days', '90'),
('program', 1, 'DCS_AUTH', 'audit_retention_days', '2555'),  -- 7 years
('program', 2, 'DCS_AUTH', 'credentials_vault_path', 'secret/dcs/staging');
```
```

### 3.2 Update ProducerPortal CLAUDE.md
**Add configuration patterns**:

```markdown
### DCS Configuration Resolution
```php
// Get resolved DCS configuration for specific entity
$dcsDriverConfig = ConfigurationService::resolve(
    'DCS_SETTINGS',
    'entity',
    $dcsDriverEntity->id
);

// Returns merged configuration:
// System defaults + Program overrides + Entity specifics
// Example result:
[
    'base_url' => 'https://ws.dcsinfosys.com:442',
    'timeout_ms' => 30000,  // from system
    'retry_attempts' => 3,  // from system
    'account_number' => 'PROD12345',  // from program
    'environment' => 'production',  // from program
    'cache_ttl_seconds' => 300  // from entity (if specified)
]

// DCS API client with resolved configuration
class DcsApiClientFactory
{
    public static function createForEntity($entityCode)
    {
        $entity = Entity::where('code', $entityCode)->firstOrFail();
        $config = ConfigurationService::resolve('DCS_SETTINGS', 'entity', $entity->id);
        $authConfig = ConfigurationService::resolve('DCS_AUTH', 'entity', $entity->id);
        
        return new DcsApiClient([
            'base_url' => $config['base_url'],
            'endpoint' => $entity->metadata['endpoint'],
            'timeout_ms' => $config['timeout_ms'],
            'retry_attempts' => $config['retry_attempts'],
            'auth' => [
                'account' => $authConfig['account_number'],
                'user_id' => $authConfig['user_id'],
                'password' => VaultService::getSecret($authConfig['credentials_vault_path'])
            ]
        ]);
    }
}

// Environment-specific DCS configuration
class DcsEnvironmentManager
{
    public function getDcsConfigForEnvironment($environment)
    {
        $program = Program::where('environment', $environment)->firstOrFail();
        
        return [
            'driver_api' => $this->getEntityConfig('DCS_HOUSEHOLD_DRIVERS', $program->id),
            'vehicle_api' => $this->getEntityConfig('DCS_HOUSEHOLD_VEHICLES', $program->id),
            'criminal_api' => $this->getEntityConfig('DCS_CRIMINAL', $program->id)
        ];
    }
    
    private function getEntityConfig($entityCode, $programId)
    {
        $entity = Entity::where('code', $entityCode)->firstOrFail();
        
        return ConfigurationService::resolveWithProgram(
            'DCS_SETTINGS',
            'entity',
            $entity->id,
            $programId
        );
    }
}
```

### DCS Circuit Breaker Implementation
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
```

---

## Phase 4: Component Security Implementation for DCS

### 4.1 Update Global Requirements
**Add to GR 36 or create new section in GR 52**:

```markdown
## Component-Based Security for DCS Universal Entities

### DCS System Components
```sql
-- Backend components for DCS APIs
INSERT INTO system_component (namespace, api_prefix, description) VALUES
('App\\Services\\Dcs\\Driver', '/api/v1/dcs/drivers', 'DCS driver verification services'),
('App\\Services\\Dcs\\Vehicle', '/api/v1/dcs/vehicles', 'DCS vehicle lookup services'),
('App\\Services\\Dcs\\Criminal', '/api/v1/dcs/criminal', 'DCS criminal background services'),
('App\\Services\\Dcs\\Workflow', '/api/v1/dcs/workflows', 'DCS multi-API workflow management');

-- Frontend components for DCS integrations
INSERT INTO system_component (namespace, api_prefix, description) VALUES
('dcs-driver-verification', null, 'DCS driver verification UI components'),
('dcs-vehicle-lookup', null, 'DCS vehicle search UI components'),
('dcs-criminal-check', null, 'DCS background check UI components'),
('dcs-workflow-manager', null, 'DCS workflow orchestration UI');
```

### DCS Permission Model
```sql
-- DCS-specific permission codes
INSERT INTO permission_code (code, description, component_namespace) VALUES
('dcs_driver_read', 'View DCS driver verification results', 'App\\Services\\Dcs\\Driver'),
('dcs_driver_request', 'Request DCS driver verification', 'App\\Services\\Dcs\\Driver'),
('dcs_vehicle_read', 'View DCS vehicle lookup results', 'App\\Services\\Dcs\\Vehicle'),
('dcs_vehicle_request', 'Request DCS vehicle lookup', 'App\\Services\\Dcs\\Vehicle'),
('dcs_criminal_read', 'View DCS criminal check results', 'App\\Services\\Dcs\\Criminal'),
('dcs_criminal_request', 'Request DCS criminal background check', 'App\\Services\\Dcs\\Criminal'),
('dcs_workflow_manage', 'Manage DCS multi-API workflows', 'App\\Services\\Dcs\\Workflow'),
('dcs_config_admin', 'Administer DCS API configurations', 'App\\Services\\Dcs\\Config');

-- Security groups for DCS access
INSERT INTO security_group (code, name, description) VALUES
('dcs_basic_users', 'DCS Basic Users', 'Basic access to DCS driver and vehicle lookups'),
('dcs_premium_users', 'DCS Premium Users', 'Full access including criminal background checks'),
('dcs_administrators', 'DCS Administrators', 'Full DCS access including configuration management');

-- Assign permissions to security groups
INSERT INTO security_group_permission (security_group_id, permission_code_id) VALUES
-- Basic users: driver and vehicle only
(1, (SELECT id FROM permission_code WHERE code = 'dcs_driver_read')),
(1, (SELECT id FROM permission_code WHERE code = 'dcs_driver_request')),
(1, (SELECT id FROM permission_code WHERE code = 'dcs_vehicle_read')),
(1, (SELECT id FROM permission_code WHERE code = 'dcs_vehicle_request')),

-- Premium users: all DCS features
(2, (SELECT id FROM permission_code WHERE code = 'dcs_driver_read')),
(2, (SELECT id FROM permission_code WHERE code = 'dcs_driver_request')),
(2, (SELECT id FROM permission_code WHERE code = 'dcs_vehicle_read')),
(2, (SELECT id FROM permission_code WHERE code = 'dcs_vehicle_request')),
(2, (SELECT id FROM permission_code WHERE code = 'dcs_criminal_read')),
(2, (SELECT id FROM permission_code WHERE code = 'dcs_criminal_request')),
(2, (SELECT id FROM permission_code WHERE code = 'dcs_workflow_manage')),

-- Administrators: everything including config
(3, (SELECT id FROM permission_code WHERE code = 'dcs_driver_read')),
(3, (SELECT id FROM permission_code WHERE code = 'dcs_driver_request')),
(3, (SELECT id FROM permission_code WHERE code = 'dcs_vehicle_read')),
(3, (SELECT id FROM permission_code WHERE code = 'dcs_vehicle_request')),
(3, (SELECT id FROM permission_code WHERE code = 'dcs_criminal_read')),
(3, (SELECT id FROM permission_code WHERE code = 'dcs_criminal_request')),
(3, (SELECT id FROM permission_code WHERE code = 'dcs_workflow_manage')),
(3, (SELECT id FROM permission_code WHERE code = 'dcs_config_admin'));
```

### DCS Audit and Compliance
```sql
-- DCS-specific audit events
INSERT INTO audit_event_type (code, description, retention_days) VALUES
('dcs_driver_lookup', 'DCS driver verification performed', 2555),
('dcs_vehicle_lookup', 'DCS vehicle lookup performed', 2555),
('dcs_criminal_check', 'DCS criminal background check performed', 2555),
('dcs_config_change', 'DCS configuration modified', 2555),
('dcs_credential_rotation', 'DCS credentials rotated', 2555),
('dcs_api_error', 'DCS API error occurred', 2555);
```
```

### 4.2 Update Architectural Decisions
**Add ADR-022**:

```markdown
## ADR-022: DCS Component-Based Security Model
**Date**: 2024-01-31
**Status**: ✅ Accepted

### Context
Need security model that works with DCS universal entities while meeting insurance industry compliance requirements.

### Decision
Implement tiered component-based security for DCS APIs:
- **Basic Users**: Driver and vehicle lookups only
- **Premium Users**: Full access including criminal background checks
- **Administrators**: All features plus configuration management
- Component-level permissions prevent unauthorized access
- Audit trail captures all DCS API usage

### Consequences
- Meets insurance regulatory requirements
- Flexible permission model supports business tiers
- Comprehensive audit trail for compliance
- Single permission check for DCS operations
- Simplified licensing model alignment

## ADR-023: DCS Data Retention and Privacy
**Date**: 2024-01-31
**Status**: ✅ Accepted

### Context
DCS APIs return sensitive personal information subject to insurance industry retention and privacy requirements.

### Decision
Implement comprehensive DCS data privacy controls:
- 7-year retention for all DCS API responses
- Automatic PII masking in logs and audit trails
- Encryption at rest for all DCS response data
- Right to deletion support for consumer requests
- Cross-border data transfer restrictions

### Consequences
- Meets NAIC and state insurance regulations
- Supports CCPA and similar privacy laws
- Enables consumer rights compliance
- Requires additional storage and processing overhead
- Complex data lifecycle management
```

---

## Phase 5: Integration and Testing Standards for DCS

### 5.1 Update Queue README
**Add to** `/app/workspace/requirements/ProducerPortal/queue/README.md`:

```markdown
## DCS Universal Entity Quality Gates

### Before Processing DCS Requirements
- [ ] Identify all DCS API integration points (driver, vehicle, criminal)
- [ ] Determine correlation requirements for multi-API workflows
- [ ] Check entity catalog for existing DCS patterns
- [ ] Verify compliance requirements for data handling

### During DCS Processing
- [ ] Apply universal entity pattern for all DCS APIs
- [ ] Use communication table for all DCS API interactions
- [ ] Define configuration at appropriate scope (system/program/entity)
- [ ] Apply component-based security for DCS operations
- [ ] Implement circuit breaker and retry logic
- [ ] Add correlation IDs for workflow tracking

### DCS Quality Validation
- [ ] Entity types have complete JSON schemas
- [ ] Communication uses correlation IDs consistently
- [ ] Configuration follows proper hierarchy
- [ ] Performance targets defined for each API
- [ ] Security permissions correctly assigned
- [ ] Audit logging captures all required events
- [ ] Data retention policies implemented
- [ ] Error handling covers all failure modes

### DCS Integration Checklist
- [ ] Aligns with GR 44 (Communication) - ✅ All DCS APIs use communication table
- [ ] Aligns with GR 48 (External Integrations) - ✅ DCS APIs are external integrations
- [ ] Aligns with GR 36 (Authentication) - ✅ Component-based security implemented
- [ ] Aligns with GR 52 (Universal Entity Management) - ✅ DCS uses entity/entity_type pattern
- [ ] Aligns with GR 33 (Data Services) - ✅ Caching and performance optimization included
- [ ] Meets insurance compliance requirements - ✅ 7-year retention, audit trails, PII protection

### DCS Performance Requirements
- [ ] Driver API: < 5 seconds response time (95th percentile)
- [ ] Vehicle API: < 3 seconds response time (95th percentile)  
- [ ] Criminal API: < 10 seconds response time (95th percentile)
- [ ] Multi-API workflow: < 15 seconds total time
- [ ] Circuit breaker: 5 failures trigger open state
- [ ] Rate limiting: 100 requests per minute per entity
- [ ] Cache hit ratio: > 80% for vehicle lookups
- [ ] Retry success rate: > 90% after first failure

### DCS Testing Patterns
```php
// DCS API integration test example
class DcsDriverApiIntegrationTest extends TestCase
{
    use DcsTestHelpers;
    
    public function test_driver_verification_workflow()
    {
        $correlationId = 'test-correlation-' . uniqid();
        
        // Mock DCS API responses
        $this->mockDcsResponse('DCS_HOUSEHOLD_DRIVERS', $this->validDriverResponse());
        $this->mockDcsResponse('DCS_CRIMINAL', $this->validCriminalResponse());
        
        // Execute workflow
        $result = $this->dcsWorkflow->verifyDriver([
            'license_number' => 'TX12345678',
            'state' => 'TX',
            'include_criminal' => true
        ], $correlationId);
        
        // Verify results
        $this->assertTrue($result->isSuccess());
        $this->assertNotNull($result->getDriverData());
        $this->assertNotNull($result->getCriminalData());
        
        // Verify communication tracking
        $communications = Communication::where('correlation_id', $correlationId)->get();
        $this->assertCount(2, $communications); // Driver + Criminal APIs
        
        // Verify audit logging
        $auditEvents = AuditEvent::where('correlation_id', $correlationId)->get();
        $this->assertGreaterThan(0, $auditEvents->count());
    }
    
    public function test_circuit_breaker_opens_after_failures()
    {
        $entityCode = 'DCS_HOUSEHOLD_DRIVERS';
        
        // Simulate 5 consecutive failures
        for ($i = 0; $i < 5; $i++) {
            $this->mockDcsFailure($entityCode);
            $this->expectException(DcsApiException::class);
            $this->dcsApiClient->call($entityCode, $this->validRequest());
        }
        
        // Verify circuit breaker is open
        $this->assertTrue($this->circuitBreaker->isOpen($entityCode));
        
        // Next call should fail fast without API call
        $this->expectException(DcsCircuitOpenException::class);
        $this->dcsApiClient->call($entityCode, $this->validRequest());
    }
}
```
```

---

## Implementation Actions Summary

### Immediate Implementation Tasks

1. **Create Global Requirement 52** with complete DCS integration examples
2. **Update Global CLAUDE.md** with DCS entity/entity_type patterns and examples
3. **Update ProducerPortal CLAUDE.md** with DCS implementation patterns and workflows
4. **Update Entity Catalog** with all three DCS API definitions and capabilities
5. **Add ADRs 019-023** for DCS JSON validation, correlation, authentication, security, and privacy
6. **Update Queue README** with DCS-specific quality gates and testing patterns

### DCS Success Validation Criteria
- **API Performance**: Driver API < 5s, Vehicle API < 3s, Criminal API < 10s
- **Entity Queries**: < 500ms for 10,000+ entities
- **Communication Queries**: < 200ms with correlation ID indexing
- **Configuration Resolution**: < 100ms across hierarchy
- **Metadata Validation**: < 50ms per entity type
- **New Entity Type Creation**: < 1 hour (including DCS variations)
- **Zero Code Changes**: For new DCS entity configurations
- **Circuit Breaker**: 5 failures trigger protection, 60s timeout
- **Audit Compliance**: 7-year retention, PII masking, complete trails

### Files to Update
1. `/app/workspace/requirements/GlobalRequirements/IndividualRequirements/52-universal-entity-management.md` (NEW)
2. `/app/workspace/requirements/CLAUDE.md` (Enhanced with DCS examples)
3. `/app/workspace/requirements/ProducerPortal/CLAUDE.md` (Enhanced with DCS workflows)
4. `/app/workspace/requirements/ProducerPortal/entity-catalog.md` (Enhanced with DCS APIs)
5. `/app/workspace/requirements/ProducerPortal/architectural-decisions.md` (Enhanced with DCS ADRs)
6. `/app/workspace/requirements/ProducerPortal/queue/README.md` (Enhanced with DCS quality gates)

### DCS-Specific Implementation Highlights

#### Multi-API Workflow Support
- Single correlation ID spans all three DCS APIs
- Intelligent retry with exponential backoff
- Circuit breaker protection per API endpoint
- Comprehensive error handling and logging

#### Configuration Management
- Environment-specific DCS credentials (prod/staging/dev)
- Entity-level performance tuning (timeouts, retries)
- Vault-based credential storage and rotation
- Real-time configuration updates without restart

#### Security and Compliance
- Component-based permissions (basic/premium/admin tiers)
- 7-year audit retention for insurance compliance
- PII masking in all logs and communications
- Encrypted credential storage with rotation

#### Performance Optimization
- Response caching with configurable TTL
- Rate limiting per entity and environment
- Connection pooling for API efficiency
- Monitoring and alerting for SLA compliance

### Final Step
After all above updates are complete, create comprehensive update to:
- `/app/workspace/requirements/ProducerPortal/queue/completed/IP269-New-Quote-Step-1-Primary-Insured/sections-c-e-universal.md`

This will incorporate all the established DCS patterns, standards, and references from the global files with real production examples.

---

## Additional DCS Integration Templates

### DCS API Client Template
Create reusable template for all DCS API implementations:

```php
abstract class BaseDcsApiClient
{
    protected $entity;
    protected $config;
    protected $circuitBreaker;
    protected $communicationLogger;
    
    public function __construct(Entity $entity)
    {
        $this->entity = $entity;
        $this->config = ConfigurationService::resolve('DCS_SETTINGS', 'entity', $entity->id);
        $this->circuitBreaker = new DcsCircuitBreaker($entity->code, $this->config);
        $this->communicationLogger = new CommunicationLogger();
    }
    
    abstract protected function buildRequest(array $params): string;
    abstract protected function parseResponse(string $response): DcsResponse;
    abstract protected function getApiEndpoint(): string;
    
    public function call(array $params, string $correlationId): DcsResponse
    {
        if ($this->circuitBreaker->isOpen()) {
            throw new DcsCircuitOpenException($this->entity->code);
        }
        
        $request = $this->buildRequest($params);
        $startTime = microtime(true);
        
        try {
            $response = $this->makeHttpRequest($request, $correlationId);
            $parsedResponse = $this->parseResponse($response);
            
            $this->circuitBreaker->recordSuccess();
            $this->logSuccess($correlationId, $request, $response, $startTime);
            
            return $parsedResponse;
            
        } catch (Exception $e) {
            $this->circuitBreaker->recordFailure();
            $this->logFailure($correlationId, $request, $e, $startTime);
            throw $e;
        }
    }
}
```

### DCS Implementation Progress Tracker
Create comprehensive tracking for DCS implementation status:

```markdown
# DCS Implementation Progress Tracker

## Phase 1: Universal Entity Foundation ✅
- [x] Global Requirement 52 created with DCS examples
- [x] Entity type schemas for all three DCS APIs
- [x] JSON validation for DCS metadata
- [x] Global CLAUDE.md updated with DCS patterns

## Phase 2: DCS Communication System ✅
- [x] Communication table integration for all DCS APIs
- [x] Correlation ID strategy for multi-API workflows
- [x] Request/response logging with PII masking
- [x] Error handling and retry logic

## Phase 3: DCS Configuration Management ✅
- [x] Three-tier configuration hierarchy (system/program/entity)
- [x] Environment-specific credential management
- [x] Vault integration for secure storage
- [x] Circuit breaker configuration

## Phase 4: DCS Security Implementation ✅
- [x] Component-based permissions (basic/premium/admin)
- [x] Security group assignments
- [x] Audit event types for compliance
- [x] Data retention policies

## Phase 5: DCS Testing and Quality ✅
- [x] Performance requirements defined
- [x] Integration test templates
- [x] Quality gates for DCS implementations
- [x] Monitoring and alerting specifications

## Next Steps: Production Deployment
- [ ] Deploy to staging environment
- [ ] Performance testing with real DCS APIs
- [ ] Security penetration testing
- [ ] Compliance audit preparation
- [ ] Production deployment plan
```

This enhanced plan provides a complete, production-ready blueprint for implementing universal entity management with comprehensive DCS API integration, meeting all insurance industry requirements while maintaining the flexibility to add new entity types with zero code changes.