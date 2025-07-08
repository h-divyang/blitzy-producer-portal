# IP269-New-Quote-Step-1-Named-Insured - Updated Approach

## Cross-Requirement Decisions Incorporated

### ✅ **From Primary Insured Step:**
- **Quote Creation**: Quote entity already created in Step 1 (immediate creation)
- **DCS Integration**: DCS data already populated (name, address, license, ID, DOB)
- **Producer Context**: Producer always attached to quote
- **Data Persistence**: Store and propagate results across models (no caching needed)
- **External Integration**: DCS-first approach established

### 🔄 **Impact on Named Insured Step:**
- This step focuses on **reviewing and enriching** DCS-populated data
- **Read-only fields** come from DCS (already populated in Step 1)
- **Additional fields** require manual entry and validation
- **Data verification** may trigger additional DCS calls if needed

## Questions Requiring Clarification

### Business Logic Questions
1. **Data Verification**: Should additional DCS APIs be used to verify/enrich customer information beyond the initial search?
2. **Field Override Authority**: Can agents modify read-only fields if they identify errors in the DCS data?
3. **Discount Eligibility**: How is discount eligibility determined - business rules engine, external service, or manual selection?
4. **Prior Insurance Validation**: Should prior insurance information be verified against external sources (like ISO or CLUE)?
5. **Notification Preferences**: How do notification preferences integrate with existing communication systems?

### Technical Implementation Questions
1. **Data Enrichment Workflow**: Should additional external data enrichment happen automatically or on-demand?
2. **Field Validation Rules**: What are the specific validation rules for each field (email format, phone format, etc.)?
3. **Error Handling**: How should system handle partial failures in external data enrichment?
4. **Cross-field Validation**: How should dependencies be handled (e.g., email required for paperless discount)?

### Integration Specifics
1. **Additional DCS APIs**: Beyond initial search, are other DCS services needed for verification?
2. **Discount Rule Engine**: Is there an existing discount calculation system to integrate with?
3. **Prior Insurance Sources**: Integration with ISO, CLUE, or other insurance history databases?

## Infrastructure Review

### Existing Codebase Patterns
**Models Available:**
- `Driver` model - DCS data already populated from Step 1
- `User` model - Has address, phone, email fields for contact information
- `CommunicationPreference` model - For notification preferences
- Map tables for relationships (map_user_policy_driver)

**Missing Entities Identified:**
- Gender, marital status reference tables
- Prior insurance entity
- Discount/discount eligibility entities
- Housing information entity

**Service Layer:**
- `UserPreferencesService` - Can be extended for notification preferences
- `VerificationService` - Exists for contact verification
- **New Required**: Data enrichment and validation services

### API Pattern Alignment
**Existing Endpoints:**
- User preference endpoints in portal_api.php
- Verification endpoints for email/phone
- Standard RESTful patterns for entity management

**New Endpoints Needed:**
- Data enrichment endpoints
- Discount eligibility endpoints
- Prior insurance validation endpoints

### Service Layer Integration Points
**Business Logic Organization:**
- Extend existing verification services for data enrichment
- Create DiscountService for eligibility calculation
- Integrate with existing CommunicationPreference patterns

## Approved Requirements Cross-Reference

### Similar Existing Requirements
- **IP269-Quotes-Search** - Contains driver and contact information patterns
- **User Verification** patterns from existing codebase
- Communication preference patterns already established

### Reusable Patterns Identified
1. **Contact Information**: Use existing phone, email, address entity patterns
2. **Verification Workflow**: Leverage existing verification service patterns
3. **Reference Tables**: Use established _type suffix pattern for gender, marital status
4. **Map Tables**: Follow existing relationship tracking patterns

### Consistency Validation
- Contact information storage aligns with existing patterns
- Verification workflow consistent with existing verification service
- Notification preferences use existing communication preference structure

## Updated Implementation Approach

### Entity Strategy (Based on Cross-Requirement Decisions)
1. **Quote Data Enhancement**:
   - Quote already created with DCS data populated
   - Focus on supplementing with additional information
   - Maintain data persistence across workflow steps

2. **Read-Only Field Handling**:
   - DCS-populated fields displayed as read-only
   - Potential override capability (pending clarification)
   - Clear indication of data source (DCS vs. manual)

3. **Additional Data Collection**:
   - New reference entities for gender, marital status, housing
   - Prior insurance tracking entity
   - Discount eligibility management

### Integration Patterns to Apply
1. **DCS Integration** (GR-53): Additional verification calls if needed
2. **Universal Entity Management** (GR-52): For additional external data services
3. **Communication Architecture** (GR-44): For notification preference management

### Workflow Implementation
1. **Data Review**: Display DCS data → Review accuracy → Flag discrepancies
2. **Data Enhancement**: Additional fields → Validation → Cross-field rules
3. **Verification Pipeline**: Contact verification → Discount eligibility → Continue

### Global Requirements Alignment
- **GR-53**: DCS Integration Architecture - for additional verification services
- **GR-52**: Universal Entity Management - for external data enrichment services
- **GR-44**: Communication Architecture - for notification preferences
- **GR-04**: Validation & Data Handling - for field validation rules

## Detailed Implementation Plan

### Phase 1: Database Schema Enhancement
1. **New Reference Tables**:
   ```sql
   - gender (code, name, description)
   - marital_status (code, name, description) 
   - housing_type (code, name, description)
   ```

2. **Prior Insurance Entity**:
   ```sql
   - prior_insurance (quote_id, company_name, expiration_date, months_insured)
   ```

3. **Discount Management**:
   ```sql
   - discount_type (code, name, description, requirements)
   - quote_discount_eligibility (quote_id, discount_type_id, eligible, reason)
   ```

### Phase 2: Data Enhancement Service
1. **Data Review Service**:
   - Display DCS-populated data as read-only
   - Flag potential data quality issues
   - Handle override requests (if permitted)

2. **Additional DCS Integration** (if required):
   - Enhanced driver verification calls
   - Address validation services
   - Identity confirmation services

3. **Validation Services**:
   - Enhanced field validation with business rules
   - Cross-field validation (e.g., email required for paperless discount)
   - Real-time validation feedback

### Phase 3: API Development
1. **Data Review Endpoints**:
   ```
   GET /api/v1/quotes/{id}/insured-data - Get populated DCS data
   PUT /api/v1/quotes/{id}/insured-data - Update additional fields
   POST /api/v1/quotes/{id}/verify-data - Trigger additional verification
   ```

2. **Discount and Validation Endpoints**:
   ```
   GET /api/v1/quotes/{id}/eligible-discounts - Get discount eligibility
   POST /api/v1/quotes/{id}/validate-fields - Validate cross-field rules
   POST /api/v1/quotes/{id}/prior-insurance - Add/validate prior insurance
   ```

### Phase 4: Business Rules Implementation
1. **Cross-field Validation**:
   - Email required for paperless enrollment
   - Phone required for SMS notifications
   - Address validation for certain discounts

2. **Discount Eligibility**:
   - Dynamic discount calculation based on data
   - Real-time eligibility checking
   - Clear messaging for requirements

### Infrastructure Integration Points
- **DCS APIs**: Additional verification calls using existing patterns
- **Database**: Extend quote data with additional entities
- **Validation**: Laravel request validation with cross-field rules
- **UI Components**: Read-only display with override capabilities
- **Performance**: Direct entity storage consistent with Step 1 approach

### Risk Considerations
1. **Data Quality**: Handling conflicts between DCS data and user corrections
2. **Validation Complexity**: Managing complex cross-field validation rules
3. **Discount Logic**: Ensuring accurate and fair discount eligibility determination
4. **User Experience**: Clear indication of read-only vs. editable fields

### Dependencies
1. **Step 1 Completion**: DCS data populated and quote created
2. **Business Rules Definition**: Clear discount eligibility and validation rules
3. **Reference Data Setup**: Gender, marital status, housing type data
4. **External Services**: Potential additional DCS or other verification services

## Updated Recommendation

**Proceed with data enhancement and validation approach:**

1. **Build on Step 1 DCS integration** - Extend with additional verification if needed
2. **Implement comprehensive field validation** with cross-field business rules
3. **Create robust discount eligibility system** with clear requirements
4. **Design clear data source indicators** to distinguish DCS vs. manual data
5. **Maintain data persistence pattern** established in Step 1

**Architecture Decision**: Focus on enhancing and validating DCS-populated data while adding comprehensive additional information collection. Use existing verification patterns while adding discount and prior insurance management.

**Integration Strategy**: Leverage Step 1 DCS integration while adding specific business logic for data review, validation, and enhancement.

**Next Steps**: 
1. **Clarify field override policies** for DCS-populated data
2. **Define discount eligibility rules** and calculation methods
3. **Specify additional verification requirements** beyond Step 1 DCS search
4. **Proceed to implementation** once business rules clarified