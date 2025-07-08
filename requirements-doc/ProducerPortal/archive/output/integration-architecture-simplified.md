# Simplified Universal Entity Management Architecture

## Executive Summary

Based on feedback in `prompt8.md`, this document provides a **dramatically simplified** architecture that focuses on core functionality without over-engineering. The approach eliminates licensing complexity, simplifies configuration management, and provides clear guidance on ENUMs vs reference tables.

**Key Simplifications**:
1. **No Licensing System**: Focus on core functionality association only
2. **Simple Component System**: Backend-frontend-security association only
3. **Scope-Based Configuration**: System → Program → Entity (no complex inheritance)
4. **Direct Foreign Keys**: Simple communication associations
5. **Clear ENUM Guidelines**: Use reference tables only when necessary

---

## 1. Simplified Component System

### 1.1 Core Purpose

**User Requirements**:
- Associate backend functionality with frontend functionality
- Support security group access control
- Keep it as simple as possible

### 1.2 Minimal Component Architecture

```sql
-- Simple system components for backend-frontend association
CREATE TABLE system_component (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT,
  
  -- Backend association
  backend_namespace VARCHAR(100), -- e.g., 'App\Services\QuoteService'
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
);

-- Component permissions for security groups
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
);
```

### 1.3 Sample Component Data

```sql
-- Core system components
INSERT INTO system_component (code, name, description, backend_namespace, api_prefix, frontend_route, ui_component, permission_code, status_id, created_by) VALUES
('QUOTE_MANAGEMENT', 'Quote Management', 'Quote creation and management', 'App\\Services\\QuoteService', '/api/v1/quotes', '/quotes', 'QuoteManagement', 'quotes.manage', 1, 1),
('ENTITY_MANAGEMENT', 'Entity Management', 'External entity management', 'App\\Services\\EntityService', '/api/v1/entities', '/entities', 'EntityManagement', 'entities.manage', 1, 1),
('INTEGRATION_MANAGEMENT', 'Integration Management', 'Third-party integration management', 'App\\Services\\IntegrationService', '/api/v1/integrations', '/integrations', 'IntegrationManagement', 'integrations.manage', 1, 1),
('USER_MANAGEMENT', 'User Management', 'User and security management', 'App\\Services\\UserService', '/api/v1/users', '/users', 'UserManagement', 'users.manage', 1, 1),
('CONFIGURATION', 'System Configuration', 'System configuration management', 'App\\Services\\ConfigService', '/api/v1/config', '/config', 'ConfigurationManagement', 'config.manage', 1, 1);
```

**Benefits**:
- **Simple Security**: Easy to assign component access to security groups
- **Frontend-Backend Mapping**: Clear association between UI and API
- **Permission Control**: Granular access control per component
- **No Over-Engineering**: Just what's needed for basic system organization

---

## 2. Simplified Configuration System

### 2.1 Core Purpose

**User Requirements**:
- System configuration, entity configuration, program configuration
- Systematic and flexible
- As simple as possible

### 2.2 Three-Level Configuration Architecture

```sql
-- Configuration types (what can be configured)
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
);

-- Simple scope-based configuration
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
);
```

### 2.3 Simple Configuration Resolution

**Resolution Order**: Entity → Program → System (if not found at entity level, check program, then system)

```sql
-- Function to get configuration value
-- Example: Get API timeout for entity ID 5
-- 1. Check entity configuration first
-- 2. If not found, check program configuration  
-- 3. If not found, use system configuration
-- 4. If not found, use default from configuration_type

-- Sample configuration types
INSERT INTO configuration_type (code, name, description, default_values, status_id, created_by) VALUES
('API_SETTINGS', 'API Settings', 'API timeout and retry settings', 
 '{"timeout_seconds": 30, "retry_attempts": 3, "rate_limit_per_minute": 100}', 1, 1),
('EMAIL_SETTINGS', 'Email Settings', 'Email configuration', 
 '{"smtp_host": "smtp.example.com", "smtp_port": 587, "from_address": "noreply@example.com"}', 1, 1),
('INTEGRATION_SETTINGS', 'Integration Settings', 'Third-party integration settings', 
 '{"cache_ttl_hours": 24, "enable_webhooks": false}', 1, 1);

-- Sample configurations
INSERT INTO configuration (configuration_type_id, scope_type, scope_id, config_data, status_id, created_by) VALUES
-- System-wide API settings
((SELECT id FROM configuration_type WHERE code = 'API_SETTINGS'), 'system', NULL, 
 '{"timeout_seconds": 30, "retry_attempts": 3, "rate_limit_per_minute": 100}', 1, 1),

-- Program-specific API settings (override for program 1)
((SELECT id FROM configuration_type WHERE code = 'API_SETTINGS'), 'program', 1, 
 '{"timeout_seconds": 45, "retry_attempts": 5}', 1, 1),

-- Entity-specific settings (override for entity 5)
((SELECT id FROM configuration_type WHERE code = 'API_SETTINGS'), 'entity', 5, 
 '{"timeout_seconds": 60}', 1, 1);
```

**Benefits**:
- **Simple Hierarchy**: Entity overrides Program overrides System
- **Easy to Understand**: Clear scope levels
- **Flexible**: JSON allows any configuration structure
- **No Complex Inheritance**: Just simple override behavior

---

## 3. ENUM vs Reference Table Philosophy

### 3.1 User Insight Analysis

**User Feedback**: "I just feel like using ENUMs in general are not standardized enough and may be a result of over-complexity. I'm normally seeing enums that are a result of not properly using the _type table"

**Analysis**: User is absolutely correct. ENUMs are often used when a proper reference table with a `_type` pattern would be better.

### 3.2 Clear Guidelines

#### ✅ **Use Reference Tables When**:
- Values need descriptions or additional metadata
- Values might be added/removed at runtime
- Values need status management (active/inactive)
- Values need configuration or relationships
- Values need audit trails

#### ❌ **Use ENUMs Only When**:
- Values are truly static and will never change
- Values are binary or tri-state concepts
- No additional metadata is needed
- Performance is absolutely critical

### 3.3 Recommended Approach

```sql
-- ✅ GOOD: Reference table for communication channels
CREATE TABLE communication_channel (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT,
  is_real_time BOOLEAN DEFAULT FALSE,
  requires_authentication BOOLEAN DEFAULT TRUE,
  default_timeout_seconds INT DEFAULT 30,
  status_id BIGINT UNSIGNED NOT NULL,
  UNIQUE KEY unique_code (code)
);

-- ❌ AVOID: ENUM for communication channels
-- channel ENUM('api', 'email', 'sms', 'phone') 
-- Why avoid? Channels need metadata, timeouts, authentication rules

-- ✅ ACCEPTABLE: ENUM for simple binary/tri-state concepts
scope_type ENUM('system', 'program', 'entity') NOT NULL
-- Why acceptable? These are fixed architectural concepts that won't change

-- ❌ AVOID: ENUM for business concepts
-- status ENUM('active', 'inactive', 'pending')
-- Why avoid? Business statuses need descriptions, workflows, audit trails

-- ✅ GOOD: Reference table for statuses
CREATE TABLE status (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT,
  is_active BOOLEAN DEFAULT TRUE,
  is_default BOOLEAN DEFAULT FALSE,
  sort_order INT DEFAULT 0,
  UNIQUE KEY unique_code (code)
);
```

### 3.4 Migration Strategy

**Immediate**: Convert business-related ENUMs to reference tables
```sql
-- Convert these ENUMs to reference tables
communication_channel (was channel ENUM)
communication_status (was status ENUM)  
entity_category (was category ENUM)
```

**Keep**: Architectural ENUMs that are truly static
```sql
-- Keep these ENUMs
scope_type ENUM('system', 'program', 'entity')
direction ENUM('inbound', 'outbound')
```

---

## 4. Simplified Communication Context

### 4.1 User Simplification Request

**User Feedback**: "this whole business context might be overkill, all we need to do is simply associate the communication with one or many of the following: policy_id, loss_id, claimant_id, entity_id, bank_id, etc."

**Recommendation**: Use direct foreign key associations - much simpler and more performant.

### 4.2 Simple Communication Table

```sql
CREATE TABLE communication (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  communication_type_id BIGINT UNSIGNED NOT NULL,
  
  -- Source and target (simplified)
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
  
  -- Simple business context (direct foreign keys)
  policy_id BIGINT UNSIGNED NULL,
  loss_id BIGINT UNSIGNED NULL,
  claimant_id BIGINT UNSIGNED NULL,
  entity_id BIGINT UNSIGNED NULL,
  bank_id BIGINT UNSIGNED NULL,
  quote_id BIGINT UNSIGNED NULL,
  driver_id BIGINT UNSIGNED NULL,
  
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
  FOREIGN KEY (policy_id) REFERENCES policy(id),
  FOREIGN KEY (loss_id) REFERENCES loss(id),
  FOREIGN KEY (claimant_id) REFERENCES claimant(id),
  FOREIGN KEY (entity_id) REFERENCES entity(id),
  FOREIGN KEY (bank_id) REFERENCES bank(id),
  FOREIGN KEY (quote_id) REFERENCES quote(id),
  FOREIGN KEY (driver_id) REFERENCES driver(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  INDEX idx_source (source_type, source_id),
  INDEX idx_target (target_type, target_id),
  INDEX idx_channel (channel_id),
  INDEX idx_comm_status (communication_status_id),
  INDEX idx_correlation (correlation_id),
  
  -- Business context indexes
  INDEX idx_policy (policy_id),
  INDEX idx_loss (loss_id),
  INDEX idx_claimant (claimant_id),
  INDEX idx_entity (entity_id),
  INDEX idx_bank (bank_id),
  INDEX idx_quote (quote_id),
  INDEX idx_driver (driver_id)
);
```

### 4.3 Benefits of Simple Approach

- **No Polymorphic Complexity**: Direct foreign keys are faster and simpler
- **Clear Relationships**: Easy to understand what each communication relates to
- **Simple Queries**: No complex JOINs through context tables
- **Performance**: Direct indexes on foreign keys
- **Multiple Contexts**: Can associate with multiple business entities directly

**Usage Examples**:
```sql
-- Find all communications for a specific quote
SELECT * FROM communication WHERE quote_id = 123;

-- Find all communications for a specific entity and quote
SELECT * FROM communication WHERE entity_id = 5 AND quote_id = 123;

-- Find all API communications for a loss
SELECT c.* FROM communication c 
JOIN communication_channel ch ON c.channel_id = ch.id 
WHERE c.loss_id = 456 AND ch.code = 'API';
```

---

## 5. Complete Simplified Schema

### 5.1 Core Universal Tables

```sql
-- Entity types (simplified)
CREATE TABLE entity_type (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT,
  category_id BIGINT UNSIGNED NOT NULL,
  metadata_schema JSON,
  
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (category_id) REFERENCES entity_category(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  UNIQUE KEY unique_code (code)
);

-- Entities (simplified)
CREATE TABLE entity (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  entity_type_id BIGINT UNSIGNED NOT NULL,
  code VARCHAR(50) NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT,
  metadata JSON,
  
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
  UNIQUE KEY unique_type_code (entity_type_id, code)
);
```

### 5.2 Reference Tables (Following _type Pattern)

```sql
-- Entity categories
CREATE TABLE entity_category (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT,
  sort_order INT DEFAULT 0,
  status_id BIGINT UNSIGNED NOT NULL,
  UNIQUE KEY unique_code (code)
);

-- Communication types
CREATE TABLE communication_type (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT,
  status_id BIGINT UNSIGNED NOT NULL,
  UNIQUE KEY unique_code (code)
);

-- Communication channels
CREATE TABLE communication_channel (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT,
  is_real_time BOOLEAN DEFAULT FALSE,
  default_timeout_seconds INT DEFAULT 30,
  status_id BIGINT UNSIGNED NOT NULL,
  UNIQUE KEY unique_code (code)
);

-- Communication status
CREATE TABLE communication_status (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT,
  is_final_state BOOLEAN DEFAULT FALSE,
  is_error_state BOOLEAN DEFAULT FALSE,
  status_id BIGINT UNSIGNED NOT NULL,
  UNIQUE KEY unique_code (code)
);
```

### 5.3 Sample Reference Data

```sql
-- Entity categories
INSERT INTO entity_category (code, name, description, sort_order, status_id) VALUES
('INTEGRATION', 'API Integration', 'Third-party API integrations', 1, 1),
('PARTNER', 'Business Partner', 'Business partners (attorneys, body shops)', 2, 1),
('VENDOR', 'Vendor', 'Service vendors', 3, 1),
('SYSTEM', 'System', 'Internal system entities', 4, 1);

-- Communication channels
INSERT INTO communication_channel (code, name, description, is_real_time, default_timeout_seconds, status_id) VALUES
('API', 'API', 'HTTP API calls', TRUE, 30, 1),
('EMAIL', 'Email', 'Email communication', FALSE, 60, 1),
('SMS', 'SMS', 'SMS messaging', TRUE, 15, 1),
('PHONE', 'Phone', 'Phone calls', TRUE, 300, 1),
('MAIL', 'Mail', 'Physical mail', FALSE, 86400, 1);

-- Communication status
INSERT INTO communication_status (code, name, description, is_final_state, is_error_state, status_id) VALUES
('PENDING', 'Pending', 'Waiting to be processed', FALSE, FALSE, 1),
('PROCESSING', 'Processing', 'Currently being processed', FALSE, FALSE, 1),
('COMPLETED', 'Completed', 'Successfully completed', TRUE, FALSE, 1),
('FAILED', 'Failed', 'Processing failed', TRUE, TRUE, 1),
('TIMEOUT', 'Timeout', 'Request timed out', TRUE, TRUE, 1);
```

---

## 6. Implementation Guidelines

### 6.1 Core Principles

1. **Keep It Simple**: Avoid over-engineering
2. **Use Reference Tables**: For business concepts that need metadata
3. **Direct Foreign Keys**: For business context associations
4. **Scope-Based Config**: Simple three-level hierarchy
5. **Clear Security**: Component-based permissions

### 6.2 Development Approach

**Phase 1**: Core universal tables (entity_type, entity)
**Phase 2**: Reference tables following _type pattern
**Phase 3**: Simple configuration system
**Phase 4**: Communication system with direct FK associations
**Phase 5**: Component system for security and UI association

### 6.3 Success Criteria

- **Simplicity**: No complex inheritance or polymorphic relationships
- **Performance**: Direct foreign keys and proper indexing
- **Maintainability**: Clear patterns and minimal complexity
- **Security**: Component-based access control
- **Flexibility**: JSON configuration and metadata

---

## 7. Final Recommendations

### 7.1 Architecture Summary

This simplified architecture provides:

✅ **Universal Entity Management** without over-engineering  
✅ **Simple Component System** for backend-frontend-security association  
✅ **Scope-Based Configuration** with clear hierarchy  
✅ **Reference Tables** following proper _type patterns  
✅ **Direct Business Context** associations for performance  
✅ **Clear Guidelines** for ENUM vs reference table decisions  

### 7.2 Key Benefits

- **Reduced Complexity**: No licensing, inheritance, or polymorphic relationships
- **Better Performance**: Direct foreign keys and simple queries
- **Easier Maintenance**: Clear patterns and minimal complexity
- **Flexible Configuration**: JSON-based with simple scope resolution
- **Security Ready**: Component-based permission system

### 7.3 Implementation Priority

1. **Entity Management**: Core entity_type and entity tables
2. **Reference Tables**: Replace ENUMs with proper _type tables
3. **Configuration**: Simple scope-based configuration system
4. **Communication**: Direct FK approach for business context
5. **Components**: Security and UI association system

This architecture maintains the benefits of Universal Entity Management while dramatically reducing complexity and focusing on practical, maintainable solutions.

---

**Recommendation**: Proceed with this simplified architecture that eliminates over-engineering while providing all necessary functionality for the universal entity management system.