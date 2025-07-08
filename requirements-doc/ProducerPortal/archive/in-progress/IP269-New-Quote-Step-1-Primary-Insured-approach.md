# IP269-New-Quote-Step-1-Primary-Insured - Suggested Approach

## Questions Requiring Clarification

### Business Logic Questions
1. **Program Availability Logic**: What specific criteria determine program availability beyond effective date? Are there geographic restrictions, producer restrictions, or other business rules?
   2. When we address the producer manager, we will be able to assign the programs they are able to write.
   3. When we address the program manager, we will be able to manage when program / rate changes go live as well as define overall programs available in the system.
3. **30-Day Rule**: Is the 30-day future date restriction a hard limit or can certain user roles override it? What's the business rationale?
   4. This should be a program level configuration that can be set in the program manager when we get there.
   5. Also the effective date can not be in the past. Treated the same way.
3. **Driver Search Scope**: When searching for existing drivers, should we search across all producers or only within the current producer's portfolio?
   4. We need to run dcs only at this point and not search internally.
      5. if the drivers license information is found we will prefill all name, address, drivers license, id card, and date of birth information.
5. **Match Confidence**: What confidence threshold should trigger automatic matches vs requiring manual review?
   5. the end user determines it's a match by looking at the name and address that was just stored.
6. **Address Modification**: When "Address Incorrect" is selected, should we update the driver's master record or only the quote-specific address?
   7. In this case both as this represents the named insured.

### Technical Questions
1. **DCS Integration Specifics**: 
   - What exact DCS endpoints should we use for driver verification?
     - Reference Aime/workspace/requirements/TpaManager/Dcs/Household Drivers API Documentation Version 2.7 conv.html
   - What data fields does DCS return that we need to map?
     - Reference Aime/workspace/requirements/TpaManager/Dcs/Household Drivers API Documentation Version 2.7 conv.html
   - How should we handle partial matches from DCS?
     - There shouldn't be partial matches I dont think.
     - Reference Aime/workspace/requirements/TpaManager/Dcs/Household Drivers API Documentation Version 2.7 conv.html
2. **Performance Requirements**:
   - What's the acceptable response time for driver search?
     - Reference Aime/workspace/requirements/TpaManager/Dcs/Household Drivers API Documentation Version 2.7 conv.html
   - Should we implement type-ahead search or search-on-submit?
     - This search will be done on submit of the form.
   - How many results should we display in the match modal?
     - It should only reference 1 result at this point.
3. **Caching Strategy**:
   - How long should we cache DCS verification results?
     - Suggestion?
   - Should program availability be cached, and for how long?
     - Program availability should never be cached. We want to always look to evaluate at the time of quote.
4. **Security Considerations**:
   - Are there specific PII masking requirements for license numbers in logs?
     - Suggestion on best practice?
   - What audit trail requirements exist for driver searches?
     - Suggestions?

### Edge Cases and Validation Rules
1. What happens if DCS is unavailable - do we allow manual entry only?
   2. The process would look and feel the same just act as if it were a no-hit
2. How do we handle duplicate license numbers across different states?
   3. the license table should have a state_id right?
3. What validation rules apply to international license numbers?
   4. None at this time.
4. How should the system behave if no programs are available for the selected effective date?
   5. There will not be a program available in the dropdown.

## Suggested Implementation Approach

### Overview
Implement a robust quote initiation system that prioritizes data reuse through intelligent driver matching while providing flexible manual entry options. The approach emphasizes user experience through progressive disclosure based on license type selection and leverages external verification services with proper fallback mechanisms.

### Entity Strategy
**Entities to reuse:**
- `quote` - Main quote entity with workflow tracking
- `driver` - Existing driver profiles  
- `license` - License information with state tracking
- `address` - Standardized address storage
- `name` - Name components with computed full name
- `program` - Insurance program configurations
- All standard reference tables (license_type, address_type, etc.)

**Potential new entities:** None identified - existing entity catalog appears complete

**Relationship modifications:** None required - existing map tables sufficient

### Integration Approach
**External services required:**
1. **DCS Driver Verification API** (GR-53)
   - Use DCS_HOUSEHOLD_DRIVERS entity type
   - Implement circuit breaker with 5-failure threshold
   - Cache results for 24 hours
   
2. **Smarty Streets Address Validation** (GR-48)
   - For non-US addresses and standardization
   - Cache validated addresses for 30 days

**Integration patterns to apply:**
- Universal Entity Management (GR-52) for all external services
- Communication tracking (GR-44) with correlation IDs
- Circuit breaker patterns with graceful degradation

### Workflow Considerations
**State transitions:**
1. Quote created in DRAFT status
2. Primary insured search/selection
3. Update to PRIMARY_INSURED_COMPLETE
4. Ready for next step (Named Insured details)

**Validation points:**
- Effective date validation (≤ 30 days)
- License format validation by state
- Required field validation based on license type
- Program availability validation

**Error handling:**
- Graceful DCS fallback to manual entry
- Clear user messaging for validation failures
- Maintain form state during errors

## Detailed Game Plan

### Step 1: Entity Analysis
1. Confirm all entities exist in catalog ✓
2. Verify no schema modifications needed ✓
3. Document entity relationships for quote creation
4. Identify indexes needed for search performance

### Step 2: Global Requirements Alignment
**Primary GRs to apply:**
- **GR-52**: Universal Entity Management (external integrations)
- **GR-04**: Validation & Data Handling (form validation)
- **GR-18**: Workflow Requirements (quote state management)
- **GR-53**: DCS Integration Architecture (driver verification)
- **GR-07**: Reusable Components (form components)
- **GR-20**: Application Business Logic (program selection)

### Step 3: Backend Implementation (Section C)
1. **Quote Creation API**
   - POST /api/v1/quotes
   - Validate effective date
   - Return available programs
   
2. **Driver Search API**
   - POST /api/v1/quotes/{id}/primary-insured/search
   - Implement dual search (license vs personal info)
   - Integrate DCS verification
   - Return matches with confidence scores
   
3. **Driver Selection API**
   - POST /api/v1/quotes/{id}/primary-insured/select
   - Handle 3 selection scenarios
   - Create appropriate associations
   - Update workflow state

### Step 4: Database Schema (Section E)
1. Use existing tables without modification
2. Ensure proper indexes on:
   - license.license_number + state_id
   - name.first_name + last_name
   - program.effective_dates
3. Implement audit trail via existing created_by/updated_by

### Step 5: Quality Validation
1. Verify all use cases covered
2. Confirm integration fallbacks work
3. Validate performance requirements met
4. Ensure security requirements addressed

## Risk Considerations

### Technical Risks
1. **DCS API Reliability**: Mitigated by circuit breaker and caching
2. **Search Performance**: Mitigated by proper indexing and result limiting
3. **Data Quality**: Mitigated by validation and standardization

### Business Risks
1. **Duplicate Records**: Clear matching logic and confidence scoring
2. **Program Availability**: Real-time validation with clear messaging
3. **User Errors**: Progressive disclosure and clear field labels

### Mitigation Strategies
- Comprehensive error handling with user-friendly messages
- Fallback options for all external dependencies
- Extensive logging for troubleshooting
- Performance monitoring on all APIs

## Dependencies
- DCS API credentials and endpoint access
- Smarty Streets API configuration
- Program configuration data must be current
- License validation rules by state

## Recommendation

**Proceed with implementation** using existing entity catalog without modifications. The requirement is well-defined and aligns perfectly with established patterns. Key considerations:

1. **Prioritize User Experience**: Implement progressive disclosure for license type selection to minimize cognitive load
2. **Implement Robust Fallbacks**: Ensure system remains functional even when external services fail
3. **Focus on Performance**: Driver search is critical path - optimize queries and implement caching
4. **Maintain Flexibility**: Design APIs to accommodate future search criteria additions

The approach leverages all applicable Global Requirements while maintaining simplicity through maximum entity reuse. No architectural decisions are required as this follows established patterns.

## Next Steps Upon Approval
1. Generate complete requirement file with Sections C & E
2. Include comprehensive integration specifications
3. Document all API endpoints with examples
4. Create detailed schema documentation
5. Move to completed subdirectory