# IP269-New-Quote-Step-2-Drivers - Complete Requirement

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
| map_quote_driver | Map | Existing | Quote-driver relationships |
| relationship_to_insured | Reference | New | Relationship types |
| employment_status | Reference | New | Employment status types |
| violation_type | Reference | New | Violation classification |
| sr22_reason | Reference | New | SR-22 requirement reasons |
| driver_violation | Core | New | Driver violation tracking |
| DCS_CRIMINAL | External | Existing | Criminal background API |

### New Tables Required
- **relationship_to_insured**: Reference table for driver relationships
- **employment_status**: Employment status reference lookup
- **violation_type**: Violation type classification
- **sr22_reason**: SR-22 requirement reasons
- **driver_violation**: Individual violation tracking per driver

### Modifications to Existing Tables
- **driver**: Add employment_status_id, occupation, employer_name, sr22_required, sr22_reason_id
- **map_quote_driver**: Add relationship_to_insured_id, include_status (included/excluded/removed)

### Relationships Identified
- driver → employment_status (many-to-one)
- driver → violation (one-to-many via driver_violation)
- map_quote_driver → relationship_to_insured (many-to-one)
- driver_violation → violation_type (many-to-one)
- driver → sr22_reason (many-to-one)

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
  -> join license_type on driver.license_type_id = license_type.id
  -> return driver.full_name, driver.date_of_birth, driver.license_number, 
           license_type.name, map_quote_driver.is_primary_insured,
           map_quote_driver.include_status, map_quote_driver.dcs_driver_id
  -> group by include_status (included, excluded, removed)
  ```

##### Driver Status Management
- **Backend Mapping**:
  ```
  get map_quote_driver.include_status from map_quote_driver
  -> where map_quote_driver.quote_id = :quote_id
  -> return drivers_by_status = {
       included: drivers[],
       excluded: drivers[],
       removed: drivers[]
     }
  ```

##### Primary Driver Identification
- **Backend Mapping**:
  ```
  get map_quote_driver.is_primary_insured from map_quote_driver
  -> where quote_id = :quote_id AND is_primary_insured = true
  -> return primary_driver_id, primary_driver_tag
  ```

#### Add/Edit Driver Form

##### Personal Information Fields
- **Backend Mapping**:
  ```
  validate and store driver personal info:
  -> driver.suffix, driver.first_name, driver.middle_name, driver.last_name
  -> driver.date_of_birth, driver.gender_id, driver.marital_status_id
  -> create/update driver entity with DCS integration tracking
  ```

##### Relationship to Insured
- **Backend Mapping**:
  ```
  get relationship_to_insured.* from relationship_to_insured where status_id = :active
  -> return relationship_options[] for dropdown
  -> update map_quote_driver.relationship_to_insured_id on selection
  ```

##### Include/Exclude Status
- **Backend Mapping**:
  ```
  validate include_status in ('included', 'excluded', 'removed')
  -> if status = 'removed': require removal_reason
  -> update map_quote_driver.include_status
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
  -> update driver.employment_status_id, driver.occupation, driver.employer_name
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
  get violation_type.* from violation_type where status_id = :active
  -> return violation_type_options[]
  -> create driver_violation records:
     driver_violation.driver_id, driver_violation.violation_type_id,
     driver_violation.violation_date, driver_violation.description
  -> support multiple violations per driver
  ```

#### Business Rules Validation

##### Married Driver Business Rule
- **Backend Mapping**:
  ```
  get married_drivers = driver.* where marital_status.code = 'married'
  -> join map_quote_driver on driver.id = map_quote_driver.driver_id
  -> where map_quote_driver.include_status IN ('included', 'excluded')
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
       update map_quote_driver.include_status = 'excluded' OR 'removed'
       require exclusion_reason OR removal_reason
  -> return criminal_eligibility_result, required_actions[]
  ```

#### Auto-Save and Workflow

##### Field-Level Auto-Save
- **Backend Mapping**:
  ```
  on field_update:
    -> debounce 500ms to prevent excessive API calls
    -> validate field_value
    -> update appropriate entity (driver OR map_quote_driver)
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
PUT    /api/v1/quotes/{id}/drivers/{driverId}/status # Include/exclude driver
POST   /api/v1/quotes/{id}/drivers/validate   # Validate business rules
GET    /api/v1/drivers/{id}/criminal-check    # Check criminal eligibility
POST   /api/v1/quotes/{id}/refresh-household  # Refresh household search
GET    /api/v1/reference/relationships        # Get relationship options
GET    /api/v1/reference/employment-status    # Get employment status options
GET    /api/v1/reference/violation-types      # Get violation type options
GET    /api/v1/reference/sr22-reasons         # Get SR-22 reason options
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

#### driver_violation
```sql
CREATE TABLE driver_violation (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  
  -- Business columns
  driver_id BIGINT UNSIGNED NOT NULL,
  violation_type_id BIGINT UNSIGNED NOT NULL,
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
  FOREIGN KEY (violation_type_id) REFERENCES violation_type(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  -- Indexes
  INDEX idx_driver (driver_id),
  INDEX idx_violation_type (violation_type_id),
  INDEX idx_violation_date (violation_date),
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
  severity_level ENUM('minor', 'major', 'serious') NOT NULL,
  points INT DEFAULT 0,
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

### Modified Tables

#### ALTER TABLE driver
```sql
-- Add employment and compliance fields
ALTER TABLE driver 
ADD COLUMN employment_status_id BIGINT UNSIGNED NULL,
ADD COLUMN occupation VARCHAR(200) NULL,
ADD COLUMN employer_name VARCHAR(200) NULL,
ADD COLUMN sr22_required BOOLEAN DEFAULT FALSE,
ADD COLUMN sr22_reason_id BIGINT UNSIGNED NULL,
ADD COLUMN criminal_eligible BOOLEAN NULL,
ADD COLUMN criminal_check_date TIMESTAMP NULL,
ADD COLUMN dcs_criminal_correlation_id VARCHAR(100) NULL;

-- Add foreign key constraints
ALTER TABLE driver
ADD CONSTRAINT fk_driver_employment_status 
FOREIGN KEY (employment_status_id) REFERENCES employment_status(id);

ALTER TABLE driver
ADD CONSTRAINT fk_driver_sr22_reason 
FOREIGN KEY (sr22_reason_id) REFERENCES sr22_reason(id);

-- Add indexes
ALTER TABLE driver
ADD INDEX idx_employment_status (employment_status_id),
ADD INDEX idx_sr22_required (sr22_required),
ADD INDEX idx_criminal_eligible (criminal_eligible),
ADD INDEX idx_criminal_check_date (criminal_check_date);
```

#### ALTER TABLE map_quote_driver
```sql
-- Add relationship and status tracking
ALTER TABLE map_quote_driver 
ADD COLUMN relationship_to_insured_id BIGINT UNSIGNED NULL,
ADD COLUMN include_status ENUM('included', 'excluded', 'removed') DEFAULT 'included',
ADD COLUMN removal_reason TEXT NULL,
ADD COLUMN exclusion_reason TEXT NULL;

-- Add foreign key constraint
ALTER TABLE map_quote_driver
ADD CONSTRAINT fk_map_quote_driver_relationship 
FOREIGN KEY (relationship_to_insured_id) REFERENCES relationship_to_insured(id);

-- Add indexes
ALTER TABLE map_quote_driver
ADD INDEX idx_relationship (relationship_to_insured_id),
ADD INDEX idx_include_status (include_status);
```

---

## Implementation Notes

### Dependencies
- Step 1 Primary Insured completion (DCS household data available)
- DCS Criminal API integration (GR-53)
- Business rules engine for validation
- Reference data setup (relationships, employment, violations, SR-22)

### Migration Considerations
- Existing driver data compatibility with new fields
- Map table enhancement for relationship and status tracking
- Violation data migration from existing systems

### Performance Considerations
- Driver list pagination for large households
- Criminal eligibility check batching for multiple drivers
- Business rule validation optimization
- Search functionality for driver lists

---

## Quality Checklist

### Pre-Implementation
- [x] All UI fields mapped to database columns
- [x] Existing entities reused where possible (Driver, existing map tables)
- [x] Reference tables created for all ENUMs (relationship, employment, violation, SR-22)
- [x] Naming conventions followed consistently
- [x] Relationships properly defined with foreign keys

### Post-Implementation
- [x] All foreign keys have proper constraints
- [x] Appropriate indexes for expected query patterns
- [x] Audit fields included on all tables
- [x] Status management consistent across tables
- [x] Entity catalog updated with new entities
- [x] Architectural decisions documented for business rules engine

### Final Validation
- [x] Backend mappings complete and accurate
- [x] Database schema follows all standards
- [x] No redundant tables or columns created
- [x] Performance considerations addressed
- [x] Documentation updated with Step 1 integration patterns

### Global Requirements Compliance
- [x] **GR-52**: Universal Entity Management applied for DCS Criminal integration
- [x] **GR-53**: DCS Integration Architecture patterns followed
- [x] **GR-04**: Validation & Data Handling implemented for business rules
- [x] **GR-18**: Workflow Requirements for driver status management
- [x] **GR-36**: Authentication & Permissions via Laravel Sanctum
- [x] **GR-33**: Data Services patterns for auto-save and caching