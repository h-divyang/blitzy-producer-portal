# Requirements Architecture Reorganization Gameplan

## Overview
This gameplan outlines all changes needed to reorganize the requirements architecture to:
1. Fix the queue process to retain files in organized subdirectories
2. Establish GlobalRequirements as the single source of truth
3. Transform context files into lightweight references and translators
4. Integrate integration specifications into sections-c-e.md

---

## 1. QUEUE PROCESS UPDATES

### 1.1 Current Problem
- Files generated during in-progress status are deleted when done
- No organized structure for in-progress files
- Separate integration-spec.md files that should be integrated

### 1.2 New File Organization Structure
```
queue/
├── pending/
│   └── [original requirement files remain here]
├── in-progress/
│   └── IP269-New-Quote-Step-1-Primary-Insured/
│       ├── analysis-notes.md
│       ├── sections-c-e.md (includes integration specs)
│       └── implementation-summary.md
└── completed/
    └── IP269-New-Quote-Step-1-Primary-Insured/
        ├── original-requirement.md
        ├── analysis-notes.md
        ├── sections-c-e.md (final version)
        └── implementation-summary.md
```

### 1.3 Process Changes
- Create subdirectory in `in-progress/` when requirement moves from pending
- Keep all working files organized in subdirectory
- Move entire subdirectory contents to `completed/` when done
- No more separate integration-spec.md files

---

## 2. GLOBAL REQUIREMENTS AUDIT & ENHANCEMENT

### 2.1 Create New DCS Global Requirement
**File**: `/app/workspace/requirements/GlobalRequirements/IndividualRequirements/53-dcs-integration-architecture.md`

**Content to Extract From**:
- IP269 integration-spec.md
- DCS sections from ProducerPortal/CLAUDE.md
- DCS content from entity-catalog.md
- DCS examples from 52-universal-entity-management.md

**Structure**:
```markdown
# 53.0 DCS Integration Architecture

## Overview
Complete Data Capture Solutions (DCS) API integration specifications

## DCS API Endpoints
### DCS Household Drivers API v2.7
- Endpoint: https://ws.dcsinfosys.com:442/apidevV2.7/DcsSearchApi/HouseholdDrivers
- Entity Type: DCS_HOUSEHOLD_DRIVERS
- Authentication: HTTP Basic

### DCS Household Vehicles API v2.3
- Endpoint: https://ws.dcsinfosys.com:442/apidevV2.3/DcsSearchApi/HouseholdVehicles
- Entity Type: DCS_HOUSEHOLD_VEHICLES

### DCS Criminal Background API v1.0
- Endpoint: https://ws.dcsinfosys.com:442/apidevV2.8/DcsSearchApi/Criminal
- Entity Type: DCS_CRIMINAL

## Authentication & Security
## Configuration Management
## Circuit Breaker & Error Handling
## Performance Requirements
## Testing Patterns
## Compliance & Audit
```

### 2.2 Audit Existing Global Requirements
- Review all 52 files for topic granularity
- Split multi-topic files into focused requirements
- Each file should cover exactly one concept

### 2.3 Content Extraction Plan
**From Context Files to Global Requirements**:
- CLAUDE.md → Extract to appropriate GRs
- ProducerPortal/CLAUDE.md → DCS content to GR-53
- entity-catalog.md → Entity specs to GR-52
- architectural-decisions.md → Implementation details to relevant GRs

---

## 3. CONTEXT FILE RESTRUCTURING

### 3.1 Global CLAUDE.md (Currently Empty)
**Transform to**: Reference aggregator and translator

**New Content Structure**:
```markdown
# Requirements Generation Standards

## Overview
This document serves as a high-level guide, referencing specific global requirements for detailed implementation.

## Core Principles
- GlobalRequirements serve as single source of truth
- Context files translate and aggregate references
- Consistent patterns across all requirements

## Quick Reference Guide
- Database Design → See GR-02, GR-03, GR-19, GR-41
- Universal Entity Management → See GR-52
- DCS Integration → See GR-53
- Communication Architecture → See GR-44
- Security & Authentication → See GR-01, GR-36
- Workflow Requirements → See GR-18
- External Integrations → See GR-48

## Template Usage → See requirement-template.md
```

### 3.2 ProducerPortal CLAUDE.md
**Remove**:
- DCS Universal Entity Implementation Standards section
- DCS Entity Creation Pattern code
- DCS Multi-API Workflow Patterns code
- DCS Configuration Resolution code
- DCS Circuit Breaker Implementation code

**Keep & Transform**:
- Domain Context (as-is)
- Core Business Concepts (as-is)
- Established Entity Patterns (with references)
- Add: "For DCS integration details → See GR-53"
- Add: "For universal entity patterns → See GR-52"

### 3.3 Entity Catalog
**Remove**:
- Complete DCS API entity type definitions
- Workflow patterns implementation
- Detailed specifications

**Keep**:
- Entity summaries
- Relationships overview
- Usage patterns

**Add**:
- "For complete DCS specifications → See GR-53"
- "For entity type schemas → See GR-52"

### 3.4 Architectural Decisions
**For ADR-019 through ADR-023 (DCS-related)**:
- Keep decision rationale only
- Remove implementation details
- Add: "Implementation details in GR-53"

---

## 4. TEMPLATE & DOCUMENTATION UPDATES

### 4.1 Requirement Template Updates
**File**: `/app/workspace/requirements/ProducerPortal/templates/requirement-template.md`

**Add to Pre-Analysis Checklist**:
```markdown
### Global Requirements Alignment
- [ ] Review applicable global requirements
- [ ] Note which GRs apply to this requirement
- [ ] Ensure patterns align with global standards
```

**Update Section C Structure**:
```markdown
## Field Mappings (Section C)

### Backend Mappings
[Existing pattern]

### Implementation Architecture
[Details about how the requirement is implemented]

### Integration Specifications (if applicable)
#### API Integration Points
[API details, endpoints, authentication]

#### Configuration Management
[Configuration hierarchy and patterns]

#### Security Implementation
[Security patterns and component permissions]

#### Performance & Monitoring
[Performance requirements and monitoring approach]

#### Error Handling
[Circuit breaker, retry logic, graceful degradation]
```

### 4.2 README.md Updates
**Add**:
- New subdirectory organization process
- File retention policy
- Integration spec inclusion in sections-c-e.md
- Reference to template for standard output

---

## 5. IP269 CONTENT MIGRATION

### 5.1 Extract DCS Content to GR-53
**From**: integration-spec.md
**Extract**: All DCS-specific patterns and implementations

### 5.2 Merge Integration Content into sections-c-e.md
**Action**: Take remaining integration content and merge into Section C
**Structure**: Follow updated template format

### 5.3 Align sections-c-e.md with Template
**Current Issues**:
- Not following template structure exactly
- Missing clear section headers
- Integration content in separate file

**Fix**:
- Restructure to match template exactly
- Include integration specs in Section C
- Ensure all sections properly labeled

---

## 6. IMPLEMENTATION TASKS

### Task 1: Create DCS Global Requirement
1. Create 53-dcs-integration-architecture.md
2. Extract all DCS content from various files
3. Organize into comprehensive DCS specification

### Task 2: Restructure Context Files
1. Rebuild global CLAUDE.md as reference aggregator
2. Clean ProducerPortal CLAUDE.md - remove implementations, add references
3. Simplify entity-catalog.md to summaries + references
4. Update architectural-decisions.md to reference implementations

### Task 3: Update Templates and Process
1. Update requirement-template.md with integration section
2. Update README.md with new queue process
3. Document template usage in CLAUDE files

### Task 4: Migrate IP269 Content
1. Extract DCS content to new global requirement
2. Merge integration-spec.md into sections-c-e.md
3. Restructure sections-c-e.md to match template

### Task 5: Implement Queue Organization
1. Create IP269 subdirectory in in-progress
2. Organize existing files properly
3. Update process documentation

---

## 7. VALIDATION CHECKLIST

### Content Integrity
- [ ] No information lost during migration
- [ ] All references accurate and complete
- [ ] DCS content fully captured in GR-53
- [ ] Integration specs included in sections-c-e.md

### Process Functionality
- [ ] Queue subdirectory structure works correctly
- [ ] Files retained in organized manner
- [ ] Template produces consistent outputs
- [ ] Documentation accurately reflects new process

### Reference Accuracy
- [ ] All context files properly reference GRs
- [ ] No broken references
- [ ] Clear path from context to details
- [ ] Cross-references validated

---

## 8. EXPECTED OUTCOMES

### Immediate Benefits
1. **Organized Queue**: Files retained in clear structure
2. **Single Source of Truth**: GlobalRequirements authoritative
3. **Integrated Specs**: No separate integration files
4. **Clear References**: Easy navigation from context to details

### Long-term Benefits
1. **Maintainability**: Changes propagate through references
2. **Consistency**: Template ensures uniform output
3. **Efficiency**: Organized files improve workflow
4. **Clarity**: Each GR focused on single topic

---

## 9. FILES AFFECTED

### New Files (2)
1. This gameplan document
2. 53-dcs-integration-architecture.md

### Major Restructuring (5)
1. /app/workspace/requirements/CLAUDE.md
2. /app/workspace/requirements/ProducerPortal/CLAUDE.md
3. /app/workspace/requirements/ProducerPortal/entity-catalog.md
4. /app/workspace/requirements/ProducerPortal/architectural-decisions.md
5. /app/workspace/requirements/ProducerPortal/queue/README.md

### Updates (3)
1. /app/workspace/requirements/ProducerPortal/templates/requirement-template.md
2. IP269 sections-c-e.md (merge integration content)
3. IP269 implementation-summary.md (remove duplicate content)

### Process Changes
- Queue organization with subdirectories
- File retention instead of deletion
- Integration specs in main sections file

---

## 10. IMPLEMENTATION SEQUENCE

All tasks to be completed in this session:

1. **First**: Create this gameplan for approval
2. **Second**: Create DCS global requirement
3. **Third**: Restructure all context files
4. **Fourth**: Update template and process docs
5. **Fifth**: Migrate IP269 content
6. **Sixth**: Implement queue organization

This comprehensive plan addresses all requirements from prompt17.md and ensures a cleaner, more maintainable architecture.