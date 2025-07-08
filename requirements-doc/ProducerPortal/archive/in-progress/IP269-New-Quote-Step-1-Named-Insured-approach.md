# IP269-New-Quote-Step-1-Named-Insured - Suggested Approach

## Questions Requiring Clarification

### Business Logic Questions
1. **Relationship to Primary Insured**: Is the Named Insured always the same as the Primary Insured from the previous step, or can they be different individuals?
   2. They are the same.
2. **Read-Only Fields Logic**: Why are certain fields read-only when a record is found? Can agents override this in certain scenarios?
   3. No. Readonly fields are there to prevent the agent from modfiying them.
3. **Prior Insurance Validation**: 
   - Is there a minimum/maximum for "Number of Months Insured"?
     - This will be defined in the Program Manager when we get to it.
   - Should we validate expiration dates are in the past?
     - Not yet.
   - Do we need to verify prior insurance through external services?
     - Not yet.
4. **Discount Eligibility**:
   - What are all the available discount types?
     - This will be defined in the Program Manager.
   - What validation rules apply to each discount?
     - This will be defined in the Program Manager.
   - Are discounts mutually exclusive or stackable?
     - This will be defined in the Program Manager.
5. **Notification Preferences**: What are the available notification channels and are there defaults?
   6. This will be defined in the Program Manager.

### Technical Questions
1. **Data Enrichment Sources**: Which third-party services should be used for data enrichment beyond DCS?
   2. Address verification will run on submission of this page.
3. **Email Validation**: Should we validate email addresses in real-time or just format validation?
   4. Format for now.
3. **Phone Number Validation**: Do we need carrier lookup or just format validation?
   4. format validation
4. **Address Modification**: When address is modified, should we re-run address standardization?
   5. only on submission of the page.
5. **Field Dependencies**: Are there other field dependencies like the paperless/email example?
   6. These will be outlined in the Program Manager.

### Edge Cases and Validation Rules
1. How do we handle conflicting data between our records and third-party sources?
   2. Example?
2. What if the named insured is a minor - different validation rules?
   3. Not at this point.
3. International phone number support requirements?
   4. Example?
4. What happens if required discount validation fails after selection?
   5. Dont worry about this right now.

## Suggested Implementation Approach

### Overview
Build upon the Primary Insured selection to create a comprehensive driver profile by enriching with additional personal information, contact details, and insurance history. The approach emphasizes data quality through validation while maintaining flexibility for manual overrides when necessary.

### Entity Strategy
**Entities to reuse:**
- `driver` - Enhanced with additional profile fields
- `phone` - For primary and alternate numbers
- `email` - With validation tracking
- `address` - May need modification from Primary Insured step
- `prior_insurance` - May need new entity for insurance history
- `discount` - Reference table for available discounts
- `notification_preference` - Reference table

**Potential new entities:**
- `prior_insurance` - Track insurance history (company, dates, duration)
- `driver_discount` - Map table for driver-discount associations
- `housing_type` - Reference table for housing status

**Relationship modifications:**
- Add map_driver_phone for multiple phone numbers
- Add map_driver_discount for discount associations

### Integration Approach
**External services required:**
1. **Email Validation Service** (e.g., SendGrid Validation API)
   - Real-time validation for email addresses
   - Prevent invalid emails for paperless discount

2. **Phone Validation Service** (e.g., Twilio Lookup)
   - Format validation and carrier lookup
   - Support for international numbers

3. **Insurance History Verification** (TBD)
   - Verify prior insurance claims
   - May use industry databases

**Integration patterns to apply:**
- Reuse Universal Entity Management patterns
- Implement validation caching to reduce API calls
- Graceful degradation for validation services

### Workflow Considerations
**State transitions:**
1. Continue from PRIMARY_INSURED_COMPLETE
2. Enrich named insured profile
3. Update to NAMED_INSURED_COMPLETE
4. Ready for Step 2 (Additional Drivers)

**Validation points:**
- Email required if paperless discount selected
- Phone number format validation
- Prior insurance date logic
- Discount eligibility rules

**Error handling:**
- Field-level validation with inline errors
- Confirmation modals for data changes
- Clear dependency error messages

## Detailed Game Plan

### Step 1: Entity Analysis
1. Confirm if `prior_insurance` entity exists
2. Verify discount management approach
3. Check for existing housing_type references
4. Document new relationships needed

### Step 2: Global Requirements Alignment
**Primary GRs to apply:**
- **GR-04**: Validation & Data Handling (extensive field validation)
- **GR-07**: Reusable Components (form sections and modals)
- **GR-20**: Application Business Logic (discount eligibility)
- **GR-11**: Accessibility (form field dependencies)
- **GR-44**: Communication Architecture (notification preferences)

### Step 3: Backend Implementation (Section C)
1. **Get Named Insured Details API**
   - GET /api/v1/quotes/{id}/named-insured
   - Return enriched driver data
   
2. **Update Named Insured API**
   - PUT /api/v1/quotes/{id}/named-insured
   - Validate all fields and dependencies
   - Handle discount eligibility
   
3. **Validation Services**
   - POST /api/v1/validation/email
   - POST /api/v1/validation/phone
   - Implement caching layer

### Step 4: Database Schema (Section E)
1. **New Tables Required:**
   - `prior_insurance` (if not exists)
   - `map_driver_discount`
   - Reference tables for housing_type, discount_type
   
2. **Modifications:**
   - Ensure driver table has gender_id, marital_status_id
   - Add notification_preference_id to driver

3. **Indexes:**
   - Email validation lookups
   - Discount eligibility queries

### Step 5: Quality Validation
1. Test all field dependencies
2. Verify validation service fallbacks
3. Confirm discount eligibility logic
4. Validate confirmation modal flows

## Risk Considerations

### Technical Risks
1. **Validation Service Reliability**: Cache results and allow manual override
2. **Complex Field Dependencies**: Clear UI indicators and error messages
3. **Data Conflicts**: Clear precedence rules for data sources

### Business Risks
1. **Incorrect Discount Application**: Robust eligibility validation
2. **Missing Required Data**: Progressive form completion indicators
3. **Prior Insurance Fraud**: Consider verification requirements

### Mitigation Strategies
- Comprehensive field-level validation
- Clear visual indicators for dependencies
- Audit trail for all data modifications
- Manual override capabilities with logging

## Dependencies
- Email validation service setup
- Phone validation service setup  
- Complete discount eligibility rules
- Prior insurance verification requirements
- Clear field dependency matrix

## Recommendation

**Proceed with implementation** but first clarify:
1. Complete list of available discounts and eligibility rules
2. Prior insurance verification requirements
3. Whether prior_insurance entity already exists
4. Specific validation service preferences

The requirement builds naturally on Step 1 Primary Insured and can leverage much of the same infrastructure. Key focus areas:

1. **Data Quality**: Implement robust validation while maintaining usability
2. **Clear Dependencies**: Visual indicators for field relationships
3. **Flexible Enrichment**: Allow both automated and manual data entry
4. **Audit Compliance**: Track all data sources and modifications

## Next Steps Upon Approval
1. Clarify outstanding questions
2. Confirm entity additions needed
3. Generate complete requirement with validation logic
4. Document all field dependencies
5. Create comprehensive test scenarios