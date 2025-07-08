# Requirements Generation Standards

## Complete Architecture Integration

### Repository Structure
- **Global Requirements**: `/app/workspace/requirements/GlobalRequirements/IndividualRequirements/`
- **Domain-Specific Standards**: `/app/workspace/requirements/[Domain]/CLAUDE.md`
- **Approved Requirements**: `/app/workspace/requirements/[Domain]/approved-requirements/`
- **Infrastructure Codebase**: `/app/workspace/blitzy-requirements/` (staging branch)

### Cross-Reference Standards
Every requirement must verify against:
1. Applicable Global Requirements
2. Domain-specific approved requirements  
3. Existing infrastructure patterns
4. Current codebase implementations

### Infrastructure Integration

#### Repository Access Procedures
Before processing any requirement:
```bash
cd /app/workspace/blitzy-requirements
git checkout staging
git pull origin staging
```

#### Cross-Reference Standards
Every requirement must validate against existing infrastructure:

**Database Schema Validation:**
- Check `src/backend/database/migrations/` for existing table patterns
- Review `src/backend/app/Models/` for entity relationships
- Validate naming conventions against established patterns

**API Pattern Validation:**
- Review `src/backend/routes/api.php` and `src/backend/routes/portal_api.php`
- Check `src/backend/app/Http/Controllers/` for endpoint patterns
- Validate RESTful conventions and response formats

**Service Layer Validation:**
- Review `src/backend/app/Services/` for business logic patterns
- Check integration with external services
- Validate authentication and authorization patterns

#### Performance Benchmarks
From actual infrastructure analysis:
- Entity queries: <500ms target
- API response times: <200ms for standard operations
- Database connections: Connection pooling via Laravel

### Quality Assurance Integration
Enhanced quality gates include:
- [ ] Global Requirements compliance verified
- [ ] Approved requirements consistency confirmed
- [ ] Infrastructure patterns aligned
- [ ] Git repository currency validated

## Overview
This document serves as a high-level guide and reference aggregator, pointing to specific global requirements for detailed implementation. All detailed specifications are maintained in the GlobalRequirements/IndividualRequirements directory.

## Core Principles
- GlobalRequirements/IndividualRequirements serve as the single source of truth
- Context files translate and aggregate references for practical use
- Consistent patterns across all requirements and domains
- Each global requirement focuses on exactly one topic for clarity

## Quick Reference Guide

### Database & Data Management
- **Database Design Principles** → See GR-02, GR-41
- **Table Relationships** → See GR-03, GR-19
- **Data Services & Caching** → See GR-33
- **Data Security** → See GR-24

### Entity & Architecture Patterns
- **Universal Entity Management** → See GR-52
- **External Integrations Catalog** → See GR-48
- **DCS Integration Architecture** → See GR-53
- **Microservice Architecture** → See GR-38

### Communication & Messaging
- **Communication Architecture** → See GR-44
- **Event-Driven Messaging** → See GR-49
- **Real-time Updates** → See GR-21

### Security & Authentication
- **Identity & Access Management** → See GR-01
- **Authentication & Permissions** → See GR-36
- **Security Considerations** → See GR-12
- **Component-Based Security** → See GR-52

### Workflow & Business Logic
- **Policy Reinstatement Process** → See GR-64
- **Workflow Requirements** → See GR-18
- **Business Logic Standards** → See GR-20
- **Locking & Action Tracking** → See GR-37

### Testing & Quality
- **Testing Requirements** → See GR-05, GR-10
- **Performance Requirements** → See GR-08, GR-27
- **Validation & Data Handling** → See GR-04

### Infrastructure & Deployment
- **Technology Standards** → See GR-00
- **Docker Requirements** → See GR-28, GR-29, GR-30
- **Deployment & Orchestration** → See GR-32
- **Disaster Recovery** → See GR-50

### Compliance & Documentation
- **Compliance & Audit** → See GR-51
- **Documentation Standards** → See GR-14
- **API Gateway Architecture** → See GR-47
- **SR22/SR26 Financial Responsibility Filing** → See GR-10

## Section C Requirements (Backend Mappings)

### Format Standards
Backend mappings should use clear arrow notation for query paths:
```
get [entity].id from [table]
-> get [related_entity] by [entity].[foreign_key]
-> return [fields], [transformations]
```

### Integration Specifications
When applicable, Section C should include:
- API Integration Points
- Configuration Management patterns
- Security Implementation details
- Performance & Monitoring approach
- Error Handling strategies

For detailed integration patterns → See GR-48, GR-53

## Section E Requirements (Database Schema)

### Table Organization
Group tables by type in this order:
1. Core Tables - Main business entities
2. Reference Tables - Lookup/type tables
3. Relationship Tables - map_* tables
4. Supporting Tables - Reusable entities

### Standards References
- Naming conventions → See GR-41
- Audit fields → See GR-02
- Status management → See GR-19
- Index patterns → See GR-33

## Quality Checklist

### Pre-Implementation
- [ ] **Review applicable global requirements** (GR-52, GR-48, GR-44, GR-41, GR-19, GR-64)
- [ ] **Check GR-64 for reinstatement patterns** if policy lifecycle involved
- [ ] **Check SR22/SR26 filing requirements** if financial responsibility filing needed (GR-10)
- [ ] **Check domain-specific approved requirements** for reusable patterns
- [ ] **Review infrastructure codebase** for existing implementations
- [ ] **Check if entity is external** - Use universal entity pattern if so (GR-52)
- [ ] **Verify patterns align with standards** and Global Requirements
- [ ] **Check entity catalog for reuse** - Avoid creating duplicate entities
- [ ] **Confirm naming conventions** following GR-41 standards
- [ ] **Validate communication patterns** align with GR-44 if applicable
- [ ] **Review DCS integration requirements** if driver/vehicle data involved (GR-53)
- [ ] **Validate against existing API patterns** using infrastructure codebase
- [ ] **Assess database schema consistency** with current codebase

### Implementation
- [ ] **Follow template structure** → See requirement-template.md
- [ ] **Include integration specs in Section C** with Global Requirements references
- [ ] **Reference global requirements appropriately** (GR-XX format)
- [ ] **Maintain consistency with existing patterns**
- [ ] **Use universal entity management** for all external entities (APIs, attorneys, body shops, vendors)
- [ ] **Implement JSON metadata schemas** for entity types requiring UI generation
- [ ] **Apply three-level configuration hierarchy** (entity → program → system) if configuration needed
- [ ] **Use polymorphic communication tracking** with correlation IDs for external communications

### Post-Implementation
- [ ] **Update entity catalog** if new entities discovered during implementation
- [ ] **Document architectural decisions** if new patterns emerge
- [ ] **Ensure all Global Requirements references are accurate** (GR-XX format)
- [ ] **Validate against approved requirements patterns** for consistency
- [ ] **Test against infrastructure codebase** (API consistency, schema alignment)
- [ ] **Validate against performance standards** (GR-52: <500ms entity queries, <200ms communication queries)
- [ ] **Confirm compliance requirements** met (7-year retention, audit logging, PII masking)
- [ ] **Test universal entity patterns** work correctly with metadata schemas
- [ ] **Verify communication templates** function with insurance-specific helpers
- [ ] **Document infrastructure integration points** if new patterns established
- [ ] **Update approved requirements library** if new reusable patterns created

## Template Usage
For consistent requirement documentation → See `/app/workspace/requirements/ProducerPortal/templates/requirement-template.md`

All requirements follow a single consolidated file approach including:
- Pre-Analysis Checklist with Global Requirements alignment
- Entity Analysis with reuse validation
- Complete Field Mappings (Section C) with implementation architecture
- Integration Specifications for external services
- Database Schema (Section E) with performance optimization
- Quality validation and compliance verification

## Domain-Specific Standards
For ProducerPortal-specific patterns → See `/app/workspace/requirements/ProducerPortal/CLAUDE.md`