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

## Performance Requirements
- Entity queries: < 500ms for 10,000+ entities
- Communication queries: < 200ms with correlation ID indexing
- Configuration resolution: < 100ms across hierarchy
- Metadata validation: < 50ms per entity type
- DCS API calls: Driver < 5s, Vehicle < 3s, Criminal < 10s
- Circuit breaker: 5 failures trigger protection, 60s timeout

## Compliance Requirements
- Data retention: 7 years for insurance regulatory compliance
- Audit logging: All DCS API interactions with PII masking
- Encryption: All credentials and sensitive data at rest
- Access control: Component-based permissions with role separation
- Privacy: Support for consumer data rights and deletion requests