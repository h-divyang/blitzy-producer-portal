# #1 - Quotes Search

## **A) WHY - Vision and Purpose**

The **Quotes Search & Flyout** experience empowers users to efficiently manage, review, and act on insurance quotes. It provides a **centralized location to view quote activity**, quickly access **detailed quote breakdowns**, and take action through **intuitive mobile and desktop views**. This enables faster decision-making, clearer communication, and streamlined operations across devices.

---

## **B) WHAT - Core Requirements**

### **1. Quotes Dashboard (Desktop & Mobile)**

- **List of quotes** with key details in table view (desktop) and list (mobile):
    - Quote Status, with color indicators
    - Date & Time Submitted
    - Effective Date
    - Insured Name
    - Producer Number
    - Vehicles
    - Drivers
    - Requoted From
        - This will include a **policy number** and link to previous policy, if applicable
- **Search & Filter Capabilities**
    - Filter by date submitted, offices, or status
    - Sort by selecting the column headers, sorting column will be indicated by downward arrow
    - Text search by name or quote number
- Start **New Quote** button

### **2. Quote Flyout Panel (Expanded View)**

Upon selection of a quote, the side panel flyout will present, showing a summary view of the quote.

- **Header Section**:
    - Insured name
    - Quote status
    - Requoted From, if applicable
    - Submitted On
    - Effective date
- **See Full Details:**
    - This button will take the user into the quoting experience where they left off for quotes in progress
- **Insured Details**:
    - Insured Name
    - Primary Phone Number
    - Alternate Phone Number
    - Email
    - Notification Preference
    - Address
- **Suspenses:**
    - This will present any suspenses that are currently open on this quote that require action by the producer to resolve
- **Vehicles:**
    - Year
    - Make
    - Model
    - VIN
    - Plate
- **Drivers:**
    - Name
    - Primary Driver Tag
    - Included vs Excluded Status
    - Date of Birth
    - State of License Issue
    - Identification Prefix
    - Identification Number
- **Documents Section**:
    - This includes documents attached to the quote (e.g. Proof of Prior Insurance, Driver’s License Image, Vehicle Photos, etc.)
- **Coverage Breakdown**
    - Individual policy coverages (BI, PD, MedPay, Comp, Collision, etc.) with limits and deductibles
- **Rate Information**
    - Prior Policy - if they held insurance previously, this will indicate ‘Yes’
    - Rate Chart - the rate chart used for their quote
    - Company - the insurance carrier associated with the program they were quoted on

---

## **C) HOW - Planning & Implementation**

### Backend Field Mappings

### **1. Quotes Dashboard List View**

### **Quote Status**

- **Backend Mapping**:

    ```
    get quote.id from quote
    -> get status by quote.status_id
    -> get status_type by status.status_type_id where status_type.code = 'QUOTE_STATUS'
    -> return status.name, status.code
    
    ```


### **Date & Time Submitted**

- **Backend Mapping**:

    ```
    get quote.id from quote
    -> quote.created_at
    -> convert to user timezone using user.timezone_preference
    
    ```


### **Effective Date**

- **Backend Mapping**:

    ```
    get quote.id from quote
    -> quote.effective_date
    
    ```


### **Insured Name**

- **Backend Mapping**:

    ```
    get quote.id from quote
    -> get map_quote_driver by quote.id where status_id = active
    -> get driver by map_quote_driver.driver_id where driver.is_named_insured = true
    -> get name by driver.name_id
    -> concat name.last_name, ', ', name.first_name, COALESCE(CONCAT(' ', name.middle_name), '')
    
    ```


### **Agent Number**

- **Backend Mapping**:

    ```
    get quote.id from quote
    -> get map_producer_quote by quote.id where status_id = active
    -> get producer by map_producer_quote.producer_id
    -> producer.producer_code
    
    ```


### **Vehicles**

- **Backend Mapping**:

    ```
    get quote.id from quote
    -> count(*) from map_quote_vehicle where quote_id = quote.id and status_id = active
    
    ```


### **Drivers**

- **Backend Mapping**:

    ```
    get quote.id from quote
    -> count(*) from map_quote_driver mqd
    -> join driver_type dt on mqd.driver_type_id = dt.id
    -> where mqd.quote_id = quote.id and mqd.status_id = active and dt.code != 'EXCLUDED'
    
    ```


### **Requoted From**

- **Backend Mapping**:

    ```
    get quote.id from quote
    -> if quote.original_policy_id is not null:
       get policy by quote.original_policy_id
       -> get policy_prefix by policy.policy_prefix_id
       -> concat policy_prefix.value, policy.number
    -> else: return null
    
    ```


### **2. Search & Filter Implementation**

### **Date Range Filter**

- **Backend Query**:

    ```sql
    WHERE quote.created_at >= :start_date
    AND quote.created_at <= :end_date
    
    ```


### **Office/Producer Filter**

- **Backend Query**:

    ```sql
    JOIN map_producer_quote mpq ON quote.id = mpq.quote_id
    WHERE mpq.producer_id IN (:selected_producer_ids)
    AND mpq.status_id = :active_status_id
    
    ```


### **Status Filter**

- **Backend Query**:

    ```sql
    WHERE quote.status_id IN (:selected_status_ids)
    
    ```


### **Text Search**

- **Backend Query**:

    ```sql
    WHERE (
      quote.quote_number LIKE CONCAT('%', :search_term, '%')
      OR EXISTS (
        SELECT 1 FROM map_quote_driver mqd
        JOIN driver d ON mqd.driver_id = d.id
        JOIN name n ON d.name_id = n.id
        WHERE mqd.quote_id = quote.id
        AND d.is_named_insured = true
        AND mqd.status_id = :active_status_id
        AND CONCAT(n.first_name, ' ', n.last_name) LIKE CONCAT('%', :search_term, '%')
      )
      OR EXISTS (
        SELECT 1 FROM map_quote_vehicle mqv
        JOIN vehicle v ON mqv.vehicle_id = v.id
        WHERE mqv.quote_id = quote.id
        AND mqv.status_id = :active_status_id
        AND v.vin LIKE CONCAT('%', :search_term, '%')
      )
    )
    
    ```


### **3. Quote Flyout Panel Detailed Mappings**

### **Header Section**

### **Insured Name**

- **Backend Mapping**:

    ```
    get quote.id
    -> get map_quote_driver by quote.id where status_id = active
    -> get driver by map_quote_driver.driver_id where driver.is_named_insured = true
    -> get name by driver.name_id
    -> concat COALESCE(name.prefix + ' ', ''), name.first_name, COALESCE(' ' + name.middle_name, ''), ' ', name.last_name, COALESCE(' ' + name.suffix, '')
    
    ```


### **Quote Status**

- **Backend Mapping**:

    ```
    get quote.id
    -> get status by quote.status_id
    -> return status.name, status.code, status.description
    
    ```


### **Requoted From Information**

- **Backend Mapping**:

    ```
    get quote.id
    -> get requote_type by quote.requote_type_id
    -> get policy by quote.original_policy_id
    -> get policy_prefix by policy.policy_prefix_id
    -> return concat('Requoted from Policy #', policy_prefix.value, policy.number, ' (', requote_type.name, ')')
    
    ```


### **Submitted On**

- **Backend Mapping**:

    ```
    get quote.id
    -> quote.created_at
    -> format with user timezone from user.timezone_preference
    
    ```


### **Effective Date**

- **Backend Mapping**:

    ```
    get quote.id
    -> quote.effective_date
    
    ```


### **Insured Details Section**

### **Primary Phone Number**

- **Backend Mapping**:

    ```
    get quote.id
    -> get map_quote_driver by quote.id where status_id = active
    -> get driver by map_quote_driver.driver_id where driver.is_named_insured = true
    -> get map_driver_phone by driver.id where is_primary = true and status_id = active
    -> get phone by map_driver_phone.phone_id
    -> get phone_type by phone.phone_type_id
    -> format phone.country_code + phone.number + COALESCE(' ext. ' + phone.extension, '')
    
    ```


### **Alternate Phone Number**

- **Backend Mapping**:

    ```
    get quote.id
    -> get map_quote_driver by quote.id where status_id = active
    -> get driver by map_quote_driver.driver_id where driver.is_named_insured = true
    -> get map_driver_phone by driver.id where is_primary = false and status_id = active limit 1
    -> get phone by map_driver_phone.phone_id
    -> format phone.country_code + phone.number
    
    ```


### **Email Address**

- **Backend Mapping**:

    ```
    get quote.id
    -> get map_quote_driver by quote.id where status_id = active
    -> get driver by map_quote_driver.driver_id where driver.is_named_insured = true
    -> get map_driver_email by driver.id where is_primary = true and status_id = active
    -> get email by map_driver_email.email_id
    -> get email_type by email.email_type_id
    -> return email.address, email.is_verified
    
    ```


### **Notification Preference**

- **Backend Mapping**:

    ```
    get quote.id
    -> get map_quote_driver by quote.id where status_id = active
    -> get driver by map_quote_driver.driver_id where driver.is_named_insured = true
    -> get notification_preference by driver.notification_preference_id
    -> notification_preference.name
    
    ```


### **Mailing Address**

- **Backend Mapping**:

    ```
    get quote.id
    -> get map_quote_driver by quote.id where status_id = active
    -> get driver by map_quote_driver.driver_id where driver.is_named_insured = true
    -> get map_driver_address by driver.id where is_primary = true and status_id = active
    -> get address by map_driver_address.address_id
    -> get address_type by address.address_type_id
    -> format:
       line1: address.line1
       line2: address.line2 (if not null)
       line3: address.city + ', ' + address.state + ' ' + address.zip + COALESCE('-' + address.zip4, '')
    
    ```


### **Suspenses Section**

### **Active Suspenses List**

- **Backend Mapping**:

    ```
    get quote.id
    -> get map_quote_suspense by quote.id where status_id = active
    -> get suspense by map_quote_suspense.suspense_id
    -> get suspense_type by suspense.suspense_type_id
    -> get assigned_user by suspense.assigned_to
    -> get suspense_status by suspense.status_id where status.code != 'RESOLVED'
    -> order by suspense.due_date asc
    -> return:
       suspense_type.name,
       suspense.description,
       suspense.due_date,
       assigned_user.first_name + ' ' + assigned_user.last_name,
       datediff(suspense.due_date, current_date) as days_until_due
    
    ```


### **Vehicles Section**

### **Vehicle Details List**

- **Backend Mapping**:

    ```
    get quote.id
    -> get map_quote_vehicle by quote.id where status_id = active
    -> get vehicle by map_quote_vehicle.vehicle_id
    -> get vehicle_registration by vehicle.id where status_id = active
    -> get state by vehicle_registration.state_id
    -> order by vehicle.year desc
    -> return:
       vehicle.year,
       vehicle.make,
       vehicle.model,
       vehicle.vin,
       vehicle_registration.plate_number,
       state.name
    
    ```


### **Drivers Section**

### **Driver Information List**

- **Backend Mapping**:

    ```
    get quote.id
    -> get map_quote_driver by quote.id where status_id = active
    -> get driver by map_quote_driver.driver_id
    -> get driver_type by driver.driver_type_id
    -> get name by driver.name_id
    -> get map_driver_license by driver.id where status_id = active
    -> get license by map_driver_license.license_id
    -> get license_type by license.license_type_id
    -> get state by license.state_id
    -> get relationship_to_insured by driver.relationship_to_insured_id
    -> order by driver.is_named_insured desc
    -> return:
       concat(name.first_name, ' ', name.last_name),
       driver_type.name,
       driver.date_of_birth,
       state.name,
       license_type.name,
       license.license_number,
       relationship_to_insured.name
    
    ```


### **Documents Section**

### **Document List**

- **Backend Mapping**:

    ```
    get quote.id
    -> get map_quote_document by quote.id where status_id = active
    -> get document by map_quote_document.document_id
    -> get document_type by document.document_type_id
    -> order by document.created_at desc
    -> return:
       document_type.name,
       document.file_name,
       document.file_size,
       document.created_at,
       document.file_path
    
    ```


### **Coverage Breakdown Section**

### **Coverage Details**

- **Backend Mapping**:

    ```
    get quote.id
    -> get map_quote_coverage by quote.id where status_id = active
    -> get coverage by map_quote_coverage.coverage_id
    -> get coverage_type by coverage.coverage_type_id
    -> get coverage_limit by map_quote_coverage.limit_id
    -> get deductible by map_quote_coverage.deductible_id
    -> order by coverage_type.name
    -> return:
       coverage_type.name,
       coverage_type.code,
       coverage_limit.limit_value,
       coverage_limit.limit_type,
       deductible.deductible_value
    
    ```


### **Rate Information Section**

### **Prior Policy Indicator**

- **Backend Mapping**:

    ```
    get quote.id
    -> if quote.original_policy_id is not null then 'Yes' else 'No'
    -> if 'Yes': get policy by quote.original_policy_id -> get carrier by policy.carrier_id -> return policy details
    
    ```


### **Program Information**

- **Backend Mapping**:

    ```
    get quote.id
    -> get program by quote.program_id
    -> get carrier by program.carrier_id
    -> return:
       program.name,
       program.rate_version,
       program.rate_effective_date,
       carrier.name
    
    ```


### **Carrier Information**

- **Backend Mapping**:

    ```
    get quote.id
    -> get program by quote.program_id
    -> get carrier by program.carrier_id
    -> return:
       carrier.name,
       carrier.code,
       carrier.contact_phone,
       carrier.website
    
    ```


### **4. API Endpoint Specifications**

### **Quote List API**

```
GET /api/v1/quotes
Parameters:
  - page: integer (default: 1)
  - per_page: integer (default: 25, max: 100)
  - sort_by: string (created_at, status, effective_date, insured_name)
  - sort_order: string (asc, desc) (default: desc)
  - filters[status_ids][]: array of integers
  - filters[producer_ids][]: array of integers
  - filters[date_from]: date (YYYY-MM-DD)
  - filters[date_to]: date (YYYY-MM-DD)
  - search: string (searches quote number, insured name, VIN)

Response: {
  "data": [...quotes],
  "meta": {
    "current_page": 1,
    "total_pages": 10,
    "total_count": 250,
    "per_page": 25
  }
}

```

### **Quote Flyout API**

```
GET /api/v1/quotes/{quote_id}/flyout

Response: {
  "quote": {...},
  "insured": {...},
  "vehicles": [...],
  "drivers": [...],
  "coverages": [...],
  "suspenses": [...],
  "documents": [...],
  "rate_info": {...}
}

```

---

## **D) User Experience (UX) & Flows**

### **1. Viewing Quotes (Desktop)**

1. User lands on the **Quotes Home Dashboard**
2. Scrolls or searches for a specific quote
3. Clicks a row to open the **Quote Flyout Panel** on the right
4. To open the quote, the user can click “Open Quote” to enter the Quoting workflow where they left off

### **2. Viewing Quotes (Mobile)**

1. User opens mobile menu > Quotes
2. Taps on a quote row from the list
3. Flyout expands full screen with quote details
4. To open the quote, the user can click “Open Quote” to enter the Quoting workflow where they left off

### **3. Starting a New Quote**

1. Clicks the **“+ Start New Quote”** button
2. Redirected to quote intake form

## **E) Master Schema Tables**

### **1. Core Quote Tables**

### **quote**

```sql
CREATE TABLE quote (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  quote_number VARCHAR(50) UNIQUE NOT NULL,
  status_id BIGINT UNSIGNED NOT NULL,
  program_id BIGINT UNSIGNED NOT NULL,

  -- Requote tracking
  requote_type_id BIGINT UNSIGNED NULL,
  original_policy_id BIGINT UNSIGNED NULL,

  -- Date fields
  effective_date DATE NOT NULL,
  expiration_date DATE NULL,

  -- Workflow timestamps
  bound_at TIMESTAMP NULL,
  expired_at TIMESTAMP NULL,

  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  -- Foreign key constraints
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (program_id) REFERENCES program(id),
  FOREIGN KEY (requote_type_id) REFERENCES requote_type(id),
  FOREIGN KEY (original_policy_id) REFERENCES policy(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),

  -- Performance indexes
  INDEX idx_status (status_id),
  INDEX idx_created_date (created_at DESC),
  INDEX idx_effective_date (effective_date),
  INDEX idx_quote_number (quote_number),
  INDEX idx_program (program_id),
  INDEX idx_original_policy (original_policy_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

```

### **driver**

```sql
CREATE TABLE driver (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  name_id BIGINT UNSIGNED NOT NULL,
  date_of_birth DATE NOT NULL,
  gender_id BIGINT UNSIGNED NULL,
  marital_status_id BIGINT UNSIGNED NULL,
  occupation_id BIGINT UNSIGNED NULL,
  driver_type_id BIGINT UNSIGNED NOT NULL,
  notification_preference_id BIGINT UNSIGNED NULL,
  relationship_to_insured_id BIGINT UNSIGNED NULL,
  years_licensed INT NULL,
  is_named_insured BOOLEAN DEFAULT FALSE,
  status_id BIGINT UNSIGNED NOT NULL,

  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  -- Foreign key constraints
  FOREIGN KEY (name_id) REFERENCES name(id),
  FOREIGN KEY (gender_id) REFERENCES gender(id),
  FOREIGN KEY (marital_status_id) REFERENCES marital_status(id),
  FOREIGN KEY (occupation_id) REFERENCES occupation(id),
  FOREIGN KEY (driver_type_id) REFERENCES driver_type(id),
  FOREIGN KEY (notification_preference_id) REFERENCES notification_preference(id),
  FOREIGN KEY (relationship_to_insured_id) REFERENCES relationship_to_insured(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),

  -- Indexes
  INDEX idx_name (name_id),
  INDEX idx_dob (date_of_birth),
  INDEX idx_driver_type (driver_type_id),
  INDEX idx_named_insured (is_named_insured),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

```

### **vehicle**

```sql
CREATE TABLE vehicle (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  vin VARCHAR(17) NOT NULL,
  year INT NOT NULL,
  make VARCHAR(50) NOT NULL,
  model VARCHAR(50) NOT NULL,
  vehicle_use_type_id BIGINT UNSIGNED NULL,
  garaging_address_id BIGINT UNSIGNED NULL,
  annual_mileage INT NULL,
  business_use BOOLEAN DEFAULT FALSE,
  is_primary_vehicle BOOLEAN DEFAULT FALSE,
  status_id BIGINT UNSIGNED NOT NULL,

  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  -- Foreign key constraints
  FOREIGN KEY (vehicle_use_type_id) REFERENCES vehicle_use_type(id),
  FOREIGN KEY (garaging_address_id) REFERENCES address(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),

  -- Indexes
  UNIQUE INDEX idx_vin (vin),
  INDEX idx_year_make_model (year, make, model),
  INDEX idx_use_type (vehicle_use_type_id),
  INDEX idx_primary (is_primary_vehicle),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

```

### **license**

```sql
CREATE TABLE license (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  license_type_id BIGINT UNSIGNED NOT NULL,
  license_number VARCHAR(50) NOT NULL,
  state_id BIGINT UNSIGNED NOT NULL,
  license_class VARCHAR(10) NULL,
  issue_date DATE NULL,
  expiration_date DATE NULL,
  is_verified BOOLEAN DEFAULT FALSE,
  verified_at TIMESTAMP NULL,
  verification_method VARCHAR(50) NULL,
  status_id BIGINT UNSIGNED NOT NULL,

  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  -- Foreign key constraints
  FOREIGN KEY (license_type_id) REFERENCES license_type(id),
  FOREIGN KEY (state_id) REFERENCES state(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),

  -- Indexes
  INDEX idx_license_state (license_number, state_id),
  INDEX idx_license_type (license_type_id),
  INDEX idx_expiration (expiration_date),
  INDEX idx_verified (is_verified),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

```

### **document**

```sql
CREATE TABLE document (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  document_type_id BIGINT UNSIGNED NOT NULL,
  file_name VARCHAR(255) NOT NULL,
  file_path TEXT NOT NULL,
  file_size BIGINT NOT NULL,
  mime_type VARCHAR(100) NOT NULL,
  file_hash VARCHAR(64) NULL,
  description TEXT NULL,
  status_id BIGINT UNSIGNED NOT NULL,

  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  -- Foreign key constraints
  FOREIGN KEY (document_type_id) REFERENCES document_type(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),

  -- Indexes
  INDEX idx_type (document_type_id),
  INDEX idx_created (created_at DESC),
  INDEX idx_hash (file_hash),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

```

### **suspense**

```sql
CREATE TABLE suspense (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  suspense_type_id BIGINT UNSIGNED NOT NULL,
  description TEXT NOT NULL,
  due_date DATE NULL,

  -- Assignment tracking
  assigned_to BIGINT UNSIGNED NULL,
  assigned_by BIGINT UNSIGNED NULL,
  assigned_at TIMESTAMP NULL,

  -- Status tracking
  status_id BIGINT UNSIGNED NOT NULL,

  -- Resolution tracking
  resolved_by BIGINT UNSIGNED NULL,
  resolved_at TIMESTAMP NULL,
  resolution_notes TEXT NULL,

  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  -- Foreign key constraints
  FOREIGN KEY (suspense_type_id) REFERENCES suspense_type(id),
  FOREIGN KEY (assigned_to) REFERENCES user(id),
  FOREIGN KEY (assigned_by) REFERENCES user(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (resolved_by) REFERENCES user(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),

  -- Indexes
  INDEX idx_type (suspense_type_id),
  INDEX idx_assigned (assigned_to, status_id),
  INDEX idx_due_date (due_date, status_id),
  INDEX idx_status (status_id),
  INDEX idx_created (created_at DESC)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

```

### **vehicle_registration**

```sql
CREATE TABLE vehicle_registration (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  vehicle_id BIGINT UNSIGNED NOT NULL,
  plate_number VARCHAR(20) NOT NULL,
  state_id BIGINT UNSIGNED NOT NULL,
  registration_date DATE NOT NULL,
  expiration_date DATE NOT NULL,
  status_id BIGINT UNSIGNED NOT NULL,

  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  -- Foreign key constraints
  FOREIGN KEY (vehicle_id) REFERENCES vehicle(id),
  FOREIGN KEY (state_id) REFERENCES state(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),

  -- Indexes
  INDEX idx_vehicle (vehicle_id),
  INDEX idx_plate_state (plate_number, state_id),
  INDEX idx_expiration (expiration_date),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

```

### **2. Reference Tables**

### **requote_type**

```sql
CREATE TABLE requote_type (
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

### **driver_type**

```sql
CREATE TABLE driver_type (
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

### **license_type**

```sql
CREATE TABLE license_type (
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

### **suspense_type**

```sql
CREATE TABLE suspense_type (
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

### **document_type**

```sql
CREATE TABLE document_type (
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

### **notification_preference**

```sql
CREATE TABLE notification_preference (
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

### **marital_status**

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

### **gender**

```sql
CREATE TABLE gender (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(10) UNIQUE NOT NULL,
  name VARCHAR(50) NOT NULL,
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

### **occupation**

```sql
CREATE TABLE occupation (
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

### **vehicle_use_type**

```sql
CREATE TABLE vehicle_use_type (
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

### **relationship_to_insured**

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

### **phone_type**

```sql
CREATE TABLE phone_type (
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

### **email_type**

```sql
CREATE TABLE email_type (
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

### **address_type**

```sql
CREATE TABLE address_type (
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

### **3. Relationship Tables (map_*)**

### **map_producer_quote**

```sql
CREATE TABLE map_producer_quote (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  producer_id BIGINT UNSIGNED NOT NULL,
  quote_id BIGINT UNSIGNED NOT NULL,
  assignment_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  status_id BIGINT UNSIGNED NOT NULL,

  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  -- Foreign key constraints
  FOREIGN KEY (producer_id) REFERENCES producer(id),
  FOREIGN KEY (quote_id) REFERENCES quote(id) ON DELETE CASCADE,
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),

  -- Constraints
  UNIQUE KEY unique_producer_quote (producer_id, quote_id),

  -- Indexes
  INDEX idx_producer (producer_id),
  INDEX idx_quote (quote_id),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

```

### **map_quote_driver**

```sql
CREATE TABLE map_quote_driver (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  quote_id BIGINT UNSIGNED NOT NULL,
  driver_id BIGINT UNSIGNED NOT NULL,
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
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

```

### **map_quote_vehicle**

```sql
CREATE TABLE map_quote_vehicle (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  quote_id BIGINT UNSIGNED NOT NULL,
  vehicle_id BIGINT UNSIGNED NOT NULL,
  status_id BIGINT UNSIGNED NOT NULL,

  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  -- Foreign key constraints
  FOREIGN KEY (quote_id) REFERENCES quote(id) ON DELETE CASCADE,
  FOREIGN KEY (vehicle_id) REFERENCES vehicle(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),

  -- Constraints
  UNIQUE KEY unique_quote_vehicle (quote_id, vehicle_id),

  -- Indexes
  INDEX idx_quote (quote_id),
  INDEX idx_vehicle (vehicle_id),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

```

### **map_driver_license**

```sql
CREATE TABLE map_driver_license (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  driver_id BIGINT UNSIGNED NOT NULL,
  license_id BIGINT UNSIGNED NOT NULL,
  status_id BIGINT UNSIGNED NOT NULL,

  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  -- Foreign key constraints
  FOREIGN KEY (driver_id) REFERENCES driver(id),
  FOREIGN KEY (license_id) REFERENCES license(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),

  -- Constraints
  UNIQUE KEY unique_driver_license (driver_id, license_id),

  -- Indexes
  INDEX idx_driver (driver_id),
  INDEX idx_license (license_id),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

```

### **map_quote_document**

```sql
CREATE TABLE map_quote_document (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  quote_id BIGINT UNSIGNED NOT NULL,
  document_id BIGINT UNSIGNED NOT NULL,
  status_id BIGINT UNSIGNED NOT NULL,

  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  -- Foreign key constraints
  FOREIGN KEY (quote_id) REFERENCES quote(id) ON DELETE CASCADE,
  FOREIGN KEY (document_id) REFERENCES document(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),

  -- Constraints
  UNIQUE KEY unique_quote_document (quote_id, document_id),

  -- Indexes
  INDEX idx_quote (quote_id),
  INDEX idx_document (document_id),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

```

### **map_driver_phone**

```sql
CREATE TABLE map_driver_phone (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  driver_id BIGINT UNSIGNED NOT NULL,
  phone_id BIGINT UNSIGNED NOT NULL,
  is_primary BOOLEAN DEFAULT FALSE,
  status_id BIGINT UNSIGNED NOT NULL,

  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  -- Foreign key constraints
  FOREIGN KEY (driver_id) REFERENCES driver(id),
  FOREIGN KEY (phone_id) REFERENCES phone(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),

  -- Constraints
  UNIQUE KEY unique_driver_phone (driver_id, phone_id),

  -- Indexes
  INDEX idx_driver (driver_id),
  INDEX idx_phone (phone_id),
  INDEX idx_primary (driver_id, is_primary),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

```

### **map_driver_email**

```sql
CREATE TABLE map_driver_email (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  driver_id BIGINT UNSIGNED NOT NULL,
  email_id BIGINT UNSIGNED NOT NULL,
  is_primary BOOLEAN DEFAULT FALSE,
  status_id BIGINT UNSIGNED NOT NULL,

  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  -- Foreign key constraints
  FOREIGN KEY (driver_id) REFERENCES driver(id),
  FOREIGN KEY (email_id) REFERENCES email(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),

  -- Constraints
  UNIQUE KEY unique_driver_email (driver_id, email_id),

  -- Indexes
  INDEX idx_driver (driver_id),
  INDEX idx_email (email_id),
  INDEX idx_primary (driver_id, is_primary),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

```

### **map_driver_address**

```sql
CREATE TABLE map_driver_address (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  driver_id BIGINT UNSIGNED NOT NULL,
  address_id BIGINT UNSIGNED NOT NULL,
  is_primary BOOLEAN DEFAULT FALSE,
  status_id BIGINT UNSIGNED NOT NULL,

  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  -- Foreign key constraints
  FOREIGN KEY (driver_id) REFERENCES driver(id),
  FOREIGN KEY (address_id) REFERENCES address(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),

  -- Constraints
  UNIQUE KEY unique_driver_address (driver_id, address_id),

  -- Indexes
  INDEX idx_driver (driver_id),
  INDEX idx_address (address_id),
  INDEX idx_primary (driver_id, is_primary),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

```

### **map_quote_coverage**

```sql
CREATE TABLE map_quote_coverage (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  quote_id BIGINT UNSIGNED NOT NULL,
  coverage_id BIGINT UNSIGNED NOT NULL,
  limit_id BIGINT UNSIGNED NULL,
  deductible_id BIGINT UNSIGNED NULL,
  status_id BIGINT UNSIGNED NOT NULL,

  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  -- Foreign key constraints
  FOREIGN KEY (quote_id) REFERENCES quote(id) ON DELETE CASCADE,
  FOREIGN KEY (coverage_id) REFERENCES coverage(id),
  FOREIGN KEY (limit_id) REFERENCES coverage_limit(id),
  FOREIGN KEY (deductible_id) REFERENCES deductible(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),

  -- Constraints
  UNIQUE KEY unique_quote_coverage (quote_id, coverage_id),

  -- Indexes
  INDEX idx_quote (quote_id),
  INDEX idx_coverage (coverage_id),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

```

### **map_quote_suspense**

```sql
CREATE TABLE map_quote_suspense (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  quote_id BIGINT UNSIGNED NOT NULL,
  suspense_id BIGINT UNSIGNED NOT NULL,
  status_id BIGINT UNSIGNED NOT NULL,

  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  -- Foreign key constraints
  FOREIGN KEY (quote_id) REFERENCES quote(id) ON DELETE CASCADE,
  FOREIGN KEY (suspense_id) REFERENCES suspense(id) ON DELETE CASCADE,
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),

  -- Constraints
  UNIQUE KEY unique_quote_suspense (quote_id, suspense_id),

  -- Indexes
  INDEX idx_quote (quote_id),
  INDEX idx_suspense (suspense_id),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

```

### **map_policy_suspense**

```sql
CREATE TABLE map_policy_suspense (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  policy_id BIGINT UNSIGNED NOT NULL,
  suspense_id BIGINT UNSIGNED NOT NULL,
  status_id BIGINT UNSIGNED NOT NULL,

  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  -- Foreign key constraints
  FOREIGN KEY (policy_id) REFERENCES policy(id) ON DELETE CASCADE,
  FOREIGN KEY (suspense_id) REFERENCES suspense(id) ON DELETE CASCADE,
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),

  -- Constraints
  UNIQUE KEY unique_policy_suspense (policy_id, suspense_id),

  -- Indexes
  INDEX idx_policy (policy_id),
  INDEX idx_suspense (suspense_id),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

```

### **4. Supporting Entity Tables**

### **phone**

```sql
CREATE TABLE phone (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  country_code VARCHAR(5) DEFAULT '+1',
  number VARCHAR(20) NOT NULL,
  extension VARCHAR(10) NULL,
  phone_type_id BIGINT UNSIGNED NOT NULL,
  can_sms BOOLEAN DEFAULT FALSE,
  is_verified BOOLEAN DEFAULT FALSE,
  verified_at TIMESTAMP NULL,
  status_id BIGINT UNSIGNED NOT NULL,

  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  -- Foreign key constraints
  FOREIGN KEY (phone_type_id) REFERENCES phone_type(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),

  -- Indexes
  INDEX idx_number (number),
  INDEX idx_country_number (country_code, number),
  INDEX idx_phone_type (phone_type_id),
  INDEX idx_verified (is_verified),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

```

### **email**

```sql
CREATE TABLE email (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  address VARCHAR(255) NOT NULL,
  email_type_id BIGINT UNSIGNED NOT NULL,
  is_valid BOOLEAN DEFAULT TRUE,
  is_verified BOOLEAN DEFAULT FALSE,
  verified_at TIMESTAMP NULL,
  bounce_count INT DEFAULT 0,
  last_verified TIMESTAMP NULL,
  status_id BIGINT UNSIGNED NOT NULL,

  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  -- Foreign key constraints
  FOREIGN KEY (email_type_id) REFERENCES email_type(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),

  -- Indexes
  UNIQUE INDEX idx_address (address),
  INDEX idx_email_type (email_type_id),
  INDEX idx_verified (is_verified),
  INDEX idx_valid (is_valid),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

```

### **address**

```sql
CREATE TABLE address (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  line1 VARCHAR(255) NOT NULL,
  line2 VARCHAR(255) NULL,
  city VARCHAR(100) NOT NULL,
  state CHAR(2) NOT NULL,
  zip VARCHAR(10) NOT NULL,
  zip4 VARCHAR(4) NULL,
  county VARCHAR(100) NULL,
  country CHAR(2) DEFAULT 'US',
  address_type_id BIGINT UNSIGNED NOT NULL,
  is_primary BOOLEAN DEFAULT FALSE,
  latitude DECIMAL(10,8) NULL,
  longitude DECIMAL(11,8) NULL,
  is_validated BOOLEAN DEFAULT FALSE,
  validated_at TIMESTAMP NULL,
  validation_service VARCHAR(50) NULL,
  status_id BIGINT UNSIGNED NOT NULL,

  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  -- Foreign key constraints
  FOREIGN KEY (address_type_id) REFERENCES address_type(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),

  -- Indexes
  INDEX idx_zip (zip),
  INDEX idx_state_city (state, city),
  INDEX idx_geocode (latitude, longitude),
  INDEX idx_address_type (address_type_id),
  INDEX idx_primary (is_primary),
  INDEX idx_validated (is_validated),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

```

### **name**

```sql
CREATE TABLE name (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  prefix VARCHAR(10) NULL,
  first_name VARCHAR(100) NOT NULL,
  middle_name VARCHAR(100) NULL,
  last_name VARCHAR(100) NOT NULL,
  suffix VARCHAR(10) NULL,
  full_name VARCHAR(255) GENERATED ALWAYS AS (
    TRIM(CONCAT_WS(' ', prefix, first_name, middle_name, last_name, suffix))
  ) STORED,
  status_id BIGINT UNSIGNED NOT NULL,

  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

  -- Foreign key constraints
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),

  -- Indexes
  INDEX idx_last_first (last_name, first_name),
  INDEX idx_full_name (full_name),
  INDEX idx_first_name (first_name),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

```