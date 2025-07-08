# IP269-New-Quote-Step-1-Primary-Insured - Updated Approach

## Stakeholder Decisions Incorporated

### ✅ **Confirmed Business Logic:**
- **Quote Creation**: Quote entity created immediately when Step 1 begins
- **Program Availability**: Managed through producer manager (program assignments) and program manager (rate changes/availability)
- **Data Source**: DCS only for search (no internal database search)
- **Data Population**: DCS provides name, address, drivers license, ID card, and date of birth
- **Quote Number**: Assigned at Step 1
- **Producer Context**: Always attached to quote (producer portal, future direct-to-consumer)
- **Effective Date Rules**: Program-level configuration + no past dates allowed
- **External Integration**: DCS only for this step
- **Data Persistence**: Store and propagate results across models (no caching needed)
- **Duplicate Detection**: Not implemented at this time
- **Address Standardization**: Required but not in this specific requirement

## Remaining Questions Requiring Clarification

### Performance Requirements
1. **Search Response Time**: What are the acceptable performance requirements for DCS search functionality? **[Stakeholder suggestion requested]**

### Business Logic Details  
2. **Search Matching Criteria**: What constitutes a "match" in the DCS search functionality? Exact license number match, or multiple criteria validation?

## Infrastructure Review

### Existing Codebase Patterns
**Models Available:**
- `User` model - Has first_name, last_name, email, phone, address fields
- `Driver` model - Has first_name, last_name, middle_name, date_of_birth, license_number, license_state
- `Policy` model - Core insurance policy entity
- `MapUserPolicyDriver` - Association between users, policies, and drivers

**Service Layer:**
- `PolicyService` - Existing policy validation and business logic
- `UserPreferencesService` - User settings management
- **New Required**: QuoteService for quote management logic

**Database Schema Patterns:**
- Standard Laravel patterns with status_id references
- SoftDeletes for audit trails
- Relationship tracking via map_ tables

### API Pattern Alignment
**Existing Route Organization:**
- `routes/portal_api.php` - Perfect location for quote endpoints
- RESTful patterns established: GET, POST, PUT, DELETE
- Authentication via Laravel Sanctum tokens

## Approved Requirements Cross-Reference

### Similar Existing Requirements
- **IP269-Quotes-Search** - Has established quote entity patterns and search functionality
- Driver and vehicle association patterns already defined

### Reusable Patterns Identified
1. **Entity Relationships**: Use existing map table patterns (map_quote_driver)
2. **Status Management**: Use established status_id pattern throughout
3. **Address Handling**: Use existing address entity patterns

## Updated Implementation Approach

### Entity Strategy (Based on Stakeholder Decisions)
1. **Quote Entity**:
   - Create quote immediately on Step 1 start
   - Include producer_id (always attached)
   - Program relationship with availability rules
   - Effective date validation (no past dates + program rules)
   - Quote number generation at creation

2. **Program Management**:
   - Producer-to-program assignment table
   - Program availability and rate change management
   - Program-level configuration for validation rules

3. **DCS Integration**:
   - DCS search as primary data source
   - Store DCS results in quote/driver entities
   - No internal database search required

### Integration Patterns to Apply
1. **DCS Integration** (GR-53): For household driver search and data enrichment
2. **Universal Entity Management** (GR-52): For DCS API integration patterns
3. **Laravel Patterns**: Standard Eloquent relationships and service layer

### Workflow Implementation
1. **Quote Lifecycle**: Create → DCS Search → Data Population → Validation → Continue
2. **DCS Integration**: License-based search → Data enrichment → Entity population
3. **Validation Pipeline**: Effective date (no past + program rules) → Program availability → Continue

### Global Requirements Alignment
- **GR-53**: DCS Integration Architecture - for household driver search
- **GR-52**: Universal Entity Management - for DCS API integration
- **GR-41**: Table Schema Requirements - for naming conventions
- **GR-19**: Table Relationships - for map table patterns

## Detailed Implementation Plan

### Phase 1: Database Schema
1. **Quote Table** (Create immediately):
   ```sql
   - id, quote_number (generated at Step 1)
   - producer_id (always attached)
   - program_id, effective_date
   - status_id, created_by, updated_by
   - dcs_search_completed, dcs_data_populated
   ```

2. **Program Management**:
   ```sql
   - program table with availability rules
   - producer_program assignment table
   - program_configuration for validation rules
   ```

3. **DCS Integration Tracking**:
   ```sql
   - Store DCS search results in driver entities
   - Track DCS data source and completion status
   ```

### Phase 2: DCS Integration Service
1. **DCS Search Service**:
   - Integrate with DCS Household Drivers API (GR-53)
   - License-based search functionality
   - Data enrichment: name, address, license, ID, DOB
   - Store results directly in quote/driver entities

2. **Quote Service**:
   - Immediate quote creation on Step 1 start
   - Quote number generation
   - Producer assignment and validation
   - Program availability checking

### Phase 3: API Development
1. **Quote Management Endpoints**:
   ```
   POST /api/v1/quotes/start - Create quote immediately
   POST /api/v1/quotes/{id}/dcs-search - Trigger DCS search
   POST /api/v1/quotes/{id}/select-insured - Select DCS result
   GET /api/v1/quotes/{id}/programs - Get available programs
   ```

2. **Validation Endpoints**:
   ```
   POST /api/v1/quotes/{id}/validate-effective-date - Check date rules
   GET /api/v1/producers/{id}/programs - Get producer programs
   ```

### Phase 4: Business Rules Implementation
1. **Effective Date Validation**:
   - No past dates allowed
   - Program-specific date range rules
   - Program manager configuration integration

2. **Program Availability**:
   - Producer-to-program assignment checking
   - Program availability date ranges
   - Real-time program status validation

### Infrastructure Integration Points
- **DCS APIs**: Use Universal Entity Management patterns for integration
- **Database**: Immediate quote creation with Laravel Eloquent
- **Authentication**: Producer context via Laravel Sanctum
- **Validation**: Laravel request validation with program-specific rules
- **Performance**: Direct entity storage (no caching required per stakeholder)

### Risk Considerations
1. **DCS Performance**: Impact of DCS search response times on user experience
2. **Data Quality**: Handling partial or missing DCS data
3. **Program Rules**: Complex program availability and validation logic
4. **Quote Creation**: Managing immediate quote creation and potential abandonment

### Dependencies
1. **DCS API Access**: Household Drivers API integration (GR-53)
2. **Program Management**: Producer-to-program assignment system
3. **Quote Number Generation**: Sequence or algorithm for quote numbers
4. **Business Rules Engine**: Program-level configuration management

## Updated Recommendation

**Proceed with DCS-first implementation approach:**

1. **Implement immediate quote creation** - Follow stakeholder decision
2. **Integrate DCS Household Drivers API** using GR-53 patterns
3. **Build program management** with producer assignments and availability rules
4. **Create comprehensive validation** with program-level configuration
5. **Design for data persistence** across quote workflow steps

**Architecture Decision**: Use DCS as primary data source with immediate quote creation and producer-program management. Leverage existing infrastructure patterns while adding DCS integration capabilities.

**Performance Considerations**: Request stakeholder guidance on acceptable DCS search response times to optimize user experience.

**Next Steps**: 
1. **Clarify performance requirements** for DCS search functionality
2. **Define search matching criteria** for DCS results
3. **Proceed to full requirement implementation** once clarifications received