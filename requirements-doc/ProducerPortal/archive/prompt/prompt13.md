that didn't quite provide what I wanted. 

Files To Analyze:
* Aime/workspace/requirements/ProducerPortal/prompt/output/next-steps-integration-plan.md
* Aime/workspace/requirements/ProducerPortal/prompt/prompt12.md
* /app/workspace/requirements/CLAUDE.md
* /app/workspace/requirements/ProducerPortal/CLAUDE.md
* /app/workspace/requirements/ProducerPortal/entity-catalog.md
* /app/workspace/requirements/ProducerPortal/architectural-decisions.md
* Aime/workspace/requirements/ProducerPortal/queue/README.md
* Aime/workspace/requirements/GlobalRequirements/IndividualRequirements

### 1. Update sections-c-e-universal.md
**File**: `/app/workspace/requirements/ProducerPortal/queue/completed/IP269-New-Quote-Step-1-Primary-Insured/sections-c-e-universal.md`
- this should be done last so it can implement everything below, appropriatly.

**Add Implementation Sections**:
- **Phase 1 Implementation Details** (Weeks 1-2): Core universal tables setup
    - entity_category, entity_type, entity tables implementation
    - JSON schema validation implementation for DCS integration
    - Basic CRUD operations with metadata handling
    - Sample DCS entity creation and validation
      - go ahead and make a plan to do all of these now.
      - implmeemt them in global requirements, claude files, entity-catalog.md, architectural-decisions.md

- **Phase 2 Implementation Details** (Weeks 3-4): Communication system implementation
    - communication_type, communication_channel, communication_status tables
    - communication table with polymorphic relationships
    - Communication service layer with correlation tracking
    - DCS driver verification communication implementation
      - go ahead and make a plan to do all of these now.
      - implmeemt them in global requirements, claude files, entity-catalog.md, architectural-decisions.md

- **Phase 3 Implementation Details** (Weeks 5-6): Configuration system implementation
    - configuration_type and configuration tables
    - Scope-based configuration resolution (system → program → entity)
    - Configuration management UI for DCS settings
    - Entity-level configuration overrides
      - go ahead and make a plan to do all of these now.
      - implmeemt them in global requirements, claude files, entity-catalog.md, architectural-decisions.md

- **Phase 4 Implementation Details** (Weeks 7-8): Component security implementation
    - system_component and system_component_permission tables
    - Component-based security middleware
    - UI access control integration for entity management
    - Permission validation for DCS operations
      - go ahead and make a plan to do all of these now.
      - implmeemt them in global requirements, claude files, entity-catalog.md, architectural-decisions.md

- **Phase 5 Implementation Details** (Weeks 9-10): Integration and testing
    - Integration with Global Requirements 44 and 48
    - Performance optimization and testing with DCS
    - End-to-end quote creation flow validation
    - Documentation completion
      - go ahead and make a plan to do all of these now.
      - implmeemt them in global requirements, claude files, entity-catalog.md, architectural-decisions.md

**Add Immediate Actions Section**:
- Begin Phase 1 Implementation: Start with core universal tables
- Validate JSON Schemas: Test DCS entity type definitions with sample data
- Setup Development Environment: Prepare for universal entity development
- Create DCS Integration Entity: First practical implementation
  - go ahead and make a plan to do all of these now.
  - implmeemt them in global requirements, claude files, entity-catalog.md, architectural-decisions.md

**Add Success Validation Criteria**:
- Performance targets:
    - Entity listing queries: <500ms for 10,000+ entities
    - Communication queries: <200ms for large datasets
    - Configuration resolution: <100ms for complex hierarchies
    - Metadata validation: <50ms per entity
- Development efficiency metrics:
    - New entity type creation: <1 hour end-to-end
    - UI automatically supports new entity types
    - Zero code changes for adding external entity types
- Architecture validation criteria:
    - Communication table has <15 columns
    - Entity operations require <5 table JOINs
    - Configuration resolution in <3 queries
    - System handles 10,000+ entities efficiently
- go ahead and make a plan to do all of these now.

implmement anything and eveyrthing from a global / reusable perspective in the following areas only so we can have a managebable set of guidelines that can be used globally.
* /app/workspace/requirements/CLAUDE.md
* /app/workspace/requirements/ProducerPortal/CLAUDE.md
* /app/workspace/requirements/ProducerPortal/entity-catalog.md
* /app/workspace/requirements/ProducerPortal/architectural-decisions.md
* Aime/workspace/requirements/ProducerPortal/queue/README.md
* Aime/workspace/requirements/GlobalRequirements/IndividualRequirements

once all of this has been defined these areas, we should reoutline sections-c-e-universal.md

let's redo Aime/workspace/requirements/ProducerPortal/prompt/output/simplified-integration-plan.md with a new plan.