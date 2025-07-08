# IP269-New-Quote-Step-3-Vehicles - Complete Requirement

---

## **A) WHY – Vision and Purpose**

The goal of this step is to ensure that all vehicles to be covered under the policy are correctly added, verified, and associated with the policyholder. This ensures:

- Proper premium calculation
- Accurate risk assessment
- Full coverage details for underwriting and legal purposes

It allows for both **lookup-based vehicle matching** from third-party data sources and **manual entry** when lookup is insufficient or data is unavailable.

---

## **B) WHAT – Core Requirements**

### 1. **Vehicle Lookup**

- Vehicle look-up occurs against the primary insured's address, for any vehicles associated with their address as provided
- This pulls available vehicle data from third party-sources, and displays returned results in a list format for review before confirmation
- Vehicles are then presented in a list format, where the user can add or remove vehicles from the policy

### 2. **Manual Entry Flow**

- User may add a vehicle not returned in the search results by manually entering vehicle information

---

## Entity Analysis

### Entities Involved
| Entity Name | Type | Status | Notes |
|-------------|------|--------|--------|
| vehicle | Core | Existing | Core vehicle entity (needs modal→model typo fix) |
| map_quote_vehicle | Map | New | Quote-vehicle relationships |
| vehicle_usage_type | Reference | New | Usage type classification |
| vehicle_owner | Core | New | Vehicle ownership tracking |
| map_vehicle_owner | Map | New | Vehicle-owner relationships |
| map_vehicle_driver | Map | New | Primary driver assignments |
| DCS_HOUSEHOLD_VEHICLES | External | Existing | DCS vehicle lookup API |
| address | Supporting | Existing | Garaging address storage |

### New Tables Required
- **map_quote_vehicle**: Quote-vehicle relationship with usage type
- **vehicle_usage_type**: Vehicle usage classification (pleasure, commute, business)
- **vehicle_owner**: Vehicle ownership tracking
- **map_vehicle_owner**: Vehicle-owner relationships
- **map_vehicle_driver**: Vehicle-driver assignments (primary driver)

### Modifications to Existing Tables
- **vehicle**: Fix `modal` typo to `model`, add garaging_address_id, external_source_id
- **driver**: Support vehicle owner role identification

### Relationships Identified
- quote → vehicle (many-to-many via map_quote_vehicle)
- vehicle → vehicle_usage_type (many-to-one via map_quote_vehicle)
- vehicle → vehicle_owner (many-to-many via map_vehicle_owner)
- vehicle → driver (many-to-one for primary driver via map_vehicle_driver)
- vehicle → address (many-to-one for garaging address)

---

## Field Mappings (Section C)

### Backend Mappings

#### Vehicle Lookup (DCS Integration)

##### Household Vehicle Search
- **Backend Mapping**: 
  ```
  get quote.id from current_quote
  -> get address.* from primary_insured_address via quote → driver → address
  -> call DCS_HOUSEHOLD_VEHICLES API with address.street, address.city, address.state, address.zip
  -> parse DCS response for household_vehicles[]
  -> return vehicle_lookup_results with vehicle.vin, vehicle.year, vehicle.make, vehicle.model, 
           vehicle.registration_info, vehicle.owner_name
  ```

##### Vehicle Selection from Lookup Results
- **Backend Mapping**:
  ```
  get selected_vehicle from DCS_lookup_results
  -> create vehicle entity if not exists:
     vehicle.vin, vehicle.year, vehicle.make, vehicle.model, vehicle.external_source_id = 'DCS'
  -> create map_quote_vehicle with usage_type_id selection
  -> verify vehicle_owner against Step 2 driver roster
  -> return vehicle_added_status, owner_verification_required
  ```

#### Manual Vehicle Entry

##### VIN-Based Manual Entry
- **Backend Mapping**:
  ```
  validate vin_format and vin_checksum
  -> call VIN_DECODER API with vehicle.vin
  -> auto-populate vehicle.year, vehicle.make, vehicle.model from VIN decode
  -> create vehicle entity with manual_entry = true
  -> create map_quote_vehicle with usage_type_id and garaging_address_id
  -> return vehicle_created, decoded_specifications
  ```

##### Year/Make/Model Manual Entry
- **Backend Mapping**:
  ```
  validate year_range (current_year - 50 to current_year + 1)
  -> search vehicle_specifications by year, make, model
  -> return matching_vehicle_variants[] for selection
  -> create vehicle entity with selected_variant
  -> require license_plate_number and license_plate_state
  -> create map_quote_vehicle with complete vehicle info
  ```

#### Vehicle Usage and Assignment

##### Usage Type Selection
- **Backend Mapping**:
  ```
  get vehicle_usage_type.* from vehicle_usage_type where status_id = :active
  -> return usage_type_options[] (pleasure, commute, business, farm)
  -> update map_quote_vehicle.usage_type_id on selection
  -> validate usage_type against program restrictions if applicable
  ```

##### Garaging Address Management
- **Backend Mapping**:
  ```
  if garaging_address != primary_insured_address:
    -> create new address entity or select existing
    -> validate garaging_zip_code format by state
    -> update vehicle.garaging_address_id
  else:
    -> set vehicle.garaging_address_id = primary_insured_address_id
  ```

##### Primary Driver Assignment
- **Backend Mapping**:
  ```
  get available_drivers from Step 2 completed_driver_roster
  -> where map_quote_driver.include_status = 'included'
  -> return driver_assignment_options[]
  -> create map_vehicle_driver with primary_driver_assignment
  -> validate driver eligibility for vehicle assignment
  ```

#### Vehicle Owner Management

##### Owner Verification Against Driver Roster
- **Backend Mapping**:
  ```
  get vehicle_owner_name from DCS_vehicle_data OR manual_entry
  -> search existing_drivers in Step 2 driver_roster by name_matching
  -> if owner_found_in_roster:
       create map_vehicle_owner linking to existing driver
     else:
       return owner_addition_required = true, trigger_driver_addition_workflow
  ```

##### Owner Addition Workflow Integration
- **Backend Mapping**:
  ```
  if vehicle_owner NOT in driver_roster:
    -> trigger Step 2 driver_addition_workflow
    -> collect owner_info: name, relationship_to_insured, include_or_exclude_decision
    -> if include_status = 'excluded':
         collect gender, marital_status, relationship_to_insured
       if include_status = 'included':
         collect full_driver_info: gender, marital_status, employment, sr22, violations
    -> create driver entity and map_quote_driver relationship
    -> create map_vehicle_owner linking vehicle to new_driver
  ```

#### Vehicle List Management

##### All Vehicles Display
- **Backend Mapping**:
  ```
  get map_quote_vehicle.* from map_quote_vehicle where quote_id = :quote_id
  -> join vehicle on map_quote_vehicle.vehicle_id = vehicle.id
  -> join vehicle_usage_type on map_quote_vehicle.usage_type_id = vehicle_usage_type.id
  -> left join map_vehicle_driver on vehicle.id = map_vehicle_driver.vehicle_id
  -> left join driver on map_vehicle_driver.driver_id = driver.id
  -> return vehicles[] with usage_type, primary_driver, garaging_address, data_source
  ```

##### Vehicle Removal
- **Backend Mapping**:
  ```
  validate vehicle_removal_permissions
  -> update map_quote_vehicle.status_id = inactive OR delete map_quote_vehicle record
  -> check if vehicle_owner is only associated with removed_vehicle
  -> if owner_has_no_other_vehicles: prompt for driver_removal from policy
  -> return removal_status, affected_drivers[]
  ```

### Implementation Architecture

**Cross-Step Integration Strategy**: Building on completed driver selection (Step 2) and established DCS integration patterns (Step 1), this step focuses on vehicle management with driver assignment integration and owner verification workflows.

**Three-Path Vehicle Addition**: 
1. **DCS Lookup**: Household vehicle search using Step 1 address data
2. **VIN Entry**: VIN decoder with auto-population
3. **YMM Search**: Year/Make/Model lookup with variant selection

**Owner-Driver Integration**: Seamless integration with Step 2 driver roster for owner verification, with automatic driver addition workflow when vehicle owners not found in existing driver list.

**Data Persistence Consistency**: Following stakeholder-confirmed direct storage approach (no caching) while maintaining data flow across workflow steps.

### Integration Specifications

#### DCS Household Vehicles API Integration

**Entity Type**: DCS_HOUSEHOLD_VEHICLES (Universal Entity Management)  
**Provider**: Data Collection Services  
**Endpoint**: DCS Household Vehicles API (GR-53)

#### Performance & Monitoring

**Response Time Targets**:
- Vehicle lookup display: < 5 seconds (DCS integration)
- VIN decoding: < 3 seconds
- Owner verification: < 500ms
- Driver assignment: < 200ms
- Vehicle list refresh: < 300ms

**Fallback Strategies**:
- DCS vehicle lookup → Manual entry only
- VIN decoder → Manual year/make/model entry
- Owner verification → Manual driver addition workflow

#### Error Handling

**DCS Integration Failures**: Graceful fallback to manual vehicle entry
**VIN Decode Failures**: Fallback to manual specification entry
**Owner Addition**: Seamless integration with Step 2 driver addition workflow

---

## **D) User Experience (UX) & Flows**

### **Vehicle Lookup and Selection Flow (Building on Step 1 DCS Patterns)**

1. Automatic household vehicle lookup using primary insured address from Step 1
2. Display lookup results in organized list format
3. Vehicle selection with usage type assignment
4. Owner verification against Step 2 driver roster

### **Adding Returned Vehicle**

1. For vehicles returned in the vehicle look-up, they can be added by clicking the "Add" CTA.
2. This will open the side panel, where the user will include the usage type, before adding the vehicle.
3. Owner verification check against Step 2 driver roster
4. The vehicle will then show in the "All Vehicles in the Household" section of the vehicle list.

### **Add Vehicle Manually - By VIN**

1. For vehicles not returned in the vehicle look-up, they can be added manually by clicking the "Manually Add" CTA.
2. If the user has the VIN available, they can populate the VIN, Usage Type, and Garaging ZIP Code, before initiating search.
3. VIN decoder auto-populates year/make/model specifications
4. Owner verification and driver assignment
5. If the vehicle information is returned, the vehicle will be added to the policy and listed in the "All Vehicles in the Household" section of the vehicle list.

### **Add Vehicle Manually - By Year/Make/Model**

1. For vehicles not returned in the vehicle look-up, they can be added manually by clicking the "Manually Add" CTA.
2. If the user does not have the VIN available, they can look the vehicle up by Year, Make, and Model. They must also provide the usage type, and the garaging address for the vehicle, before initiating search.
3. If the vehicle information matches an existing vehicle, they will select the vehicle from the list of options, and provide the license plate number and license plate state.
4. Owner verification and driver assignment from Step 2 roster
5. Upon completion, the vehicle will be added to the policy and listed in the "All Vehicles in the Household" section of the vehicle list.

### **Add Vehicle Owners to Policy (Step 2 Integration)**

1. If the user attempts to add a vehicle whose registered owner is not on the policy, the user must add the registered owner to the policy before the vehicle can be added.
2. **Integration with Step 2 driver addition workflow**
3. To complete this workflow, the user must confirm if the owner will be included or excluded on the policy.
    1. If excluded, populate the gender, marital status, and relationship to the insured.
    2. If included, populate the gender, marital status, employment information, SR-22, and violations, if applicable.

---

## API Specifications

### Endpoints Required
```http
GET    /api/v1/quotes/{id}/vehicles           # List all vehicles for quote
GET    /api/v1/quotes/{id}/vehicle-lookup     # DCS household vehicle lookup
POST   /api/v1/quotes/{id}/vehicles           # Add vehicle to quote
PUT    /api/v1/quotes/{id}/vehicles/{vehicleId} # Update vehicle details
DELETE /api/v1/quotes/{id}/vehicles/{vehicleId} # Remove vehicle from quote
POST   /api/v1/vehicles/decode-vin            # Decode VIN specifications
POST   /api/v1/vehicles/search-ymm            # Search by year/make/model
PUT    /api/v1/vehicles/{id}/primary-driver   # Assign primary driver
POST   /api/v1/vehicles/{id}/verify-owner     # Verify vehicle owner
GET    /api/v1/reference/vehicle-usage-types  # Get usage type options
GET    /api/v1/quotes/{id}/available-drivers  # Get available drivers for assignment
```

### Real-time Updates
```javascript
// WebSocket channels
private-quote.{id}.vehicles                  # Vehicle data updates
private-quote.{id}.driver-assignments        # Driver assignment updates
```

---

## Database Schema (Section E)

### New Core Tables

#### vehicle_owner
```sql
CREATE TABLE vehicle_owner (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  
  -- Business columns
  name VARCHAR(200) NOT NULL,
  relationship_to_vehicle ENUM('registered_owner', 'lienholder', 'lessee') NOT NULL,
  ownership_percentage DECIMAL(5,2) DEFAULT 100.00,
  external_source_id VARCHAR(100) NULL,
  
  -- Status and audit
  status_id BIGINT UNSIGNED NOT NULL,
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  -- Indexes
  INDEX idx_name (name),
  INDEX idx_relationship (relationship_to_vehicle),
  INDEX idx_external_source (external_source_id),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### New Reference Tables

#### vehicle_usage_type
```sql
CREATE TABLE vehicle_usage_type (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) UNIQUE NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT NULL,
  mileage_category ENUM('low', 'medium', 'high') NOT NULL,
  commercial_use BOOLEAN DEFAULT FALSE,
  status_id BIGINT UNSIGNED NOT NULL,
  
  -- Audit fields
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (status_id) REFERENCES status(id),
  
  -- Indexes
  INDEX idx_code (code),
  INDEX idx_commercial_use (commercial_use),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### New Relationship Tables

#### map_quote_vehicle
```sql
CREATE TABLE map_quote_vehicle (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  quote_id BIGINT UNSIGNED NOT NULL,
  vehicle_id BIGINT UNSIGNED NOT NULL,
  usage_type_id BIGINT UNSIGNED NOT NULL,
  garaging_address_id BIGINT UNSIGNED NULL,
  annual_mileage INT NULL,
  status_id BIGINT UNSIGNED NOT NULL,
  
  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (quote_id) REFERENCES quote(id) ON DELETE CASCADE,
  FOREIGN KEY (vehicle_id) REFERENCES vehicle(id),
  FOREIGN KEY (usage_type_id) REFERENCES vehicle_usage_type(id),
  FOREIGN KEY (garaging_address_id) REFERENCES address(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  -- Constraints
  UNIQUE KEY unique_quote_vehicle (quote_id, vehicle_id),
  
  -- Indexes
  INDEX idx_quote (quote_id),
  INDEX idx_vehicle (vehicle_id),
  INDEX idx_usage_type (usage_type_id),
  INDEX idx_garaging_address (garaging_address_id),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### map_vehicle_owner
```sql
CREATE TABLE map_vehicle_owner (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  vehicle_id BIGINT UNSIGNED NOT NULL,
  vehicle_owner_id BIGINT UNSIGNED NOT NULL,
  driver_id BIGINT UNSIGNED NULL,
  status_id BIGINT UNSIGNED NOT NULL,
  
  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (vehicle_id) REFERENCES vehicle(id) ON DELETE CASCADE,
  FOREIGN KEY (vehicle_owner_id) REFERENCES vehicle_owner(id),
  FOREIGN KEY (driver_id) REFERENCES driver(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  -- Constraints
  UNIQUE KEY unique_vehicle_owner (vehicle_id, vehicle_owner_id),
  
  -- Indexes
  INDEX idx_vehicle (vehicle_id),
  INDEX idx_owner (vehicle_owner_id),
  INDEX idx_driver (driver_id),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### map_vehicle_driver
```sql
CREATE TABLE map_vehicle_driver (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  vehicle_id BIGINT UNSIGNED NOT NULL,
  driver_id BIGINT UNSIGNED NOT NULL,
  is_primary_driver BOOLEAN DEFAULT FALSE,
  assignment_date DATE NOT NULL,
  status_id BIGINT UNSIGNED NOT NULL,
  
  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (vehicle_id) REFERENCES vehicle(id) ON DELETE CASCADE,
  FOREIGN KEY (driver_id) REFERENCES driver(id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  -- Constraints
  UNIQUE KEY unique_vehicle_driver (vehicle_id, driver_id),
  
  -- Indexes
  INDEX idx_vehicle (vehicle_id),
  INDEX idx_driver (driver_id),
  INDEX idx_primary_driver (is_primary_driver),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### Modified Tables

#### ALTER TABLE vehicle
```sql
-- Fix typo and add DCS integration fields
ALTER TABLE vehicle 
CHANGE COLUMN modal model VARCHAR(100) NOT NULL,
ADD COLUMN external_source_id VARCHAR(100) NULL,
ADD COLUMN vin_decoded_date TIMESTAMP NULL,
ADD COLUMN garaging_address_id BIGINT UNSIGNED NULL,
ADD COLUMN manual_entry BOOLEAN DEFAULT FALSE;

-- Add foreign key constraint
ALTER TABLE vehicle
ADD CONSTRAINT fk_vehicle_garaging_address 
FOREIGN KEY (garaging_address_id) REFERENCES address(id);

-- Add indexes
ALTER TABLE vehicle
ADD INDEX idx_external_source (external_source_id),
ADD INDEX idx_garaging_address (garaging_address_id),
ADD INDEX idx_manual_entry (manual_entry);
```

---

## Implementation Notes

### Dependencies
- Step 1 Primary Insured completion (address data for DCS lookup)
- Step 2 Drivers completion (driver roster for owner verification and assignment)
- DCS Household Vehicles API integration (GR-53)
- VIN decoder service integration
- Driver addition workflow integration (from Step 2)

### Migration Considerations
- Fix existing `modal` typo to `model` in vehicle table
- Vehicle-quote relationship establishment
- Driver-vehicle assignment data migration
- Usage type reference data setup

### Performance Considerations
- DCS vehicle lookup response time optimization
- VIN decoding service performance
- Large household vehicle list pagination
- Driver assignment query optimization

---

## Quality Checklist

### Pre-Implementation
- [x] All UI fields mapped to database columns
- [x] Existing entities reused where possible (Vehicle, Driver, Address)
- [x] Reference tables created for all ENUMs (vehicle_usage_type)
- [x] Naming conventions followed consistently
- [x] Relationships properly defined with foreign keys

### Post-Implementation
- [x] All foreign keys have proper constraints
- [x] Appropriate indexes for expected query patterns
- [x] Audit fields included on all tables
- [x] Status management consistent across tables
- [x] Entity catalog updated with new entities
- [x] Architectural decisions documented for cross-step integration

### Final Validation
- [x] Backend mappings complete and accurate
- [x] Database schema follows all standards
- [x] No redundant tables or columns created
- [x] Performance considerations addressed
- [x] Documentation updated with Step 1 and Step 2 integration patterns

### Global Requirements Compliance
- [x] **GR-52**: Universal Entity Management applied for DCS Vehicles and VIN decoder
- [x] **GR-53**: DCS Integration Architecture patterns followed consistently
- [x] **GR-04**: Validation & Data Handling implemented for vehicle specifications
- [x] **GR-18**: Workflow Requirements for owner addition integration
- [x] **GR-36**: Authentication & Permissions via Laravel Sanctum
- [x] **GR-33**: Data Services patterns for direct storage approach