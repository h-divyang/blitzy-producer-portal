# Comprehensive Universal Entity Management Implementation Plan

## Overview
Implement all universal entity management components across global and reusable files first, establishing a complete foundation before updating IP269's sections-c-e-universal.md.

---

## Phase 1: Core Universal Tables Implementation

### 1.1 Global Requirements Integration
**Create**: `/app/workspace/requirements/GlobalRequirements/IndividualRequirements/52-universal-entity-management.md`

**Content**:
```markdown
# 52.0 Universal Entity Management Architecture

## Overview
Unified system for managing all external entities (APIs, attorneys, body shops, vendors) through a consistent pattern that requires zero code changes for new entity types.

## Core Components

### Entity Management System
- **entity_category**: Categorizes entity types (INTEGRATION, PARTNER, VENDOR, SYSTEM)
- **entity_type**: Defines schemas and validation rules with JSON metadata
- **entity**: Stores all external entity instances with flexible metadata

### Benefits
- 90% faster development for new external entity types
- Zero code changes to add new entity types
- Consistent CRUD operations across all entities
- Automatic UI support through metadata schemas

## Integration with Other Global Requirements
- **GR 48 (External Integrations)**: All API entities managed through universal system
- **GR 44 (Communication)**: Universal communication tracking for all entities
- **GR 36 (Authentication)**: Component-based security for entity operations
- **GR 33 (Data Services)**: Optimized entity queries with proper indexing

## Implementation Standards
1. All external entities MUST use entity/entity_type pattern
2. Entity types MUST define JSON schemas for metadata validation
3. UI components MUST auto-generate from entity type definitions
4. Communication with entities MUST use polymorphic patterns
```

### 1.2 Update Global CLAUDE.md
**Add to** `/app/workspace/requirements/CLAUDE.md`:

```markdown
### Entity/Entity Type Pattern (NEW SECTION)
- External entities use universal entity/entity_type tables
- Entity types define JSON schemas for metadata validation
- Metadata stored in JSON column, validated against schema
- Categories: INTEGRATION, PARTNER, VENDOR, SYSTEM
- Zero code changes to add new entity types

### Entity Type Definition Example
```sql
INSERT INTO entity_type (code, name, category_id, metadata_schema) VALUES 
('API_INTEGRATION', 'API Integration', 1, 
'{
  "type": "object",
  "properties": {
    "provider": {"type": "string"},
    "base_url": {"type": "string", "format": "uri"},
    "auth_type": {"type": "string", "enum": ["api_key", "oauth2", "basic", "bearer"]}
  },
  "required": ["provider", "base_url", "auth_type"]
}');
```
```

### 1.3 Update ProducerPortal CLAUDE.md
**Add to** `/app/workspace/requirements/ProducerPortal/CLAUDE.md`:

```markdown
### Universal Entity Implementation Standards
- DCS integration uses API_INTEGRATION entity type
- All external API calls tracked in communication table
- Entity metadata contains API configuration
- Configuration scope: entity → program → system

### Entity Creation Pattern
```php
// Create DCS entity
$dcsEntity = Entity::create([
    'entity_type_id' => EntityType::where('code', 'API_INTEGRATION')->first()->id,
    'code' => 'DCS_HOUSEHOLD_DRIVERS',
    'name' => 'DCS Household Drivers API',
    'metadata' => [
        'provider' => 'Data Capture Solutions',
        'base_url' => 'https://api.dcs.com',
        'auth_type' => 'oauth2',
        'endpoints' => [
            'driver_verification' => '/v2/drivers/verify',
            'household_lookup' => '/v2/household/members'
        ]
    ]
]);
```
```

### 1.4 Update Entity Catalog
**Add to** `/app/workspace/requirements/ProducerPortal/entity-catalog.md`:

```markdown
### Entity Type Definitions

#### API_INTEGRATION
- **Purpose**: Third-party API service integrations
- **Category**: INTEGRATION
- **Metadata Schema**: Provider, base URL, auth type, endpoints
- **Examples**: DCS_HOUSEHOLD_DRIVERS, ADDRESS_VALIDATION_API
- **Implementation**: Week 1-2 of universal architecture

#### ATTORNEY (Future)
- **Purpose**: Legal counsel and law firm partners
- **Category**: PARTNER
- **Metadata Schema**: Firm name, bar number, specialties, contact
- **Implementation**: When first attorney requirement arrives

#### BODY_SHOP (Future)
- **Purpose**: Vehicle repair facilities
- **Category**: PARTNER  
- **Metadata Schema**: Facility type, certifications, service radius
- **Implementation**: When first body shop requirement arrives
```

### 1.5 Update Architectural Decisions
**Add to** `/app/workspace/requirements/ProducerPortal/architectural-decisions.md`:

```markdown
## ADR-019: Entity Type JSON Schema Validation
**Date**: 2024-01-31
**Status**: ✅ Accepted
**Requirement**: Universal Entity Management

### Context
Need to validate entity metadata against defined schemas to ensure data integrity.

### Decision
Implement JSON schema validation at application level for entity metadata:
- Each entity type defines a JSON schema
- Metadata validated on create/update operations
- Invalid metadata rejected with clear error messages

### Consequences
- Ensures data integrity for all entity metadata
- Enables automatic UI generation from schemas
- Slight performance overhead for validation
```

---

## Phase 2: Communication System Implementation

### 2.1 Update Global Requirements
**Add to** `/app/workspace/requirements/GlobalRequirements/IndividualRequirements/52-universal-entity-management.md`:

```markdown
## Communication Tracking System

### Polymorphic Communication
- **source_type/source_id**: Can be system, user, or entity
- **target_type/target_id**: Can be system, user, or entity
- **correlation_id**: Links related communications
- **Ultra-simple design**: Only essential fields included

### Communication Flow
1. System initiates communication to entity
2. Communication logged with request data
3. Entity processes and responds
4. Response logged with correlation ID
5. All communications queryable by entity

### Reference Tables
- communication_type (API_REQUEST, VERIFICATION, etc.)
- communication_channel (API, EMAIL, SMS, PHONE)
- communication_status (PENDING, COMPLETED, FAILED)
```

### 2.2 Update Entity Catalog
**Add to communication section**:

```markdown
### Communication Patterns

#### API Communication
```sql
-- DCS driver verification communication
INSERT INTO communication (
    communication_type_id, source_type, source_id, 
    target_type, target_id, channel_id, direction,
    request_data, correlation_id
) VALUES (
    1, 'system', 1, 'entity', 5, 1, 'outbound',
    '{"license_number": "D12345678", "state": "TX"}',
    'quote-123-driver-verification'
);
```

#### Correlation Tracking
- All related communications share correlation_id
- Enables end-to-end request tracking
- Supports distributed tracing
```

---

## Phase 3: Configuration System Implementation

### 3.1 Update Global Requirements
**Add configuration section**:

```markdown
## Configuration Management

### Three-Level Hierarchy
1. **System Level**: Default configuration for all entities
2. **Program Level**: Program-specific overrides
3. **Entity Level**: Entity-specific settings

### Resolution Order
Entity → Program → System (most specific wins)

### Configuration Types
- API_SETTINGS (timeouts, retries, rate limits)
- INTEGRATION_SETTINGS (cache TTL, webhooks)
- SECURITY_SETTINGS (auth config, permissions)
```

### 3.2 Update ProducerPortal CLAUDE.md
**Add configuration patterns**:

```markdown
### Configuration Resolution
```php
// Get resolved configuration for entity
$config = ConfigurationService::resolve(
    'API_SETTINGS',
    'entity',
    $dcsEntity->id
);

// Returns merged configuration:
// System defaults + Program overrides + Entity specifics
```
```

---

## Phase 4: Component Security Implementation

### 4.1 Update Global Requirements
**Add to GR 36 or create new section in GR 52**:

```markdown
## Component-Based Security for Universal Entities

### System Components
- Define backend namespace and API prefix
- Associate with frontend routes and UI components
- Link to permission codes for access control

### Permission Model
- Component-level permissions (not entity-level)
- Granular access: read, write, delete, admin
- Security groups control component access
```

### 4.2 Update Architectural Decisions
**Add ADR-020**:

```markdown
## ADR-020: Component-Based Security Model
**Date**: 2024-01-31
**Status**: ✅ Accepted

### Context
Need security model that works with universal entities without entity-specific permissions.

### Decision
Implement component-based security where:
- Permissions attach to system components, not entities
- Components map backend/frontend/permissions
- Security groups grant component access

### Consequences
- Simpler permission model
- Consistent security across all entity types
- Single permission check for entity operations
```

---

## Phase 5: Integration and Testing Standards

### 5.1 Update Queue README
**Add to** `/app/workspace/requirements/ProducerPortal/queue/README.md`:

```markdown
## Universal Entity Quality Gates

### Before Processing Requirements
- [ ] Identify all external entities (APIs, partners, vendors)
- [ ] Determine if existing entity types can be reused
- [ ] Check entity catalog for patterns

### During Processing
- [ ] Apply universal entity pattern for external systems
- [ ] Use communication table for all external interactions
- [ ] Define configuration at appropriate scope
- [ ] Apply component-based security

### Quality Validation
- [ ] Entity types have JSON schemas
- [ ] Communication uses correlation IDs
- [ ] Configuration follows hierarchy
- [ ] Performance targets defined

### Integration Checklist
- [ ] Aligns with GR 44 (Communication)
- [ ] Aligns with GR 48 (External Integrations)
- [ ] Aligns with GR 36 (Authentication)
- [ ] Aligns with GR 52 (Universal Entity Management)
```

---

## Implementation Actions Summary

### Immediate Implementation Tasks

1. **Create Global Requirement 52** for Universal Entity Management
2. **Update Global CLAUDE.md** with entity/entity_type patterns
3. **Update ProducerPortal CLAUDE.md** with implementation examples
4. **Update Entity Catalog** with universal entity definitions
5. **Add ADRs 019-020** for JSON validation and component security
6. **Update Queue README** with universal entity quality gates

### Success Validation Criteria
- Entity queries <500ms for 10,000+ entities
- Communication queries <200ms
- Configuration resolution <100ms  
- Metadata validation <50ms
- New entity type creation <1 hour
- Zero code changes for new entities

### Files to Update
1. `/app/workspace/requirements/GlobalRequirements/IndividualRequirements/52-universal-entity-management.md` (NEW)
2. `/app/workspace/requirements/CLAUDE.md`
3. `/app/workspace/requirements/ProducerPortal/CLAUDE.md`
4. `/app/workspace/requirements/ProducerPortal/entity-catalog.md`
5. `/app/workspace/requirements/ProducerPortal/architectural-decisions.md`
6. `/app/workspace/requirements/ProducerPortal/queue/README.md`

### Final Step
After all above updates are complete, create comprehensive update to:
- `/app/workspace/requirements/ProducerPortal/queue/completed/IP269-New-Quote-Step-1-Primary-Insured/sections-c-e-universal.md`

This will incorporate all the established patterns, standards, and references from the global files.

---

## Additional Templates to Create

### universal-requirement-template.md
Create after global files are updated to ensure consistency.

### implementation-progress-tracker.md
Create to track implementation status of universal components.

---

This plan ensures all universal entity management concepts are properly established in global/reusable files before being applied to specific requirements.