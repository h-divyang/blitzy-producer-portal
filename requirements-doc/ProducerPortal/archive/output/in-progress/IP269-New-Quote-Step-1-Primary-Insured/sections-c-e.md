# IP269 - New Quote Step 1: Primary Insured - Sections C & E

## Pre-Analysis Checklist

### Initial Review
- [x] Read base requirement document completely
- [x] Identify all UI elements and data fields mentioned
- [x] Note workflow states and transitions described
- [x] List relationships to existing entities

### Global Requirements Alignment
- [x] Review applicable global requirements
- [x] GR 01 (IAM), GR 18 (Workflow), GR 33 (Data Services), GR 36 (Authentication)
- [x] GR 44 (Communication), GR 48 (External Integrations), GR 52 (Universal Entity)
- [x] GR 53 (DCS Integration) for driver verification patterns

### Cross-Reference Check
- [x] Review entity catalog for reusable entities
- [x] Check architectural decisions for relevant patterns
- [x] Review related requirements for shared entities
- [x] Ensure patterns align with global standards

### Compliance Verification
- [x] Verify alignment with CLAUDE.md standards
- [x] Check naming convention compliance
- [x] Validate reference table approach for ENUMs
- [x] Ensure status_id usage instead of is_active

---

## Entity Analysis

### Entities Involved
| Entity Name | Type | Status | Notes |
|-------------|------|--------|--------|
| quote | Core | Existing | Main business entity with state management |
| driver | Core | Existing | Driver identity with is_named_insured field |
| license | Supporting | Existing | License verification with state/type references |
| program | Reference | Existing | Program selection with date validation |
| entity | Universal | Existing | DCS integration management |
| communication | Universal | Existing | External API tracking |
| name | Supporting | Existing | Reusable person name entity |
| address | Supporting | Existing | Reusable address entity |

### New Tables Required
- **None** - All functionality achieved through existing entities

### Modifications to Existing Tables
- **None** - All required fields already present

### Relationships Identified
- quote → program (many-to-one)
- quote → driver (many-to-many via map_quote_driver)
- driver → license (one-to-many)
- driver → name (many-to-one)
- driver → address (many-to-many via map_driver_address)
- entity → communication (polymorphic via source/target)

---

## Field Mappings (Section C)

### Backend Mappings

#### Effective Date & Program Selection

##### Effective Date Field
- **Backend Mapping**:
  ```
  get program.id from program 
  -> get program_effective_date_rule by program.id
  -> validate effective_date <= (current_date + program_effective_date_rule.max_days_future)
  -> return validation_result, available_programs
  ```

##### Program Dropdown
- **Backend Mapping**:
  ```
  get program.id from program
  -> where program.status_id = :active_status_id
  -> where program.effective_date <= :selected_effective_date
  -> where program.expiration_date >= :selected_effective_date
  -> return program.id, program.name, program.code
  ```

#### Search & Match Functionality

##### License Type Selection
- **Backend Mapping**:
  ```
  get license_type.id from license_type
  -> where license_type.status_id = :active_status_id
  -> return license_type.id, license_type.code, license_type.name
  ```

##### US License Search
- **Backend Mapping**:
  ```
  get driver.id from driver
  -> join license on driver.id = license.driver_id
  -> where license.license_number = :license_number
  -> where license.state_id = :state_id
  -> where license.status_id = :active_status_id
  -> where driver.status_id = :active_status_id
  -> return driver.id, license.id, match_confidence_score
  ```

##### Non-US License Search
- **Backend Mapping**:
  ```
  get driver.id from driver
  -> join name on driver.name_id = name.id
  -> join map_driver_address on driver.id = map_driver_address.driver_id
  -> join address on map_driver_address.address_id = address.id
  -> where name.first_name LIKE :first_name
  -> where name.last_name LIKE :last_name
  -> where address.street_address LIKE :street_address
  -> where address.city LIKE :city
  -> where address.state_id = :state_id
  -> where address.country_id = :country_id
  -> where address.zip_code = :zip_code
  -> where driver.status_id = :active_status_id
  -> return driver.id, name.full_name, address.formatted_address, match_confidence_score
  ```

#### Search Results Display

##### Profile Card Data
- **Backend Mapping**:
  ```
  get driver.id from driver
  -> join name on driver.name_id = name.id
  -> join map_driver_address on driver.id = map_driver_address.driver_id
  -> join address on map_driver_address.address_id = address.id
  -> where map_driver_address.is_primary = true
  -> where driver.status_id = :active_status_id
  -> return name.first_name, name.middle_name, name.last_name, 
           driver.date_of_birth, address.street_address, address.city, 
           address.state_name, address.zip_code
  ```

#### Quote Creation

##### New Quote Record
- **Backend Mapping**:
  ```
  create quote
  -> set quote.program_id = :selected_program_id
  -> set quote.effective_date = :selected_effective_date
  -> set quote.status_id = :draft_status_id
  -> set quote.created_by = :current_user_id
  -> return quote.id
  ```

##### Associate Named Insured
- **Backend Mapping**:
  ```
  create map_quote_driver
  -> set map_quote_driver.quote_id = :quote_id
  -> set map_quote_driver.driver_id = :selected_driver_id
  -> set map_quote_driver.status_id = :active_status_id
  -> update driver set is_named_insured = true where id = :selected_driver_id
  -> return map_quote_driver.id
  ```

### Implementation Architecture

The Primary Insured step leverages the universal entity management architecture for external integrations, particularly DCS driver verification. The implementation follows these key patterns:

1. **Entity Management**: Uses existing entity/entity_type tables for DCS integration
2. **Communication Tracking**: All external API calls tracked in communication table
3. **Configuration Hierarchy**: System → Program → Entity scope resolution
4. **Component Security**: Backend-frontend-security association for permissions

### Integration Specifications

#### DCS Driver Verification Integration

##### API Integration Points
- **Endpoint**: DCS Household Drivers API v2.7
- **Entity Type**: `DCS_HOUSEHOLD_DRIVERS`
- **Authentication**: HTTP Basic (Account:User:Password)
- **Purpose**: Real-time driver verification during quote creation
- **For complete DCS specifications** → See GR-53

##### Configuration Management
Configuration follows the universal three-level hierarchy:
- **System Level**: Default DCS settings (timeout, retry attempts)
- **Program Level**: Environment-specific credentials and overrides
- **Entity Level**: API-specific endpoints and capabilities
- **Resolution**: Entity overrides Program overrides System

##### Security Implementation
Component-based security for DCS access:
```
Component: DCS_INTEGRATION
Actions: DRIVER_VERIFICATION, VEHICLE_LOOKUP, CRIMINAL_CHECK
Scope: Program-level permissions
Groups: Basic Users (driver only), Premium Users (all features)
```

##### Performance & Monitoring
- **Response Time Target**: < 5 seconds (95th percentile)
- **Circuit Breaker**: Opens after 5 consecutive failures
- **Retry Logic**: 3 attempts with exponential backoff
- **Caching**: 5-minute TTL for successful verifications
- **Monitoring**: Real-time metrics and alerting

##### Error Handling
- **Graceful Degradation**: Allow manual entry if DCS unavailable
- **Circuit Breaker Pattern**: Prevents cascade failures
- **Correlation IDs**: Track multi-step workflows
- **Audit Logging**: Complete trail with PII masking

#### DCS Integration Workflow
```
1. User enters license number and state
2. System searches local driver records
3. If no match, initiate DCS verification:
   a. Get DCS entity configuration
   b. Create communication record with correlation ID
   c. Call DCS API with circuit breaker protection
   d. Store response in communication table
   e. Process and return verification results
4. Display results or allow manual entry
```

---

## API Specifications

### Endpoints Required
```http
# Quote Management
GET    /api/v1/quotes                    # List quotes with pagination
POST   /api/v1/quotes                    # Create new quote
GET    /api/v1/quotes/{id}              # Get quote details
PUT    /api/v1/quotes/{id}              # Update quote

# Program Selection
GET    /api/v1/programs                  # List available programs
GET    /api/v1/programs/available        # Programs for effective date

# Driver Search
POST   /api/v1/drivers/search           # Search existing drivers
POST   /api/v1/drivers/verify           # DCS verification

# Reference Data
GET    /api/v1/license-types            # License type options
GET    /api/v1/states                   # State options
GET    /api/v1/countries                # Country options
```

### Real-time Updates
```javascript
// WebSocket channels
private-quote.{quote_id}                 # Quote updates
private-driver-verification.{correlation_id}  # DCS verification status
```

---

## Database Schema (Section E)

### New Core Tables
**None Required** - All functionality uses existing tables

### New Reference Tables
**None Required** - All reference tables already exist

### New Relationship Tables
**None Required** - map_quote_driver already exists

### Modified Tables
**None Required** - All tables have required fields

### Existing Table Validation

#### Core Tables

##### quote
```sql
-- Verified existing fields:
-- program_id BIGINT UNSIGNED NOT NULL
-- effective_date DATE NOT NULL
-- status_id BIGINT UNSIGNED NOT NULL
-- created_by BIGINT UNSIGNED NOT NULL
-- All audit fields present
-- Foreign keys and indexes confirmed
```

##### driver
```sql
-- Verified existing fields:
-- name_id BIGINT UNSIGNED NOT NULL
-- date_of_birth DATE
-- is_named_insured BOOLEAN DEFAULT FALSE
-- status_id BIGINT UNSIGNED NOT NULL
-- All audit fields present
-- Foreign keys and indexes confirmed
```

##### license
```sql
-- Verified existing fields:
-- driver_id BIGINT UNSIGNED NOT NULL
-- license_number VARCHAR(50)
-- license_type_id BIGINT UNSIGNED NOT NULL
-- state_id BIGINT UNSIGNED NOT NULL
-- status_id BIGINT UNSIGNED NOT NULL
-- All audit fields present
-- Foreign keys and indexes confirmed
```

#### Universal Entity Tables

##### entity
```sql
-- Verified for DCS integration:
-- entity_type_id BIGINT UNSIGNED NOT NULL
-- code VARCHAR(50) UNIQUE NOT NULL
-- name VARCHAR(100) NOT NULL
-- metadata JSON
-- status_id BIGINT UNSIGNED NOT NULL
-- All audit fields present
```

##### communication
```sql
-- Verified for API tracking:
-- source_type VARCHAR(50) NOT NULL
-- source_id BIGINT UNSIGNED NOT NULL
-- target_type VARCHAR(50) NOT NULL
-- target_id BIGINT UNSIGNED NOT NULL
-- correlation_id VARCHAR(100)
-- request_data JSON
-- response_data JSON
-- status_id BIGINT UNSIGNED NOT NULL
-- All audit fields present
```

#### Reference Tables

##### license_type
```sql
-- Verified codes exist:
-- 'US_LICENSE' - US Driver's License
-- 'INTERNATIONAL' - International License
-- 'PASSPORT' - Passport ID
-- All with active status_id
```

##### program
```sql
-- Verified fields:
-- code VARCHAR(50) UNIQUE NOT NULL
-- name VARCHAR(100) NOT NULL
-- effective_date DATE NOT NULL
-- expiration_date DATE NOT NULL
-- All audit fields present
```

### Performance Indexes
All required indexes already exist:
- quote: PRIMARY KEY, idx_program, idx_status, idx_effective_date
- driver: PRIMARY KEY, idx_name, idx_status, idx_named_insured
- license: PRIMARY KEY, idx_driver, idx_license_number, idx_state
- entity: PRIMARY KEY, idx_entity_type, idx_code, idx_status
- communication: PRIMARY KEY, idx_source, idx_target, idx_correlation

---

## Implementation Notes

### Dependencies
- Existing entity catalog entities
- DCS entity types already configured
- Communication tracking infrastructure
- Component-based security system

### Migration Considerations
- No data migration required
- All tables and infrastructure exist
- DCS credentials need vault configuration

### Performance Considerations
- License number search uses existing index
- DCS calls use circuit breaker pattern
- Caching reduces repeated API calls
- Correlation IDs enable workflow tracking

---

## Quality Checklist

### Pre-Implementation
- [x] All UI fields mapped to database columns
- [x] Existing entities reused (100% reuse)
- [x] Reference tables already exist
- [x] Naming conventions followed
- [x] Relationships properly defined

### Post-Implementation
- [x] All foreign keys have proper constraints
- [x] Appropriate indexes already exist
- [x] Audit fields on all tables
- [x] Status management consistent
- [x] Entity catalog already complete
- [x] Architectural decisions documented

### Final Validation
- [x] Backend mappings complete and accurate
- [x] Database schema uses only existing tables
- [x] No redundant tables or columns created
- [x] Performance considerations addressed
- [x] Integration specifications included in Section C
- [x] Global requirements properly referenced