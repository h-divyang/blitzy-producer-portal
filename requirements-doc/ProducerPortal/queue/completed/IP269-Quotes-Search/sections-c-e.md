# IP269-Quotes-Search: Sections C & E - Complete Implementation

## **C) HOW - Planning & Implementation**

### Front End Mapping of Fields

#### **1. Quotes Dashboard List View**

##### **Quote Status**
- **Frontend Display**: Status badge with standardized color coding
- **Backend Mapping**: 
  ```
  get quote.id from quote where tenant_id = current_tenant_id
  -> get status by quote.status_id 
  -> get status_type by status.status_type_id where status_type.code = 'QUOTE_STATUS'
  -> return status.name, status.code
  ```

##### **Date & Time Submitted**
- **Frontend Display**: MM/DD/YYYY HH:MM AM/PM format with timezone conversion
- **Backend Mapping**: 
  ```
  get quote.id from quote where tenant_id = current_tenant_id
  -> quote.submitted_at
  -> convert to user timezone using user.timezone_preference
  ```

##### **Effective Date**
- **Frontend Display**: MM/DD/YYYY format
- **Backend Mapping**: 
  ```
  get quote.id from quote where tenant_id = current_tenant_id
  -> quote.effective_date
  ```

##### **Insured Name**
- **Frontend Display**: "Last, First Middle" format
- **Backend Mapping**: 
  ```
  get quote.id from quote where tenant_id = current_tenant_id
  -> get map_quote_driver by quote.id where is_named_insured = true and status_id = active
  -> get driver by map_quote_driver.driver_id
  -> get name by driver.name_id
  -> concat name.last_name, ', ', name.first_name, COALESCE(CONCAT(' ', name.middle_name), '')
  ```

##### **Agent Number**
- **Frontend Display**: Producer code identifier
- **Backend Mapping**: 
  ```
  get quote.id from quote where tenant_id = current_tenant_id
  -> get map_producer_quote by quote.id where relationship_type = 'primary' and status_id = active
  -> get producer by map_producer_quote.producer_id
  -> producer.producer_code
  ```

##### **Vehicles**
- **Frontend Display**: Count of active vehicles on quote
- **Backend Mapping**: 
  ```
  get quote.id from quote where tenant_id = current_tenant_id
  -> count(*) from map_quote_vehicle where quote_id = quote.id and status_id = active
  ```

##### **Drivers**
- **Frontend Display**: Count of active included drivers
- **Backend Mapping**: 
  ```
  get quote.id from quote where tenant_id = current_tenant_id
  -> count(*) from map_quote_driver mqd 
  -> join driver_type dt on mqd.driver_type_id = dt.id 
  -> where mqd.quote_id = quote.id and mqd.status_id = active and dt.code != 'EXCLUDED'
  ```

##### **Requoted From**
- **Frontend Display**: Policy number as clickable link (when applicable)
- **Backend Mapping**: 
  ```
  get quote.id from quote where tenant_id = current_tenant_id
  -> if quote.original_policy_id is not null:
     get policy by quote.original_policy_id
     -> get policy_prefix by policy.policy_prefix_id
     -> concat policy_prefix.value, policy.number
  -> else: return null
  ```

#### **2. Search & Filter Implementation**

##### **Date Range Filter**
- **Frontend Input**: Date picker components for start and end dates
- **Backend Query**: 
  ```sql
  WHERE quote.submitted_at >= :start_date 
  AND quote.submitted_at <= :end_date
  ```

##### **Office/Producer Filter**
- **Frontend Input**: Multi-select dropdown populated from user's accessible producers
- **Backend Query**: 
  ```sql
  JOIN map_producer_quote mpq ON quote.id = mpq.quote_id
  WHERE mpq.producer_id IN (:selected_producer_ids) 
  AND mpq.status_id = :active_status_id
  ```

##### **Status Filter**
- **Frontend Input**: Multi-select checkboxes with color indicators
- **Backend Query**: 
  ```sql
  WHERE quote.status_id IN (:selected_status_ids)
  ```

##### **Text Search**
- **Frontend Input**: Single search field with autocomplete
- **Backend Query**: 
  ```sql
  WHERE (
    quote.quote_number LIKE CONCAT('%', :search_term, '%')
    OR EXISTS (
      SELECT 1 FROM map_quote_driver mqd
      JOIN driver d ON mqd.driver_id = d.id
      JOIN name n ON d.name_id = n.id
      WHERE mqd.quote_id = quote.id
      AND mqd.is_named_insured = true
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

##### **Sorting Implementation**
- **Column Headers**: Clickable with sort direction indicators
- **Backend Sorting**: 
  ```sql
  ORDER BY 
    CASE WHEN :sort_column = 'submitted_at' THEN quote.submitted_at END :sort_direction,
    CASE WHEN :sort_column = 'status' THEN status.name END :sort_direction,
    CASE WHEN :sort_column = 'effective_date' THEN quote.effective_date END :sort_direction,
    CASE WHEN :sort_column = 'insured_name' THEN 
      (SELECT CONCAT(n.last_name, ', ', n.first_name) 
       FROM map_quote_driver mqd 
       JOIN driver d ON mqd.driver_id = d.id 
       JOIN name n ON d.name_id = n.id 
       WHERE mqd.quote_id = quote.id AND mqd.is_named_insured = true LIMIT 1)
    END :sort_direction
  ```

#### **3. Quote Flyout Panel Detailed Mappings**

##### **Header Section**

###### **Insured Name**
- **Frontend Display**: Full formatted name
- **Backend Mapping**: 
  ```
  get quote.id
  -> get map_quote_driver by quote.id where is_named_insured = true and status_id = active
  -> get driver by map_quote_driver.driver_id
  -> get name by driver.name_id
  -> concat COALESCE(name.prefix + ' ', ''), name.first_name, COALESCE(' ' + name.middle_name, ''), ' ', name.last_name, COALESCE(' ' + name.suffix, '')
  ```

###### **Quote Status**
- **Frontend Display**: Status name with color indicator and description
- **Backend Mapping**: 
  ```
  get quote.id
  -> get status by quote.status_id
  -> return status.name, status.code, status.description
  ```

###### **Requoted From Information**
- **Frontend Display**: "Requoted from Policy #ABC123 (Renewal)" format
- **Backend Mapping**: 
  ```
  get quote.id
  -> get requote_type by quote.requote_type_id
  -> get policy by quote.original_policy_id
  -> get policy_prefix by policy.policy_prefix_id
  -> return concat('Requoted from Policy #', policy_prefix.value, policy.number, ' (', requote_type.name, ')')
  ```

###### **Submitted On**
- **Frontend Display**: "MM/DD/YYYY at HH:MM AM/PM" with timezone
- **Backend Mapping**: 
  ```
  get quote.id
  -> quote.submitted_at
  -> format with user timezone from user.timezone_preference
  ```

###### **Effective Date**
- **Frontend Display**: "MM/DD/YYYY" format
- **Backend Mapping**: 
  ```
  get quote.id
  -> quote.effective_date
  ```

##### **Insured Details Section**

###### **Primary Phone Number**
- **Frontend Display**: "(XXX) XXX-XXXX" format with extension if applicable
- **Backend Mapping**: 
  ```
  get quote.id
  -> get map_quote_driver by quote.id where is_named_insured = true and status_id = active
  -> get driver by map_quote_driver.driver_id
  -> get map_driver_phone by driver.id where is_primary = true and status_id = active
  -> get phone by map_driver_phone.phone_id
  -> format phone.country_code + phone.number + COALESCE(' ext. ' + phone.extension, '')
  ```

###### **Alternate Phone Number**
- **Frontend Display**: "(XXX) XXX-XXXX" format
- **Backend Mapping**: 
  ```
  get quote.id
  -> get map_quote_driver by quote.id where is_named_insured = true and status_id = active
  -> get driver by map_quote_driver.driver_id
  -> get map_driver_phone by driver.id where is_primary = false and status_id = active limit 1
  -> get phone by map_driver_phone.phone_id
  -> format phone.country_code + phone.number
  ```

###### **Email Address**
- **Frontend Display**: Standard email format with verification status
- **Backend Mapping**: 
  ```
  get quote.id
  -> get map_quote_driver by quote.id where is_named_insured = true and status_id = active
  -> get driver by map_quote_driver.driver_id
  -> get map_driver_email by driver.id where is_primary = true and status_id = active
  -> get email by map_driver_email.email_id
  -> return email.address, map_driver_email.is_verified
  ```

###### **Notification Preference**
- **Frontend Display**: "Email", "Text", or "Phone" preference
- **Backend Mapping**: 
  ```
  get quote.id
  -> get map_quote_driver by quote.id where is_named_insured = true and status_id = active
  -> get driver by map_quote_driver.driver_id
  -> get notification_preference by driver.notification_preference_id
  -> notification_preference.name
  ```

###### **Mailing Address**
- **Frontend Display**: Multi-line formatted address
- **Backend Mapping**: 
  ```
  get quote.id
  -> get map_quote_driver by quote.id where is_named_insured = true and status_id = active
  -> get driver by map_quote_driver.driver_id
  -> get map_driver_address by driver.id where address_type = 'mailing' and is_primary = true and status_id = active
  -> get address by map_driver_address.address_id
  -> format:
     line1: address.line1
     line2: address.line2 (if not null)
     line3: address.city + ', ' + address.state + ' ' + address.zip + COALESCE('-' + address.zip4, '')
  ```

##### **Suspenses Section**

###### **Active Suspenses List**
- **Frontend Display**: Expandable list with priority indicators, due dates, and assignment
- **Backend Mapping**: 
  ```
  get quote.id
  -> get map_quote_suspense by quote.id where status_id = active
  -> get suspense by map_quote_suspense.suspense_id
  -> get suspense_type by suspense.suspense_type_id
  -> get assigned_user by suspense.assigned_to
  -> get suspense_status by suspense.status_id where status.code != 'RESOLVED'
  -> order by field(suspense.priority, 'critical', 'high', 'medium', 'low'), suspense.due_date asc
  -> return:
     suspense_type.name,
     suspense.description,
     suspense.priority,
     suspense.due_date,
     assigned_user.first_name + ' ' + assigned_user.last_name,
     datediff(suspense.due_date, current_date) as days_until_due
  ```

##### **Vehicles Section**

###### **Vehicle Details List**
- **Frontend Display**: Tabular format with year, make, model, VIN, and current registration
- **Backend Mapping**: 
  ```
  get quote.id
  -> get map_quote_vehicle by quote.id where status_id = active
  -> get vehicle by map_quote_vehicle.vehicle_id
  -> get vehicle_registration by vehicle.id where is_current = true and status_id = active
  -> order by map_quote_vehicle.is_primary_vehicle desc, vehicle.year desc
  -> return:
     vehicle.year,
     vehicle.make,
     vehicle.model,
     vehicle.trim,
     vehicle.vin,
     vehicle_registration.plate_number,
     vehicle_registration.registration_state,
     map_quote_vehicle.is_primary_vehicle
  ```

##### **Drivers Section**

###### **Driver Information List**
- **Frontend Display**: Table with driver details, status, and license information
- **Backend Mapping**: 
  ```
  get quote.id
  -> get map_quote_driver by quote.id where status_id = active
  -> get driver by map_quote_driver.driver_id
  -> get driver_type by map_quote_driver.driver_type_id
  -> get name by driver.name_id
  -> get map_driver_license by driver.id where is_primary = true and status_id = active
  -> get license by map_driver_license.license_id
  -> get license_type by license.license_type_id
  -> order by map_quote_driver.is_primary_driver desc, map_quote_driver.is_named_insured desc
  -> return:
     concat(name.first_name, ' ', name.last_name),
     map_quote_driver.is_primary_driver,
     driver_type.name,
     driver.date_of_birth,
     license.state_issued,
     license_type.name,
     license.license_number,
     map_quote_driver.relationship_to_named_insured
  ```

##### **Documents Section**

###### **Document List**
- **Frontend Display**: Categorized document list with upload dates and download links
- **Backend Mapping**: 
  ```
  get quote.id
  -> get map_quote_document by quote.id where status_id = active
  -> get document by map_quote_document.document_id
  -> get document_type by document.document_type_id
  -> order by document_type.category, document.created_at desc
  -> return:
     document_type.name,
     document_type.category,
     document.file_name,
     document.file_size,
     document.created_at,
     map_quote_document.document_purpose,
     map_quote_document.is_required,
     document.file_path
  ```

##### **Coverage Breakdown Section**

###### **Coverage Details**
- **Frontend Display**: Coverage types with limits, deductibles, and premiums
- **Backend Mapping**: 
  ```
  get quote.id
  -> get map_quote_coverage by quote.id where is_included = true and status_id = active
  -> get coverage by map_quote_coverage.coverage_id
  -> get coverage_type by coverage.coverage_type_id
  -> get coverage_limit by map_quote_coverage.limit_id
  -> get deductible by map_quote_coverage.deductible_id
  -> order by coverage_type.display_order, coverage_type.name
  -> return:
     coverage_type.name,
     coverage_type.code,
     coverage_limit.limit_value,
     coverage_limit.limit_type,
     deductible.deductible_value,
     map_quote_coverage.premium
  ```

##### **Rate Information Section**

###### **Prior Policy Indicator**
- **Frontend Display**: "Yes" or "No" with policy details if applicable
- **Backend Mapping**: 
  ```
  get quote.id
  -> if quote.original_policy_id is not null then 'Yes' else 'No'
  -> if 'Yes': get policy by quote.original_policy_id -> get carrier by policy.carrier_id -> return policy details
  ```

###### **Program Information**
- **Frontend Display**: Program name and rate version details
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

###### **Carrier Information**
- **Frontend Display**: Insurance carrier name and contact information
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

#### **4. API Endpoint Specifications**

##### **Quote List API**
```http
GET /api/v1/quotes
Parameters:
  - page: integer (default: 1)
  - per_page: integer (default: 25, max: 100)
  - sort_by: string (submitted_at, status, effective_date, insured_name)
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
  },
  "filters_applied": {...},
  "available_filters": {
    "statuses": [...],
    "producers": [...]
  }
}
```

##### **Quote Search API**
```http
POST /api/v1/quotes/search
Body: {
  "search_term": "string",
  "filters": {
    "status_ids": [1, 2, 3],
    "producer_ids": [10, 20],
    "date_range": {
      "from": "2024-01-01",
      "to": "2024-12-31"
    },
    "coverage_types": ["LIABILITY", "COMPREHENSIVE"]
  },
  "sort": {
    "field": "submitted_at",
    "direction": "desc"
  },
  "pagination": {
    "page": 1,
    "per_page": 25
  }
}

Response: Same as GET /api/v1/quotes
```

##### **Quote Flyout API**
```http
GET /api/v1/quotes/{quote_id}/flyout

Response: {
  "quote": {
    "id": 12345,
    "quote_number": "Q2024-001234",
    "status": {...},
    "dates": {...}
  },
  "insured": {...},
  "vehicles": [...],
  "drivers": [...],
  "coverages": [...],
  "suspenses": [...],
  "documents": [...],
  "rate_info": {...},
  "timeline": [...]
}
```

##### **Real-time WebSocket Channels**
```javascript
// Channel subscriptions
private-tenant.{tenant_id}.quotes        // All quotes for tenant
private-quote.{quote_id}                 // Specific quote updates
private-producer.{producer_id}.quotes    // Producer's quotes

// Event types
quote.created
quote.updated
quote.status_changed
quote.bound
quote.expired
suspense.created
suspense.assigned
suspense.resolved
```

#### **5. Performance Optimization Strategies**

##### **Database Query Optimization**
- Implement covering indexes for common query patterns
- Use database query result caching for expensive joins
- Implement read replicas for search queries
- Use database connection pooling

##### **Application-Level Caching**
```php
// Redis cache implementation
Cache::remember("quote_flyout_{$quoteId}", 600, function() use ($quoteId) {
    return $this->buildFlyoutData($quoteId);
});

Cache::remember("quote_search_{$tenantId}_{$cacheKey}", 300, function() use ($searchParams) {
    return $this->performSearch($searchParams);
});
```

##### **Search Performance**
- Initial: Optimized MariaDB full-text search with strategic indexes
- Future: Elasticsearch integration via Laravel Scout for advanced search capabilities
- Implement search result pagination with cursor-based pagination for large datasets

---

## **E) Master Schema Tables**

### **1. Core Quote Tables**

#### **quote**
```sql
CREATE TABLE quote (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  tenant_id BIGINT UNSIGNED NOT NULL,
  quote_number VARCHAR(50) UNIQUE NOT NULL,
  status_id BIGINT UNSIGNED NOT NULL,
  program_id BIGINT UNSIGNED NOT NULL,
  
  -- Requote tracking
  requote_type_id BIGINT UNSIGNED NULL,
  original_policy_id BIGINT UNSIGNED NULL,
  
  -- Date fields
  effective_date DATE NOT NULL,
  expiration_date DATE NULL,
  
  -- Financial fields
  total_premium DECIMAL(10,2) DEFAULT 0.00,
  total_fees DECIMAL(10,2) DEFAULT 0.00,
  total_taxes DECIMAL(10,2) DEFAULT 0.00,
  
  -- Workflow timestamps
  submitted_at TIMESTAMP NULL,
  bound_at TIMESTAMP NULL,
  expired_at TIMESTAMP NULL,
  
  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (tenant_id) REFERENCES tenant(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (program_id) REFERENCES program(id),
  FOREIGN KEY (requote_type_id) REFERENCES requote_type(id),
  FOREIGN KEY (original_policy_id) REFERENCES policy(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  -- Performance indexes
  INDEX idx_tenant_status (tenant_id, status_id),
  INDEX idx_submitted_date (submitted_at DESC),
  INDEX idx_effective_date (effective_date),
  INDEX idx_quote_number (quote_number),
  INDEX idx_tenant_submitted (tenant_id, submitted_at DESC),
  INDEX idx_program (program_id),
  INDEX idx_original_policy (original_policy_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### **driver**
```sql
CREATE TABLE driver (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  name_id BIGINT UNSIGNED NOT NULL,
  date_of_birth DATE NOT NULL,
  gender ENUM('M', 'F', 'X') NULL,
  marital_status_id BIGINT UNSIGNED NULL,
  notification_preference_id BIGINT UNSIGNED NULL,
  occupation VARCHAR(100) NULL,
  years_licensed INT NULL,
  status_id BIGINT UNSIGNED NOT NULL,
  
  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (name_id) REFERENCES name(id),
  FOREIGN KEY (marital_status_id) REFERENCES marital_status(id),
  FOREIGN KEY (notification_preference_id) REFERENCES notification_preference(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  -- Indexes
  INDEX idx_name (name_id),
  INDEX idx_dob (date_of_birth),
  INDEX idx_status (status_id),
  INDEX idx_marital_status (marital_status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### **vehicle**
```sql
CREATE TABLE vehicle (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  vin VARCHAR(17) NOT NULL,
  year INT NOT NULL,
  make VARCHAR(50) NOT NULL,
  model VARCHAR(50) NOT NULL,
  trim VARCHAR(50) NULL,
  body_type VARCHAR(50) NULL,
  engine_type VARCHAR(50) NULL,
  fuel_type VARCHAR(30) NULL,
  color VARCHAR(30) NULL,
  purchase_date DATE NULL,
  purchase_price DECIMAL(10,2) NULL,
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
  UNIQUE INDEX idx_vin (vin),
  INDEX idx_year_make_model (year, make, model),
  INDEX idx_status (status_id),
  INDEX idx_make_model (make, model)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### **license**
```sql
CREATE TABLE license (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  license_type_id BIGINT UNSIGNED NOT NULL,
  license_number VARCHAR(50) NOT NULL,
  state_issued CHAR(2) NOT NULL,
  license_class VARCHAR(10) NULL,
  issue_date DATE NULL,
  expiration_date DATE NULL,
  restrictions TEXT NULL,
  endorsements TEXT NULL,
  verification_status ENUM('unverified', 'verified', 'failed') DEFAULT 'unverified',
  verification_date TIMESTAMP NULL,
  additional_data JSON NULL,
  status_id BIGINT UNSIGNED NOT NULL,
  
  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (license_type_id) REFERENCES license_type(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  -- Indexes
  INDEX idx_license_state (license_number, state_issued),
  INDEX idx_license_type (license_type_id),
  INDEX idx_expiration (expiration_date),
  INDEX idx_status (status_id),
  INDEX idx_verification (verification_status)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### **document**
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
  INDEX idx_status (status_id),
  INDEX idx_hash (file_hash)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### **suspense**
```sql
CREATE TABLE suspense (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  suspense_type_id BIGINT UNSIGNED NOT NULL,
  description TEXT NOT NULL,
  priority ENUM('low', 'medium', 'high', 'critical') DEFAULT 'medium',
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
  INDEX idx_priority_status (priority, status_id),
  INDEX idx_status (status_id),
  INDEX idx_created (created_at DESC)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### **vehicle_registration**
```sql
CREATE TABLE vehicle_registration (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  vehicle_id BIGINT UNSIGNED NOT NULL,
  plate_number VARCHAR(20) NOT NULL,
  registration_state CHAR(2) NOT NULL,
  registration_date DATE NOT NULL,
  expiration_date DATE NOT NULL,
  registration_type ENUM('standard', 'commercial', 'temporary', 'dealer') DEFAULT 'standard',
  is_current BOOLEAN DEFAULT TRUE,
  status_id BIGINT UNSIGNED NOT NULL,
  
  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (vehicle_id) REFERENCES vehicle(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  -- Indexes
  INDEX idx_vehicle_current (vehicle_id, is_current),
  INDEX idx_plate_state (plate_number, registration_state),
  INDEX idx_expiration (expiration_date),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### **2. Reference Tables**

#### **requote_type**
```sql
CREATE TABLE requote_type (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) UNIQUE NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT NULL,
  requires_original_policy BOOLEAN DEFAULT TRUE,
  workflow_rules JSON NULL,
  display_order INT DEFAULT 0,
  status_id BIGINT UNSIGNED NOT NULL,
  
  -- Audit fields
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (status_id) REFERENCES status(id),
  
  -- Indexes
  INDEX idx_code (code),
  INDEX idx_status (status_id),
  INDEX idx_display_order (display_order)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### **driver_type**
```sql
CREATE TABLE driver_type (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) UNIQUE NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT NULL,
  mga_specific_code VARCHAR(50) NULL,
  business_rules JSON NULL,
  display_order INT DEFAULT 0,
  status_id BIGINT UNSIGNED NOT NULL,
  
  -- Audit fields
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (status_id) REFERENCES status(id),
  
  -- Indexes
  INDEX idx_code (code),
  INDEX idx_mga_code (mga_specific_code),
  INDEX idx_status (status_id),
  INDEX idx_display_order (display_order)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### **license_type**
```sql
CREATE TABLE license_type (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) UNIQUE NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT NULL,
  required_fields JSON NULL,
  validation_rules JSON NULL,
  display_order INT DEFAULT 0,
  status_id BIGINT UNSIGNED NOT NULL,
  
  -- Audit fields
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (status_id) REFERENCES status(id),
  
  -- Indexes
  INDEX idx_code (code),
  INDEX idx_status (status_id),
  INDEX idx_display_order (display_order)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### **suspense_type**
```sql
CREATE TABLE suspense_type (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) UNIQUE NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT NULL,
  default_priority ENUM('low', 'medium', 'high', 'critical') DEFAULT 'medium',
  auto_assign_rules JSON NULL,
  escalation_rules JSON NULL,
  sla_hours INT NULL,
  display_order INT DEFAULT 0,
  status_id BIGINT UNSIGNED NOT NULL,
  
  -- Audit fields
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (status_id) REFERENCES status(id),
  
  -- Indexes
  INDEX idx_code (code),
  INDEX idx_status (status_id),
  INDEX idx_priority (default_priority),
  INDEX idx_display_order (display_order)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### **document_type**
```sql
CREATE TABLE document_type (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) UNIQUE NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT NULL,
  category VARCHAR(50) NULL,
  allowed_extensions JSON NULL,
  max_file_size BIGINT NULL,
  is_required_default BOOLEAN DEFAULT FALSE,
  display_order INT DEFAULT 0,
  status_id BIGINT UNSIGNED NOT NULL,
  
  -- Audit fields
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (status_id) REFERENCES status(id),
  
  -- Indexes
  INDEX idx_code (code),
  INDEX idx_category (category),
  INDEX idx_status (status_id),
  INDEX idx_display_order (display_order)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### **notification_preference**
```sql
CREATE TABLE notification_preference (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) UNIQUE NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT NULL,
  channels JSON NOT NULL, -- ["email", "sms", "phone", "portal"]
  display_order INT DEFAULT 0,
  status_id BIGINT UNSIGNED NOT NULL,
  
  -- Audit fields
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (status_id) REFERENCES status(id),
  
  -- Indexes
  INDEX idx_code (code),
  INDEX idx_status (status_id),
  INDEX idx_display_order (display_order)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### **marital_status**
```sql
CREATE TABLE marital_status (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) UNIQUE NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT NULL,
  display_order INT DEFAULT 0,
  status_id BIGINT UNSIGNED NOT NULL,
  
  -- Audit fields
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (status_id) REFERENCES status(id),
  
  -- Indexes
  INDEX idx_code (code),
  INDEX idx_status (status_id),
  INDEX idx_display_order (display_order)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### **3. Relationship Tables (map_*)**

#### **map_producer_quote**
```sql
CREATE TABLE map_producer_quote (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  producer_id BIGINT UNSIGNED NOT NULL,
  quote_id BIGINT UNSIGNED NOT NULL,
  relationship_type ENUM('primary', 'secondary', 'servicing') DEFAULT 'primary',
  commission_percentage DECIMAL(5,2) NULL,
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
  UNIQUE KEY unique_producer_quote_type (producer_id, quote_id, relationship_type),
  
  -- Indexes
  INDEX idx_producer (producer_id),
  INDEX idx_quote (quote_id),
  INDEX idx_relationship (relationship_type),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### **map_quote_driver**
```sql
CREATE TABLE map_quote_driver (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  quote_id BIGINT UNSIGNED NOT NULL,
  driver_id BIGINT UNSIGNED NOT NULL,
  driver_type_id BIGINT UNSIGNED NOT NULL,
  is_primary_driver BOOLEAN DEFAULT FALSE,
  is_named_insured BOOLEAN DEFAULT FALSE,
  relationship_to_named_insured VARCHAR(50) NULL,
  status_id BIGINT UNSIGNED NOT NULL,
  
  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (quote_id) REFERENCES quote(id) ON DELETE CASCADE,
  FOREIGN KEY (driver_id) REFERENCES driver(id),
  FOREIGN KEY (driver_type_id) REFERENCES driver_type(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  -- Constraints
  UNIQUE KEY unique_quote_driver (quote_id, driver_id),
  UNIQUE KEY unique_named_insured (quote_id, is_named_insured) WHERE is_named_insured = TRUE,
  
  -- Indexes
  INDEX idx_quote (quote_id),
  INDEX idx_driver (driver_id),
  INDEX idx_driver_type (driver_type_id),
  INDEX idx_primary (quote_id, is_primary_driver),
  INDEX idx_named_insured (quote_id, is_named_insured),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### **map_quote_vehicle**
```sql
CREATE TABLE map_quote_vehicle (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  quote_id BIGINT UNSIGNED NOT NULL,
  vehicle_id BIGINT UNSIGNED NOT NULL,
  is_primary_vehicle BOOLEAN DEFAULT FALSE,
  garaging_address_id BIGINT UNSIGNED NULL,
  annual_mileage INT NULL,
  business_use BOOLEAN DEFAULT FALSE,
  vehicle_use_type ENUM('pleasure', 'commute', 'business', 'farm') DEFAULT 'pleasure',
  status_id BIGINT UNSIGNED NOT NULL,
  
  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (quote_id) REFERENCES quote(id) ON DELETE CASCADE,
  FOREIGN KEY (vehicle_id) REFERENCES vehicle(id),
  FOREIGN KEY (garaging_address_id) REFERENCES address(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  -- Constraints
  UNIQUE KEY unique_quote_vehicle (quote_id, vehicle_id),
  
  -- Indexes
  INDEX idx_quote (quote_id),
  INDEX idx_vehicle (vehicle_id),
  INDEX idx_primary (quote_id, is_primary_vehicle),
  INDEX idx_garaging (garaging_address_id),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### **map_driver_license**
```sql
CREATE TABLE map_driver_license (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  driver_id BIGINT UNSIGNED NOT NULL,
  license_id BIGINT UNSIGNED NOT NULL,
  is_primary BOOLEAN DEFAULT TRUE,
  verified_date TIMESTAMP NULL,
  verification_method VARCHAR(50) NULL,
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
  UNIQUE KEY unique_primary_license (driver_id, is_primary) WHERE is_primary = TRUE,
  
  -- Indexes
  INDEX idx_driver (driver_id),
  INDEX idx_license (license_id),
  INDEX idx_primary (driver_id, is_primary),
  INDEX idx_verified (verified_date),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### **map_quote_document**
```sql
CREATE TABLE map_quote_document (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  quote_id BIGINT UNSIGNED NOT NULL,
  document_id BIGINT UNSIGNED NOT NULL,
  document_purpose VARCHAR(50) NOT NULL,
  is_required BOOLEAN DEFAULT FALSE,
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
  INDEX idx_purpose (document_purpose),
  INDEX idx_required (quote_id, is_required),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### **map_driver_phone**
```sql
CREATE TABLE map_driver_phone (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  driver_id BIGINT UNSIGNED NOT NULL,
  phone_id BIGINT UNSIGNED NOT NULL,
  phone_type ENUM('mobile', 'home', 'work', 'other') DEFAULT 'mobile',
  is_primary BOOLEAN DEFAULT FALSE,
  can_sms BOOLEAN DEFAULT FALSE,
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
  INDEX idx_type (phone_type),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### **map_driver_email**
```sql
CREATE TABLE map_driver_email (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  driver_id BIGINT UNSIGNED NOT NULL,
  email_id BIGINT UNSIGNED NOT NULL,
  email_type ENUM('personal', 'work', 'other') DEFAULT 'personal',
  is_primary BOOLEAN DEFAULT FALSE,
  is_verified BOOLEAN DEFAULT FALSE,
  verified_at TIMESTAMP NULL,
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
  INDEX idx_verified (is_verified),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### **map_driver_address**
```sql
CREATE TABLE map_driver_address (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  driver_id BIGINT UNSIGNED NOT NULL,
  address_id BIGINT UNSIGNED NOT NULL,
  address_type ENUM('mailing', 'garaging', 'previous', 'business') DEFAULT 'mailing',
  is_primary BOOLEAN DEFAULT FALSE,
  effective_date DATE NULL,
  end_date DATE NULL,
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
  UNIQUE KEY unique_driver_address_type (driver_id, address_id, address_type),
  
  -- Indexes
  INDEX idx_driver (driver_id),
  INDEX idx_address (address_id),
  INDEX idx_primary (driver_id, is_primary),
  INDEX idx_type (address_type),
  INDEX idx_effective (effective_date),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### **map_quote_coverage**
```sql
CREATE TABLE map_quote_coverage (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  quote_id BIGINT UNSIGNED NOT NULL,
  coverage_id BIGINT UNSIGNED NOT NULL,
  limit_id BIGINT UNSIGNED NULL,
  deductible_id BIGINT UNSIGNED NULL,
  premium DECIMAL(10,2) DEFAULT 0.00,
  is_included BOOLEAN DEFAULT TRUE,
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
  INDEX idx_included (quote_id, is_included),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### **map_quote_suspense**
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

#### **map_policy_suspense**
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

#### **map_loss_suspense**
```sql
CREATE TABLE map_loss_suspense (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  loss_id BIGINT UNSIGNED NOT NULL,
  suspense_id BIGINT UNSIGNED NOT NULL,
  status_id BIGINT UNSIGNED NOT NULL,
  
  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (loss_id) REFERENCES loss(id) ON DELETE CASCADE,
  FOREIGN KEY (suspense_id) REFERENCES suspense(id) ON DELETE CASCADE,
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  -- Constraints
  UNIQUE KEY unique_loss_suspense (loss_id, suspense_id),
  
  -- Indexes
  INDEX idx_loss (loss_id),
  INDEX idx_suspense (suspense_id),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### **4. Supporting Entity Tables**

#### **phone**
```sql
CREATE TABLE phone (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  country_code VARCHAR(5) DEFAULT '+1',
  number VARCHAR(20) NOT NULL,
  extension VARCHAR(10) NULL,
  is_mobile BOOLEAN DEFAULT FALSE,
  carrier VARCHAR(50) NULL,
  is_verified BOOLEAN DEFAULT FALSE,
  verified_at TIMESTAMP NULL,
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
  INDEX idx_number (number),
  INDEX idx_country_number (country_code, number),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### **email**
```sql
CREATE TABLE email (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  address VARCHAR(255) NOT NULL,
  domain VARCHAR(255) GENERATED ALWAYS AS (SUBSTRING_INDEX(address, '@', -1)) STORED,
  is_valid BOOLEAN DEFAULT TRUE,
  bounce_count INT DEFAULT 0,
  last_verified TIMESTAMP NULL,
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
  UNIQUE INDEX idx_address (address),
  INDEX idx_domain (domain),
  INDEX idx_valid (is_valid),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### **address**
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
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  -- Indexes
  INDEX idx_zip (zip),
  INDEX idx_state_city (state, city),
  INDEX idx_geocode (latitude, longitude),
  INDEX idx_validated (is_validated),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### **name**
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

### **5. Technology Stack Requirements**

#### **Required Technologies**
- **Backend Framework**: Laravel 12.x with PHP 8.4+
- **Database**: MariaDB 12.x LTS with read replicas for search performance
- **Cache Layer**: Redis 7.x for session management and query result caching
- **Search Infrastructure**: 
  - Phase 1: Optimized MariaDB queries with full-text search indexes
  - Phase 2: Elasticsearch 8.x integration via Laravel Scout
- **Real-time Communications**: Laravel Echo + Pusher for WebSocket connections
- **API Management**: Kong API Gateway for security and rate limiting
- **Queue Processing**: Laravel Horizon with Redis backend for background tasks
- **File Storage**: S3-compatible storage for document management
- **CDN**: CloudFront for static asset delivery

#### **Development & Monitoring Tools**
- **API Documentation**: OpenAPI/Swagger specifications
- **Development Debugging**: Laravel Telescope
- **Query Performance**: Laravel Debugbar
- **Testing Framework**: PHPUnit with Laravel Test Helpers
- **Code Quality**: PHP_CodeSniffer and Laravel Pint

#### **Performance Optimization**
- **Database Indexes**: Covering indexes for all common query patterns
- **Query Caching**: Redis-based result caching with 5-minute TTL
- **Read Replicas**: MariaDB read replicas for search and reporting queries
- **Connection Pooling**: Database connection pooling for high concurrency
- **CDN Integration**: CloudFront for static assets and document serving

#### **Security & Compliance**
- **Authentication**: Laravel Passport/Sanctum for API authentication
- **Authorization**: Role-based access control (RBAC) with producer permissions
- **Data Encryption**: Database encryption at rest for sensitive data
- **Audit Logging**: Comprehensive action logging using existing action table
- **Rate Limiting**: API rate limiting via Kong Gateway
- **CORS Configuration**: Proper cross-origin resource sharing setup