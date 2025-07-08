# Universal Entity Management Integration Patterns Reference

## Overview
This document provides comprehensive patterns and standards for implementing Universal Entity Management across all external entities in the Producer Portal. It serves as a reference for consistent implementation of APIs, attorneys, body shops, vendors, and other external entities.

---

## Universal Entity Types

### Standard Entity Categories
```sql
-- Core entity categories for external entities
INSERT INTO entity_category (code, name, description, sort_order, status_id, created_by) VALUES
('INTEGRATION', 'API Integration', 'Third-party API integrations', 1, 1, 1),
('PARTNER', 'Business Partner', 'Business partners (attorneys, body shops)', 2, 1, 1),
('VENDOR', 'Vendor', 'Service vendors', 3, 1, 1),
('SYSTEM', 'System', 'Internal system entities', 4, 1, 1);
```

### Common Entity Types with JSON Schemas

#### API_INTEGRATION
```json
{
  "type": "object",
  "properties": {
    "provider": {"type": "string"},
    "api_version": {"type": "string"},
    "base_url": {"type": "string", "format": "uri"},
    "auth_type": {"type": "string", "enum": ["api_key", "oauth2", "basic", "bearer"]},
    "timeout_seconds": {"type": "integer", "minimum": 1, "maximum": 300},
    "rate_limit_per_minute": {"type": "integer", "minimum": 1}
  },
  "required": ["provider", "base_url", "auth_type"]
}
```

#### ATTORNEY
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

#### BODY_SHOP
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

#### VENDOR
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

---

## API Endpoint Patterns

### Universal Entity Management APIs

#### Entity Type Operations
```
GET    /api/v1/entity-types              # List all entity types
POST   /api/v1/entity-types              # Create new entity type
GET    /api/v1/entity-types/{id}         # Get entity type details
PUT    /api/v1/entity-types/{id}         # Update entity type
DELETE /api/v1/entity-types/{id}         # Deactivate entity type

# Category filtering
GET    /api/v1/entity-types?category=INTEGRATION
GET    /api/v1/entity-types?category=PARTNER
```

#### Entity Operations
```
GET    /api/v1/entities                  # List entities with filtering
POST   /api/v1/entities                  # Create new entity
GET    /api/v1/entities/{id}             # Get entity details
PUT    /api/v1/entities/{id}             # Update entity
DELETE /api/v1/entities/{id}             # Deactivate entity

# Type filtering
GET    /api/v1/entities?type=API_INTEGRATION
GET    /api/v1/entities?type=ATTORNEY
GET    /api/v1/entities?type=BODY_SHOP

# Search by metadata
GET    /api/v1/entities?search=provider:DCS
GET    /api/v1/entities?search=specialties:collision_repair
```

#### Configuration Management
```
GET    /api/v1/configuration/{type}      # Get resolved configuration
POST   /api/v1/configuration/{type}      # Update configuration
GET    /api/v1/configuration-types       # List configuration types

# Scope-specific operations
GET    /api/v1/configuration/{type}/system
GET    /api/v1/configuration/{type}/program/{id}
GET    /api/v1/configuration/{type}/entity/{id}
```

#### Communication Management
```
GET    /api/v1/communications            # List communications
POST   /api/v1/communications            # Create communication record
GET    /api/v1/communications/{id}       # Get communication details

# Entity-specific communications
GET    /api/v1/communications?target_type=entity&target_id={id}
GET    /api/v1/communications?source_type=entity&source_id={id}

# Correlation tracking
GET    /api/v1/communications?correlation_id={id}
```

---

## Integration with Global Requirements

### Reference to Global Requirement 48 (External Integrations)
Universal Entity Management integrates with established Apache Camel integration platform:
- All entity communications route through Camel for transformation and routing
- Error handling and retry mechanisms leveraged from existing patterns
- Monitoring and health checks utilize established infrastructure

### Reference to Global Requirement 44 (Communication Architecture)
Universal communication table complements existing communication infrastructure:
- SendGrid email integrations use entity communication logging
- Twilio SMS integrations tracked through universal communication
- HashiCorp Vault credentials referenced in entity metadata

### Alignment with Global Standards
- **Security**: Follows Global Requirement 36 authentication patterns
- **Database**: Aligns with Global Requirement 33 caching strategies
- **API**: Consistent with Global Requirement 47 API gateway patterns
- **Monitoring**: Integrates with Global Requirement 25 observability standards

---

## Implementation Timeline

### Phase 1: Core Universal Tables (Weeks 1-2)
1. Create entity_category, entity_type, entity tables
2. Implement JSON schema validation
3. Basic CRUD operations with metadata handling

### Phase 2: Communication System (Weeks 3-4)
1. Create communication reference tables
2. Implement communication table with polymorphic relationships
3. Communication service layer with correlation tracking

### Phase 3: Configuration System (Weeks 5-6)
1. Create configuration_type and configuration tables
2. Implement scope-based configuration resolution
3. UI for configuration management

### Phase 4: Component Security (Weeks 7-8)
1. Create system_component and permission tables
2. Implement component-based security middleware
3. UI access control integration

### Phase 5: Integration and Testing (Weeks 9-10)
1. Integration with existing global requirements
2. Performance optimization and testing
3. Documentation completion

---

## Success Metrics

### Performance Targets
- Entity listing queries: <500ms for 10,000+ entities
- Communication queries: <200ms for large datasets
- Configuration resolution: <100ms for complex hierarchies
- Metadata validation: <50ms per entity

### Development Efficiency
- New entity type creation: <1 hour end-to-end
- UI automatically supports new entity types
- Zero code changes for adding external entity types
- Consistent patterns across all entity management

### Maintainability Goals
- Single source of truth for all external entities
- Clear separation of concerns through component system
- Simple configuration hierarchy with predictable resolution
- Comprehensive audit trail for all entity interactions

This reference document ensures consistent implementation of Universal Entity Management patterns across all Producer Portal requirements and provides clear integration points with existing global infrastructure.