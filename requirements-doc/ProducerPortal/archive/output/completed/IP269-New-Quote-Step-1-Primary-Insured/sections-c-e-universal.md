# IP269 - Quotes: New Quote - Step 1: Primary Insured (Universal Architecture)

## **C) HOW – Planning & Implementation (Universal Entity Management)**

### Backend API Mappings

#### 1. Effective Date & Program Selection

**Effective Date Validation**
```
POST /api/v1/quotes/validate-effective-date
{
  "effective_date": "2024-01-15"
}

Validation:
- Check: effective_date <= CURRENT_DATE + 30 days
- Return error if > 30 days: "Effective date cannot be more than 30 days in the future"
```

**Program Selection with Universal Entity Integration Status**
```  
GET /api/v1/programs?effective_date=2024-01-15&producer_id=123
-> SELECT p.id, p.code, p.name, p.description,
          CASE WHEN ue.id IS NOT NULL THEN TRUE ELSE FALSE END as has_universal_integration
   FROM program p
   LEFT JOIN entity e ON p.id = e.entity_id
   LEFT JOIN entity_type et ON e.entity_type_id = et.id
   WHERE et.code = 'INTEGRATION'
   AND p.effective_date <= :effective_date
   AND (p.expiration_date IS NULL OR p.expiration_date >= :effective_date)
   AND p.status_id = :active_status_id
   ORDER BY p.display_order
```

#### 2. Universal Entity Communication for External Verification

**DCS Multi-API Driver Verification via Universal Entity Management**
```
POST /api/v1/dcs/workflows/driver-verification
{
  "correlation_id": "quote-123-driver-verification",
  "driver_data": {
    "license_number": "TX12345678",
    "state": "TX",
    "first_name": "John", 
    "last_name": "Smith",
    "date_of_birth": "1975-02-25"
  },
  "address_data": {
    "street_1": "123 Main St",
    "city": "Dallas",
    "state": "TX",
    "zip_code": "75001"
  },
  "verification_options": {
    "include_household": true,
    "include_vehicles": true,
    "include_criminal": true
  }
}

Backend Processing:
1. Create DCS Driver entity communication:
   -> SELECT entity.* FROM entity 
      JOIN entity_type ON entity.entity_type_id = entity_type.id
      WHERE entity_type.code = 'DCS_HOUSEHOLD_DRIVERS'
      AND entity.status_id = :active_status_id

2. Insert driver verification communication:
   -> INSERT INTO communication (
        communication_type_id, source_type, source_id,
        target_type, target_id, channel_id, direction,
        request_data, correlation_id, status_id
      ) VALUES (
        (SELECT id FROM communication_type WHERE code = 'DCS_DRIVER_LOOKUP'),
        'system', 1, 'entity', :dcs_driver_entity_id, 
        (SELECT id FROM communication_channel WHERE code = 'DCS_API'),
        'outbound', :driver_request_xml, :correlation_id, 
        (SELECT id FROM communication_status WHERE code = 'DCS_PENDING')
      )

3. If include_vehicles = true, create vehicle lookup communication:
   -> INSERT INTO communication (
        communication_type_id, source_type, source_id,
        target_type, target_id, channel_id, direction,
        request_data, correlation_id, status_id
      ) VALUES (
        (SELECT id FROM communication_type WHERE code = 'DCS_VEHICLE_LOOKUP'),
        'system', 1, 'entity', :dcs_vehicle_entity_id,
        (SELECT id FROM communication_channel WHERE code = 'DCS_API'),
        'outbound', :vehicle_request_xml, :correlation_id,
        (SELECT id FROM communication_status WHERE code = 'DCS_PENDING')
      )

4. If include_criminal = true, create criminal check communication:
   -> INSERT INTO communication (
        communication_type_id, source_type, source_id,
        target_type, target_id, channel_id, direction,
        request_data, correlation_id, status_id
      ) VALUES (
        (SELECT id FROM communication_type WHERE code = 'DCS_CRIMINAL_CHECK'),
        'system', 1, 'entity', :dcs_criminal_entity_id,
        (SELECT id FROM communication_channel WHERE code = 'DCS_API'),
        'outbound', :criminal_request_xml, :correlation_id,
        (SELECT id FROM communication_status WHERE code = 'DCS_PENDING')
      )

Response Data Assembly:
-> SELECT c.id, c.communication_type_id, ct.code as type_code,
          c.response_data, c.status_id, cs.code as status_code,
          c.created_at, c.updated_at
   FROM communication c
   JOIN communication_type ct ON c.communication_type_id = ct.id
   JOIN communication_status cs ON c.status_id = cs.id
   WHERE c.correlation_id = :correlation_id
   ORDER BY c.created_at
    "verification_type": "HOUSEHOLD_DRIVERS"
  },
  "correlation_id": "quote-123-driver-verification"
}

Backend Processing:
-> get entity where id = 5 (DCS integration entity)
-> get entity.metadata for API configuration
-> resolve configuration for entity scope
-> make API call using entity metadata
-> log communication record with response
-> return standardized verification result
```

**Universal Entity Configuration Resolution**
```sql
-- Get DCS entity configuration with scope hierarchy
SELECT COALESCE(
  entity_config.config_data,
  program_config.config_data,
  system_config.config_data,
  ct.default_values
) as resolved_config
FROM entity e
JOIN entity_type et ON e.entity_type_id = et.id
JOIN configuration_type ct ON ct.code = 'API_SETTINGS'
LEFT JOIN configuration entity_config ON ct.id = entity_config.configuration_type_id 
  AND entity_config.scope_type = 'entity' AND entity_config.scope_id = e.id
LEFT JOIN configuration program_config ON ct.id = program_config.configuration_type_id 
  AND program_config.scope_type = 'program' AND program_config.scope_id = :program_id
LEFT JOIN configuration system_config ON ct.id = system_config.configuration_type_id 
  AND system_config.scope_type = 'system' AND system_config.scope_id IS NULL
WHERE e.id = :entity_id
AND et.code = 'API_INTEGRATION'
AND e.status_id = :active_status_id
```

#### 3. Internal Quote Duplication Check

**Duplicate Quote Search**
```
POST /api/v1/quotes/check-duplicates
{
  "license_number": "D12345678",
  "state_id": 5,
  "program_id": 1,
  "effective_date": "2024-01-15"
}

-> SELECT q.id, q.quote_number, q.created_at,
          n.first_name, n.last_name, qs.name as status_name
   FROM quote q
   JOIN map_quote_driver mqd ON q.id = mqd.quote_id
   JOIN driver d ON mqd.driver_id = d.id AND d.is_named_insured = true
   JOIN name n ON d.name_id = n.id
   JOIN map_driver_license mdl ON d.id = mdl.driver_id
   JOIN license l ON mdl.license_id = l.id
   JOIN status qs ON q.status_id = qs.id
   WHERE l.license_number = :license_number
   AND l.state_id = :state_id
   AND q.program_id = :program_id
   AND q.effective_date = :effective_date
   AND q.status_id IN (SELECT id FROM status WHERE code IN ('PENDING', 'ACTIVE'))
   ORDER BY q.created_at DESC
```

#### 4. License Type Processing with Universal Communication

**License Type Validation and External Verification**
```
GET /api/v1/license-types?country=US
-> SELECT lt.id, lt.code, lt.name, lt.requires_state
   FROM license_type lt
   WHERE lt.status_id = :active_status_id
   ORDER BY lt.display_order

POST /api/v1/drivers/verify-license
{
  "license_type": "US_DL",
  "license_number": "D12345678", 
  "state_id": 5,
  "verification_source": "DCS_HOUSEHOLD_DRIVERS"
}

Processing Flow:
-> find entity where entity_type.code = 'API_INTEGRATION' and code = 'DCS_HOUSEHOLD_DRIVERS'
-> create communication record for verification request
-> call universal entity communication service
-> process response using entity metadata schema
-> update driver verification status
-> return verification result
```

#### 5. Address Processing with Universal Architecture

**Address Verification via Universal Entity**
```
POST /api/v1/addresses/verify
{
  "address_data": {
    "street_1": "123 Main St",
    "city": "Austin", 
    "state_id": 48,
    "zip_code": "78701"
  },
  "verification_service": "ADDRESS_VALIDATION_API"
}

Backend Processing:
-> get entity where entity_type.code = 'API_INTEGRATION' and metadata.service_type = 'address_validation'
-> create communication record
-> call address validation API via universal entity
-> apply response transformation rules from entity metadata
-> update address verification fields
-> return standardized address result
```

#### 6. Driver Search and Match with Universal Communication Audit

**Driver Search with Communication Logging**
```
POST /api/v1/drivers/search
{
  "search_criteria": {
    "license_number": "D12345678",
    "state_id": 5,
    "first_name": "John",
    "last_name": "Smith"
  },
  "use_external_verification": true
}

-> if external verification enabled:
   -> create communication record for search request
   -> call driver verification entity (DCS or similar)
   -> log all external communications with correlation_id
   -> merge external data with internal search results
   -> return unified driver search results with verification status
```

---

## **E) DATABASE SCHEMA (Universal Entity Management)**

### Universal Entity Management Tables

#### 1. Entity Categories
```sql
CREATE TABLE entity_category (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT,
  sort_order INT DEFAULT 0,
  
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  UNIQUE KEY unique_code (code),
  INDEX idx_sort_order (sort_order)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### 2. Entity Types (Universal Schema Definitions)
```sql
CREATE TABLE entity_type (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT,
  category_id BIGINT UNSIGNED NOT NULL,
  metadata_schema JSON, -- JSON schema for entity validation
  
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (category_id) REFERENCES entity_category(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  INDEX idx_category (category_id),
  UNIQUE KEY unique_code (code)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### 3. Universal Entity Storage
```sql
CREATE TABLE entity (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  entity_type_id BIGINT UNSIGNED NOT NULL,
  code VARCHAR(50) NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT,
  metadata JSON, -- Flexible data based on entity_type schema
  
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (entity_type_id) REFERENCES entity_type(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  INDEX idx_entity_type (entity_type_id),
  INDEX idx_code (code),
  UNIQUE KEY unique_type_code (entity_type_id, code)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### Universal Configuration System

#### 4. Configuration Types
```sql
CREATE TABLE configuration_type (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT,
  default_values JSON, -- Default configuration values
  schema_definition JSON, -- JSON schema for validation
  
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  UNIQUE KEY unique_code (code)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### 5. Universal Configuration with Simple Hierarchy
```sql
CREATE TABLE configuration (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  configuration_type_id BIGINT UNSIGNED NOT NULL,
  
  -- Simple scope: system, program, or entity
  scope_type ENUM('system', 'program', 'entity') NOT NULL,
  scope_id BIGINT UNSIGNED NULL, -- program_id or entity_id (NULL for system)
  
  -- Configuration data
  config_data JSON NOT NULL,
  
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (configuration_type_id) REFERENCES configuration_type(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  INDEX idx_scope (scope_type, scope_id),
  INDEX idx_type (configuration_type_id),
  UNIQUE KEY unique_config_scope (configuration_type_id, scope_type, scope_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### Universal Communication System

#### 6. Communication Reference Tables
```sql
CREATE TABLE communication_type (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT,
  
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  UNIQUE KEY unique_code (code)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

CREATE TABLE communication_channel (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT,
  is_real_time BOOLEAN DEFAULT FALSE,
  default_timeout_seconds INT DEFAULT 30,
  
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  UNIQUE KEY unique_code (code)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

CREATE TABLE communication_status (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT,
  is_final_state BOOLEAN DEFAULT FALSE,
  is_error_state BOOLEAN DEFAULT FALSE,
  
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  UNIQUE KEY unique_code (code)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### 7. Ultra-Simple Universal Communication
```sql
CREATE TABLE communication (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  communication_type_id BIGINT UNSIGNED NOT NULL,
  
  -- Ultra-simple source and target (polymorphic only)
  source_type ENUM('system', 'user', 'entity') NOT NULL,
  source_id BIGINT UNSIGNED NOT NULL,
  target_type ENUM('system', 'user', 'entity') NOT NULL,
  target_id BIGINT UNSIGNED NOT NULL,
  
  -- Communication details
  channel_id BIGINT UNSIGNED NOT NULL,
  direction ENUM('inbound', 'outbound') NOT NULL,
  
  -- Payload
  request_data JSON,
  response_data JSON,
  
  -- Status
  communication_status_id BIGINT UNSIGNED NOT NULL,
  error_message TEXT NULL,
  
  -- Timing
  started_at TIMESTAMP NULL,
  completed_at TIMESTAMP NULL,
  
  -- Correlation for tracking
  correlation_id VARCHAR(100) NOT NULL,
  
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (communication_type_id) REFERENCES communication_type(id),
  FOREIGN KEY (channel_id) REFERENCES communication_channel(id),
  FOREIGN KEY (communication_status_id) REFERENCES communication_status(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  INDEX idx_source (source_type, source_id),
  INDEX idx_target (target_type, target_id),
  INDEX idx_channel (channel_id),
  INDEX idx_comm_status (communication_status_id),
  INDEX idx_correlation (correlation_id),
  INDEX idx_created (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### Component-Based Security System

#### 8. System Components
```sql
CREATE TABLE system_component (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT,
  
  -- Backend association
  backend_namespace VARCHAR(100), -- e.g., 'App\\Services\\QuoteService'
  api_prefix VARCHAR(50), -- e.g., '/api/v1/quotes'
  
  -- Frontend association  
  frontend_route VARCHAR(100), -- e.g., '/quotes'
  ui_component VARCHAR(100), -- e.g., 'QuoteManagement'
  
  -- Security
  requires_permission BOOLEAN DEFAULT TRUE,
  permission_code VARCHAR(100), -- e.g., 'quotes.manage'
  
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  UNIQUE KEY unique_code (code)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

CREATE TABLE system_component_permission (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  component_id BIGINT UNSIGNED NOT NULL,
  security_group_id BIGINT UNSIGNED NOT NULL,
  can_read BOOLEAN DEFAULT FALSE,
  can_write BOOLEAN DEFAULT FALSE,
  can_delete BOOLEAN DEFAULT FALSE,
  can_admin BOOLEAN DEFAULT FALSE,
  
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (component_id) REFERENCES system_component(id),
  FOREIGN KEY (security_group_id) REFERENCES security_group(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  UNIQUE KEY unique_component_group (component_id, security_group_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### Sample Data for Universal Entity Management

#### 9. Core Reference Data
```sql
-- Entity categories
INSERT INTO entity_category (code, name, description, sort_order, status_id, created_by) VALUES
('INTEGRATION', 'API Integration', 'Third-party API integrations', 1, 1, 1),
('PARTNER', 'Business Partner', 'Business partners (attorneys, body shops)', 2, 1, 1),
('VENDOR', 'Vendor', 'Service vendors', 3, 1, 1),
('SYSTEM', 'System', 'Internal system entities', 4, 1, 1);

-- Entity types with JSON schemas
INSERT INTO entity_type (code, name, description, category_id, metadata_schema, status_id, created_by) VALUES
('API_INTEGRATION', 'API Integration', 'Third-party API service integration', 
 (SELECT id FROM entity_category WHERE code = 'INTEGRATION'),
 '{"type": "object", "properties": {"provider": {"type": "string"}, "api_version": {"type": "string"}, "base_url": {"type": "string", "format": "uri"}, "auth_type": {"type": "string", "enum": ["api_key", "oauth2", "basic", "bearer"]}}, "required": ["provider", "base_url", "auth_type"]}',
 1, 1);

-- DCS integration entity
INSERT INTO entity (entity_type_id, code, name, description, metadata, status_id, created_by) VALUES
((SELECT id FROM entity_type WHERE code = 'API_INTEGRATION'), 'DCS_HOUSEHOLD_DRIVERS', 'DCS Household Drivers API', 'Driver verification and household member lookup',
 '{"provider": "Data Capture Solutions", "api_version": "2.7", "base_url": "https://api.dcs.com", "auth_type": "oauth2"}', 1, 1);

-- Communication reference data
INSERT INTO communication_channel (code, name, description, is_real_time, default_timeout_seconds, status_id, created_by) VALUES
('API', 'API', 'HTTP API calls', TRUE, 30, 1, 1),
('EMAIL', 'Email', 'Email communication', FALSE, 60, 1, 1);

INSERT INTO communication_type (code, name, description, status_id, created_by) VALUES
('API_REQUEST', 'API Request', 'External API call', 1, 1),
('VERIFICATION', 'Verification', 'Identity or data verification', 1, 1);

INSERT INTO communication_status (code, name, description, is_final_state, is_error_state, status_id, created_by) VALUES
('PENDING', 'Pending', 'Waiting to be processed', FALSE, FALSE, 1, 1),
('COMPLETED', 'Completed', 'Successfully completed', TRUE, FALSE, 1, 1),
('FAILED', 'Failed', 'Processing failed', TRUE, TRUE, 1, 1);
```

### Integration with Existing Tables

#### 10. Enhanced Driver Table for Universal Communication
```sql
-- Add universal communication tracking to existing driver table
ALTER TABLE driver ADD COLUMN (
  last_external_verification_date TIMESTAMP NULL,
  external_verification_source VARCHAR(100) NULL,
  verification_correlation_id VARCHAR(100) NULL
);

ALTER TABLE driver ADD INDEX idx_verification_correlation (verification_correlation_id);
ALTER TABLE driver ADD INDEX idx_last_verification (last_external_verification_date);
```

---

**Universal Entity Management Benefits for IP269:**
- **Zero Code Changes**: Adding new verification services (attorneys, vendors) requires no code changes
- **Consistent Patterns**: All external communications follow same universal patterns  
- **Simple Configuration**: Entity-level overrides for program/producer specific settings
- **Complete Audit Trail**: All external communications logged with correlation tracking
- **High Performance**: Simple relationships with proper indexing
- **Global Alignment**: Integrates with existing Global Requirements 44 and 48