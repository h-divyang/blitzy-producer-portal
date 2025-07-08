# Requirements Queue Management

## Overview
This queue system manages the processing of Producer Portal requirements through a standardized workflow. Requirements move through different stages from pending to completion.

## Directory Structure

```
queue/
├── pending/           # Requirements waiting to be processed
├── in-progress/       # Requirements currently being worked on
└── completed/         # Finished requirements with generated sections C & E
```

## Workflow Process

### 1. Adding Requirements to Queue
Place requirement documents in the `pending/` directory:
```bash
# Copy requirement to pending queue
cp /path/to/IP269-Something.md queue/pending/

# Or create symlink to avoid duplication
ln -s /path/to/IP269-Something.md queue/pending/
```

### 2. Processing Requirements

#### Single Requirement Workflow

##### Step 1: Move to In-Progress
```bash
# Move requirement to in-progress
mv queue/pending/IP269-Something.md queue/in-progress/
```

##### Step 2: Create Approach File
Create `queue/in-progress/IP269-Something-approach.md` with:
- Questions requiring clarification
- Suggested implementation approach
- Detailed game plan for approval
- Risk considerations and dependencies

##### Step 3: Review and Approve Approach
- Review approach file with stakeholders
- Iterate based on feedback
- Get explicit approval before proceeding

##### Step 4: Process Requirement (After Approval)
- Generate comprehensive requirement following template
- Include Sections C & E with full technical details
- Add integration specifications if applicable

##### Step 5: Complete and Move to Subdirectory
```bash
# Create requirement-specific subdirectory
mkdir -p queue/completed/IP269-Something/

# Move completed files
mv queue/in-progress/IP269-Something.md queue/completed/IP269-Something/
mv queue/in-progress/IP269-Something-approach.md queue/completed/IP269-Something/
# Create processing notes in completed subdirectory
```

#### Batch Processing
```bash
# Process multiple related requirements
mv queue/pending/IP269-*.md queue/in-progress/

# Identify shared entities across requirements
# Generate consolidated entity definitions
# Process each requirement using shared patterns
```

### 3. File Naming Conventions

#### In Progress
```
queue/in-progress/
├── IP269-Something.md                    # Original requirement
└── IP269-Something-approach.md           # Suggested approach file (NEW)
```

#### Completed
```
queue/completed/
├── IP269-Something/                      # Requirement-specific subdirectory
│   ├── IP269-Something.md                    # Complete consolidated requirement
│   └── IP269-Something-processing-notes.md  # Light processing reference
```

## Architecture Integration

### Global Requirements Alignment
All requirements must align with Global Requirements from:
`/app/workspace/requirements/GlobalRequirements/IndividualRequirements/`

Core references for every requirement:
- **GR-52**: Universal Entity Management
- **GR-04**: Validation & Data Handling  
- **GR-18**: Workflow Requirements
- **GR-44**: Communication Architecture
- **GR-48**: External Integrations Catalog
- **GR-53**: DCS Integration Architecture
- [Additional GRs as applicable]

### Approved Requirements Cross-Reference
Before processing any requirement, review existing approved requirements:
`/app/workspace/requirements/ProducerPortal/approved-requirements/`

Validation steps:
- Check for similar functionality patterns
- Identify reusable entity definitions
- Validate consistency with established approaches
- Document deviations and rationale

### Infrastructure Cross-Reference
Mandatory verification against existing codebase:
`/app/workspace/blitzy-requirements/` (staging branch)

Pre-processing checklist:
- [ ] Git repository updated (git pull)
- [ ] Existing entity models reviewed
- [ ] Database schema consistency verified
- [ ] API patterns identified for reuse
- [ ] Service layer integration points documented

### Git Update Procedures
Before starting any requirement analysis:
```bash
cd /app/workspace/blitzy-requirements
git checkout staging
git pull origin staging
```

**Alignment Validation:**
- [ ] Check which global requirements apply to current requirement
- [ ] Ensure ProducerPortal patterns align with global standards
- [ ] Cross-reference with `/app/workspace/requirements/CLAUDE.md` (global)
- [ ] Validate consistency with `/app/workspace/requirements/ProducerPortal/CLAUDE.md`
- [ ] Review approved requirements for reusable patterns
- [ ] Update `/app/workspace/requirements/ProducerPortal/architectural-decisions.md` if needed
- [ ] Ensure `/app/workspace/requirements/ProducerPortal/entity-catalog.md` reflects global patterns

## Quality Gates

### Enhanced Pre-Analysis Checklist

#### Repository Updates
- [ ] Global requirements reviewed for updates
- [ ] Approved requirements directory checked
- [ ] Blitzy requirements repository updated (git pull)

#### Architecture Alignment  
- [ ] Global Requirements applicability assessed
- [ ] Approved requirements patterns identified
- [ ] Infrastructure integration points documented
- [ ] Existing codebase patterns reviewed

#### Cross-Reference Validation
- [ ] Entity models consistency verified
- [ ] API patterns alignment confirmed
- [ ] Database schema impacts assessed
- [ ] Service layer integration planned

### Before Starting (Pending → In-Progress)
- [ ] Requirement document is complete
- [ ] Related requirements identified
- [ ] Dependencies noted
- [ ] Resources allocated
- [ ] **Enhanced pre-analysis checklist completed**
- [ ] **Global requirements reviewed for applicability**
- [ ] **Global standards alignment confirmed**

### Approach Approval Enhancement
- [ ] Approach file created with questions and suggestions
- [ ] Business logic questions identified
- [ ] Technical approach outlined
- [ ] Entity reuse strategy defined
- [ ] Integration requirements documented
- [ ] Risk considerations addressed
- [ ] **Infrastructure review completed**
- [ ] **Approved requirements alignment documented**
- [ ] **Global requirements compliance verified**
- [ ] **Infrastructure integration confirmed**
- [ ] **Existing codebase patterns leveraged**
- [ ] **Approach reviewed and approved by stakeholders**

### Before Completion (In-Progress → Completed)
- [ ] Complete requirement file generated following template structure
- [ ] Pre-Analysis Checklist completed with Global Requirements alignment
- [ ] Entity Analysis section complete with reuse validation
- [ ] Field Mappings (Section C) with backend implementation details
- [ ] Integration Specifications included (if external services involved)
- [ ] Database Schema (Section E) with all tables and relationships
- [ ] Quality checklist passed
- [ ] **Global requirements alignment validated**
- [ ] **Cross-references to global standards documented**
- [ ] Performance and security requirements addressed
- [ ] All technical architecture details consolidated

## Batch Processing Strategy

### Related Requirements Grouping
Group requirements that share entities or workflows:
```
Group 1: Quote Creation Flow
├── IP269-New-Quote-Step-1-Primary-Insured.md (completed - with integration architecture)
├── IP269-New-Quote-Step-2-Drivers.md
├── IP269-New-Quote-Step-3-Vehicles.md
└── IP269-New-Quote-Step-4-Coverages.md

Group 2: Quote Management
├── IP269-Quotes-Search.md (completed)
├── IP269-Quote-Details.md
└── IP269-Quote-Actions.md

Group 3: Integration-Heavy Requirements
├── All DCS-related integrations (leverage established patterns)
├── Document processing workflows
└── External verification services
```

### Shared Entity Management
1. **Identify Common Entities**: Review all requirements in group
2. **Define Once**: Create shared entity definitions
3. **Reference Consistently**: Use same patterns across requirements
4. **Update Catalog**: Maintain central entity catalog

## Commands Reference

### Queue Status
```bash
# Check queue status
find queue/ -name "*.md" | wc -l
ls -la queue/pending/
ls -la queue/in-progress/
ls -la queue/completed/
```

### Move Requirements
```bash
# Start processing requirement
mv queue/pending/[requirement].md queue/in-progress/

# Complete requirement with subdirectory
mkdir -p queue/completed/[requirement]/
mv queue/in-progress/[requirement].md queue/completed/[requirement]/
mv queue/in-progress/[requirement]-approach.md queue/completed/[requirement]/
```

### Batch Operations
```bash
# Process all quote-related requirements
mv queue/pending/IP269-*Quote*.md queue/in-progress/

# Complete batch
for req in queue/in-progress/IP269-*.md; do
    basename=$(basename "$req" .md | sed 's/-approach$//')
    mkdir -p "queue/completed/$basename/"
    mv "queue/in-progress/$basename.md" "queue/completed/$basename/"
    mv "queue/in-progress/$basename-approach.md" "queue/completed/$basename/"
done
```

## Monitoring & Metrics

### Track Progress
- Number of requirements in each stage
- Average processing time per requirement
- Entity reuse percentage
- Quality checkpoint pass rates

### Continuous Improvement
- Update templates based on learnings
- Refine quality checklists
- Optimize batch processing strategies
- Improve entity catalog organization

## Getting Started

### Process First Requirement
1. **Add to queue**: 
   ```bash
   ln -s ../../input/IP269/IP269-New-Quote-Step-1-Primary-Insured.md queue/pending/
   ```

2. **Review templates**:
   - Study `templates/requirement-template.md`
   - Review `CLAUDE.md` standards
   - Check `entity-catalog.md`

3. **Begin processing**:
   ```bash
   mv queue/pending/IP269-New-Quote-Step-1-Primary-Insured.md queue/in-progress/
   ```

4. **Follow streamlined process**:
   - Use requirement template
   - **Review applicable global requirements**
   - Reference entity catalog
   - **Ensure global standards alignment**
   - Apply established patterns
   - Generate sections C & E

5. **Complete and document**:
   ```bash
   mkdir -p queue/completed/IP269-New-Quote-Step-1-Primary-Insured/
   mv queue/in-progress/IP269-New-Quote-Step-1-Primary-Insured.md queue/completed/IP269-New-Quote-Step-1-Primary-Insured/
   mv queue/in-progress/IP269-New-Quote-Step-1-Primary-Insured-approach.md queue/completed/IP269-New-Quote-Step-1-Primary-Insured/
   ```

This queue system enables efficient processing of single requirements or batch processing of related requirements while maintaining quality and consistency.

## External Integration Quality Gates

### For DCS and Other External Integrations
For complete DCS integration specifications → See GR-53

### Integration Processing Checklist
- [ ] Identify all API integration points
- [ ] Apply universal entity pattern → See GR-52
- [ ] Use communication table for tracking → See GR-44
- [ ] Configure at appropriate scope → See GR-52
- [ ] Apply component-based security → See GR-36
- [ ] Implement circuit breaker patterns → See GR-53

### Global Requirements Alignment
- [ ] GR 44 (Communication Architecture)
- [ ] GR 48 (External Integrations Catalog)
- [ ] GR 52 (Universal Entity Management)
- [ ] GR 53 (DCS Integration Architecture)
- [ ] GR 36 (Authentication & Security)
- [ ] GR 33 (Data Services & Performance)

## Approach File Template

Each requirement must have an approach file created before processing. The approach file should include:

### Enhanced Approach File Requirements

Each approach file must include:

#### Infrastructure Review Section
- [ ] Existing codebase patterns identified
- [ ] Database schema impacts assessed
- [ ] Service layer modifications documented
- [ ] API consistency verified
- [ ] Migration requirements identified

#### Approved Requirements Alignment
- [ ] Similar requirements reviewed
- [ ] Pattern reuse validated
- [ ] Consistency maintained
- [ ] Deviations justified

#### Global Requirements Compliance
- [ ] All applicable GRs identified
- [ ] Implementation patterns verified
- [ ] Architecture alignment confirmed

### Structure
```markdown
# [Requirement ID] - Suggested Approach

## Questions Requiring Clarification
- Business logic questions
- Technical implementation questions  
- Integration specifics
- Edge cases and validation rules

## Infrastructure Review
- Existing codebase patterns
- Database schema consistency check
- API pattern alignment
- Service layer integration points

## Approved Requirements Cross-Reference
- Similar existing requirements
- Reusable patterns identified
- Consistency validation
- Deviation rationale

## Suggested Implementation Approach
- Overview of approach
- Entity reuse strategy
- Integration patterns to apply
- Workflow considerations
- Global Requirements alignment

## Detailed Game Plan
- Step-by-step implementation plan
- Global Requirements alignment
- Infrastructure integration points
- Risk considerations
- Dependencies

## Recommendation
Clear recommendation on how to proceed with full architecture alignment
```

## Infrastructure Validation Examples

### Specific Infrastructure Cross-Reference Procedures

#### Database Schema Validation
**Before defining new entities, check:**
```bash
# Check existing models for similar patterns
ls /app/workspace/blitzy-requirements/src/backend/app/Models/
grep -r "class.*Model" /app/workspace/blitzy-requirements/src/backend/app/Models/

# Review existing migrations for table patterns
ls /app/workspace/blitzy-requirements/src/backend/database/migrations/ | grep -E "(policy|user|document|payment)"

# Check existing relationships
grep -r "belongsTo\|hasMany\|hasOne" /app/workspace/blitzy-requirements/src/backend/app/Models/
```

#### API Pattern Validation
**Before defining new endpoints, verify existing patterns:**
```bash
# Check existing API routes
cat /app/workspace/blitzy-requirements/src/backend/routes/api.php
cat /app/workspace/blitzy-requirements/src/backend/routes/portal_api.php

# Review controller patterns
ls /app/workspace/blitzy-requirements/src/backend/app/Http/Controllers/Api/
grep -r "class.*Controller" /app/workspace/blitzy-requirements/src/backend/app/Http/Controllers/
```

#### Service Layer Validation  
**Before implementing business logic, check existing services:**
```bash
# Review existing service patterns
ls /app/workspace/blitzy-requirements/src/backend/app/Services/
cat /app/workspace/blitzy-requirements/src/backend/app/Services/PolicyService.php | head -20
cat /app/workspace/blitzy-requirements/src/backend/app/Services/PaymentService.php | head -20
```

### Concrete Infrastructure Examples

#### Example 1: Policy-Related Requirement
When processing a policy requirement:
1. **Check existing Policy model**: `/app/workspace/blitzy-requirements/src/backend/app/Models/Policy.php`
2. **Review policy migrations**: Look for policy-related migration files
3. **Check PolicyService**: `/app/workspace/blitzy-requirements/src/backend/app/Services/PolicyService.php`
4. **Validate API routes**: Check `portal_api.php` for policy endpoints
5. **Document consistency**: Use existing `map_policy_*` table patterns

#### Example 2: Payment Processing Requirement
When processing payment requirements:
1. **Examine Payment model**: `/app/workspace/blitzy-requirements/src/backend/app/Models/Payment.php`
2. **Review PaymentService**: `/app/workspace/blitzy-requirements/src/backend/app/Services/PaymentService.php`
3. **Check payment types**: Review `payment_method_type` reference table
4. **API integration**: Look for existing payment controller patterns
5. **Transaction handling**: Use existing `transaction` and `map_policy_transaction` patterns

#### Example 3: Document Management Requirement
When processing document requirements:
1. **Study DocumentManager**: `/app/workspace/blitzy-requirements/src/backend/app/Services/DocumentManager.php`
2. **Review Document model**: Check existing document/file relationship patterns
3. **File storage**: Use existing Laravel filesystem abstraction
4. **Access control**: Follow `map_user_document` access patterns
5. **Audit trail**: Use existing `map_document_action` tracking

### Infrastructure Validation Checklist

#### Pre-Analysis Infrastructure Review
- [ ] **Git repository updated**: `cd /app/workspace/blitzy-requirements && git pull`
- [ ] **Existing models reviewed**: Check for similar entity patterns
- [ ] **Migration files analyzed**: Understand existing table structures
- [ ] **Service patterns identified**: Review business logic organization
- [ ] **API routes examined**: Understand endpoint conventions
- [ ] **Reference tables cataloged**: Document existing _type tables

#### Architecture Alignment Validation
- [ ] **Laravel patterns followed**: Framework conventions maintained
- [ ] **Eloquent relationships**: Use established relationship patterns
- [ ] **Service layer compliance**: Business logic in appropriate services
- [ ] **API organization**: Routes in correct files with proper prefixes
- [ ] **Naming conventions**: Follow existing entity naming patterns
- [ ] **Status management**: Use existing `status` table pattern

#### Performance and Consistency Checks
- [ ] **Database performance**: Follow existing indexing patterns
- [ ] **API response format**: Match existing JSON structure
- [ ] **Authentication**: Use Laravel Sanctum token patterns
- [ ] **File storage**: Use Laravel filesystem abstraction
- [ ] **Error handling**: Follow existing error response patterns
- [ ] **Audit trails**: Use established logging and tracking patterns

## Template Usage

All requirements must follow the standard template structure defined in:
`/app/workspace/requirements/ProducerPortal/templates/requirement-template.md`

### Key Template Sections
- **Pre-Analysis Checklist**: Global Requirements alignment and compliance verification
- **Entity Analysis**: Entity mappings, reuse validation, and relationship identification
- **Field Mappings (Section C)**: Backend mappings, implementation architecture, and integration specifications
- **API Specifications**: Endpoint definitions and real-time update channels
- **Database Schema (Section E)**: Complete table definitions following established patterns
- **Implementation Notes**: Dependencies, performance considerations, and quality validation

### Single File Approach
Requirements are now processed into a single comprehensive file that includes:
- All original requirement content (sections A, B, D)
- Complete technical implementation (sections C, E)
- Integration specifications for external services
- Performance, security, and architecture details
- Quality checklists and validation criteria

For complete template details → See requirement-template.md