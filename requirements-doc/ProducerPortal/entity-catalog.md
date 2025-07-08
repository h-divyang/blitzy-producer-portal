# Producer Portal Entity Catalog

## Overview
This catalog documents all entities discovered and defined across Producer Portal requirements. It serves as a reference to ensure consistency and reuse across different requirements.

**Current Status**: 60+ entities documented across 5 major categories
**Last Updated**: Enhanced with Universal Entity Management Architecture (GR-52)

---

## Core Business Entities

### quote
- **Purpose**: Main quote entity for insurance applications
- **Key Fields**: quote_number, status_id, program_id, effective_date, expiration_date, producer_id
- **Used By**: Quote workflow, producer dashboard
- **Relationships**: 
  - Has many drivers (map_quote_driver)
  - Has many vehicles (map_quote_vehicle)
  - Has many documents (map_quote_document)
  - Has many suspenses (map_quote_suspense)
  - Belongs to program
  - Belongs to producer
- **Requirements**: IP269-Quotes-Search, IP269-New-Quote-Step-1-Primary-Insured

### driver
- **Purpose**: Person who drives vehicles on the policy
- **Key Fields**: name_id, date_of_birth, driver_type_id, is_named_insured
- **Used By**: Quotes, Policies
- **Relationships**:
  - Belongs to name
  - Has many licenses (map_driver_license)
  - Has many phones (map_driver_phone)
  - Has many emails (map_driver_email)
  - Has many addresses (map_driver_address)
- **Requirements**: IP269-Quotes-Search

### vehicle
- **Purpose**: Insured vehicles
- **Key Fields**: vin, year, make, model, vehicle_use_type_id, is_primary_vehicle
- **Used By**: Quotes, Policies
- **Relationships**:
  - Has many registrations
  - Belongs to vehicle_use_type
- **Requirements**: IP269-Quotes-Search

### license
- **Purpose**: Driver's license information
- **Key Fields**: license_type_id, license_number, state_id, expiration_date
- **Used By**: Driver verification
- **Relationships**:
  - Belongs to license_type
  - Belongs to state
- **Requirements**: IP269-Quotes-Search

### document
- **Purpose**: File attachments for quotes/policies
- **Key Fields**: document_type_id, file_name, file_path, file_size
- **Used By**: Quote workflow, policy servicing
- **Relationships**:
  - Belongs to document_type
  - Associated via map tables
- **Requirements**: IP269-Quotes-Search

### suspense
- **Purpose**: Tasks/requirements that need resolution
- **Key Fields**: suspense_type_id, description, due_date, assigned_to
- **Used By**: Quote workflow, policy servicing, claims
- **Relationships**:
  - Belongs to suspense_type
  - Associated via map tables
- **Requirements**: IP269-Quotes-Search

### program
- **Purpose**: Insurance program configuration and availability
- **Key Fields**: code, name, effective_date, expiration_date, display_order
- **Used By**: Quote creation, program selection
- **Relationships**:
  - Has many quotes
- **Requirements**: IP269-New-Quote-Step-1-Primary-Insured

### address
- **Purpose**: Physical addresses for drivers and properties
- **Key Fields**: address_type_id, street_1, city, state_id, zip_code, is_verified
- **Used By**: Driver profiles, policy addresses
- **Relationships**:
  - Belongs to address_type
  - Belongs to state
  - Belongs to country
  - Associated via map tables
- **Requirements**: IP269-New-Quote-Step-1-Primary-Insured

---

## Universal Entity Management Entities

### entity_category
- **Purpose**: Categorizes types of entities for organizational purposes
- **Key Fields**: code, name, description, sort_order
- **Used By**: Entity type classification, UI organization
- **Relationships**: Has many entity_types
- **Requirements**: Universal Entity Management Architecture

### entity_type
- **Purpose**: Defines schemas and structures for entity categories
- **Key Fields**: code, name, category_id, metadata_schema
- **Used By**: Entity creation, validation, UI generation
- **Relationships**: Belongs to entity_category, has many entities
- **Requirements**: Universal Entity Management Architecture

### entity
- **Purpose**: Universal storage for all external entities (APIs, attorneys, body shops, vendors)
- **Key Fields**: entity_type_id, code, name, metadata
- **Used By**: All external entity management
- **Relationships**: Belongs to entity_type, polymorphic communication target/source
- **Requirements**: Universal Entity Management Architecture

### External Integration Entity Types

#### DCS Integration APIs (Data Capture Solutions)
Based on GR-52 Universal Entity Management Architecture and GR-53 DCS Integration Architecture:

##### DCS_HOUSEHOLD_DRIVERS
- **Purpose**: Driver verification and household lookup API
- **Provider**: Data Capture Solutions
- **API Version**: v2.7
- **Capabilities**: driver_verification, household_lookup, criminal_history, alias_search
- **Complete Metadata Schema**:
```json
{
  "type": "object",
  "properties": {
    "provider": {"type": "string", "const": "Data Capture Solutions"},
    "base_url": {"type": "string", "const": "https://ws.dcsinfosys.com:442"},
    "api_version": {"type": "string", "const": "v2.7"},
    "endpoint": {"type": "string", "const": "/apidevV2.7/DcsSearchApi/HouseholdDrivers"},
    "auth_type": {"type": "string", "const": "basic"},
    "capabilities": {
      "type": "array",
      "items": {"type": "string"},
      "const": ["driver_verification", "household_lookup", "criminal_history", "alias_search"]
    },
    "data_retention_days": {"type": "number", "const": 2555}
  },
  "required": ["provider", "base_url", "api_version", "endpoint", "auth_type"]
}
```

##### DCS_HOUSEHOLD_VEHICLES
- **Purpose**: Vehicle data and VIN decoding API
- **Provider**: Data Capture Solutions
- **API Version**: v2.3
- **Capabilities**: vehicle_lookup, vin_decoding, registration_data, household_vehicles
- **Complete Metadata Schema**:
```json
{
  "type": "object",
  "properties": {
    "provider": {"type": "string", "const": "Data Capture Solutions"},
    "base_url": {"type": "string", "const": "https://ws.dcsinfosys.com:442"},
    "api_version": {"type": "string", "const": "v2.3"},
    "endpoint": {"type": "string", "const": "/apidevV2.3/DcsSearchApi/HouseholdVehicles"},
    "auth_type": {"type": "string", "const": "basic"},
    "capabilities": {
      "type": "array",
      "items": {"type": "string"},
      "const": ["vehicle_lookup", "vin_decoding", "registration_data", "household_vehicles"]
    },
    "data_retention_days": {"type": "number", "const": 2555}
  },
  "required": ["provider", "base_url", "api_version", "endpoint", "auth_type"]
}
```

##### DCS_CRIMINAL
- **Purpose**: Criminal background verification API
- **Provider**: Data Capture Solutions
- **API Version**: v1.0
- **Capabilities**: criminal_history, background_check, offense_details, court_records
- **Complete Metadata Schema**:
```json
{
  "type": "object",
  "properties": {
    "provider": {"type": "string", "const": "Data Capture Solutions"},
    "base_url": {"type": "string", "const": "https://ws.dcsinfosys.com:442"},
    "api_version": {"type": "string", "const": "v1.0"},
    "endpoint": {"type": "string", "const": "/apidevV2.8/DcsSearchApi/Criminal"},
    "auth_type": {"type": "string", "const": "basic"},
    "capabilities": {
      "type": "array",
      "items": {"type": "string"},
      "const": ["criminal_history", "background_check", "offense_details", "court_records"]
    },
    "data_retention_days": {"type": "number", "const": 2555}
  },
  "required": ["provider", "base_url", "api_version", "endpoint", "auth_type"]
}
```

#### Communication Service Entity Types
Based on GR-44 Comprehensive Communication Architecture:

##### SENDGRID_EMAIL
- **Purpose**: Email delivery service integration
- **Provider**: SendGrid
- **Capabilities**: transactional_email, template_management, delivery_tracking, bounce_handling
- **Complete Metadata Schema**:
```json
{
  "type": "object",
  "properties": {
    "provider": {"type": "string", "const": "SendGrid"},
    "base_url": {"type": "string", "const": "https://api.sendgrid.com"},
    "api_version": {"type": "string", "const": "v3"},
    "auth_type": {"type": "string", "const": "bearer"},
    "capabilities": {
      "type": "array",
      "items": {"type": "string"},
      "const": ["transactional_email", "template_management", "delivery_tracking", "bounce_handling"]
    },
    "rate_limit_per_minute": {"type": "number", "const": 600}
  },
  "required": ["provider", "base_url", "api_version", "auth_type"]
}
```

##### TWILIO_SMS
- **Purpose**: SMS and voice communication service
- **Provider**: Twilio
- **Capabilities**: sms_messaging, voice_calls, phone_verification, delivery_receipts
- **Complete Metadata Schema**:
```json
{
  "type": "object",
  "properties": {
    "provider": {"type": "string", "const": "Twilio"},
    "base_url": {"type": "string", "const": "https://api.twilio.com"},
    "api_version": {"type": "string", "const": "2010-04-01"},
    "auth_type": {"type": "string", "const": "basic"},
    "capabilities": {
      "type": "array",
      "items": {"type": "string"},
      "const": ["sms_messaging", "voice_calls", "phone_verification", "delivery_receipts"]
    },
    "rate_limit_per_minute": {"type": "number", "const": 1000}
  },
  "required": ["provider", "base_url", "api_version", "auth_type"]
}
```

#### Future Entity Types
Based on Universal Entity Management patterns from GR-52:

##### ATTORNEY
- **Purpose**: Legal counsel partners
- **Complete Metadata Schema**:
```json
{
  "type": "object",
  "properties": {
    "firm_name": {"type": "string"},
    "bar_number": {"type": "string"},
    "specialties": {"type": "array", "items": {"type": "string"}},
    "contact_person": {"type": "string"},
    "hourly_rate": {"type": "number", "minimum": 0},
    "retainer_required": {"type": "boolean"}
  },
  "required": ["firm_name", "contact_person"]
}
```

##### BODY_SHOP
- **Purpose**: Vehicle repair facilities
- **Complete Metadata Schema**:
```json
{
  "type": "object",
  "properties": {
    "facility_type": {"type": "string"},
    "certifications": {"type": "array", "items": {"type": "string"}},
    "service_radius_miles": {"type": "integer", "minimum": 0},
    "specialties": {"type": "array", "items": {"type": "string"}},
    "preferred_insurers": {"type": "array", "items": {"type": "string"}},
    "capacity_vehicles_per_week": {"type": "integer", "minimum": 0}
  },
  "required": ["facility_type"]
}
```

##### VENDOR
- **Purpose**: General service providers
- **Complete Metadata Schema**:
```json
{
  "type": "object",
  "properties": {
    "service_type": {"type": "string"},
    "capabilities": {"type": "array", "items": {"type": "string"}},
    "service_area": {"type": "string"},
    "business_hours": {"type": "string"},
    "emergency_available": {"type": "boolean"},
    "insurance_requirements": {"type": "object"}
  },
  "required": ["service_type"]
}
```

### system_component
- **Purpose**: Associates backend functionality with frontend and security
- **Key Fields**: code, backend_namespace, api_prefix, frontend_route, permission_code
- **Used By**: Security groups, UI routing, API organization
- **Relationships**: Has many system_component_permissions
- **Requirements**: Universal Entity Management Architecture

### system_component_permission
- **Purpose**: Granular permissions for system components by security group
- **Key Fields**: component_id, security_group_id, can_read, can_write, can_delete, can_admin
- **Used By**: Access control, permission checking
- **Relationships**: Belongs to system_component and security_group
- **Requirements**: Universal Entity Management Architecture

### configuration_type
- **Purpose**: Defines what aspects of the system can be configured
- **Key Fields**: code, name, default_values, schema_definition
- **Used By**: Configuration management, validation
- **Relationships**: Has many configurations
- **Requirements**: Universal Entity Management Architecture

### configuration
- **Purpose**: Stores configuration values with simple scope hierarchy (system/program/entity)
- **Key Fields**: configuration_type_id, scope_type, scope_id, config_data
- **Used By**: Runtime configuration resolution
- **Relationships**: Belongs to configuration_type
- **Requirements**: Universal Entity Management Architecture

### communication_type
- **Purpose**: Classifies types of communications (API calls, emails, SMS, etc.)
- **Key Fields**: code, name, description
- **Used By**: Communication logging, routing
- **Relationships**: Used by communication table
- **Requirements**: Universal Entity Management Architecture

### communication_channel
- **Purpose**: Defines available communication channels and their properties
- **Key Fields**: code, name, is_real_time, default_timeout_seconds
- **Used By**: Communication routing, timeout management
- **Relationships**: Used by communication table
- **Requirements**: Universal Entity Management Architecture

### communication_status
- **Purpose**: Tracks status of communications (pending, processing, completed, failed)
- **Key Fields**: code, name, is_final_state, is_error_state
- **Used By**: Communication tracking, error handling
- **Relationships**: Used by communication table
- **Requirements**: Universal Entity Management Architecture

### communication
- **Purpose**: Universal logging for all external communications with polymorphic source/target
- **Key Fields**: source_type, source_id, target_type, target_id, correlation_id, request_data, response_data
- **Used By**: All external communications (API, email, phone, etc.)
- **Relationships**: Polymorphic source/target relationships to any entity
- **Requirements**: Universal Entity Management Architecture

---

## Infrastructure Entities
*Existing entities from blitzy-requirements codebase analysis*

### Core Insurance Entities

#### policy
- **Purpose**: Insurance policy contracts
- **Key Fields**: policy_prefix_id, number, policy_type_id, effective_date, expiration_date, status_id, cancellation_reason, reinstatement_date
- **Infrastructure Pattern**: Laravel Eloquent model with SoftDeletes
- **Relationships**: Has many transactions, belongs to policy_type and policy_prefix, has many reinstatement_calculations
- **Current Usage**: Insured Portal, payment processing, reinstatement workflow
- **Reinstatement Support**: Supports GR-64 reinstatement process with status tracking and premium calculations

#### user 
- **Purpose**: System users (policyholders, agents, staff)
- **Key Fields**: first_name, last_name, email, status_id
- **Infrastructure Pattern**: Laravel Authenticatable with Sanctum tokens
- **Relationships**: Has many policies via map tables, has user preferences
- **Current Usage**: Authentication, portal access

#### producer
- **Purpose**: Insurance producers/agents
- **Key Fields**: producer_code_id, producer_type_id, name, status_id  
- **Infrastructure Pattern**: Standard Eloquent model
- **Relationships**: Has many policies, belongs to producer_code and producer_type
- **Current Usage**: Producer assignment, commission tracking

#### payment
- **Purpose**: Payment transactions
- **Key Fields**: amount, payment_method_id, policy_id, transaction_date, status_id
- **Infrastructure Pattern**: Eloquent model with transaction tracking
- **Relationships**: Belongs to policy and payment_method
- **Current Usage**: Payment processing, history tracking

#### transaction
- **Purpose**: Financial transactions for policies
- **Key Fields**: policy_id, amount, transaction_type, effective_date, status_id
- **Infrastructure Pattern**: Eloquent model with audit fields
- **Relationships**: Belongs to policy via map table
- **Current Usage**: Premium calculations, financial reporting

#### reinstatement_calculation
- **Purpose**: Stores reinstatement premium calculations for audit and replay
- **Key Fields**: policy_id, cancellation_date, reinstatement_date, original_premium, lapse_days, adjusted_premium, total_due
- **Infrastructure Pattern**: Laravel Eloquent model with audit fields
- **Relationships**: Belongs to policy
- **Current Usage**: Reinstatement workflow, audit trails, premium adjustments
- **Requirements**: GR-64 Policy Reinstatement with Lapse Process

### Document Management Entities

#### document
- **Purpose**: File attachments and documents
- **Key Fields**: name, file_type_id, file_path, status_id
- **Infrastructure Pattern**: Eloquent model with file storage integration
- **Relationships**: Has many files, associated via map tables
- **Current Usage**: Document upload, policy documentation

#### file
- **Purpose**: Physical file storage references
- **Key Fields**: document_id, original_filename, file_path, file_size, file_type_id
- **Infrastructure Pattern**: File storage abstraction
- **Relationships**: Belongs to document and file_type
- **Current Usage**: File storage, download management

### Supporting Infrastructure Entities

#### verification
- **Purpose**: Identity and contact verification tracking
- **Key Fields**: verification_type, contact_value, status_id, verified_at
- **Infrastructure Pattern**: Polymorphic verification system
- **Relationships**: Can verify any entity type
- **Current Usage**: Email/phone verification

#### user_preference
- **Purpose**: User-specific settings and preferences
- **Key Fields**: user_id, preference_key, preference_value
- **Infrastructure Pattern**: Key-value preference storage
- **Relationships**: Belongs to user
- **Current Usage**: Portal customization, notification preferences

#### communication_preference
- **Purpose**: Communication method preferences
- **Key Fields**: code, name, description, is_default
- **Infrastructure Pattern**: Reference table
- **Relationships**: Used by user preferences
- **Current Usage**: Notification routing

### Infrastructure Map Tables

#### map_user_policy_driver
- **Purpose**: Associates users with policies and drivers
- **Key Fields**: user_id, policy_id, driver_id, status_id
- **Infrastructure Pattern**: Many-to-many with pivot data
- **Current Usage**: Policy access control

#### map_policy_transaction
- **Purpose**: Associates policies with transactions
- **Key Fields**: policy_id, transaction_id, status_id
- **Infrastructure Pattern**: Relationship tracking
- **Current Usage**: Financial transaction history

#### map_user_document  
- **Purpose**: Associates users with documents
- **Key Fields**: user_id, document_id, access_level, status_id
- **Infrastructure Pattern**: Document access control
- **Current Usage**: Document permissions

#### map_document_action
- **Purpose**: Tracks actions performed on documents
- **Key Fields**: document_id, action_id, user_id, performed_at
- **Infrastructure Pattern**: Audit trail for documents
- **Current Usage**: Document workflow tracking

### Infrastructure Reference Types

#### policy_type
- **Purpose**: Classification of policy types
- **Key Fields**: code, name, description, status_id
- **Infrastructure Pattern**: Standard reference table
- **Values**: AUTO, HOME, UMBRELLA, COMMERCIAL
- **Current Usage**: Policy categorization

#### payment_method_type
- **Purpose**: Payment method classifications
- **Key Fields**: code, name, description, status_id
- **Infrastructure Pattern**: Reference table
- **Values**: CREDIT_CARD, BANK_ACCOUNT, CHECK, CASH
- **Current Usage**: Payment processing

#### verification_status_type
- **Purpose**: Verification status classifications
- **Key Fields**: code, name, description, is_verified
- **Infrastructure Pattern**: Reference table with boolean flags
- **Values**: PENDING, VERIFIED, FAILED, EXPIRED
- **Current Usage**: Verification workflow

#### status
- **Purpose**: Universal status tracking for all entities
- **Key Fields**: code, name, description, is_active
- **Infrastructure Pattern**: Universal reference table
- **Values**: ACTIVE, INACTIVE, PENDING, DELETED
- **Current Usage**: Entity state management across entire system

---

## Communication Service Entities

### SENDGRID_EMAIL Entity Type
- **Purpose**: Email delivery service integration for insurance communications
- **Category**: API_INTEGRATION → COMMUNICATION_SERVICE  
- **Complete Metadata Schema**:
```json
{
  "type": "object",
  "properties": {
    "provider": {"type": "string", "const": "SendGrid"},
    "base_url": {"type": "string", "const": "https://api.sendgrid.com"},
    "api_version": {"type": "string", "const": "v3"},
    "auth_type": {"type": "string", "const": "bearer"},
    "capabilities": {
      "type": "array", 
      "items": {"type": "string"},
      "const": ["transactional_email", "marketing_campaigns", "template_engine", "analytics", "bounce_handling"]
    },
    "insurance_templates": {
      "type": "object",
      "properties": {
        "quote_confirmation": {"type": "string"},
        "policy_renewal": {"type": "string"},
        "payment_reminder": {"type": "string"},
        "claim_update": {"type": "string"},
        "document_request": {"type": "string"}
      }
    },
    "rate_limits": {
      "type": "object",
      "properties": {
        "requests_per_second": {"type": "number", "const": 600},
        "monthly_limit": {"type": "number", "const": 40000}
      }
    }
  },
  "required": ["provider", "base_url", "api_version", "auth_type"]
}
```
- **Requirements**: GR-44 Communication Architecture

### TWILIO_SMS Entity Type  
- **Purpose**: SMS and voice communication service for insurance notifications
- **Category**: API_INTEGRATION → COMMUNICATION_SERVICE
- **Complete Metadata Schema**:
```json
{
  "type": "object", 
  "properties": {
    "provider": {"type": "string", "const": "Twilio"},
    "base_url": {"type": "string", "const": "https://api.twilio.com"},
    "api_version": {"type": "string", "const": "2010-04-01"},
    "auth_type": {"type": "string", "const": "basic"},
    "capabilities": {
      "type": "array",
      "items": {"type": "string"}, 
      "const": ["sms_messaging", "voice_calls", "verification", "notifications", "two_factor_auth"]
    },
    "insurance_use_cases": {
      "type": "array",
      "items": {"type": "string"},
      "const": ["payment_reminders", "appointment_confirmations", "claim_updates", "policy_expiration", "emergency_notifications"]
    },
    "message_types": {
      "type": "object",
      "properties": {
        "transactional": {"type": "boolean", "const": true},
        "promotional": {"type": "boolean", "const": false},
        "verification": {"type": "boolean", "const": true}
      }
    }
  },
  "required": ["provider", "base_url", "api_version", "auth_type"]
}
```
- **Requirements**: GR-44 Communication Architecture

---

## Reference/Type Entities

### requote_type
- **Purpose**: Classification of requote reasons
- **Key Fields**: code, name, description
- **Values**: RENEWAL, REWRITE, REINSTATE, COMPETITOR
- **Requirements**: IP269-Quotes-Search

### driver_type
- **Purpose**: Driver classification (included, excluded, etc.)
- **Key Fields**: code, name, description
- **Values**: INCLUDED, EXCLUDED, LISTED_ONLY, OCCASIONAL
- **Requirements**: IP269-Quotes-Search

### license_type
- **Purpose**: Type of license (driver, producer, business)
- **Key Fields**: code, name, description, requires_state
- **Values**: US_DL, INTL_DL, NO_LICENSE, CDL
- **Requirements**: IP269-Quotes-Search, IP269-New-Quote-Step-1-Primary-Insured

### suspense_type
- **Purpose**: Category of suspense/task
- **Key Fields**: code, name, description
- **Values**: DOCUMENT, PHOTO, INFORMATION, APPROVAL, PAYMENT
- **Requirements**: IP269-Quotes-Search

### document_type
- **Purpose**: Classification of documents
- **Key Fields**: code, name, description
- **Values**: DRIVERS_LICENSE, VEHICLE_PHOTO, PROOF_OF_PRIOR, etc.
- **Requirements**: IP269-Quotes-Search

### address_type
- **Purpose**: Classification of address types
- **Key Fields**: code, name, description
- **Values**: HOME, MAILING, BUSINESS, GARAGING
- **Requirements**: IP269-New-Quote-Step-1-Primary-Insured

### notification_preference
- **Purpose**: How users prefer to be contacted
- **Key Fields**: code, name, description
- **Values**: EMAIL, SMS, PHONE, PORTAL
- **Requirements**: IP269-Quotes-Search

### marital_status
- **Purpose**: Driver marital status
- **Key Fields**: code, name, description
- **Values**: SINGLE, MARRIED, DIVORCED, WIDOWED
- **Requirements**: IP269-Quotes-Search

### gender
- **Purpose**: Driver gender
- **Key Fields**: code, name, description
- **Values**: M, F, X
- **Requirements**: IP269-Quotes-Search

### occupation
- **Purpose**: Driver occupation
- **Key Fields**: code, name, description
- **Values**: Various occupation types
- **Requirements**: IP269-Quotes-Search

### vehicle_use_type
- **Purpose**: How vehicle is used
- **Key Fields**: code, name, description
- **Values**: PLEASURE, COMMUTE, BUSINESS, FARM
- **Requirements**: IP269-Quotes-Search

### relationship_to_insured
- **Purpose**: Family/other relationships to named insured
- **Key Fields**: code, name, description
- **Values**: SELF, SPOUSE, CHILD, PARENT, OTHER
- **Requirements**: IP269-Quotes-Search

### phone_type
- **Purpose**: Type of phone number
- **Key Fields**: code, name, description
- **Values**: MOBILE, HOME, WORK, OTHER
- **Requirements**: IP269-Quotes-Search

### email_type
- **Purpose**: Type of email address
- **Key Fields**: code, name, description
- **Values**: PERSONAL, WORK, OTHER
- **Requirements**: IP269-Quotes-Search

### address_type
- **Purpose**: Type of address
- **Key Fields**: code, name, description
- **Values**: MAILING, GARAGING, PREVIOUS, BUSINESS
- **Requirements**: IP269-Quotes-Search

---

## Supporting Entities

### phone
- **Purpose**: Phone number storage
- **Key Fields**: country_code, number, phone_type_id, can_sms, is_verified
- **Used By**: Driver contact info
- **Relationships**: Belongs to phone_type
- **Requirements**: IP269-Quotes-Search

### email
- **Purpose**: Email address storage
- **Key Fields**: address, email_type_id, is_verified, is_valid
- **Used By**: Driver contact info
- **Relationships**: Belongs to email_type
- **Requirements**: IP269-Quotes-Search

### address
- **Purpose**: Physical address storage
- **Key Fields**: line1, line2, city, state, zip, address_type_id, is_primary
- **Used By**: Driver addresses, vehicle garaging
- **Relationships**: Belongs to address_type
- **Requirements**: IP269-Quotes-Search

### name
- **Purpose**: Person name storage
- **Key Fields**: prefix, first_name, middle_name, last_name, suffix, full_name (computed)
- **Used By**: Drivers, users
- **Relationships**: None (base entity)
- **Requirements**: IP269-Quotes-Search

### vehicle_registration
- **Purpose**: Vehicle registration/plate information
- **Key Fields**: vehicle_id, plate_number, state_id, registration_date, expiration_date
- **Used By**: Vehicle tracking
- **Relationships**: Belongs to vehicle and state
- **Requirements**: IP269-Quotes-Search

---

## Relationship Entities (map_*)

### map_producer_quote
- **Purpose**: Associates producers with quotes
- **Key Fields**: producer_id, quote_id, assignment_date
- **Requirements**: IP269-Quotes-Search

### map_quote_driver
- **Purpose**: Associates drivers with quotes
- **Key Fields**: quote_id, driver_id
- **Requirements**: IP269-Quotes-Search

### map_quote_vehicle
- **Purpose**: Associates vehicles with quotes
- **Key Fields**: quote_id, vehicle_id
- **Requirements**: IP269-Quotes-Search

### map_driver_license
- **Purpose**: Associates drivers with licenses
- **Key Fields**: driver_id, license_id
- **Requirements**: IP269-Quotes-Search

### map_quote_document
- **Purpose**: Associates documents with quotes
- **Key Fields**: quote_id, document_id
- **Requirements**: IP269-Quotes-Search

### map_driver_phone
- **Purpose**: Associates drivers with phone numbers
- **Key Fields**: driver_id, phone_id, is_primary
- **Requirements**: IP269-Quotes-Search

### map_driver_email
- **Purpose**: Associates drivers with email addresses
- **Key Fields**: driver_id, email_id, is_primary
- **Requirements**: IP269-Quotes-Search

### map_driver_address
- **Purpose**: Associates drivers with addresses
- **Key Fields**: driver_id, address_id, is_primary
- **Requirements**: IP269-Quotes-Search

### map_quote_coverage
- **Purpose**: Associates coverages with quotes
- **Key Fields**: quote_id, coverage_id, limit_id, deductible_id
- **Requirements**: IP269-Quotes-Search

### map_quote_suspense
- **Purpose**: Associates suspenses with quotes
- **Key Fields**: quote_id, suspense_id
- **Requirements**: IP269-Quotes-Search

### map_policy_suspense
- **Purpose**: Associates suspenses with policies
- **Key Fields**: policy_id, suspense_id
- **Requirements**: IP269-Quotes-Search

---

## Entity Reuse Guidelines

### Universal Entity Management First (Based on GR-52)
1. **Check if entity is external** - Use universal entity pattern (entity/entity_type/entity_category)
2. **Verify global requirements alignment** - Check GR-52, GR-48, GR-44 for existing patterns
3. **Check this catalog** for existing entities before creating new ones
4. **Review similar requirements** for established patterns
5. **Consider extending existing** rather than creating new entities
6. **Validate with CLAUDE.md standards** and global requirement references

### Universal vs. Specific Entity Patterns (GR-52 Standards)
- **External Entities** (APIs, attorneys, body shops, vendors): ALWAYS use universal entity pattern
  - Provides 90% faster development for new external entity types
  - Zero code changes required to add new entity types
  - Consistent CRUD operations across all entities
  - Automatic UI support through JSON metadata schemas
- **Internal Business Entities** (quotes, drivers, vehicles): Use specific tables for performance
- **Supporting Entities** (phone, email, address): Use specific reusable tables
- **Reference Tables**: Use _type suffix pattern for all classifications
- **Communication Tracking**: Universal communication table for ALL external communications

### Recommended Entity Categories (Based on GR-52)
- **INTEGRATION**: Third-party API integrations (DCS, SendGrid, Twilio)
- **PARTNER**: Business partners (attorneys, body shops, vendors)
- **VENDOR**: Service vendors and suppliers
- **SYSTEM**: Internal system entities and configurations

### Global Requirements Integration Patterns
- **DCS APIs** (GR-53): Use DCS_HOUSEHOLD_DRIVERS, DCS_HOUSEHOLD_VEHICLES, DCS_CRIMINAL entity types
- **Communication Services** (GR-44): Use SENDGRID_EMAIL, TWILIO_SMS entity types
- **External Integrations** (GR-48): Follow Apache Camel integration patterns
- **Database Standards** (GR-41): Follow table schema requirements and naming conventions
- **Security Components** (GR-52): Use system_component for permission-based access control
- **Policy Reinstatement** (GR-64): Use reinstatement_calculation for premium calculations and audit trails

### Common Reusable Patterns
- **External Entities**: entity/entity_type/entity_category with JSON metadata schemas
- **Contact Info**: phone, email, address (separate reusable entities)
- **Person Info**: name (separate entity for consistency)
- **Status Tracking**: status_id in all tables following GR-19 patterns
- **Type Classification**: Reference tables for all enums following GR-41
- **Relationships**: map_ tables for associations following GR-19
- **Communications**: Universal communication table with polymorphic source/target (GR-44)
- **Configuration**: Three-level hierarchy (system/program/entity) following GR-52

### Entity Naming Patterns (GR-41 Compliance)
- **Universal entities**: entity, entity_type, entity_category, communication, configuration
- **Core entities**: Business domain names (quote, driver, vehicle) - singular nouns
- **Reference entities**: _type suffix (driver_type, phone_type, address_type)
- **Relationship entities**: map_ prefix (map_quote_driver, map_entity_action)
- **Supporting entities**: Generic names (phone, email, address, name)
- **Communication entities**: communication, communication_type, communication_channel, communication_status

### Performance and Compliance Standards
- **Entity queries**: < 500ms for 10,000+ entities (GR-52)
- **Communication queries**: < 200ms with correlation ID indexing (GR-52)
- **Configuration resolution**: < 100ms across scope hierarchy (GR-52)
- **Metadata validation**: < 50ms per entity type (GR-52)
- **Data retention**: 7 years for insurance regulatory compliance (GR-52)

---

## Update Process

When working on new requirements:

1. **Review Global Requirements first** - Check GR-52, GR-48, GR-44, GR-41, GR-19 for existing patterns
2. **Review this catalog** before defining any new entities
3. **Prioritize universal entity patterns** for all external entities (APIs, partners, vendors)
4. **Update catalog** with any new entities discovered during implementation
5. **Note requirements** that use each entity for impact analysis
6. **Maintain relationships** section for each entity
7. **Update patterns** if new ones emerge from Global Requirements analysis
8. **Validate compliance** with performance and naming standards

### Global Requirements Reference Checklist
Before creating new entities, verify alignment with:
- **GR-52**: Universal Entity Management - for all external entities
- **GR-48**: External Integrations Catalog - for API integrations
- **GR-44**: Communication Architecture - for communication entities
- **GR-41**: Table Schema Requirements - for naming and structure
- **GR-19**: Table Relationships - for mapping and audit patterns
- **GR-53**: DCS Integration Architecture - for specific DCS APIs

This catalog should be updated with every requirement processed to maintain accuracy, usefulness, and Global Requirements compliance.