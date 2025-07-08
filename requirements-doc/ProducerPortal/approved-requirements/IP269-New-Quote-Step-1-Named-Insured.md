# IP269-New-Quote-Step-1-Named-Insured - Complete Requirement (Updated)

---

## **A) WHY – Vision and Purpose**

This section is designed to review **search results for the chosen primary insured** and supplement their profile with any **additional verified or missing customer information**. The purpose is to ensure that all relevant personal data is:

- Confirmed as accurate
- Correctly tied to the selected policy
- Enriched using third-party data sources when needed

This improves data integrity for quoting, binding, and compliance, and supports the insurance agent's ability to confidently move forward with the policy.

---

## **B) WHAT – Core Requirements**

### 1. **Primary Insured Search Results**

- Display returned data for the given individual, or allow the user to key all information manually if no result is returned
- In the case where an individual is returned, the following fields will be presented as read-only: Suffix, First Name, Middle Name, Last Name, Date of Birth, License Number, State Issuer for License, and Country

### 2. **Customer Info Panels**

- Additional sections to capture:
    - Address
    - Gender
    - Marital Status
    - Primary Phone Number
    - Alternate Phone Number
    - Email Address
    - Notification Preference
    - Prior Insurance Status
        - Has the primary insured had prior insurance? If yes, show:
            - Prior Insurance Company
            - Prior Insurance Expiration Date
            - Number of Months Insured with Prior Insurance
    - Eligible Discounts
    - Housing (Residence Type)

### 3. **Modals & Overlays**

- **"Are you sure?" confirmation modal** when switching search results or redoing a search.
- Error/warning messaging when required fields are incomplete or data mismatches occur.
    - For example, if the user selects 'Enroll in Paperless' discount without a valid email address. The email address fields will be outlined in red and must be populated before proceeding.

### 4. **Navigation**

- On mobile, form fields become stacked sections.

---

## Entity Analysis

### Entities Involved
| Entity Name | Type | Status | Notes |
|-------------|------|--------|--------|
| driver | Core | Existing | Primary insured driver (from Step 1) |
| address | Supporting | Existing | Address information |
| phone | Supporting | Existing | Phone number storage |
| email | Supporting | Existing | Email address storage |
| communication_preference | Supporting | Existing | Notification preferences |
| prior_insurance | Core | New | Prior insurance tracking |
| discount | Core | New | Specific discount instances |
| discount_type | Reference | New | Available discount types |
| quote_discount | Map | New | Quote-specific discount application |
| gender | Reference | New | Gender reference table |
| marital_status | Reference | New | Marital status reference table |
| residence_type | Reference | New | Residence type reference table |
| occupation | Core | New | Occupation information |
| occupation_type | Reference | New | Occupation type classification |

### New Tables Required
- **prior_insurance**: Track prior insurance history
- **gender**: Gender reference lookup
- **marital_status**: Marital status reference lookup
- **residence_type**: Residence type reference lookup (replaces housing_type)
- **discount**: Specific discount information
- **discount_type**: Available discount types and requirements
- **quote_discount**: Quote-specific discount application tracking
- **occupation**: Occupation details
- **occupation_type**: Occupation classification

### Modifications to Existing Tables
- **driver**: Add gender_id, marital_status_id, residence_type_id, occupation_id references
- **quote**: Add prior_insurance_id reference

### Relationships Identified
- driver → gender (many-to-one)
- driver → marital_status (many-to-one)
- driver → residence_type (many-to-one)
- driver → occupation (many-to-one)
- occupation → occupation_type (many-to-one)
- quote → prior_insurance (one-to-one)
- quote → discount (many-to-many via quote_discount)
- discount → discount_type (many-to-one)
- driver → phone (many-to-many via map_driver_phone)
- driver → email (many-to-many via map_driver_email)
- driver → communication_preference (one-to-many)

---

## Field Mappings (Section C)

### Backend Mappings

#### Primary Insured Search Results Display

##### Read-Only Personal Information
- **Backend Mapping**: 
  ```
  get driver.id from map_quote_driver where quote_id = :quote_id
  -> join driver on map_quote_driver.driver_id = driver.id
  -> where driver.is_named_insured = true
  -> get driver.* by driver.id
  -> return driver.suffix, driver.first_name, driver.middle_name, driver.last_name, 
           driver.date_of_birth, driver.license_number, license_type.name, driver.license_state
  -> display as read_only = true with DCS data source indicator
  ```

##### DCS Data Source Indicator
- **Backend Mapping**:
  ```
  get driver.dcs_driver_id from driver
  -> if dcs_driver_id IS NOT NULL:
       return data_source = 'DCS', editable = false, override_available = true
     else:
       return data_source = 'Manual', editable = true, override_available = false
  ```

#### Customer Information Panels

##### Address Information
- **Backend Mapping**:
  ```
  get driver.id from primary_insured_driver
  -> get map_driver_address by driver.id where address_type_id = 'primary'
  -> get address.* by map_driver_address.address_id
  -> return address.street, address.city, address.state, address.zip, address.country
  -> if DCS_populated: display with edit_override_option = true
  ```

##### Gender Selection
- **Backend Mapping**:
  ```
  get gender.* from gender where status_id = :active_status
  -> return gender.code, gender.name for dropdown
  -> update driver.gender_id on selection
  ```

##### Marital Status
- **Backend Mapping**:
  ```
  get marital_status.* from marital_status where status_id = :active_status
  -> return marital_status.code, marital_status.name for dropdown
  -> update driver.marital_status_id on selection
  ```

##### Phone Numbers (Primary & Alternate)
- **Backend Mapping**:
  ```
  get map_driver_phone.* from map_driver_phone where driver_id = :driver_id
  -> join phone on map_driver_phone.phone_id = phone.id
  -> join phone_type on map_driver_phone.phone_type_id = phone_type.id
  -> where phone_type.code IN ('primary', 'alternate')
  -> return phone.number, phone_type.name, phone.verified_date
  -> create/update phone entities and map_driver_phone relationships
  ```

##### Email Address
- **Backend Mapping**:
  ```
  get map_driver_email.* from map_driver_email where driver_id = :driver_id
  -> join email on map_driver_email.email_id = email.id
  -> join email_type on map_driver_email.email_type_id = email_type.id
  -> where email_type.code = 'primary'
  -> return email.address, email.verified_date
  -> create/update email entity and map_driver_email relationship
  ```

##### Notification Preferences
- **Backend Mapping**:
  ```
  get communication_preference.* from communication_preference where driver_id = :driver_id
  -> return preference.email_notifications, preference.sms_notifications, preference.phone_notifications
  -> update communication_preference on changes
  ```

##### Prior Insurance Information
- **Backend Mapping**:
  ```
  get prior_insurance.* from prior_insurance where quote_id = :quote_id
  -> return prior_insurance.company_name, prior_insurance.policy_id,
           prior_insurance.expiration_date, prior_insurance.months_insured
  -> create/update prior_insurance entity linked to quote
  ```

##### Eligible Discounts
- **Backend Mapping**:
  ```
  get discount_type.* from discount_type where status_id = :active_status
  -> check discount_type.requirements against driver/quote data
  -> create discount records for eligible types
  -> create quote_discount records linking quote to discounts
  -> return eligible_discounts[] with eligibility_reason and requirements_met boolean
  ```

##### Residence Information
- **Backend Mapping**:
  ```
  get residence_type.* from residence_type where status_id = :active_status
  -> return residence_type.code, residence_type.name for selection
  -> update driver.residence_type_id on selection
  ```

#### Validation & Cross-Field Rules

##### Paperless Discount Email Validation
- **Backend Mapping**:
  ```
  if discount_selection.code = 'paperless':
    -> validate email.address IS NOT NULL AND email.address != ''
    -> validate email.verified_date IS NOT NULL OR schedule_verification = true
    -> return validation_result, required_fields[]
  ```

##### Notification Preference Email/SMS Validation
- **Backend Mapping**:
  ```
  if communication_preference.email_notifications = true:
    -> validate primary_email exists and is valid
  if communication_preference.sms_notifications = true:
    -> validate primary_phone exists and is mobile type
  -> return validation_result, required_fields[]
  ```

### Implementation Architecture

**Data Review and Enhancement Strategy**: Building on Step 1 DCS data population, this step focuses on reviewing DCS-populated read-only fields and enhancing with additional information not available through external search.

**Cross-Field Validation Engine**: Implements comprehensive validation rules for discount eligibility (e.g., email required for paperless discount) and notification preferences.

**Data Source Distinction**: Clear UI indicators for DCS-populated vs. manually entered data, with override capabilities for correcting DCS data when necessary.

**Progressive Data Enhancement**: Builds upon the immediate quote creation and DCS integration from Step 1, maintaining data persistence across workflow steps.

#### Performance & Monitoring

**Response Time Targets**:
- Data review display: < 200ms
- Discount eligibility calculation: < 500ms
- Email verification: < 3 seconds
- Cross-field validation: < 100ms

**Caching Strategy**:
- Discount type rules: 1 hour cache
- Gender/marital status/residence lookups: 24 hour cache
- Email verification results: 7 days cache

#### Error Handling

**Validation Error Handling**: Real-time field validation with clear error messaging for required field combinations
**Service Degradation**: Email verification fallback to manual verification when service unavailable
**Data Override**: Allow manual correction of DCS data with audit trail tracking

---

## **D) User Experience (UX) & Flows**

### **Record Found Flow**

1. Agent will review the pre-populated address returned with the record and confirm it is correct before continuing. If incorrect, the agent will modify the information.
2. The agent will populate the additional details section, including their gender, marital status, primary and alternate phone numbers, email address, notification preferences, as well as prior insurance information and eligible discounts.

### **No Record Found Flow**

1. If no record is found, the agent must populate all information on the user, including their name, and date of birth, along with all fields defined in the record found workflow.

### **Data Enhancement Workflow**

1. **Review DCS Data**: Display read-only fields from Step 1 DCS search with data source indicators
2. **Address Verification**: Allow address override with "Address Incorrect" option handling from Step 1
3. **Contact Information**: Collect and verify phone/email with real-time validation
4. **Additional Details**: Capture gender, marital status, residence type, prior insurance
5. **Discount Eligibility**: Auto-calculate and display eligible discounts based on collected data
6. **Cross-Field Validation**: Enforce business rules (email for paperless, etc.)

---

## API Specifications

### Endpoints Required
```http
GET    /api/v1/quotes/{id}/primary-insured    # Get DCS-populated data for review
PUT    /api/v1/quotes/{id}/primary-insured    # Update primary insured information
POST   /api/v1/quotes/{id}/verify-email       # Verify email address
POST   /api/v1/quotes/{id}/verify-phone       # Verify phone number
GET    /api/v1/quotes/{id}/eligible-discounts # Calculate discount eligibility
PUT    /api/v1/quotes/{id}/discount-selection # Update discount selections
GET    /api/v1/reference/gender               # Get gender options
GET    /api/v1/reference/marital-status       # Get marital status options
GET    /api/v1/reference/residence-types      # Get residence type options
GET    /api/v1/reference/occupation-types     # Get occupation type options
```

### Real-time Updates
```javascript
// WebSocket channels
private-quote.{id}.primary-insured           # Primary insured data updates
private-quote.{id}.discounts                 # Discount eligibility updates
```

---

## Database Schema (Section E)

### New Core Tables

#### prior_insurance
```sql
CREATE TABLE prior_insurance (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  
  -- Business columns
  quote_id BIGINT UNSIGNED NOT NULL,
  company_name VARCHAR(200) NOT NULL,
  policy_id BIGINT UNSIGNED NULL,
  expiration_date DATE NULL,
  months_insured INT NULL,
  
  -- Status and audit
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (quote_id) REFERENCES quote(id) ON DELETE CASCADE,
  FOREIGN KEY (policy_id) REFERENCES policy(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  -- Indexes
  INDEX idx_quote (quote_id),
  INDEX idx_policy (policy_id),
  INDEX idx_status (status_id),
  INDEX idx_company_name (company_name)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### discount
```sql
CREATE TABLE discount (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  
  -- Business columns
  discount_type_id BIGINT UNSIGNED NOT NULL,
  calculation_value DECIMAL(10,2) NOT NULL,
  
  -- Status and audit
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (discount_type_id) REFERENCES discount_type(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  -- Indexes
  INDEX idx_discount_type (discount_type_id),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### occupation
```sql
CREATE TABLE occupation (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  
  -- Business columns
  occupation_type_id BIGINT UNSIGNED NOT NULL,
  employer_name VARCHAR(200) NULL,
  job_title VARCHAR(200) NULL,
  
  -- Status and audit
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (occupation_type_id) REFERENCES occupation_type(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  -- Indexes
  INDEX idx_occupation_type (occupation_type_id),
  INDEX idx_status (status_id),
  INDEX idx_employer_name (employer_name)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### New Reference Tables

#### discount_type
```sql
CREATE TABLE discount_type (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  
  -- Business columns
  code VARCHAR(50) UNIQUE NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT NULL,
  requirements JSON NULL,
  savings_type ENUM('percentage', 'fixed_amount') NOT NULL,
  base_value DECIMAL(10,2) NOT NULL,
  program_specific BOOLEAN DEFAULT FALSE,
  
  -- Status and audit
  status_id BIGINT UNSIGNED NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (status_id) REFERENCES status(id),
  
  -- Indexes
  INDEX idx_code (code),
  INDEX idx_status (status_id),
  INDEX idx_program_specific (program_specific)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### gender
```sql
CREATE TABLE gender (
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

#### marital_status
```sql
CREATE TABLE marital_status (
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

#### residence_type
```sql
CREATE TABLE residence_type (
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

#### occupation_type
```sql
CREATE TABLE occupation_type (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) UNIQUE NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT NULL,
  risk_category ENUM('low', 'medium', 'high') NULL,
  status_id BIGINT UNSIGNED NOT NULL,
  
  -- Audit fields
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (status_id) REFERENCES status(id),
  
  -- Indexes
  INDEX idx_code (code),
  INDEX idx_risk_category (risk_category),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### New Relationship Tables

#### quote_discount
```sql
CREATE TABLE quote_discount (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  quote_id BIGINT UNSIGNED NOT NULL,
  discount_id BIGINT UNSIGNED NOT NULL,
  applied BOOLEAN DEFAULT FALSE,
  requirements_met BOOLEAN DEFAULT FALSE,
  eligibility_reason TEXT NULL,
  status_id BIGINT UNSIGNED NOT NULL,
  
  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (quote_id) REFERENCES quote(id) ON DELETE CASCADE,
  FOREIGN KEY (discount_id) REFERENCES discount(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  -- Constraints
  UNIQUE KEY unique_quote_discount (quote_id, discount_id),
  
  -- Indexes
  INDEX idx_quote (quote_id),
  INDEX idx_discount (discount_id),
  INDEX idx_status (status_id),
  INDEX idx_applied (applied)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### Modified Tables

#### ALTER TABLE driver
```sql
-- Add reference fields for additional information
ALTER TABLE driver 
ADD COLUMN gender_id BIGINT UNSIGNED NULL,
ADD COLUMN marital_status_id BIGINT UNSIGNED NULL,
ADD COLUMN residence_type_id BIGINT UNSIGNED NULL,
ADD COLUMN occupation_id BIGINT UNSIGNED NULL;

-- Add foreign key constraints
ALTER TABLE driver
ADD CONSTRAINT fk_driver_gender 
FOREIGN KEY (gender_id) REFERENCES gender(id);

ALTER TABLE driver
ADD CONSTRAINT fk_driver_marital_status 
FOREIGN KEY (marital_status_id) REFERENCES marital_status(id);

ALTER TABLE driver
ADD CONSTRAINT fk_driver_residence_type 
FOREIGN KEY (residence_type_id) REFERENCES residence_type(id);

ALTER TABLE driver
ADD CONSTRAINT fk_driver_occupation 
FOREIGN KEY (occupation_id) REFERENCES occupation(id);

-- Add indexes
ALTER TABLE driver
ADD INDEX idx_gender (gender_id),
ADD INDEX idx_marital_status (marital_status_id),
ADD INDEX idx_residence_type (residence_type_id),
ADD INDEX idx_occupation (occupation_id);
```

---

## Implementation Notes

### Dependencies
- Step 1 Primary Insured completion (DCS data populated)
- Email verification service integration
- Phone verification service integration
- Discount calculation engine
- Reference data setup (gender, marital status, residence types, occupation types)

### Migration Considerations
- Existing driver data compatibility with new reference fields
- Discount type configuration and requirements setup
- Communication preference migration from existing patterns
- Split discount_type (definition) and discount (instance) tables

### Performance Considerations
- Real-time discount eligibility calculation optimization
- Email/phone verification service response times
- Cross-field validation efficiency
- Reference data caching strategy

---

## Quality Checklist

### Global Requirements Compliance
- [x] **GR-52**: Universal Entity Management applied for verification services
- [x] **GR-44**: Communication Architecture used for verification tracking
- [x] **GR-04**: Validation & Data Handling implemented for cross-field rules
- [x] **GR-36**: Authentication & Permissions via Laravel Sanctum
- [x] **GR-33**: Data Services patterns for reference data caching