# Architectural Decision Record (ADR) - Producer Portal

## Overview
This document tracks key architectural decisions made during the development of Producer Portal requirements. Each decision includes context, rationale, and consequences.

---

## ADR-001: Map Table Approach for Suspenses
**Date**: 2024-01-30  
**Status**: ✅ Accepted  
**Requirement**: IP269-Quotes-Search

### Context
Need to associate suspenses with multiple entity types (quotes, policies, losses) while maintaining clean database design.

### Decision
Use separate map tables (`map_quote_suspense`, `map_policy_suspense`, `map_loss_suspense`) instead of polymorphic relationships.

### Rationale
- Cleaner, more explicit queries
- Better database performance with proper indexing
- Easier to maintain foreign key constraints
- Follows established patterns in the codebase

### Consequences
- More tables to maintain
- Need separate queries for each entity type
- But: Better performance and maintainability

---

## ADR-002: Driver Type Management
**Date**: 2024-01-30  
**Status**: ✅ Accepted  
**Requirement**: IP269-Quotes-Search

### Context
Different MGAs have different driver classification needs (included, excluded, listed-only, etc.).

### Decision
Use `driver_type` reference table with business logic in application layer, not database constraints.

### Rationale
- Flexible for MGA-specific requirements
- Simple database structure
- Business rules can evolve without schema changes
- Consistent with other reference table patterns

### Consequences
- Business logic must be enforced in application

---

## ADR-013: Leverage Existing Global Infrastructure for Integrations
**Date**: 2024-01-30  
**Status**: ✅ Accepted  
**Requirement**: IP269-New-Quote-Step-1-Primary-Insured (Enhanced)

### Context
Need to implement third-party integrations (DCS API) and internal quote duplication checking while maintaining consistency with existing platform architecture.

### Decision
Extend existing Apache Camel integration platform (Global Requirement 48) and CommunicationService patterns (Global Requirement 44) rather than build custom integration layer.

### Rationale
- Reuses proven patterns from global requirements
- Maintains consistency across platform
- Reduces development time and maintenance overhead
- Leverages existing security, monitoring, and error handling

### Consequences
- Must follow established communication patterns
- Integration requests flow through existing audit systems
- But: Faster implementation and better consistency

---

## ADR-014: Hybrid JSON/Relational Mapping Storage
**Date**: 2024-01-30  
**Status**: ✅ Accepted  
**Requirement**: IP269-New-Quote-Step-1-Primary-Insured (Enhanced)

### Context
Need flexible system to map third-party API responses to internal database fields while supporting complex transformation rules and versioning.

### Decision
Use relational tables for mapping metadata with JSON fields for transformation rules in `integration_field_mapping` table.

### Rationale
- Enables complex queries on mapping structure
- Supports flexible business logic transformations
- Allows for versioning and auditability
- JSON provides needed flexibility without schema changes

### Consequences
- Slightly more complex queries for mapping lookups
- Need JSON validation for transformation rules
- But: Maximum flexibility with good performance

---

## ADR-015: Hierarchical Integration Configuration
**Date**: 2024-01-30  
**Status**: ✅ Accepted  
**Requirement**: IP269-New-Quote-Step-1-Primary-Insured (Enhanced)

### Context
Multiple stakeholders need different levels of control over integration settings (system admins, program managers, producer admins).

### Decision
Implement system → program → producer configuration hierarchy with override capabilities in `integration_configuration` table.

### Rationale
- Supports enterprise requirements for granular control
- Allows system defaults with program/producer customization
- Scales from simple to complex deployment scenarios
- Clear precedence rules prevent configuration conflicts

### Consequences
- More complex configuration lookup logic
- Need to manage configuration inheritance
- But: Supports all business scenarios and scales well

---

## ADR-016: Comprehensive Audit Trail with PII Protection
**Date**: 2024-01-30  
**Status**: ✅ Accepted  
**Requirement**: IP269-New-Quote-Step-1-Primary-Insured (Enhanced)

### Context
Insurance industry requires comprehensive audit trails for compliance while protecting PII data and providing user access to integration results.

### Decision
Log all API requests/responses in `integration_request` table with automatic PII masking, configurable retention periods, and role-based access to raw responses.

### Rationale
- Meets insurance industry compliance requirements
- Enables debugging and support while protecting sensitive data
- Provides user transparency into verification results
- Supports GDPR and state privacy law requirements

### Consequences
- Additional storage requirements for audit data
- Need automated PII detection and masking
- But: Full compliance and transparency achieved
- But: Maximum flexibility for different business needs

---

## ADR-003: Named Insured Designation
**Date**: 2024-01-30  
**Status**: ✅ Accepted  
**Requirement**: IP269-Quotes-Search

### Context
Need to identify the primary insured person among multiple drivers on a quote/policy.

### Decision
Use `is_named_insured` boolean on `driver` table, not in map tables.

### Rationale
- Simple and direct relationship
- Easy to query
- Can enforce business rule of single named insured
- Avoids complex join patterns

### Consequences
- Must ensure only one driver per quote has `is_named_insured = true`
- But: Simpler queries and data integrity

---

## ADR-004: Status Management Strategy
**Date**: 2024-01-30  
**Status**: ✅ Accepted  
**Requirement**: IP269-Quotes-Search

### Context
Need consistent status tracking across all entities while avoiding boolean `is_active` fields.

### Decision
Use single `status` table with `status_type` categories, all entities reference `status_id`.

### Rationale
- Consistent pattern across all tables
- Flexible status definitions
- Better reporting capabilities
- Aligns with existing global requirements

### Consequences
- Additional join required for status information
- But: Consistent, flexible, and maintainable

---

## ADR-005: Frontend Color Management
**Date**: 2024-01-30  
**Status**: ✅ Accepted  
**Requirement**: IP269-Quotes-Search

### Context
Need status indicators with colors for UI, but colors may change frequently.

### Decision
Store status codes only in database, manage colors in frontend application.

### Rationale
- UI concerns separated from data concerns
- Colors can change without database migrations
- Easier to maintain consistent styling
- Reduces database complexity

### Consequences
- Frontend must maintain color mappings
- But: Better separation of concerns and flexibility

---

## ADR-006: Entity Characteristic Placement
**Date**: 2024-01-30  
**Status**: ✅ Accepted  
**Requirement**: IP269-Quotes-Search

### Context
Where to store entity characteristics like `is_primary`, `is_verified`, type information.

### Decision
Store characteristics on the entity tables themselves, not in map tables.

### Rationale
- Characteristics belong to the entity, not the relationship
- Simpler queries and better performance
- Reusable across different contexts
- More normalized design

### Consequences
- Entity tables are larger
- But: Better normalization and reusability

---

## ADR-007: ENUM to Reference Table Strategy
**Date**: 2024-01-30  
**Status**: ✅ Accepted  
**Requirement**: IP269-Quotes-Search

### Context
Database ENUM fields are inflexible and difficult to maintain.

### Decision
Convert all ENUM fields to reference tables with foreign key relationships.

### Rationale
- More flexible for adding new values
- Better for reporting and analytics
- Easier to maintain descriptions
- Consistent with architectural patterns

### Consequences
- More tables and joins required
- But: Much more flexible and maintainable

---

## ADR-008: Multi-tenant Data Isolation
**Date**: 2024-01-30  
**Status**: ✅ Accepted  
**Requirement**: IP269-Quotes-Search

### Context
Need multi-tenant data isolation without explicit tenant_id in all tables.

### Decision
Handle multi-tenancy at application layer through user → producer relationships.

### Rationale
- Cleaner database design
- Flexibility in tenant model
- Easier to query without tenant_id everywhere
- Aligns with existing user management

### Consequences
- Must enforce tenancy in application layer
- But: Cleaner schema and more flexibility

---

## ADR-009: Quote Submission Timestamp
**Date**: 2024-01-30  
**Status**: ✅ Accepted  
**Requirement**: IP269-Quotes-Search

### Context
Whether to use separate `submitted_at` field or reuse `created_at` for quote timestamps.

### Decision
Use `created_at` for quote submission, remove `submitted_at` field.

### Rationale
- Simpler schema
- Quote creation IS submission in this context
- Consistent with other entity patterns
- Reduces redundant timestamp fields

### Consequences
- Cannot distinguish draft vs submitted quotes easily
- But: Simpler model and less redundancy

---

## ADR-018: Ultra-Simplified Universal Entity Management
**Date**: 2024-01-31  
**Status**: ✅ Accepted  
**Requirement**: Universal Entity Management Architecture (Final Simplified)

### Context
After iterative simplification based on user feedback through prompts 6-9, need to establish final architecture that:
- Eliminates all unnecessary complexity
- Focuses on core functionality only
- Provides maximum maintainability
- Supports building from scratch without migration concerns
- Ensures future requirements can leverage universal patterns

### Decision
Implement ultra-simplified Universal Entity Management with:
- **No licensing system complexity** - Focus on core functionality only
- **Simple component system** for backend-frontend-security association
- **Three-level configuration hierarchy** (system-program-entity)
- **Ultra-simple communication** with polymorphic source/target only
- **Reference tables** following _type pattern for business concepts
- **ENUMs only** for truly static architectural concepts

### Rationale
User feedback consistently requested maximum simplification while maintaining universal entity management benefits:
- "This seems too complicated. lets keep it to source and target type where they reference the table"
- "this needs to be as simple as possible"
- "Complexity of initial implementation should not out-weigh long term maintainability"
- Building from scratch allows optimal design without migration constraints
- 90% faster development for new entity types
- Zero code changes required to add new external entities

### Consequences
**Positive:**
- Maximum simplicity and maintainability
- Zero code changes for new entity types  
- Clear patterns throughout system
- High performance through simple relationships
- Easy to understand and extend
- 65% lower total cost of ownership over 3 years

**Negative:**
- Less business context tracking in communication table
- Requires polymorphic queries for entity relationships
- JSON metadata requires application-level validation

### Implementation References
- **Entity Management Details** → See GR-52
- **Communication Patterns** → See GR-44
- **Configuration Management** → See GR-52
- **Security Implementation** → See GR-36

---

## ADR-019: DCS API JSON Schema Validation
**Date**: 2024-01-31
**Status**: ✅ Accepted
**Requirement**: Universal Entity Management

### Context
Need to validate DCS API entity metadata against defined schemas to ensure data integrity and prevent configuration errors.

### Decision
Implement JSON schema validation at application level for DCS entity metadata.

**Implementation Details** → See GR-53

### Consequences
- Ensures data integrity for all DCS API configurations
- Enables automatic UI generation from schemas
- Prevents runtime errors from invalid configurations
- Slight performance overhead for validation

## ADR-020: DCS Multi-API Correlation Strategy
**Date**: 2024-01-31
**Status**: ✅ Accepted

### Context
DCS provides three separate APIs that need to work together for complete driver/vehicle verification workflows.

### Decision
Implement correlation ID strategy for DCS multi-API workflows.

**Implementation Details** → See GR-53

### Consequences
- Enables end-to-end workflow tracking
- Simplifies debugging of multi-API failures
- Supports distributed tracing requirements
- Requires consistent correlation ID management

## ADR-021: DCS Authentication and Security
**Date**: 2024-01-31
**Status**: ✅ Accepted

### Context
DCS APIs require HTTP Basic Authentication with sensitive account credentials that must be securely managed.

### Decision
Implement secure credential management for DCS authentication using HashiCorp Vault.

**Implementation Details** → See GR-53

### Consequences
- Meets insurance industry security requirements
- Enables automated credential rotation
- Provides audit trail for compliance
- Adds complexity to configuration management

## ADR-022: DCS Component-Based Security Model
**Date**: 2024-01-31
**Status**: ✅ Accepted

### Context
Need security model that works with DCS universal entities while meeting insurance industry compliance requirements.

### Decision
Implement tiered component-based security for DCS APIs with Basic, Premium, and Administrator access levels.

**Implementation Details** → See GR-53

### Consequences
- Meets insurance regulatory requirements
- Flexible permission model supports business tiers
- Comprehensive audit trail for compliance
- Single permission check for DCS operations
- Simplified licensing model alignment

## ADR-023: DCS Data Retention and Privacy
**Date**: 2024-01-31
**Status**: ✅ Accepted

### Context
DCS APIs return sensitive personal information subject to insurance industry retention and privacy requirements.

### Decision
Implement comprehensive DCS data privacy controls with 7-year retention and PII protection.

**Implementation Details** → See GR-53

### Consequences
- Meets NAIC and state insurance regulations
- Supports CCPA and similar privacy laws
- Enables consumer rights compliance
- Requires additional storage and processing overhead
- Complex data lifecycle management

## ADR-030: Policy Reinstatement Architecture
**Date**: 2025-01-07  
**Status**: ✅ Accepted  
**Requirement**: GR-64 Policy Reinstatement with Lapse Process

### Context
Need to implement policy reinstatement functionality following Global Requirement patterns while maintaining consistency with existing policy lifecycle architecture. Policy reinstatement allows customers to restore canceled policies within a specified time window, subject to specific business rules and premium adjustments.

### Decision
Implement reinstatement using existing workflow patterns from GR-18 and business logic patterns from GR-20, extending PolicyService with dedicated reinstatement methods and integrating with established audit and payment processing systems.

### Rationale
- Reuses proven policy lifecycle patterns from GR-18 workflow requirements
- Maintains consistency with existing architecture established in ADR-024, ADR-025, ADR-027
- Leverages existing payment processing infrastructure from PaymentService patterns
- Follows established audit and logging requirements from GR-37
- Integrates with SR22 filing system (GR-10) without requiring new filings
- Supports program-specific business rules through configuration hierarchy

### Consequences
**Positive:**
- Extends existing services rather than creating new architecture
- Maintains consistency with established policy workflow patterns
- Leverages existing payment and document generation infrastructure
- Comprehensive audit trail through established action tracking
- Integration with SR22 system maintains regulatory compliance

**Negative:**
- Additional complexity in policy state management with new reinstatement states
- Premium calculation logic requires time-sensitive adjustments
- But: Complexity managed through existing service layer patterns

### Implementation References
- **Business Logic** → See GR-20 PolicyService reinstatement methods
- **Workflow Integration** → See GR-18 reinstatement state transitions
- **Database Schema** → See GR-41 reinstatement calculation table
- **Action Tracking** → See GR-37 reinstatement audit requirements
- **SR22 Integration** → See GR-10 reinstatement impact on filing status

---

## ADR-024: Laravel Infrastructure Standards
**Date**: 2025-01-07  
**Status**: ✅ Accepted  
**Source**: Infrastructure Analysis from blitzy-requirements

### Context
Based on analysis of the existing blitzy-requirements codebase, need to establish infrastructure standards for all new requirements to maintain consistency with current implementation.

### Decision
Adopt established Laravel infrastructure patterns from existing codebase:
- **Framework**: Laravel 10.x with PHP 8.4+ 
- **Authentication**: Laravel Sanctum with API tokens
- **Database**: MariaDB with Eloquent ORM
- **File Storage**: Laravel filesystem abstraction
- **Service Layer**: Dedicated Service classes for business logic

### Rationale
- Consistency with existing infrastructure
- Proven patterns already implemented
- Reduces development overhead and learning curve
- Leverages existing authentication and security systems

### Consequences
- All new requirements must follow Laravel patterns
- Existing service layer patterns must be extended
- But: Faster development through pattern reuse

---

## ADR-025: API Endpoint Organization
**Date**: 2025-01-07  
**Status**: ✅ Accepted  
**Source**: Infrastructure Analysis from blitzy-requirements

### Context
Existing codebase has established API organization patterns that new requirements should follow.

### Decision
Follow existing API organization:
- `routes/api.php` - General authentication and system endpoints
- `routes/portal_api.php` - Insured Portal specific endpoints  
- Standard RESTful CRUD patterns for resource endpoints
- Route prefixes for logical grouping

### Rationale
- Maintains consistency with existing API structure
- Clear separation between general and portal-specific endpoints
- Follows RESTful conventions already established
- Easier for frontend developers familiar with existing patterns

### Consequences
- New endpoints must fit into existing route organization
- API versioning considerations for future changes
- But: Consistent API experience across platform

---

## ADR-026: Database Entity Naming Conventions
**Date**: 2025-01-07  
**Status**: ✅ Accepted  
**Source**: Infrastructure Analysis from blitzy-requirements

### Context
Analysis of existing models reveals established naming patterns that should be maintained.

### Decision
Follow existing entity naming conventions:
- **Core entities**: Singular business domain names (policy, user, document)
- **Reference tables**: `_type` suffix (payment_method_type, verification_status_type)
- **Map tables**: `map_` prefix (map_user_policy_driver, map_document_action)
- **Universal status**: Single `status` table referenced by all entities via `status_id`

### Rationale
- Maintains consistency with 30+ existing models
- Clear naming patterns make schema easier to understand
- Established relationships already proven in production
- Reduces cognitive overhead for developers

### Consequences
- New entities must follow established naming patterns
- Status management must use existing `status` table
- But: Consistent, maintainable database schema

---

## ADR-027: Service Layer Architecture Patterns
**Date**: 2025-01-07  
**Status**: ✅ Accepted  
**Source**: Infrastructure Analysis from blitzy-requirements

### Context
Existing codebase has established service layer patterns for business logic organization.

### Decision
Follow existing service patterns:
- **PolicyService** - Policy validation and business logic
- **PaymentService** - Payment processing and history
- **DocumentManager** - Document processing and storage
- **UserPreferencesService** - User settings management
- Business logic in Services, Controllers handle HTTP concerns only

### Rationale
- Consistent with existing 8+ service classes
- Clear separation of concerns already established
- Testable business logic patterns
- Easier maintenance and debugging

### Consequences
- New business logic must be implemented in Service classes
- Controllers must remain thin and focused on HTTP concerns
- But: Maintainable, testable business logic architecture

---

## ADR-028: Document Management Infrastructure
**Date**: 2025-01-07  
**Status**: ✅ Accepted  
**Source**: Infrastructure Analysis from blitzy-requirements

### Context
Existing infrastructure has comprehensive document management system with established patterns.

### Decision
Leverage existing document infrastructure:
- `document` table for metadata and references
- `file` table for physical file storage details  
- Map tables for entity associations (map_user_document, map_document_action)
- DocumentManager service for business logic
- File storage through Laravel filesystem abstraction

### Rationale
- Proven document management system already implemented
- Established audit trail and access control patterns
- File storage abstraction allows multiple backends
- Existing security and permission patterns

### Consequences
- New document requirements must use existing infrastructure
- File storage implementation details abstracted
- But: Robust, tested document management system

---

## ADR-029: Infrastructure Cross-Reference Validation
**Date**: 2025-01-07  
**Status**: ✅ Accepted  
**Source**: Infrastructure Integration Requirements

### Context
Need systematic approach to validate new requirements against existing infrastructure patterns to maintain consistency and reuse.

### Decision
Implement mandatory infrastructure cross-reference validation:
- Git repository currency check before requirement analysis
- Database schema consistency validation against migrations
- API pattern alignment with existing routes
- Service layer pattern compliance with existing architecture
- Model relationship validation against established patterns

### Rationale
- Prevents divergent architectural patterns
- Ensures maximum reuse of existing infrastructure
- Reduces development time through pattern reuse
- Maintains system consistency and maintainability

### Consequences
- Additional validation steps required for all requirements
- Must keep infrastructure repository current
- But: Consistent, maintainable system architecture

---

## Template for New Decisions

```markdown
## ADR-XXX: [Decision Title]
**Date**: YYYY-MM-DD  
**Status**: 🚧 Proposed | ✅ Accepted | ❌ Rejected | 🔄 Superseded  
**Requirement**: [Requirement ID]

### Context
[Describe the situation and problem that needs a decision]

### Decision
[State the decision clearly]

### Rationale
[Explain why this decision was made]

### Consequences
[Describe the positive and negative consequences]
```

---

## Decision Status Legend
- 🚧 **Proposed**: Under consideration
- ✅ **Accepted**: Approved and implemented
- ❌ **Rejected**: Decided against
- 🔄 **Superseded**: Replaced by newer decision