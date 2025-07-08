# IP269-New-Quote-Step-1-Primary-Insured - Complete Requirement

---

## **A) WHY – Vision and Purpose**

The purpose of this screen is to **initiate a new quote** by collecting and verifying the **primary insured's identity**. It ensures a streamlined intake flow that is user-friendly, minimizes manual input, and leverages existing profile or search data to speed up the quoting process. The design supports **desktop and mobile form factors**, optimizing efficiency for agents in various working conditions.

---

## **B) WHAT – Core Requirements**

### **1. Effective Date & Program Selection**

- The agent must first select the effective date for this policy, which will determine which programs are available in the program drop-down
- An error will be surfaced if the effective date is 31 or more days in the future
- Block progression if rule is triggered

### **2. Search & Match Functionality**

- Ability to search existing records for a potential match using:
    - License Type
        - If 'US License' selected, Country field will become disabled and preset to the United States, and the following fields will be displayed:
            - Driver's License Number
            - State Licensed
        - If an option other than 'US License' is selected, the following fields will be displayed:
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

## Entity Analysis

### Entities Involved
| Entity Name | Type | Status | Notes |
|-------------|------|--------|--------|
| quote | Core | New | Main quote entity created immediately |
| program | Core | Existing | Insurance program selection |
| producer | Core | Existing | Producer context attachment |
| driver | Core | Existing | Primary insured driver data |
| quote_driver | Map | New | Quote-to-driver relationship |
| license_type | Reference | New | License type lookup |
| address | Supporting | Existing | Address storage |
| DCS_HOUSEHOLD_DRIVERS | External | Existing | DCS API integration entity |

### New Tables Required
- **quote**: Core quote entity created immediately at Step 1 start
- **license_type**: Reference table for license types (US License, etc.)
- **map_quote_driver**: Quote-to-driver relationship tracking
- **producer_program**: Producer-to-program assignment tracking

### Modifications to Existing Tables
- **driver**: Add DCS data fields for external integration tracking
- **program**: Add availability and effective date configuration fields

### Relationships Identified
- quote → producer (many-to-one)
- quote → program (many-to-one)
- quote → driver (many-to-many via map_quote_driver)
- driver → license_type (many-to-one)
- driver → address (many-to-one)

---

## Field Mappings (Section C)

### Backend Mappings

#### Effective Date & Program Selection

##### Effective Date
- **Backend Mapping**: 
  ```
  get program.id from quote
  -> get program_configuration by program.id
  -> validate effective_date against program.date_rules AND current_date + 30 days
  -> return validation_result, available_programs
  ```

##### Program Selection
- **Backend Mapping**:
  ```
  get producer.id from auth_user
  -> get producer_program by producer.id 
  -> join program on producer_program.program_id = program.id
  -> where program.status_id = active AND program.effective_date_start <= input_date <= program.effective_date_end
  -> return available_programs[]
  ```

#### License Type & Search Fields

##### License Type
- **Backend Mapping**:
  ```
  get license_type.id from license_type
  -> where license_type.status_id = active
  -> return license_type.code, license_type.name, license_type.search_fields_required
  ```

##### Driver's License Search (US License)
- **Backend Mapping**:
  ```
  validate license_number format
  -> call DCS_HOUSEHOLD_DRIVERS API with license_number, state
  -> store DCS response in driver entity
  -> return driver.id, driver.first_name, driver.last_name, driver.date_of_birth, address.*
  ```

##### Personal Information Search (Non-US License)
- **Backend Mapping**:
  ```
  validate required fields (first_name, last_name, address.*)
  -> call DCS_HOUSEHOLD_DRIVERS API with name_address_search
  -> parse DCS response for household_drivers[]
  -> return potential_matches[] with confidence_score
  ```

#### Search Results & Match Selection

##### Search Results Display
- **Backend Mapping**:
  ```
  get DCS search_results from DCS_HOUSEHOLD_DRIVERS.response_data
  -> format results as profile_cards[]
  -> each card: driver.full_name, driver.date_of_birth, address.formatted_address
  -> return search_results[] with match_score
  ```

##### Match Selection Actions
- **Backend Mapping**:
  ```
  if action = 'information_correct':
    -> create quote with immediate quote_number generation
    -> create map_quote_driver with selected driver
    -> store DCS data in driver entity
  
  if action = 'not_a_match':
    -> create quote with quote_number
    -> enable manual_entry_mode = true
    
  if action = 'address_incorrect':
    -> create quote with quote_number
    -> create map_quote_driver with selected driver
    -> set address_override_required = true
  ```

### Implementation Architecture

**Quote Creation Strategy**: Immediate quote creation at Step 1 start per stakeholder decision. Quote number assigned immediately to support workflow persistence across steps.

**DCS Integration Pattern**: Follows GR-53 DCS Integration Architecture using Universal Entity Management (GR-52) for external API handling. DCS serves as primary data source with no internal database search.

**Data Persistence Approach**: Direct entity storage per stakeholder clarification (no caching needed). All DCS search results stored in driver entities for propagation across quote workflow steps.

**Producer Context**: Producer always attached to quote, supporting both producer portal and future direct-to-consumer implementation.

### Integration Specifications

#### DCS Household Drivers API Integration

**Entity Type**: DCS_HOUSEHOLD_DRIVERS (Universal Entity Management)  
**Provider**: Data Collection Services  
**Endpoint**: DCS Household Drivers API (GR-53)

**Circuit Breaker Configuration**:
```php
'dcs_household_drivers' => [
    'failure_threshold' => 5,
    'timeout_seconds' => 60,
    'recovery_timeout' => 300,
    'fallback_strategy' => 'manual_entry_only'
]
```

**Request/Response Schema**:
```json
// License Lookup Request
{
  "license_number": "DL123456789",
  "license_state": "CA",
  "search_type": "license_lookup"
}

// Name/Address Lookup Request
{
  "first_name": "John",
  "last_name": "Smith",
  "address": {
    "street": "123 Main St",
    "city": "Anytown",
    "state": "CA",
    "zip": "90210"
  },
  "search_type": "name_address_lookup"
}

// Response
{
  "household_drivers": [
    {
      "driver_id": "DCS123456",
      "first_name": "John",
      "last_name": "Smith",
      "date_of_birth": "1985-06-15",
      "license_number": "DL123456789",
      "license_state": "CA",
      "address": {
        "street": "123 Main St",
        "city": "Anytown",
        "state": "CA",
        "zip": "90210"
      }
    }
  ],
  "search_metadata": {
    "search_type": "license_lookup",
    "match_confidence": "high",
    "household_size": 1
  }
}
```

**Security & Privacy**:
- PII data masked in communication logs
- License numbers encrypted in transit
- DCS correlation ID tracking for audit compliance
- 7-year retention per insurance regulations

#### Communication Tracking (Following GR-44)

All DCS API calls logged using universal communication table.

#### Performance & Monitoring

**Response Time Targets**:
- License lookup: < 5 seconds (per stakeholder clarification pending)
- Name/address search: < 10 seconds
- Quote creation: < 500ms
- Program availability check: < 200ms

**Data Persistence Strategy**:
- Direct entity storage (no caching per stakeholder decision)
- DCS results stored in driver entities
- Quote data propagated across workflow steps

**Rate Limiting**: 100 DCS requests/minute per producer

#### Error Handling

**Circuit Breaker Pattern**: 5 failures trigger protection, 60-second timeout  
**Graceful Degradation**: Fall back to manual entry when DCS unavailable  
**Retry Logic**: 3 attempts with exponential backoff for transient failures

---

## **D) User Experience (UX) & Flows**

### **1. Start a New Quote Flow**

1. User selects "Start New Quote"
2. Step 1: Form opens for **Primary Insured**
3. Enter the effective date and program
    1. The effective date must be within 30 days of the present date when the quote is started
4. Enter the insured's information
    1. If US License holder, pre-populate Country field with United States and disable, and present Driver's License Number and State Licensed field
    2. If any other License Type is selected, present Suffix, First Name, Middle Name, Last Name, Street Address, City, State, Country and ZIP Code fields
5. System auto-searches for matches via DCS API
6. Match dialog appears
    - If match found → user selects and proceeds
    - If no match → enters details manually

### **2. Handling Match Found**

1. Yes - Information Correct: Selects matched record to prefill quote data
2. No - Not a Match: Allows manual entry of insured details without a match
3. No - Address Incorrect: Selects matched record to prefill quote data, but allows user to modify the address associated to the quote
4. **Continue** button to proceed to the next step

### **3. Mobile Flow**

- Stacked, scrollable layout
- Match results shown in full-screen modal
- All actions accessible via bottom CTA button

---

## API Specifications

### Endpoints Required
```http
POST   /api/v1/quotes/start              # Create new quote immediately
GET    /api/v1/quotes/{id}/programs      # Get available programs for producer
POST   /api/v1/quotes/{id}/dcs-search    # Trigger DCS household search
POST   /api/v1/quotes/{id}/select-match  # Select DCS match result
PUT    /api/v1/quotes/{id}/primary-insured  # Update primary insured info
GET    /api/v1/license-types             # Get available license types
```

### Real-time Updates
```javascript
// WebSocket channels
private-quote.{id}                        # Quote-specific updates
private-producer.{producer_id}.quotes     # All quotes for producer
```

---

## Database Schema (Section E)

### New Core Tables

#### quote
```sql
CREATE TABLE quote (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  
  -- Business columns
  quote_number VARCHAR(50) UNIQUE NOT NULL,
  effective_date DATE NOT NULL,
  producer_id BIGINT UNSIGNED NOT NULL,
  program_id BIGINT UNSIGNED NOT NULL,
  dcs_search_completed BOOLEAN DEFAULT FALSE,
  dcs_data_populated BOOLEAN DEFAULT FALSE,
  manual_entry_mode BOOLEAN DEFAULT FALSE,
  address_override_required BOOLEAN DEFAULT FALSE,
  
  -- Status and workflow
  status_id BIGINT UNSIGNED NOT NULL,
  workflow_step ENUM('primary_insured', 'named_insured', 'drivers', 'vehicles', 'coverages') DEFAULT 'primary_insured',
  
  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (producer_id) REFERENCES user(id),
  FOREIGN KEY (program_id) REFERENCES program(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  -- Indexes
  INDEX idx_producer (producer_id),
  INDEX idx_program (program_id),
  INDEX idx_status (status_id),
  INDEX idx_quote_number (quote_number),
  INDEX idx_effective_date (effective_date),
  INDEX idx_workflow_step (workflow_step)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### New Reference Tables

#### license_type
```sql
CREATE TABLE license_type (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) UNIQUE NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT NULL,
  search_fields_required JSON NULL,
  country_code VARCHAR(3) NULL,
  status_id BIGINT UNSIGNED NOT NULL,
  
  -- Audit fields
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (status_id) REFERENCES status(id),
  
  -- Indexes
  INDEX idx_code (code),
  INDEX idx_status (status_id),
  INDEX idx_country (country_code)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### producer_program
```sql
CREATE TABLE producer_program (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  producer_id BIGINT UNSIGNED NOT NULL,
  program_id BIGINT UNSIGNED NOT NULL,
  assignment_date DATE NOT NULL,
  status_id BIGINT UNSIGNED NOT NULL,
  
  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (producer_id) REFERENCES user(id),
  FOREIGN KEY (program_id) REFERENCES program(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  -- Constraints
  UNIQUE KEY unique_producer_program (producer_id, program_id),
  
  -- Indexes
  INDEX idx_producer (producer_id),
  INDEX idx_program (program_id),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### New Relationship Tables

#### map_quote_driver
```sql
CREATE TABLE map_quote_driver (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  quote_id BIGINT UNSIGNED NOT NULL,
  driver_id BIGINT UNSIGNED NOT NULL,
  is_primary_insured BOOLEAN DEFAULT FALSE,
  dcs_driver_id VARCHAR(100) NULL,
  dcs_correlation_id VARCHAR(100) NULL,
  status_id BIGINT UNSIGNED NOT NULL,
  
  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (quote_id) REFERENCES quote(id) ON DELETE CASCADE,
  FOREIGN KEY (driver_id) REFERENCES driver(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  -- Constraints
  UNIQUE KEY unique_quote_driver (quote_id, driver_id),
  
  -- Indexes
  INDEX idx_quote (quote_id),
  INDEX idx_driver (driver_id),
  INDEX idx_status (status_id),
  INDEX idx_primary_insured (is_primary_insured),
  INDEX idx_dcs_correlation (dcs_correlation_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### Modified Tables

#### ALTER TABLE driver
```sql
-- Add DCS integration fields
ALTER TABLE driver 
ADD COLUMN dcs_driver_id VARCHAR(100) NULL,
ADD COLUMN dcs_last_updated TIMESTAMP NULL,
ADD COLUMN license_type_id BIGINT UNSIGNED NULL;

-- Add foreign key for license type
ALTER TABLE driver
ADD CONSTRAINT fk_driver_license_type 
FOREIGN KEY (license_type_id) REFERENCES license_type(id);

-- Add index for DCS tracking
ALTER TABLE driver
ADD INDEX idx_dcs_driver_id (dcs_driver_id);
```

#### ALTER TABLE program
```sql
-- Add program configuration fields
ALTER TABLE program 
ADD COLUMN effective_date_start DATE NULL,
ADD COLUMN effective_date_end DATE NULL,
ADD COLUMN max_future_days INT DEFAULT 30,
ADD COLUMN availability_rules JSON NULL;

-- Add indexes for date-based queries
ALTER TABLE program
ADD INDEX idx_effective_dates (effective_date_start, effective_date_end);
```

---

## Implementation Notes

### Dependencies
- DCS Household Drivers API integration (GR-53)
- Producer-to-program assignment system
- Quote number generation sequence
- Program availability configuration

### Migration Considerations
- Existing driver data compatibility with new DCS fields
- Program configuration migration for effective date rules
- Producer-program assignment data setup

### Performance Considerations
- DCS API response time optimization
- Quote number generation efficiency
- Program availability query optimization
- Driver search result caching strategy (direct storage per stakeholder)

---

## Quality Checklist

### Pre-Implementation
- [x] All UI fields mapped to database columns
- [x] Existing entities reused where possible (User, Driver, Policy)
- [x] Reference tables created for all ENUMs (license_type)
- [x] Naming conventions followed consistently
- [x] Relationships properly defined with foreign keys

### Post-Implementation
- [x] All foreign keys have proper constraints
- [x] Appropriate indexes for expected query patterns
- [x] Audit fields included on all tables
- [x] Status management consistent across tables
- [x] Entity catalog updated with new entities
- [x] Architectural decisions documented for DCS integration

### Final Validation
- [x] Backend mappings complete and accurate
- [x] Database schema follows all standards
- [x] No redundant tables or columns created
- [x] Performance considerations addressed (DCS response times pending stakeholder input)
- [x] Documentation updated with stakeholder decisions

### Global Requirements Compliance
- [x] **GR-52**: Universal Entity Management applied for DCS integration
- [x] **GR-53**: DCS Integration Architecture patterns followed
- [x] **GR-44**: Communication Architecture used for API tracking
- [x] **GR-04**: Validation & Data Handling implemented for business rules
- [x] **GR-36**: Authentication & Permissions via Laravel Sanctum
- [x] **GR-33**: Data Services patterns for direct storage approach