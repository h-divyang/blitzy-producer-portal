# Streamlined Requirements Generation Process

## Overview
This document outlines a repeatable, efficient process for generating technical requirements (sections C & E) based on our learnings from IP269-Quotes-Search implementation.

## Process Goals
1. **Reduce iteration cycles** from 7+ prompts to 2-3 focused sessions
2. **Establish architectural patterns** upfront rather than discovering them iteratively
3. **Create reusable templates** and decision frameworks
4. **Enable parallel processing** of multiple requirements
5. **Build cumulative knowledge** that improves with each requirement

## Phase 1: Environment Setup & Standards

### 1.1 Create Master CLAUDE.md Files

#### `/app/workspace/requirements/CLAUDE.md`
```markdown
# Requirements Generation Standards

## Database Design Principles
- All tables use singular nouns (e.g., `driver` not `drivers`)
- Map tables use `map_` prefix with singular nouns
- All tables include `status_id` (never `is_active` boolean)
- All tables include audit fields: `created_by`, `updated_by`, `created_at`, `updated_at`
- No `deleted_at` fields - use action table for deletion tracking
- Foreign keys always reference `id` field of parent table

## Field Normalization Rules
- Extract ENUMs to reference tables (e.g., `phone_type` instead of ENUM)
- Move characteristics from map tables to entity tables
- Use `_id` suffix for all foreign keys
- Boolean fields start with `is_` or `can_`
- Date/time fields use appropriate types (DATE, TIMESTAMP)

## Naming Conventions
- Tables: lowercase with underscores
- Columns: lowercase with underscores
- Indexes: `idx_` prefix
- Unique constraints: `unique_` prefix
- Foreign keys: `fk_` prefix (implicit in Laravel)

## Section C Requirements
- Backend mappings only (no frontend display info)
- Use arrow notation for query paths
- Include all JOINs and conditions
- Reference actual table and column names

## Section E Requirements
- Complete CREATE TABLE statements
- Include all constraints and indexes
- Follow established patterns from existing tables
- Group tables by type: Core, Reference, Relationship (map_), Supporting
```

#### `/app/workspace/requirements/ProducerPortal/CLAUDE.md`
```markdown
# Producer Portal Specific Standards

## Domain Context
- Multi-tenant architecture (but no tenant_id in quote tables)
- Producer-based access control
- Quote → Policy → Loss lifecycle
- Rate, Quote, Bind (RQB) workflow

## Common Entities
- driver (with is_named_insured indicator)
- vehicle (with is_primary_vehicle indicator)
- license (separate from driver)
- phone, email, address (separate entities)
- suspense (using map tables for associations)

## Established Patterns
- Named insured: Boolean on driver table, not map table
- Primary indicators: On entity tables, not map tables
- Suspenses: Map table approach (map_quote_suspense, map_policy_suspense)
- Status management: Single status table with status_type
- Documents: Separate document table with map associations

## Technology Stack
- Laravel 12.x with PHP 8.4+
- MariaDB 12.x LTS
- Redis 7.x for caching
- Elasticsearch (future) via Laravel Scout
- Laravel Echo + Pusher for real-time
- Kong API Gateway
```

### 1.2 Create Architectural Decision Record (ADR)

#### `/app/workspace/requirements/ProducerPortal/architectural-decisions.md`
```markdown
# Architectural Decisions for Producer Portal

## Decision Log

### ADR-001: Map Table Approach for Suspenses
**Date**: 2024-01-30
**Status**: Accepted
**Context**: Need to associate suspenses with quotes, policies, and losses
**Decision**: Use separate map tables (map_quote_suspense, etc.) instead of polymorphic
**Consequences**: More tables but cleaner queries and better performance

### ADR-002: Driver Type Management
**Date**: 2024-01-30
**Status**: Accepted
**Context**: Need flexible driver types for different MGAs
**Decision**: driver_type reference table with business logic in application layer
**Consequences**: Simplified database, flexible business rules

### ADR-003: Named Insured Designation
**Date**: 2024-01-30
**Status**: Accepted
**Context**: Need to identify primary insured among drivers
**Decision**: is_named_insured boolean on driver table
**Consequences**: Simple queries, enforced single named insured per quote

[Continue adding decisions as they're made...]
```

## Phase 2: Requirement Analysis Framework

### 2.1 Initial Analysis Checklist
```markdown
For each new requirement (e.g., IP269-New-Quote-Step-1-Primary-Insured.md):

1. **Read Base Requirement**
   - [ ] Identify all UI elements mentioned
   - [ ] List all data fields displayed
   - [ ] Note workflow states and transitions
   - [ ] Identify relationships to existing entities

2. **Cross-Reference Existing Code**
   - [ ] Search for similar models in /app/source
   - [ ] Identify reusable patterns
   - [ ] Note any conflicts with existing structure

3. **Check Related Requirements**
   - [ ] Review other IP269 files for context
   - [ ] Identify shared entities
   - [ ] Note dependencies and integration points

4. **Architectural Alignment**
   - [ ] Verify compliance with CLAUDE.md standards
   - [ ] Check architectural decisions record
   - [ ] Identify any new patterns needed
```

### 2.2 Requirements Template

#### `/app/workspace/requirements/ProducerPortal/templates/requirement-template.md`
```markdown
# [Requirement ID] - [Requirement Name]

## Entities Involved
- List all database entities
- Note new vs existing
- Identify relationships

## New Tables Required
- Table name: purpose
- Reference tables needed
- Map tables needed

## Modifications to Existing Tables
- Table name: changes needed
- Impact analysis

## Field Mappings (Section C)
### [UI Section Name]
#### [Field Name]
- **Backend Mapping**: 
  ```
  Complete query path
  ```

## Database Schema (Section E)
### New Tables
```sql
-- Complete CREATE TABLE statements
```

### Modified Tables
```sql
-- ALTER TABLE statements
```
```

## Phase 3: Parallel Processing Strategy

### 3.1 Batch Analysis Approach
```markdown
1. **Group Related Requirements**
   - Group by workflow (e.g., all quote steps together)
   - Identify shared entities across requirements
   - Plan table structure holistically

2. **Create Shared Entity Definitions**
   - Define once, reference many times
   - Maintain consistency across requirements
   - Update master entity list

3. **Process in Parallel**
   - Assign different team members to different requirements
   - Use consistent templates and standards
   - Regular sync on shared entities
```

### 3.2 Knowledge Accumulation

#### `/app/workspace/requirements/ProducerPortal/entity-catalog.md`
```markdown
# Master Entity Catalog

## Core Entities

### driver
- **Purpose**: Store driver information
- **Key Fields**: name_id, date_of_birth, driver_type_id, is_named_insured
- **Used By**: Quotes, Policies
- **Relationships**: Has many licenses, phones, emails, addresses

### vehicle
- **Purpose**: Store vehicle information
- **Key Fields**: vin, year, make, model, is_primary_vehicle
- **Used By**: Quotes, Policies
- **Relationships**: Has many registrations

[Continue for all entities...]
```

## Phase 4: Execution Process

### 4.1 Step-by-Step Workflow

1. **Preparation** (30 minutes)
   - Read requirement document
   - Run analysis checklist
   - Review related requirements
   - Check entity catalog

2. **Design Session** (1 hour)
   - Map all fields to backend
   - Design new tables
   - Identify modifications needed
   - Document architectural decisions

3. **Implementation** (30 minutes)
   - Generate Section C mappings
   - Generate Section E schemas
   - Validate against standards
   - Update entity catalog

4. **Review** (15 minutes)
   - Check CLAUDE.md compliance
   - Verify consistency
   - Update knowledge base

### 4.2 Quality Checkpoints

```markdown
## Pre-Implementation Checklist
- [ ] All UI fields mapped to database
- [ ] No redundant tables created
- [ ] Existing patterns reused
- [ ] Naming conventions followed
- [ ] Reference tables extracted from ENUMs

## Post-Implementation Checklist
- [ ] All foreign keys defined
- [ ] Indexes optimized for queries
- [ ] Audit fields included
- [ ] Status management consistent
- [ ] Entity catalog updated
```

## Phase 5: Continuous Improvement

### 5.1 Feedback Loop
```markdown
After each requirement:
1. Update CLAUDE.md with new patterns
2. Add architectural decisions to ADR
3. Update entity catalog
4. Refine templates based on learnings
5. Document any gotchas or edge cases
```

### 5.2 Metrics to Track
- Time per requirement
- Number of revision cycles
- Reuse percentage of existing entities
- Consistency score across requirements

## Implementation for Next Requirement

### For IP269-New-Quote-Step-1-Primary-Insured.md:

1. **Pre-Analysis** (before starting)
   - Review this process document
   - Check entity catalog for `driver`, `name`, `phone`, `email`, `address`
   - Read the requirement document
   - Note this is "Step 1" - plan for subsequent steps

2. **Expected Entities**
   - Reuse: driver, name, phone, email, address
   - Potentially new: quote_session, quote_draft
   - Map tables: map_driver_phone, map_driver_email, map_driver_address

3. **Key Decisions to Make**
   - How to handle quote drafts vs submitted quotes
   - Session management for multi-step process
   - Validation rules storage

## Summary

This streamlined process reduces iteration by:
1. **Establishing standards upfront** via CLAUDE.md files
2. **Creating reusable templates** for consistency
3. **Building cumulative knowledge** in entity catalog
4. **Enabling parallel processing** with clear frameworks
5. **Minimizing back-and-forth** through comprehensive analysis

Expected outcome: 2-3 focused sessions instead of 7+ iterations, with increasing efficiency as the knowledge base grows.