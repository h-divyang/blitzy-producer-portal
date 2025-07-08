# Integration Platform Database Schema

## Overview
This schema supports the enterprise integration platform requirements, including hierarchical permission controls, versioned node mapping, and comprehensive audit logging.

## Core Integration Management Tables

### third_party_integration
```sql
CREATE TABLE third_party_integration (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT,
  provider_name VARCHAR(100) NOT NULL,
  api_version VARCHAR(20),
  base_url VARCHAR(255),
  documentation_url VARCHAR(255) NULL,
  is_active BOOLEAN DEFAULT TRUE,
  
  -- Configuration metadata
  auth_type ENUM('api_key', 'oauth2', 'basic', 'bearer') NOT NULL,
  requires_sandbox BOOLEAN DEFAULT FALSE,
  default_timeout_seconds INT DEFAULT 30,
  default_retry_attempts INT DEFAULT 3,
  
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  INDEX idx_status (status_id),
  INDEX idx_active (is_active),
  UNIQUE KEY unique_code (code)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Sample data
INSERT INTO third_party_integration (code, name, description, provider_name, api_version, auth_type, status_id, created_by) VALUES
('DCS_HOUSEHOLD_DRIVERS', 'DCS Household Drivers API', 'Driver verification and household member lookup', 'Data Capture Solutions', '2.7', 'oauth2', 1, 1),
('DCS_CRIMINAL', 'DCS Criminal API', 'Criminal background verification', 'Data Capture Solutions', '1.0', 'oauth2', 1, 1),
('DCS_HOUSEHOLD_VEHICLES', 'DCS Household Vehicles API', 'Vehicle information and verification', 'Data Capture Solutions', '2.3', 'oauth2', 1, 1);
```

### integration_configuration
```sql
CREATE TABLE integration_configuration (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  integration_id BIGINT UNSIGNED NOT NULL,
  
  -- Hierarchical configuration (system -> program -> producer)
  configuration_level ENUM('system', 'program', 'producer') NOT NULL,
  program_id BIGINT UNSIGNED NULL,
  producer_id BIGINT UNSIGNED NULL,
  
  -- Environment settings
  environment ENUM('sandbox', 'production') NOT NULL DEFAULT 'sandbox',
  endpoint_url VARCHAR(255) NOT NULL,
  
  -- Authentication (encrypted)
  auth_config JSON NOT NULL, -- Contains encrypted credentials
  
  -- Feature flags
  is_enabled BOOLEAN DEFAULT FALSE,
  is_required BOOLEAN DEFAULT FALSE,
  
  -- Performance settings
  timeout_seconds INT DEFAULT 30,
  retry_attempts INT DEFAULT 3,
  cache_ttl_hours INT DEFAULT 24,
  
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (integration_id) REFERENCES third_party_integration(id),
  FOREIGN KEY (program_id) REFERENCES program(id),
  FOREIGN KEY (producer_id) REFERENCES producer(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  INDEX idx_status (status_id),
  INDEX idx_integration (integration_id),
  INDEX idx_program (program_id),
  INDEX idx_producer (producer_id),
  INDEX idx_level (configuration_level),
  INDEX idx_enabled (is_enabled),
  UNIQUE KEY unique_integration_scope (integration_id, configuration_level, program_id, producer_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### integration_node
```sql
CREATE TABLE integration_node (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  integration_id BIGINT UNSIGNED NOT NULL,
  
  -- Node identification
  node_path VARCHAR(255) NOT NULL, -- e.g., "driver_data.personal_info.first_name"
  node_name VARCHAR(100) NOT NULL,
  node_description TEXT,
  
  -- Data type information
  data_type ENUM('string', 'integer', 'decimal', 'boolean', 'date', 'datetime', 'array', 'object') NOT NULL,
  is_required BOOLEAN DEFAULT FALSE,
  is_array BOOLEAN DEFAULT FALSE,
  
  -- Sample data for reference
  sample_value TEXT NULL,
  
  -- Versioning
  api_version VARCHAR(20) NOT NULL,
  deprecated_in_version VARCHAR(20) NULL,
  
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (integration_id) REFERENCES third_party_integration(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  INDEX idx_status (status_id),
  INDEX idx_integration (integration_id),
  INDEX idx_node_path (node_path),
  INDEX idx_api_version (api_version),
  UNIQUE KEY unique_integration_node_version (integration_id, node_path, api_version)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### integration_field_mapping
```sql
CREATE TABLE integration_field_mapping (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  integration_id BIGINT UNSIGNED NOT NULL,
  
  -- Scope (system-wide or program-specific)
  mapping_scope ENUM('system', 'program', 'producer') NOT NULL DEFAULT 'system',
  program_id BIGINT UNSIGNED NULL,
  producer_id BIGINT UNSIGNED NULL,
  
  -- Source (API node) to target (internal field) mapping
  source_node_id BIGINT UNSIGNED NOT NULL,
  target_table VARCHAR(100) NOT NULL,
  target_column VARCHAR(100) NOT NULL,
  
  -- Transformation rules (JSON for flexibility)
  transformation_rules JSON NULL, -- e.g., {"uppercase": true, "trim": true, "default": "N/A"}
  
  -- Versioning
  version_number INT NOT NULL DEFAULT 1,
  is_active_version BOOLEAN DEFAULT TRUE,
  replaced_mapping_id BIGINT UNSIGNED NULL, -- Points to previous version
  
  -- Validation rules
  validation_rules JSON NULL, -- e.g., {"required": true, "max_length": 50, "pattern": "^[A-Z0-9]+$"}
  
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (integration_id) REFERENCES third_party_integration(id),
  FOREIGN KEY (source_node_id) REFERENCES integration_node(id),
  FOREIGN KEY (program_id) REFERENCES program(id),
  FOREIGN KEY (producer_id) REFERENCES producer(id),
  FOREIGN KEY (replaced_mapping_id) REFERENCES integration_field_mapping(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  INDEX idx_status (status_id),
  INDEX idx_integration (integration_id),
  INDEX idx_source_node (source_node_id),
  INDEX idx_target (target_table, target_column),
  INDEX idx_scope (mapping_scope, program_id, producer_id),
  INDEX idx_version (version_number, is_active_version),
  UNIQUE KEY unique_active_mapping (integration_id, source_node_id, target_table, target_column, mapping_scope, program_id, producer_id, is_active_version)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

## Communication & Audit Tables

### integration_request (extends existing communication pattern)
```sql
CREATE TABLE integration_request (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  
  -- Link to integration configuration
  integration_id BIGINT UNSIGNED NOT NULL,
  configuration_id BIGINT UNSIGNED NOT NULL,
  
  -- Request context
  quote_id BIGINT UNSIGNED NULL,
  driver_id BIGINT UNSIGNED NULL,
  user_id BIGINT UNSIGNED NOT NULL,
  
  -- Request details
  endpoint VARCHAR(255) NOT NULL,
  http_method VARCHAR(10) NOT NULL DEFAULT 'POST',
  request_headers JSON,
  request_body JSON,
  
  -- Response details
  response_status_code INT NULL,
  response_headers JSON NULL,
  response_body JSON NULL,
  response_time_ms INT NULL,
  
  -- Processing status
  request_status ENUM('pending', 'sent', 'completed', 'failed', 'timeout') NOT NULL DEFAULT 'pending',
  error_message TEXT NULL,
  retry_count INT DEFAULT 0,
  
  -- Transaction tracking
  external_transaction_id VARCHAR(100) NULL,
  correlation_id VARCHAR(100) NOT NULL, -- For tracing across services
  
  -- PII protection flags
  contains_pii BOOLEAN DEFAULT TRUE,
  pii_masked BOOLEAN DEFAULT FALSE,
  retention_until DATE NULL, -- Automatic cleanup date
  
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (integration_id) REFERENCES third_party_integration(id),
  FOREIGN KEY (configuration_id) REFERENCES integration_configuration(id),
  FOREIGN KEY (quote_id) REFERENCES quote(id),
  FOREIGN KEY (driver_id) REFERENCES driver(id),
  FOREIGN KEY (user_id) REFERENCES user(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  INDEX idx_status (status_id),
  INDEX idx_integration (integration_id),
  INDEX idx_configuration (configuration_id),
  INDEX idx_quote (quote_id),
  INDEX idx_driver (driver_id),
  INDEX idx_user (user_id),
  INDEX idx_request_status (request_status),
  INDEX idx_correlation (correlation_id),
  INDEX idx_external_transaction (external_transaction_id),
  INDEX idx_retention (retention_until),
  INDEX idx_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### integration_verification_result
```sql
CREATE TABLE integration_verification_result (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  
  -- Link to the request that generated this result
  request_id BIGINT UNSIGNED NOT NULL,
  
  -- What was verified
  verification_type ENUM('license', 'address', 'identity', 'vehicle', 'household') NOT NULL,
  target_entity_type ENUM('driver', 'vehicle', 'address') NOT NULL,
  target_entity_id BIGINT UNSIGNED NOT NULL,
  
  -- Verification results
  verification_status ENUM('verified', 'partial', 'failed', 'not_found', 'error') NOT NULL,
  confidence_score DECIMAL(5,2) NULL, -- 0.00 to 100.00
  
  -- Verified data (structured)
  verified_data JSON NULL,
  discrepancies JSON NULL, -- What didn't match
  
  -- Caching information
  cache_expires_at TIMESTAMP NULL,
  is_cached_result BOOLEAN DEFAULT FALSE,
  
  -- External references
  external_verification_id VARCHAR(100) NULL,
  
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (request_id) REFERENCES integration_request(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  INDEX idx_status (status_id),
  INDEX idx_request (request_id),
  INDEX idx_verification_type (verification_type),
  INDEX idx_target_entity (target_entity_type, target_entity_id),
  INDEX idx_verification_status (verification_status),
  INDEX idx_cache_expires (cache_expires_at),
  INDEX idx_external_verification (external_verification_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

## Enhanced Entity Tables

### driver (Enhanced for integration tracking)
```sql
-- Add columns to existing driver table
ALTER TABLE driver ADD COLUMN IF NOT EXISTS (
  external_verification_status ENUM('not_verified', 'verified', 'partial', 'failed') DEFAULT 'not_verified',
  last_verification_date TIMESTAMP NULL,
  verification_source VARCHAR(100) NULL, -- e.g., 'DCS_HOUSEHOLD_DRIVERS'
  verification_confidence DECIMAL(5,2) NULL,
  external_driver_id VARCHAR(100) NULL -- Reference to external system
);

-- Add indexes for new columns
ALTER TABLE driver ADD INDEX idx_verification_status (external_verification_status);
ALTER TABLE driver ADD INDEX idx_last_verification (last_verification_date);
ALTER TABLE driver ADD INDEX idx_external_id (external_driver_id);
```

### license (Enhanced for integration tracking)
```sql
-- Add columns to existing license table
ALTER TABLE license ADD COLUMN IF NOT EXISTS (
  external_verification_status ENUM('not_verified', 'verified', 'invalid', 'expired', 'suspended') DEFAULT 'not_verified',
  last_verification_date TIMESTAMP NULL,
  verification_source VARCHAR(100) NULL,
  external_license_status VARCHAR(50) NULL, -- Raw status from external system
  verification_notes TEXT NULL
);

-- Add indexes for new columns
ALTER TABLE license ADD INDEX idx_external_verification (external_verification_status);
ALTER TABLE license ADD INDEX idx_last_verification (last_verification_date);
```

### address (Enhanced for integration tracking)
```sql
-- Add columns to existing address table
ALTER TABLE address ADD COLUMN IF NOT EXISTS (
  standardized_by_source VARCHAR(100) NULL, -- e.g., 'DCS_ADDRESS_VERIFY'
  standardization_confidence DECIMAL(5,2) NULL,
  delivery_point_validation ENUM('valid', 'invalid', 'unknown') DEFAULT 'unknown',
  last_standardization_date TIMESTAMP NULL
);

-- Add indexes for new columns
ALTER TABLE address ADD INDEX idx_standardized_by (standardized_by_source);
ALTER TABLE address ADD INDEX idx_delivery_validation (delivery_point_validation);
```

## Permission and Access Control

### integration_permission
```sql
CREATE TABLE integration_permission (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  
  -- What integration this permission applies to
  integration_id BIGINT UNSIGNED NOT NULL,
  
  -- Who has permission (user, role, or permission group)
  permission_type ENUM('user', 'role', 'group') NOT NULL,
  permission_target_id BIGINT UNSIGNED NOT NULL, -- user_id, role_id, or group_id
  
  -- What actions are allowed
  can_configure BOOLEAN DEFAULT FALSE,
  can_view_raw_responses BOOLEAN DEFAULT FALSE,
  can_manage_mappings BOOLEAN DEFAULT FALSE,
  can_enable_disable BOOLEAN DEFAULT FALSE,
  
  -- Scope limitations
  scope_level ENUM('system', 'program', 'producer') NOT NULL DEFAULT 'producer',
  program_id BIGINT UNSIGNED NULL,
  producer_id BIGINT UNSIGNED NULL,
  
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (integration_id) REFERENCES third_party_integration(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  INDEX idx_status (status_id),
  INDEX idx_integration (integration_id),
  INDEX idx_permission_target (permission_type, permission_target_id),
  INDEX idx_scope (scope_level, program_id, producer_id),
  UNIQUE KEY unique_permission (integration_id, permission_type, permission_target_id, scope_level, program_id, producer_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

## Sample Data Population

### DCS Integration Nodes (Sample)
```sql
-- Sample integration nodes for DCS Household Drivers API
INSERT INTO integration_node (integration_id, node_path, node_name, node_description, data_type, is_required, api_version, status_id, created_by) VALUES
((SELECT id FROM third_party_integration WHERE code = 'DCS_HOUSEHOLD_DRIVERS'), 'driver_data.personal_info.first_name', 'First Name', 'Driver first name', 'string', true, '2.7', 1, 1),
((SELECT id FROM third_party_integration WHERE code = 'DCS_HOUSEHOLD_DRIVERS'), 'driver_data.personal_info.last_name', 'Last Name', 'Driver last name', 'string', true, '2.7', 1, 1),
((SELECT id FROM third_party_integration WHERE code = 'DCS_HOUSEHOLD_DRIVERS'), 'driver_data.personal_info.date_of_birth', 'Date of Birth', 'Driver date of birth', 'date', true, '2.7', 1, 1),
((SELECT id FROM third_party_integration WHERE code = 'DCS_HOUSEHOLD_DRIVERS'), 'driver_data.license_info.license_number', 'License Number', 'Driver license number', 'string', true, '2.7', 1, 1),
((SELECT id FROM third_party_integration WHERE code = 'DCS_HOUSEHOLD_DRIVERS'), 'driver_data.license_info.state_code', 'License State', 'State that issued the license', 'string', true, '2.7', 1, 1),
((SELECT id FROM third_party_integration WHERE code = 'DCS_HOUSEHOLD_DRIVERS'), 'driver_data.license_info.status', 'License Status', 'Current status of the license', 'string', true, '2.7', 1, 1),
((SELECT id FROM third_party_integration WHERE code = 'DCS_HOUSEHOLD_DRIVERS'), 'driver_data.address_info.current_address.street_1', 'Address Street 1', 'Primary street address', 'string', true, '2.7', 1, 1),
((SELECT id FROM third_party_integration WHERE code = 'DCS_HOUSEHOLD_DRIVERS'), 'driver_data.address_info.current_address.city', 'Address City', 'City name', 'string', true, '2.7', 1, 1),
((SELECT id FROM third_party_integration WHERE code = 'DCS_HOUSEHOLD_DRIVERS'), 'driver_data.address_info.current_address.state', 'Address State', 'State abbreviation', 'string', true, '2.7', 1, 1),
((SELECT id FROM third_party_integration WHERE code = 'DCS_HOUSEHOLD_DRIVERS'), 'driver_data.address_info.current_address.zip_code', 'Address ZIP', 'ZIP code', 'string', true, '2.7', 1, 1),
((SELECT id FROM third_party_integration WHERE code = 'DCS_HOUSEHOLD_DRIVERS'), 'verification_status', 'Verification Status', 'Overall verification result', 'string', true, '2.7', 1, 1),
((SELECT id FROM third_party_integration WHERE code = 'DCS_HOUSEHOLD_DRIVERS'), 'confidence_score', 'Confidence Score', 'Verification confidence percentage', 'integer', true, '2.7', 1, 1);
```

---

**Schema Status:** Ready for Implementation  
**Next Steps:** 
1. Create migration files for these tables
2. Implement API endpoints for integration management
3. Build permission system integration
4. Update IP269-New-Quote-Step-1-Primary-Insured requirement