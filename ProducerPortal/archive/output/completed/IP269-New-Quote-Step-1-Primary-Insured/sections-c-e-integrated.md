# IP269 - Quotes: New Quote - Step 1: Primary Insured

## **A) WHY – Vision and Purpose**

The purpose of this screen is to **initiate a new quote** by collecting and verifying the **primary insured’s identity**. It ensures a streamlined intake flow that is user-friendly, minimizes manual input, and leverages existing profile or search data to speed up the quoting process. The design supports **desktop and mobile form factors**, optimizing efficiency for agents in various working conditions.

---

## **B) WHAT – Core Requirements**

### **1. Effective Date & Program Selection**

- The agent must first select the effective date for this policy, which will determine which programs are available in the program drop-down
- An error will be surfaced if the effective date is 31 or more days in the future
- Block progression if rule is triggered

### **2. Search & Match Functionality**

- Ability to search existing records for a potential match using:
    - License Type
        - If ‘US License’ selected, Country field will become disabled and preset to the United States, and the following fields will be displayed:
            - Driver’s License Number
            - State Licensed
        - If an option other than ‘US License’ is selected, the following fields will be displayed:
            - Suffix (Optional)
            - First Name (Required)
            - Middle Name (Optional)
            - Last Name (Required)
            - Street Address (Required)
            - City (Required)
            - State (Required)
            - Country (Required)
            - ZIP Code (Required)
- Display matching results in a modal list with selection option

### **3. Search Results**

- Profile card containing:
    - Full name
    - Date of Birth
    - Address

### **4. Action Options**

- Yes - Information Correct: Selects matched record to prefill quote data
- No - Not a Match: Allows manual entry of insured details without a match
- No - Address Incorrect: Selects matched record to prefill quote data, but allows user to modify the address associated to the quote
- **Continue** button to proceed to the next step

---

## **C) HOW – Planning & Implementation**

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

**Program Selection with Integration Status**
```  
GET /api/v1/programs?effective_date=2024-01-15&producer_id=123
-> SELECT p.id, p.code, p.name, p.description,
          ic.is_enabled as integration_enabled,
          ic.is_required as integration_required
   FROM program p
   LEFT JOIN integration_configuration ic ON p.id = ic.program_id 
                                           AND ic.integration_id = (SELECT id FROM third_party_integration WHERE code = 'DCS_HOUSEHOLD_DRIVERS')
                                           AND ic.configuration_level = 'program'
   WHERE p.effective_date <= :effective_date
   AND (p.expiration_date IS NULL OR p.expiration_date >= :effective_date)
   AND p.status_id = :active_status_id
   ORDER BY p.display_order
```

#### 2. Internal Quote Duplication Check

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
   JOIN map_quote_driver mqd ON q.id = mqd.quote_id AND mqd.is_named_insured = true
   JOIN driver d ON mqd.driver_id = d.id
   JOIN name n ON d.name_id = n.id
   JOIN map_driver_license mdl ON d.id = mdl.driver_id AND mdl.is_primary = true
   JOIN license l ON mdl.license_id = l.id
   JOIN status qs ON q.status_id = qs.id
   WHERE l.license_number = :license_number
   AND l.state_id = :state_id
   AND q.program_id = :program_id
   AND ABS(DATEDIFF(q.effective_date, :effective_date)) <= 30
   AND q.status_id NOT IN (SELECT id FROM status WHERE code IN ('CANCELLED', 'EXPIRED'))
   ORDER BY q.created_at DESC
   LIMIT 5
```

#### 3. Third-Party Integration Search (DCS Household Drivers API)

**Integration Configuration Lookup**
```
GET /api/v1/integrations/DCS_HOUSEHOLD_DRIVERS/config?program_id=1&producer_id=123
-> WITH hierarchy AS (
     SELECT *, 
            CASE configuration_level 
              WHEN 'producer' THEN 1
              WHEN 'program' THEN 2  
              WHEN 'system' THEN 3
            END as priority
     FROM integration_configuration ic
     JOIN third_party_integration tpi ON ic.integration_id = tpi.id
     WHERE tpi.code = 'DCS_HOUSEHOLD_DRIVERS'
     AND ic.is_enabled = true
     AND (ic.producer_id = :producer_id OR ic.producer_id IS NULL)
     AND (ic.program_id = :program_id OR ic.program_id IS NULL)
   )
   SELECT * FROM hierarchy 
   WHERE priority = (SELECT MIN(priority) FROM hierarchy)
   LIMIT 1
```

**DCS API Integration Call**
```
POST /api/v1/integrations/third-party-request
{
  "integration_code": "DCS_HOUSEHOLD_DRIVERS",
  "quote_context": {
    "program_id": 1,
    "producer_id": 123,
    "effective_date": "2024-01-15"
  },
  "search_criteria": {
    "license_number": "D12345678",
    "state_code": "TX",
    "license_type": "DL",
    "first_name": "John",
    "last_name": "Doe"
  },
  "verification_level": "standard"
}

Process:
1. Validate integration is enabled for program/producer
2. Get integration configuration (endpoint, auth, etc.)
3. Create integration_request record for audit
4. Route through Apache Camel to DCS API
5. Process response using configured field mappings
6. Cache results based on TTL settings
7. Create integration_verification_result records
8. Return structured response to frontend
```

**DCS Response Processing**
```
DCS Raw Response:
{
  "status": "success",
  "driver_data": {
    "personal_info": {
      "first_name": "John",
      "last_name": "Doe",
      "date_of_birth": "1980-05-15"
    },
    "license_info": {
      "license_number": "D12345678",
      "state_code": "TX",
      "status": "valid"
    },
    "address_info": {
      "current_address": {
        "street_1": "123 Main Street",
        "city": "Austin",
        "state": "TX",
        "zip_code": "78701"
      }
    }
  },
  "verification_status": "verified",
  "confidence_score": 95
}

Mapped to Internal Format:
{
  "source": "DCS_HOUSEHOLD_DRIVERS",
  "verification_status": "verified",
  "confidence_score": 95,
  "driver_data": {
    "name": {
      "first_name": "John",
      "last_name": "Doe"
    },
    "date_of_birth": "1980-05-15",
    "license": {
      "license_number": "D12345678",
      "state_id": 44, // Mapped from TX to state.id
      "status": "valid",
      "verification_status": "verified"
    },
    "address": {
      "street_1": "123 Main Street",
      "city": "Austin",
      "state_id": 44,
      "zip_code": "78701",
      "verification_status": "verified"
    }
  }
}
```

#### 4. Combined Search Results

**Unified Search Response**
```
POST /api/v1/quotes/search-for-existing-record
{
  "license_type": "US_DL",
  "license_number": "D12345678",
  "state_id": 44,
  "program_id": 1,
  "effective_date": "2024-01-15",
  "first_name": "John",
  "last_name": "Doe"
}

Response:
{
  "internal_search": {
    "duplicate_quotes_found": 1,
    "existing_drivers": [
      {
        "driver_id": 123,
        "name": "John Doe",
        "date_of_birth": "1980-05-15",
        "address": "123 Main St, Austin, TX 78701",
        "last_quote_date": "2024-01-10",
        "verification_status": "verified"
      }
    ],
    "duplicate_quotes": [
      {
        "quote_id": 456,
        "quote_number": "Q-2024-001234",
        "status": "Draft",
        "created_date": "2024-01-10"
      }
    ]
  },
  "third_party_search": {
    "integration_used": "DCS_HOUSEHOLD_DRIVERS",
    "verification_status": "verified",
    "confidence_score": 95,
    "driver_data": {
      // Mapped DCS response data
    },
    "discrepancies": [],
    "cache_status": "fresh",
    "response_time_ms": 450
  },
  "recommendations": [
    {
      "action": "use_existing_driver",
      "driver_id": 123,
      "reason": "High confidence match with verified data"
    }
  ]
}
```

#### 5. Quote Creation with Integrated Verification

**Create Quote with Verification Tracking**
```
POST /api/v1/quotes
{
  "effective_date": "2024-01-15",
  "program_id": 1,
  "driver_selection": {
    "action": "use_verified_external", // or "use_existing", "create_new"
    "external_verification_id": "dcs_12345",
    "driver_data": {
      // Complete driver/license/address data
    },
    "verification_confidence": 95
  }
}

Process:
1. Validate effective date and program
2. Create or link driver based on selection
3. Update driver verification status and timestamps
4. Create quote record with verification context
5. Link all entities via map tables
6. Set appropriate statuses and audit trails
7. Generate quote number
8. Return quote ID and next step URL
```

#### 6. Error Handling and Fallback

**Integration Failure Handling**
```
When DCS API fails:
1. Log error in integration_request table
2. Check if quote creation should continue (based on program settings)
3. If continuation allowed:
   - Present manual entry form
   - Mark driver as "pending_verification"
   - Create suspense for manual verification
4. If continuation not allowed:
   - Block quote creation
   - Present error message
   - Suggest retry or contact support

Alert Generation:
- Critical failures: Immediate notification to system administrators
- Non-critical failures: Logged for batch review
- Pattern detection: Alert on repeated failures for same integration
```

---

## **D) User Experience (UX) & Flows**

### **1. Start a New Quote Flow**

1. User selects “Start New Quote”
2. Step 1: Form opens for **Primary Insured**
3. Enter the effective date and program
    1. The effective date must be within 30 days of the present date when the quote is started
4. Enter the insured’s information
    1. If US License holder, pre-populate Country field with United States and disable, and present Driver’s License Number and State Licensed field
    2. If any other License Type is selected, present Suffix, First Name, Middle Name, Last Name, Street Address, City, State, Country and ZIP Code fields
5. System auto-searches for matches
6. Match dialog appears
    - If match found → user selects and proceeds
    - If no match → enters details manually

### **2. Handling Match Found**

1. Yes - Information Correct: Selects matched record to prefill quote data
2. No - Not a Match: Allows manual entry of insured details without a match
3. No - Address Incorrect: Selects matched record to prefill quote data, but allows user to modify the address associated to the quote
4. **Continue** button to proceed to the next step

### **4. Mobile Flow**

- Stacked, scrollable layout
- Match results shown in full-screen modal
- All actions accessible via bottom CTA button

## E) Master Schema Tables

### Core Tables

#### program (Enhanced for Integration Support)
```sql
CREATE TABLE program (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT,
  effective_date DATE NOT NULL,
  expiration_date DATE NULL,
  display_order INT DEFAULT 0,
  
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  INDEX idx_status (status_id),
  INDEX idx_effective_date (effective_date),
  INDEX idx_expiration_date (expiration_date),
  UNIQUE KEY unique_code (code)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### quote (Enhanced with Integration Tracking)
```sql
CREATE TABLE quote (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  quote_number VARCHAR(50) NOT NULL,
  program_id BIGINT UNSIGNED NOT NULL,
  effective_date DATE NOT NULL,
  expiration_date DATE NULL,
  producer_id BIGINT UNSIGNED NOT NULL,
  
  -- Integration tracking
  primary_verification_source VARCHAR(100) NULL, -- e.g., 'DCS_HOUSEHOLD_DRIVERS'
  verification_confidence DECIMAL(5,2) NULL,
  has_duplicate_check BOOLEAN DEFAULT FALSE,
  
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (program_id) REFERENCES program(id),
  FOREIGN KEY (producer_id) REFERENCES producer(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  INDEX idx_status (status_id),
  INDEX idx_program (program_id),
  INDEX idx_producer (producer_id),
  INDEX idx_effective_date (effective_date),
  INDEX idx_verification_source (primary_verification_source),
  UNIQUE KEY unique_quote_number (quote_number)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### driver (Enhanced with External Verification)
```sql
CREATE TABLE driver (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  name_id BIGINT UNSIGNED NOT NULL,
  date_of_birth DATE NULL,
  gender_id BIGINT UNSIGNED NULL,
  marital_status_id BIGINT UNSIGNED NULL,
  is_named_insured BOOLEAN DEFAULT FALSE,
  driver_type_id BIGINT UNSIGNED NOT NULL,
  relationship_to_insured_id BIGINT UNSIGNED NULL,
  
  -- External verification tracking
  external_verification_status ENUM('not_verified', 'verified', 'partial', 'failed') DEFAULT 'not_verified',
  last_verification_date TIMESTAMP NULL,
  verification_source VARCHAR(100) NULL,
  verification_confidence DECIMAL(5,2) NULL,
  external_driver_id VARCHAR(100) NULL,
  
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (name_id) REFERENCES name(id),
  FOREIGN KEY (gender_id) REFERENCES gender(id),
  FOREIGN KEY (marital_status_id) REFERENCES marital_status(id),
  FOREIGN KEY (driver_type_id) REFERENCES driver_type(id),
  FOREIGN KEY (relationship_to_insured_id) REFERENCES relationship_to_insured(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  INDEX idx_status (status_id),
  INDEX idx_name (name_id),
  INDEX idx_driver_type (driver_type_id),
  INDEX idx_named_insured (is_named_insured),
  INDEX idx_verification_status (external_verification_status),
  INDEX idx_last_verification (last_verification_date),
  INDEX idx_external_id (external_driver_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### license (Enhanced with External Verification)
```sql
CREATE TABLE license (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  license_type_id BIGINT UNSIGNED NOT NULL,
  license_number VARCHAR(50) NOT NULL,
  state_id BIGINT UNSIGNED NULL,
  country_id BIGINT UNSIGNED NOT NULL DEFAULT 1,
  issue_date DATE NULL,
  expiration_date DATE NULL,
  is_verified BOOLEAN DEFAULT FALSE,
  verified_at TIMESTAMP NULL,
  
  -- External verification tracking
  external_verification_status ENUM('not_verified', 'verified', 'invalid', 'expired', 'suspended') DEFAULT 'not_verified',
  last_verification_date TIMESTAMP NULL,
  verification_source VARCHAR(100) NULL,
  external_license_status VARCHAR(50) NULL,
  verification_notes TEXT NULL,
  
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (license_type_id) REFERENCES license_type(id),
  FOREIGN KEY (state_id) REFERENCES state(id),
  FOREIGN KEY (country_id) REFERENCES country(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  INDEX idx_status (status_id),
  INDEX idx_license_type (license_type_id),
  INDEX idx_license_number (license_number),
  INDEX idx_state (state_id),
  INDEX idx_external_verification (external_verification_status),
  INDEX idx_last_verification (last_verification_date),
  UNIQUE KEY unique_license_state (license_number, state_id, license_type_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### address (Enhanced with Standardization Tracking)
```sql
CREATE TABLE address (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  address_type_id BIGINT UNSIGNED NOT NULL,
  street_1 VARCHAR(100) NOT NULL,
  street_2 VARCHAR(100) NULL,
  city VARCHAR(50) NOT NULL,
  state_id BIGINT UNSIGNED NOT NULL,
  country_id BIGINT UNSIGNED NOT NULL DEFAULT 1,
  zip_code VARCHAR(20) NOT NULL,
  zip_4 VARCHAR(4) NULL,
  is_verified BOOLEAN DEFAULT FALSE,
  verified_at TIMESTAMP NULL,
  latitude DECIMAL(10, 8) NULL,
  longitude DECIMAL(11, 8) NULL,
  
  -- Standardization tracking
  standardized_by_source VARCHAR(100) NULL,
  standardization_confidence DECIMAL(5,2) NULL,
  delivery_point_validation ENUM('valid', 'invalid', 'unknown') DEFAULT 'unknown',
  last_standardization_date TIMESTAMP NULL,
  
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (address_type_id) REFERENCES address_type(id),
  FOREIGN KEY (state_id) REFERENCES state(id),
  FOREIGN KEY (country_id) REFERENCES country(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  INDEX idx_status (status_id),
  INDEX idx_address_type (address_type_id),
  INDEX idx_state (state_id),
  INDEX idx_standardized_by (standardized_by_source),
  INDEX idx_delivery_validation (delivery_point_validation),
  INDEX idx_verification (is_verified)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### Integration Management Tables

#### third_party_integration
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

-- Sample integrations
INSERT INTO third_party_integration (code, name, provider_name, api_version, auth_type, status_id, created_by) VALUES
('DCS_HOUSEHOLD_DRIVERS', 'DCS Household Drivers API', 'Data Capture Solutions', '2.7', 'oauth2', 1, 1),
('DCS_CRIMINAL', 'DCS Criminal API', 'Data Capture Solutions', '1.0', 'oauth2', 1, 1),
('DCS_HOUSEHOLD_VEHICLES', 'DCS Household Vehicles API', 'Data Capture Solutions', '2.3', 'oauth2', 1, 1);
```

#### integration_configuration
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
  auth_config JSON NOT NULL,
  
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
  INDEX idx_level (configuration_level),
  INDEX idx_enabled (is_enabled),
  UNIQUE KEY unique_integration_scope (integration_id, configuration_level, program_id, producer_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### integration_node
```sql
CREATE TABLE integration_node (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  integration_id BIGINT UNSIGNED NOT NULL,
  
  -- Node identification
  node_path VARCHAR(255) NOT NULL,
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
  UNIQUE KEY unique_integration_node_version (integration_id, node_path, api_version)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### integration_field_mapping
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
  transformation_rules JSON NULL,
  
  -- Versioning
  version_number INT NOT NULL DEFAULT 1,
  is_active_version BOOLEAN DEFAULT TRUE,
  replaced_mapping_id BIGINT UNSIGNED NULL,
  
  -- Validation rules
  validation_rules JSON NULL,
  
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
  INDEX idx_version (version_number, is_active_version)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### Communication & Audit Tables

#### integration_request
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
  correlation_id VARCHAR(100) NOT NULL,
  
  -- PII protection flags
  contains_pii BOOLEAN DEFAULT TRUE,
  pii_masked BOOLEAN DEFAULT FALSE,
  retention_until DATE NULL,
  
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
  INDEX idx_quote (quote_id),
  INDEX idx_driver (driver_id),
  INDEX idx_request_status (request_status),
  INDEX idx_correlation (correlation_id),
  INDEX idx_retention (retention_until)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### integration_verification_result
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
  confidence_score DECIMAL(5,2) NULL,
  
  -- Verified data (structured)
  verified_data JSON NULL,
  discrepancies JSON NULL,
  
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
  INDEX idx_cache_expires (cache_expires_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### Reference Tables (Existing + Enhanced)

#### license_type (Enhanced for Integration)
```sql
CREATE TABLE license_type (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(20) NOT NULL,
  name VARCHAR(50) NOT NULL,
  description TEXT,
  country_id BIGINT UNSIGNED NULL,
  requires_state BOOLEAN DEFAULT TRUE,
  display_order INT DEFAULT 0,
  
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (country_id) REFERENCES country(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  INDEX idx_status (status_id),
  INDEX idx_country (country_id),
  UNIQUE KEY unique_code (code)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Sample data
INSERT INTO license_type (code, name, description, requires_state, display_order, status_id, created_by) VALUES
('US_DL', 'US Driver License', 'United States Driver License', TRUE, 1, 1, 1),
('INTL_DL', 'International Driver License', 'International Driver License', FALSE, 2, 1, 1),
('NO_LICENSE', 'No License', 'Driver without license', FALSE, 3, 1, 1),
('CDL', 'Commercial Driver License', 'Commercial Driver License', TRUE, 4, 1, 1);
```

#### address_type
```sql
CREATE TABLE address_type (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(20) NOT NULL,
  name VARCHAR(50) NOT NULL,
  description TEXT,
  display_order INT DEFAULT 0,
  
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  INDEX idx_status (status_id),
  UNIQUE KEY unique_code (code)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Sample data
INSERT INTO address_type (code, name, description, display_order, status_id, created_by) VALUES
('HOME', 'Home', 'Home address', 1, 1, 1),
('MAILING', 'Mailing', 'Mailing address', 2, 1, 1),
('BUSINESS', 'Business', 'Business address', 3, 1, 1),
('GARAGING', 'Garaging', 'Vehicle garaging address', 4, 1, 1);
```

### Relationship Tables (Enhanced)

#### map_quote_driver
```sql
CREATE TABLE map_quote_driver (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  quote_id BIGINT UNSIGNED NOT NULL,
  driver_id BIGINT UNSIGNED NOT NULL,
  is_named_insured BOOLEAN DEFAULT FALSE,
  
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (quote_id) REFERENCES quote(id),
  FOREIGN KEY (driver_id) REFERENCES driver(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  INDEX idx_status (status_id),
  INDEX idx_quote (quote_id),
  INDEX idx_driver (driver_id),
  INDEX idx_named_insured (is_named_insured),
  UNIQUE KEY unique_quote_driver (quote_id, driver_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### map_driver_license
```sql
CREATE TABLE map_driver_license (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  driver_id BIGINT UNSIGNED NOT NULL,
  license_id BIGINT UNSIGNED NOT NULL,
  is_primary BOOLEAN DEFAULT FALSE,
  
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (driver_id) REFERENCES driver(id),
  FOREIGN KEY (license_id) REFERENCES license(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  INDEX idx_status (status_id),
  INDEX idx_driver (driver_id),
  INDEX idx_license (license_id),
  INDEX idx_primary (is_primary),
  UNIQUE KEY unique_driver_license (driver_id, license_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### map_driver_address
```sql
CREATE TABLE map_driver_address (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  driver_id BIGINT UNSIGNED NOT NULL,
  address_id BIGINT UNSIGNED NOT NULL,
  is_primary BOOLEAN DEFAULT FALSE,
  
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (driver_id) REFERENCES driver(id),
  FOREIGN KEY (address_id) REFERENCES address(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  INDEX idx_status (status_id),
  INDEX idx_driver (driver_id),
  INDEX idx_address (address_id),
  INDEX idx_primary (is_primary),
  UNIQUE KEY unique_driver_address (driver_id, address_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### Supporting Tables

#### country (System table)
```sql
CREATE TABLE country (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(3) NOT NULL,
  name VARCHAR(100) NOT NULL,
  full_name VARCHAR(200) NULL,
  display_order INT DEFAULT 0,
  
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  INDEX idx_status (status_id),
  UNIQUE KEY unique_code (code)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Sample data
INSERT INTO country (code, name, full_name, display_order, status_id, created_by) VALUES
('US', 'United States', 'United States of America', 1, 1, 1),
('CA', 'Canada', 'Canada', 2, 1, 1),
('MX', 'Mexico', 'Mexico', 3, 1, 1);
```

#### state (System table)
```sql
CREATE TABLE state (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  country_id BIGINT UNSIGNED NOT NULL,
  code VARCHAR(5) NOT NULL,
  name VARCHAR(100) NOT NULL,
  abbreviation VARCHAR(5) NOT NULL,
  display_order INT DEFAULT 0,
  
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (country_id) REFERENCES country(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  INDEX idx_status (status_id),
  INDEX idx_country (country_id),
  UNIQUE KEY unique_country_code (country_id, code)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### name (Reused from existing)
```sql
CREATE TABLE name (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  prefix VARCHAR(20) NULL,
  first_name VARCHAR(50) NOT NULL,
  middle_name VARCHAR(50) NULL,
  last_name VARCHAR(50) NOT NULL,
  suffix VARCHAR(20) NULL,
  
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  INDEX idx_status (status_id),
  INDEX idx_first_name (first_name),
  INDEX idx_last_name (last_name),
  INDEX idx_full_name (first_name, last_name)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### Sample Integration Node Data

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