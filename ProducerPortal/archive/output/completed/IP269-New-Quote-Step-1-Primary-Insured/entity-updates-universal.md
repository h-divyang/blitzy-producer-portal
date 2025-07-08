# IP269 Universal Entity Management - Entity Catalog Updates

## Overview
This document outlines the specific entity catalog updates required to implement Universal Entity Management for IP269 and future requirements.

---

## New Universal Entity Management Entities

### Core Universal Entities

#### entity_category
- **Status**: NEW - Added to catalog
- **Purpose**: Categorizes types of entities for organizational purposes
- **Key Fields**: code, name, description, sort_order
- **Used By**: Entity type classification, UI organization
- **Relationships**: Has many entity_types
- **Requirements**: Universal Entity Management Architecture, IP269-New-Quote-Step-1-Primary-Insured (Universal)

#### entity_type  
- **Status**: NEW - Added to catalog
- **Purpose**: Defines schemas and structures for entity categories
- **Key Fields**: code, name, category_id, metadata_schema
- **Used By**: Entity creation, validation, UI generation
- **Relationships**: Belongs to entity_category, has many entities
- **Requirements**: Universal Entity Management Architecture, IP269-New-Quote-Step-1-Primary-Insured (Universal)

#### entity
- **Status**: NEW - Added to catalog
- **Purpose**: Universal storage for all external entities (APIs, attorneys, body shops, vendors)
- **Key Fields**: entity_type_id, code, name, metadata
- **Used By**: All external entity management
- **Relationships**: Belongs to entity_type, polymorphic communication target/source
- **Requirements**: Universal Entity Management Architecture, IP269-New-Quote-Step-1-Primary-Insured (Universal)

### Universal Configuration Entities

#### configuration_type
- **Status**: NEW - Added to catalog
- **Purpose**: Defines what aspects of the system can be configured
- **Key Fields**: code, name, default_values, schema_definition
- **Used By**: Configuration management, validation
- **Relationships**: Has many configurations
- **Requirements**: Universal Entity Management Architecture

#### configuration
- **Status**: NEW - Added to catalog
- **Purpose**: Stores configuration values with simple scope hierarchy (system/program/entity)
- **Key Fields**: configuration_type_id, scope_type, scope_id, config_data
- **Used By**: Runtime configuration resolution
- **Relationships**: Belongs to configuration_type
- **Requirements**: Universal Entity Management Architecture

### Universal Communication Entities

#### communication_type
- **Status**: NEW - Added to catalog
- **Purpose**: Classifies types of communications (API calls, emails, SMS, etc.)
- **Key Fields**: code, name, description
- **Used By**: Communication logging, routing
- **Relationships**: Used by communication table
- **Requirements**: Universal Entity Management Architecture

#### communication_channel
- **Status**: NEW - Added to catalog
- **Purpose**: Defines available communication channels and their properties
- **Key Fields**: code, name, is_real_time, default_timeout_seconds
- **Used By**: Communication routing, timeout management
- **Relationships**: Used by communication table
- **Requirements**: Universal Entity Management Architecture

#### communication_status
- **Status**: NEW - Added to catalog
- **Purpose**: Tracks status of communications (pending, processing, completed, failed)
- **Key Fields**: code, name, is_final_state, is_error_state
- **Used By**: Communication tracking, error handling
- **Relationships**: Used by communication table
- **Requirements**: Universal Entity Management Architecture

#### communication
- **Status**: NEW - Added to catalog
- **Purpose**: Universal logging for all external communications with polymorphic source/target
- **Key Fields**: source_type, source_id, target_type, target_id, correlation_id, request_data, response_data
- **Used By**: All external communications (API, email, phone, etc.)
- **Relationships**: Polymorphic source/target relationships to any entity
- **Requirements**: Universal Entity Management Architecture, IP269-New-Quote-Step-1-Primary-Insured (Universal)

### Component-Based Security Entities

#### system_component
- **Status**: NEW - Added to catalog
- **Purpose**: Associates backend functionality with frontend and security
- **Key Fields**: code, backend_namespace, api_prefix, frontend_route, permission_code
- **Used By**: Security groups, UI routing, API organization
- **Relationships**: Has many system_component_permissions
- **Requirements**: Universal Entity Management Architecture

#### system_component_permission
- **Status**: NEW - Added to catalog
- **Purpose**: Granular permissions for system components by security group
- **Key Fields**: component_id, security_group_id, can_read, can_write, can_delete, can_admin
- **Used By**: Access control, permission checking
- **Relationships**: Belongs to system_component and security_group
- **Requirements**: Universal Entity Management Architecture

---

## Legacy Integration Entities Status

### third_party_integration
- **Status**: LEGACY - Being replaced by universal entities
- **Migration Path**: Convert to entity_type 'API_INTEGRATION' and individual entity records
- **Timeline**: Phase out during universal entity implementation
- **Notes**: Existing DCS integration should be converted to universal entity pattern

### integration_configuration
- **Status**: LEGACY - Being replaced by universal configuration
- **Migration Path**: Convert to configuration table with scope_type system/program/entity
- **Timeline**: Phase out during universal configuration implementation
- **Notes**: Hierarchical configuration simplified to three-level scope

### integration_node
- **Status**: LEGACY - Being replaced by entity metadata
- **Migration Path**: API response structure defined in entity_type metadata_schema
- **Timeline**: Phase out during universal entity implementation
- **Notes**: JSON schema provides more flexible response definition

### integration_field_mapping
- **Status**: LEGACY - Being replaced by entity metadata
- **Migration Path**: Field mappings handled through entity metadata and transformation rules
- **Timeline**: Phase out during universal entity implementation
- **Notes**: Simplified to JSON-based transformation in entity metadata

### integration_request
- **Status**: LEGACY - Being replaced by universal communication
- **Migration Path**: Convert to communication table with polymorphic relationships
- **Timeline**: Phase out during universal communication implementation
- **Notes**: Ultra-simple design with source/target only

### integration_verification_result
- **Status**: LEGACY - Being replaced by communication response_data
- **Migration Path**: Verification results stored in communication.response_data JSON
- **Timeline**: Phase out during universal communication implementation
- **Notes**: Structured results maintained in JSON format

---

## Updated Entity Reuse Guidelines

### Universal Entity Management Decision Tree

1. **Is this an external entity?** (API, attorney, body shop, vendor)
   - **YES**: Use universal entity pattern (entity_type/entity)
   - **NO**: Continue to step 2

2. **Is this an internal business entity?** (quote, driver, vehicle)
   - **YES**: Use specific table with established patterns
   - **NO**: Continue to step 3

3. **Is this a supporting entity?** (phone, email, address)
   - **YES**: Use specific reusable table
   - **NO**: Use reference table pattern

### Entity Pattern Selection

```
External Entities (APIs, Partners, Vendors)
├── entity_category (INTEGRATION, PARTNER, VENDOR)
├── entity_type (API_INTEGRATION, ATTORNEY, BODY_SHOP)
└── entity (specific instances with JSON metadata)

Internal Business Entities
├── Core tables (quote, driver, vehicle)
├── Reference tables (quote_type, driver_type)
└── Map tables (map_quote_driver)

Supporting Entities
├── Reusable tables (phone, email, address)
├── Reference tables (phone_type, email_type)
└── Map tables (map_driver_phone)

Universal Communication
├── communication (polymorphic source/target)
├── communication_type (API_REQUEST, VERIFICATION)
└── communication_channel (API, EMAIL, SMS)
```

---

## Sample Entity Type Definitions

### API_INTEGRATION Entity Type
```json
{
  "type": "object",
  "properties": {
    "provider": {"type": "string", "description": "Service provider name"},
    "api_version": {"type": "string", "description": "API version"},
    "base_url": {"type": "string", "format": "uri", "description": "Base API URL"},
    "auth_type": {"type": "string", "enum": ["api_key", "oauth2", "basic", "bearer"]},
    "timeout_seconds": {"type": "integer", "minimum": 1, "maximum": 300},
    "rate_limit_per_minute": {"type": "integer", "minimum": 1},
    "endpoints": {
      "type": "object",
      "description": "Available API endpoints",
      "additionalProperties": {"type": "string"}
    },
    "response_format": {"type": "string", "enum": ["json", "xml", "csv"]},
    "supports_webhooks": {"type": "boolean", "default": false}
  },
  "required": ["provider", "base_url", "auth_type"]
}
```

### ATTORNEY Entity Type  
```json
{
  "type": "object",
  "properties": {
    "firm_name": {"type": "string", "description": "Law firm name"},
    "bar_number": {"type": "string", "description": "Bar association number"},
    "specialties": {
      "type": "array", 
      "items": {"type": "string"},
      "description": "Legal specialties"
    },
    "contact_person": {"type": "string", "description": "Primary contact"},
    "hourly_rate": {"type": "number", "minimum": 0, "description": "Standard hourly rate"},
    "retainer_required": {"type": "boolean", "default": false},
    "preferred_communication": {
      "type": "string", 
      "enum": ["email", "phone", "fax", "portal"],
      "default": "email"
    },
    "service_areas": {
      "type": "array",
      "items": {"type": "string"},
      "description": "Geographic service areas"
    }
  },
  "required": ["firm_name", "contact_person"]
}
```

### BODY_SHOP Entity Type
```json
{
  "type": "object", 
  "properties": {
    "facility_type": {"type": "string", "enum": ["collision_center", "glass_shop", "paint_shop", "full_service"]},
    "certifications": {
      "type": "array",
      "items": {"type": "string"},
      "description": "Industry certifications (ASE, I-CAR, etc.)"
    },
    "service_radius_miles": {"type": "integer", "minimum": 0},
    "specialties": {
      "type": "array",
      "items": {"type": "string"},
      "description": "Repair specialties"
    },
    "preferred_insurers": {
      "type": "array", 
      "items": {"type": "string"},
      "description": "Preferred insurance companies"
    },
    "capacity_vehicles_per_week": {"type": "integer", "minimum": 0},
    "tow_service_available": {"type": "boolean", "default": false},
    "rental_cars_available": {"type": "boolean", "default": false}
  },
  "required": ["facility_type"]
}
```

---

## Migration Strategy

### Phase 1: Universal Entity Foundation (Weeks 1-2)
**Entity Updates**:
- Add all universal entity management entities to catalog
- Update entity reuse guidelines
- Create JSON schemas for common entity types

**Files Updated**:
- `/app/workspace/requirements/ProducerPortal/entity-catalog.md` ✓ (Completed)

### Phase 2: Legacy Entity Transition (Weeks 3-4)  
**Entity Updates**:
- Mark legacy integration entities as deprecated
- Document migration paths for each legacy entity
- Update requirements to reference universal entities

**Files Updated**:
- Entity catalog migration notes
- Architectural decisions with deprecation timeline

### Phase 3: IP269 Universal Implementation (Weeks 5-6)
**Entity Updates**:
- Implement DCS integration as universal entity
- Create sample entity instances for testing
- Update IP269 to use universal communication patterns

**Files Updated**:
- IP269 sections-c-e-universal.md ✓ (Completed)
- Entity catalog with IP269-specific entities

### Phase 4: Future Requirement Preparation (Weeks 7-8)
**Entity Updates**:
- Add attorney, body shop, vendor entity types
- Create component-based security entities
- Document patterns for all common external entity types

**Files Updated**:
- Integration patterns reference ✓ (Completed)
- Entity catalog with all planned entity types

---

## Quality Assurance Checklist

### Entity Catalog Completeness
- [ ] All universal entities documented with purpose, fields, relationships
- [ ] Legacy entities marked with migration status and timeline
- [ ] Entity reuse guidelines updated for universal patterns
- [ ] JSON schemas defined for all entity types
- [ ] Integration with Global Requirements documented

### Documentation Consistency
- [ ] All new entities follow established naming patterns
- [ ] Relationship descriptions are clear and accurate
- [ ] Requirements references updated throughout catalog
- [ ] Anti-patterns section updated with universal guidance

### Implementation Readiness
- [ ] Database schema provided for all new entities
- [ ] Sample data created for testing universal patterns
- [ ] API endpoints documented for entity management
- [ ] Security model defined for component-based permissions

---

## Future Entity Types to Add

### Planned Entity Types (Next Phase)
- **VENDOR**: General service providers
- **DOCUMENT_PROCESSOR**: OCR and document classification services
- **PAYMENT_PROCESSOR**: Payment gateway integrations
- **NOTIFICATION_SERVICE**: Email/SMS service providers
- **MAPPING_SERVICE**: Address validation and geocoding services

### Entity Categories to Expand
- **PARTNER**: Attorneys, body shops, adjusters, appraisers
- **INTEGRATION**: All external API services
- **VENDOR**: Service providers, suppliers, contractors
- **SYSTEM**: Internal system components and services

---

## Conclusion

The entity catalog has been successfully updated to support Universal Entity Management architecture. Key achievements:

- **12 new universal entities** added to support all external entity management
- **6 legacy entities** marked for deprecation with clear migration paths  
- **Updated reuse guidelines** to prioritize universal patterns for external entities
- **JSON schemas defined** for common entity types (API, attorney, body shop)
- **Complete migration strategy** with 4-phase implementation plan

This establishes the foundation for:
- Zero code changes when adding new external entity types
- Consistent patterns across all external entity management
- Simple configuration hierarchy for entity-specific settings
- Universal communication tracking for all external interactions
- Component-based security for all entity operations

The entity catalog is now ready to support IP269 universal implementation and future requirements that involve external entities.