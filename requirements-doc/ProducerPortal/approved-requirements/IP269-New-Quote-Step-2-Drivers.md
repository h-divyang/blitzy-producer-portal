# IP269-New-Quote-Step-2-Drivers - Complete Requirement (Updated)

---

## **A) WHY – Vision and Purpose**

This step ensures that all relevant driver information is captured and associated with the quote. Proper driver data enables accurate risk assessment, pricing, and policy eligibility. The goal is to create a seamless interface for agents to:

- Add and manage drivers,
- Handle violations and license status,
- Ensure underwriting rules and validations are enforced.

---

## **B) WHAT – Core Requirements**

The purpose of this step is to expedite including, excluding, or removing drivers from the policy. All drivers returned are associated with the address identified in Step 1, indicating there is a relationship to the primary insured.

### **1. Driver List Management**

- Display all drivers with:
    - Name
    - Primary driver tag, if applicable
    - Date of Birth
    - State of Issue License
    - Identification Type Prefix
    - Identification Number
    - Included or Excluded Status
- Add new driver (modal flow)

### **2. Add a Driver Form**

- Fields:
    - Suffix (Optional)
    - First Name
    - Middle Name (Optional)
    - Last Name
    - Date of Birth
    - Gender
    - Marital Status
    - Relationship to Insured
    - Include or Exclude in Policy
    - Driver's License Type
    - Country Licensed
    - Driver License Number
    - State Licensed
    - Employment Status
    - Occupation/Source of Income
    - Employer Name
    - SR-22 Required Status
    - Reason for SR-22
    - Violation Type
    - Violation Date

### **3. Business Rules & Validation**

- If an included driver has their Marital Status set to "Married", there must be another driver on the policy with Marital Status set to "Married", in either an included or excluded driver status
- If an included driver is deemed to be criminally ineligible to be included in a policy, they must be removed or excluded before proceeding

### **4. Save & Navigation**

- Continue button enabled only when:
    - All required fields are completed
- Progress automatically saved on each field update

---

## Entity Analysis

### Entities Involved
| Entity Name | Type | Status | Notes |
|-------------|------|--------|--------|
| driver | Core | Existing | Primary driver entity (from Step 1 DCS data) |
| driver_type | Reference | Existing | Driver classification (included/excluded/removed) |
| map_quote_driver | Map | Existing | Quote-driver relationships |
| relationship_to_insured | Reference | New | Relationship types |
| employment_status | Reference | New | Employment status types |
| occupation | Core | Existing | Occupation information |
| occupation_type | Reference | Existing | Occupation classification |
| violation | Core | New | All available violations |
| violation_type | Reference | New | Violation classification (major/minor/DUI) |
| driver_violations | Map | New | Driver-violation associations |
| map_entity_violation | Map | New | DCS-to-system violation mapping |
| sr22_reason | Reference | New | SR-22 requirement reasons |
| removal_reason | Reference | New | Driver removal reasons |
| DCS_CRIMINAL | External | Existing | Criminal background API |

### New Tables Required
- **relationship_to_insured**: Reference table for driver relationships
- **employment_status**: Employment status reference lookup
- **violation**: All available violations in the system
- **violation_type**: Violation type classification
- **driver_violations**: Individual violation tracking per driver
- **map_entity_violation**: Maps DCS-specific violations to system violations
- **sr22_reason**: SR-22 requirement reasons
- **removal_reason**: Predefined driver removal reasons

### Modifications to Existing Tables
- **driver**: Add relationship_to_insured_id, employment_status_id, sr22_required, sr22_reason_id
- **map_quote_driver**: Use existing driver_type_id for include/exclude/remove status

### Relationships Identified
- driver → relationship_to_insured (many-to-one)
- driver → employment_status (many-to-one)
- driver → occupation (many-to-one)
- driver → violation (many-to-many via driver_violations)
- violation → violation_type (many-to-one)
- driver → sr22_reason (many-to-one)
- map_quote_driver → driver_type (many-to-one)
- map_quote_driver → removal_reason (many-to-one)

---

## Field Mappings (Section C)

### Backend Mappings

#### Driver List Display

##### Household Driver Data Review
- **Backend Mapping**: 
  ```
  get quote.id from current_quote
  -> get map_quote_driver.* by quote.id
  -> join driver on map_quote_driver.driver_id = driver.id
  -> join driver_type on map_quote_driver.driver_type_id = driver_type.id
  -> join license_type on driver.license_type_id = license_type.id
  -> return driver.full_name, driver.date_of_birth, driver.license_number, 
           license_type.name, map_quote_driver.is_primary_driver,
           driver_type.code (included/excluded/removed), driver.dcs_driver_id
  -> group by driver_type.code
  ```

##### Driver Status Management
- **Backend Mapping**:
  ```
  get driver_type.* from driver_type where status_id = :active
  -> filter by code IN ('included', 'excluded', 'removed')
  -> update map_quote_driver.driver_type_id based on selection
  -> if driver_type.code = 'removed': require removal_reason_id
  ```

##### Primary Driver Identification
- **Backend Mapping**:
  ```
  get map_quote_driver.is_primary_driver from map_quote_driver
  -> where quote_id = :quote_id AND is_primary_driver = true
  -> return primary_driver_id, primary_driver_tag
  ```

#### Add/Edit Driver Form

##### Personal Information Fields
- **Backend Mapping**:
  ```
  validate and store driver personal info:
  -> driver.suffix, driver.first_name, driver.middle_name, driver.last_name
  -> driver.date_of_birth, driver.gender_id, driver.marital_status_id
  -> driver.relationship_to_insured_id
  -> create/update driver entity with DCS integration tracking
  ```

##### Relationship to Insured
- **Backend Mapping**:
  ```
  get relationship_to_insured.* from relationship_to_insured where status_id = :active
  -> return relationship_options[] for dropdown
  -> update driver.relationship_to_insured_id on selection
  ```

##### Include/Exclude Status
- **Backend Mapping**:
  ```
  get driver_type.* from driver_type where code IN ('included', 'excluded', 'removed')
  -> update map_quote_driver.driver_type_id
  -> if driver_type.code = 'removed': 
       get removal_reason.* from removal_reason where status_id = :active
       update map_quote_driver.removal_reason_id
  -> trigger business_rule_validation for married driver rule
  ```

##### License Information
- **Backend Mapping**:
  ```
  get license_type.* from license_type where status_id = :active
  -> return license_type_options[]
  -> validate driver.license_number format by license_type.validation_rules
  -> update driver.license_type_id, driver.license_number, driver.license_state
  ```

##### Employment Information
- **Backend Mapping**:
  ```
  get employment_status.* from employment_status where status_id = :active
  -> return employment_status_options[]
  -> update driver.employment_status_id
  -> if occupation required:
       get occupation_type.* from occupation_type where status_id = :active
       create/update occupation record with occupation_type_id, employer_name, job_title
       update driver.occupation_id
  ```

##### SR-22 Requirements
- **Backend Mapping**:
  ```
  if driver.sr22_required = true:
    -> get sr22_reason.* from sr22_reason where status_id = :active
    -> require sr22_reason_id selection
    -> update driver.sr22_reason_id
  else:
    -> set driver.sr22_reason_id = null
  ```

##### Violation Management
- **Backend Mapping**:
  ```
  get violation.* from violation where status_id = :active
  -> join violation_type on violation.violation_type_id = violation_type.id
  -> return violations[] grouped by violation_type.code
  -> create driver_violations records:
     driver_violations.driver_id, driver_violations.violation_id,
     driver_violations.violation_date, driver_violations.description
  -> support multiple violations per driver
  ```

#### Business Rules Validation

##### Married Driver Business Rule
- **Backend Mapping**:
  ```
  get married_drivers = driver.* where marital_status.code = 'married'
  -> join map_quote_driver on driver.id = map_quote_driver.driver_id
  -> join driver_type on map_quote_driver.driver_type_id = driver_type.id
  -> where driver_type.code IN ('included', 'excluded')
  -> count married_drivers
  -> if any married driver AND count < 2:
       return validation_error = 'Married driver requires spouse on policy'
  -> else: return validation_passed = true
  ```

##### Criminal Eligibility Check
- **Backend Mapping**:
  ```
  call DCS_CRIMINAL API for driver criminal background
  -> if criminal_eligible = false:
       get driver_type where code IN ('excluded', 'removed')
       update map_quote_driver.driver_type_id
       if code = 'removed': require removal_reason_id
  -> return criminal_eligibility_result, required_actions[]
  ```

#### Auto-Save and Workflow

##### Field-Level Auto-Save
- **Backend Mapping**:
  ```
  on field_update:
    -> debounce 500ms to prevent excessive API calls
    -> validate field_value
    -> update appropriate entity (driver OR map_quote_driver OR occupation)
    -> return save_status, validation_results
  ```

##### Continue Button Enablement
- **Backend Mapping**:
  ```
  validate all_required_fields_completed:
    -> check all included drivers have complete information
    -> validate business_rules (married driver rule, criminal eligibility)
    -> check no validation_errors exist
    -> return continue_enabled = true/false, blocking_issues[]
  ```

### Implementation Architecture

**Driver Selection and Enhancement Strategy**: Building on Step 1 DCS household data, this step focuses on driver selection from pre-populated results, status assignment (included/excluded/removed), and enhanced data collection for drivers not found in DCS search.

**Business Rules Engine**: Implements real-time validation for married driver rule and criminal eligibility checks using DCS Criminal API integration.

**Manual Addition Workflow**: When household search doesn't find all drivers, provides comprehensive manual driver addition with same data collection patterns established in previous steps.

**Auto-Save with Debouncing**: Field-level auto-save with 500ms debouncing to prevent excessive API calls while maintaining progress across workflow steps.

### Integration Specifications

#### DCS Criminal Background Integration

**Entity Type**: DCS_CRIMINAL (Universal Entity Management)  
**Provider**: Data Collection Services  
**Endpoint**: DCS Criminal Background API (GR-53)

#### Performance & Monitoring

**Response Time Targets**:
- Driver list display: < 300ms
- Criminal eligibility check: < 10 seconds
- Business rule validation: < 200ms
- Auto-save operations: < 500ms
- Driver search/filter: < 100ms

**Auto-Save Strategy**:
- 500ms debouncing for field updates
- Optimistic UI updates with rollback on failure
- Progress persistence across workflow steps

#### Error Handling

**Criminal Check Fallback**: Manual review required when DCS Criminal API unavailable
**Business Rule Enforcement**: Real-time validation with clear blocking messages
**Auto-Save Resilience**: Retry failed saves with exponential backoff

---

## **D) User Experience (UX) & Flows**

### **1. Driver List Review Flow (Building on Step 1 DCS Data)**

1. Display household drivers from Step 1 DCS search results
2. Show drivers grouped by status: Included, Excluded, Removed
3. Allow status changes and additional information collection
4. Auto-trigger criminal eligibility checks for included drivers

### **2. Add Driver Manually Flow**

1. Click "Add Driver"
2. Modal opens – user enters personal info
3. Continue to license section
4. Add violations if any
5. Auto-trigger household search refresh (may find additional "New" drivers)
6. Save and return to driver list

### **3. Edit Driver Flow**

1. Click on a driver row
2. Modal opens, requiring additional information:
    1. Gather gender, marital status, and relationship to insured, and if the driver will be included or excluded on the policy
    2. If removing from the policy, status of driver will be set to removed, and the reason for removal must be included
3. Make changes and save
4. Auto-save on field updates

### **4. Driver List Presentation**

- Drivers will be presented in three sections: Included, Excluded, or Removed
    - If the system has been configured without the ability to remove drivers, this section will be removed and the CTA will shift up to eliminate unexpected white space
- The user can search the list using the search bar to expedite finding the relevant drivers
- The list of drivers will be paginated, and the user can click to show additional drivers
- The household search will run again when the user adds drivers manually, which may result in additional new drivers being returned
    - These drivers will be returned with a "New" tag to differentiate from other drivers in the list

---

## API Specifications

### Endpoints Required
```http
GET    /api/v1/quotes/{id}/drivers           # List all drivers for quote (includes DCS data)
GET    /api/v1/quotes/{id}/household-drivers # Get DCS household data from Step 1
POST   /api/v1/quotes/{id}/drivers           # Add new driver (manual addition)
PUT    /api/v1/quotes/{id}/drivers/{driverId} # Update driver status/details
DELETE /api/v1/quotes/{id}/drivers/{driverId} # Remove driver
PUT    /api/v1/quotes/{id}/drivers/{driverId}/status # Update driver type
POST   /api/v1/quotes/{id}/drivers/validate   # Validate business rules
GET    /api/v1/drivers/{id}/criminal-check    # Check criminal eligibility
POST   /api/v1/quotes/{id}/refresh-household  # Refresh household search
GET    /api/v1/reference/relationships        # Get relationship options
GET    /api/v1/reference/employment-status    # Get employment status options
GET    /api/v1/reference/violations           # Get available violations
GET    /api/v1/reference/violation-types      # Get violation type options
GET    /api/v1/reference/sr22-reasons         # Get SR-22 reason options
GET    /api/v1/reference/removal-reasons      # Get removal reason options
```

### Real-time Updates
```javascript
// WebSocket channels
private-quote.{id}.drivers                   # Driver data updates
private-quote.{id}.business-rules           # Business rule validation updates
```

---

## Database Schema (Section E)

### New Core Tables

#### violation
```sql
CREATE TABLE violation (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  
  -- Business columns
  code VARCHAR(50) UNIQUE NOT NULL,
  name VARCHAR(200) NOT NULL,
  description TEXT NULL,
  violation_type_id BIGINT UNSIGNED NOT NULL,
  points INT DEFAULT 0,
  dcs_code VARCHAR(50) NULL,
  
  -- Status and audit
  status_id BIGINT UNSIGNED NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (violation_type_id) REFERENCES violation_type(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  
  -- Indexes
  INDEX idx_code (code),
  INDEX idx_violation_type (violation_type_id),
  INDEX idx_dcs_code (dcs_code),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### New Reference Tables

#### relationship_to_insured
```sql
CREATE TABLE relationship_to_insured (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) UNIQUE NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT NULL,
  status_id BIGINT UNSIGNED NOT NULL,
  
  -- Audit fields
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (status_id) REFERENCES status(id),
  
  -- Indexes
  INDEX idx_code (code),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### employment_status
```sql
CREATE TABLE employment_status (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) UNIQUE NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT NULL,
  status_id BIGINT UNSIGNED NOT NULL,
  
  -- Audit fields
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (status_id) REFERENCES status(id),
  
  -- Indexes
  INDEX idx_code (code),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### violation_type
```sql
CREATE TABLE violation_type (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) UNIQUE NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT NULL,
  severity_level ENUM('minor', 'major', 'DUI', 'serious') NOT NULL,
  status_id BIGINT UNSIGNED NOT NULL,
  
  -- Audit fields
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (status_id) REFERENCES status(id),
  
  -- Indexes
  INDEX idx_code (code),
  INDEX idx_severity (severity_level),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### sr22_reason
```sql
CREATE TABLE sr22_reason (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) UNIQUE NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT NULL,
  status_id BIGINT UNSIGNED NOT NULL,
  
  -- Audit fields
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (status_id) REFERENCES status(id),
  
  -- Indexes
  INDEX idx_code (code),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### removal_reason
```sql
CREATE TABLE removal_reason (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) UNIQUE NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT NULL,
  status_id BIGINT UNSIGNED NOT NULL,
  
  -- Audit fields
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (status_id) REFERENCES status(id),
  
  -- Indexes
  INDEX idx_code (code),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### New Relationship Tables

#### driver_violations
```sql
CREATE TABLE driver_violations (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  
  -- Business columns
  driver_id BIGINT UNSIGNED NOT NULL,
  violation_id BIGINT UNSIGNED NOT NULL,
  violation_date DATE NOT NULL,
  description TEXT NULL,
  disposition VARCHAR(100) NULL,
  court_date DATE NULL,
  fine_amount DECIMAL(10,2) NULL,
  
  -- Status and audit
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (driver_id) REFERENCES driver(id) ON DELETE CASCADE,
  FOREIGN KEY (violation_id) REFERENCES violation(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  -- Indexes
  INDEX idx_driver (driver_id),
  INDEX idx_violation (violation_id),
  INDEX idx_violation_date (violation_date),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### map_entity_violation
```sql
CREATE TABLE map_entity_violation (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  
  -- Business columns
  source_entity_id BIGINT UNSIGNED NOT NULL,
  system_violation_id BIGINT UNSIGNED NOT NULL,
  external_code VARCHAR(100) NOT NULL,
  
  -- Status and audit
  status_id BIGINT UNSIGNED NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (source_entity_id) REFERENCES entity(id),
  FOREIGN KEY (system_violation_id) REFERENCES violation(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  
  -- Constraints
  UNIQUE KEY unique_entity_violation (source_entity_id, external_code),
  
  -- Indexes
  INDEX idx_source_entity (source_entity_id),
  INDEX idx_system_violation (system_violation_id),
  INDEX idx_external_code (external_code),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### Modified Tables

#### ALTER TABLE driver
```sql
-- Add relationship and compliance fields
ALTER TABLE driver 
ADD COLUMN relationship_to_insured_id BIGINT UNSIGNED NULL,
ADD COLUMN employment_status_id BIGINT UNSIGNED NULL,
ADD COLUMN occupation_id BIGINT UNSIGNED NULL,
ADD COLUMN sr22_required BOOLEAN DEFAULT FALSE,
ADD COLUMN sr22_reason_id BIGINT UNSIGNED NULL,
ADD COLUMN criminal_eligible BOOLEAN NULL,
ADD COLUMN criminal_check_date TIMESTAMP NULL,
ADD COLUMN dcs_criminal_correlation_id VARCHAR(100) NULL;

-- Add foreign key constraints
ALTER TABLE driver
ADD CONSTRAINT fk_driver_relationship 
FOREIGN KEY (relationship_to_insured_id) REFERENCES relationship_to_insured(id);

ALTER TABLE driver
ADD CONSTRAINT fk_driver_employment_status 
FOREIGN KEY (employment_status_id) REFERENCES employment_status(id);

ALTER TABLE driver
ADD CONSTRAINT fk_driver_occupation 
FOREIGN KEY (occupation_id) REFERENCES occupation(id);

ALTER TABLE driver
ADD CONSTRAINT fk_driver_sr22_reason 
FOREIGN KEY (sr22_reason_id) REFERENCES sr22_reason(id);

-- Add indexes
ALTER TABLE driver
ADD INDEX idx_relationship (relationship_to_insured_id),
ADD INDEX idx_employment_status (employment_status_id),
ADD INDEX idx_occupation (occupation_id),
ADD INDEX idx_sr22_required (sr22_required),
ADD INDEX idx_criminal_eligible (criminal_eligible),
ADD INDEX idx_criminal_check_date (criminal_check_date);
```

#### ALTER TABLE map_quote_driver
```sql
-- Add removal reason tracking
ALTER TABLE map_quote_driver 
ADD COLUMN removal_reason_id BIGINT UNSIGNED NULL;

-- Add foreign key constraint
ALTER TABLE map_quote_driver
ADD CONSTRAINT fk_map_quote_driver_removal_reason 
FOREIGN KEY (removal_reason_id) REFERENCES removal_reason(id);

-- Add index
ALTER TABLE map_quote_driver
ADD INDEX idx_removal_reason (removal_reason_id);

-- Note: driver_type_id already exists for included/excluded/removed status
-- No need for include_status ENUM as driver_type table handles this
```

---

## Implementation Notes

### Dependencies
- Step 1 Primary Insured completion (DCS household data available)
- DCS Criminal API integration (GR-53)
- Business rules engine for validation
- Reference data setup (relationships, employment, violations, SR-22, removal reasons)
- Existing occupation and occupation_type tables

### Migration Considerations
- Existing driver data compatibility with new fields
- Ensure driver_type table has included/excluded/removed types
- Violation data migration from existing systems
- Map DCS violation codes to system violations

### Performance Considerations
- Driver list pagination for large households
- Criminal eligibility check batching for multiple drivers
- Business rule validation optimization
- Search functionality for driver lists

---

## Quality Checklist

### Global Requirements Compliance
- [x] **GR-52**: Universal Entity Management applied for DCS Criminal integration
- [x] **GR-53**: DCS Integration Architecture patterns followed
- [x] **GR-04**: Validation & Data Handling implemented for business rules
- [x] **GR-18**: Workflow Requirements for driver status management
- [x] **GR-36**: Authentication & Permissions via Laravel Sanctum
- [x] **GR-33**: Data Services patterns for auto-save and caching