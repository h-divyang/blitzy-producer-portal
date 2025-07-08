# IP269-New-Quote-Step-2-Drivers - Suggested Approach

## Questions Requiring Clarification

### Business Logic Questions
1. **Household Driver Search**: 
   - What defines "associated with the address" - same address, nearby, or household relationship?
     - these are the results from dcs. check the documentation in Aime/workspace/requirements/Documentation/Dcs/Household Drivers API Documentation Version 2.7 conv.html
   - When does the household search re-run automatically?
     - when the address is changed and the form / page is submitted.
   - How are "New" drivers differentiated from existing unprocessed drivers?
     - compare between first name, last name, date of birth, drivers license number from the dcs results and the drivers on the quote.
2. **Marital Status Rule**:
   - Must both married drivers be included, or can one be excluded?
     - one can be excluded.
   - What if a married driver's spouse is not in the household?
     - The spouse needs to be listed as an excluded driver.
   - How is this rule enforced in the UI?
     - Suggestion?
3. **Criminal Eligibility**:
   - What criteria determine criminal ineligibility?
     - This will be defined in the Program Manager
   - Is this checked via external service or internal rules?
     - This is done via the DCS Criminal service to get the criminal history for all included drivers based on first name, last name, and date of birth
     - We need to store the results and reference them in our validation process.
   - Can agents see why a driver is ineligible?
     - Yes.
4. **Driver Removal**:
   - What are valid removal reasons?
     - these will be defined in Program Manager
   - Can removed drivers be re-added?
     - Yes
   - Is removal different from exclusion for rating purposes?
     - Yes. Most will only have the included and excluded options available to choose from, but Remove is requested by some.
5. **SR-22 Requirements**:
   - What are all valid reasons for SR-22?
     - This will be defined in the Program Manager
   - Does SR-22 affect driver eligibility?
     - This will be defined in the Program Manager
   - State-specific SR-22 rules?
     - This will be defined in the Program Manager

### Technical Questions
1. **Violation Management**:
   - How many violations can be added per driver?
     - This will be defined in the Program Manager
   - Are violations verified through external services?
     - Violations are manually added from an internal list and/or added by external services like DCS when defined in the Program Manager
   - What violation types are supported?
     - This will be defined in the program manager.
2. **Primary Driver Logic**:
   - How is primary driver determined/assigned?
     - It is the named insured.
   - Can there be multiple primary drivers?
     - No.
   - Is primary driver per vehicle or per policy?
     - Per policy.
3. **Search Implementation**:
   - Is driver search local (in list) or backend search?
     - local
   - What fields are searchable?
     - name
4. **Pagination**:
   - How many drivers per page?
     - decided by the rows per page drop down.
   - Does pagination apply to each section separately?
5. **Configuration Options**:
   - What other features can be configured on/off besides removal?

### Edge Cases and Validation Rules
1. What if DCS returns drivers already in our system?
   2. Do not re-add them as drivers.
2. Maximum number of drivers allowed on a policy?
   3. Will be defined in Program Manager.
3. Minimum age requirements for included drivers?
   4. Will be defined in Program Manager.
4. How to handle duplicate driver entries?
   5. What do you mean?
5. Interstate license validation requirements?
   6. Not for now.

## Suggested Implementation Approach

### Overview
Implement a comprehensive driver management system that leverages household search data while providing flexible manual entry. The system must enforce complex business rules (marital status, criminal eligibility) while maintaining a smooth user experience through clear sectioning and search capabilities.

### Entity Strategy
**Entities to reuse:**
- `driver` - Core driver information
- `license` - License details
- `violation` - New entity needed for violations
- `map_quote_driver` - Enhanced with include/exclude/remove status
- All reference tables (gender, marital_status, relationship_to_insured, etc.)

**Potential new entities:**
- `violation` - Track traffic violations
- `violation_type` - Reference table for violation types
- `map_driver_violation` - Associate violations with drivers
- `sr22_reason` - Reference table for SR-22 reasons
- `removal_reason` - Reference table for removal reasons
- `employment_status` - Reference table
- `occupation` - Reference table (if not exists)

**Relationship modifications:**
- Enhance map_quote_driver with driver_status (included/excluded/removed)
- Add removal_reason_id to map_quote_driver

### Integration Approach
**External services required:**
1. **DCS Household Search** (GR-53)
   - Periodic re-runs to find new household members
   - Returns drivers associated with address
   
2. **Criminal Background Check** (TBD)
   - Determine criminal eligibility
   - May use DCS or separate service
   
3. **DMV Violation Lookup** (TBD)
   - Verify violations
   - Get violation details

**Integration patterns to apply:**
- Batch processing for household search results
- Real-time eligibility checking
- Caching of criminal/violation data

### Workflow Considerations
**State transitions:**
1. From NAMED_INSURED_COMPLETE
2. Manage driver list (add/edit/remove)
3. Validate all business rules
4. Update to DRIVERS_COMPLETE

**Validation points:**
- Marital status consistency check
- Criminal eligibility verification  
- Required fields per driver
- SR-22 documentation requirements

**Error handling:**
- Block progression for criminal ineligibility
- Warning for marital status mismatch
- Clear validation messages for violations

## Detailed Game Plan

### Step 1: Entity Analysis
1. Design violation tracking schema
2. Determine driver status management approach
3. Create reference tables for SR-22, removal reasons
4. Plan household search result handling

### Step 2: Global Requirements Alignment
**Primary GRs to apply:**
- **GR-18**: Workflow Requirements (complex state management)
- **GR-20**: Application Business Logic (eligibility rules)
- **GR-04**: Validation & Data Handling (multi-field validation)
- **GR-53**: DCS Integration (household search)
- **GR-07**: Reusable Components (driver cards, modals)
- **GR-11**: Accessibility (complex form flows)

### Step 3: Backend Implementation (Section C)
1. **List Drivers API**
   - GET /api/v1/quotes/{id}/drivers
   - Return sectioned by status
   - Include search and pagination
   
2. **Add/Update Driver API**  
   - POST/PUT /api/v1/quotes/{id}/drivers
   - Complex validation logic
   - Trigger household re-search
   
3. **Driver Eligibility Check**
   - POST /api/v1/drivers/{id}/eligibility
   - Criminal background integration
   - Real-time validation
   
4. **Violation Management**
   - CRUD operations for violations
   - Validation against DMV data

### Step 4: Database Schema (Section E)
1. **New Tables:**
   - `violation` with type, date, description
   - `violation_type` reference table
   - `map_driver_violation` associations
   - SR-22 and removal reason tables
   
2. **Modifications:**
   - Add driver_status to map_quote_driver
   - Add removal_reason_id
   - Add employment fields to driver

3. **Indexes:**
   - Driver search optimization
   - Household association queries
   - Status-based filtering

### Step 5: Quality Validation
1. Test marital status rule enforcement
2. Verify criminal eligibility blocking
3. Validate household search integration
4. Test all status transitions
5. Confirm pagination and search

## Risk Considerations

### Technical Risks
1. **Household Search Performance**: Large households may return many drivers
2. **Complex Business Rules**: Marital status and eligibility rules interaction
3. **Data Synchronization**: Keeping household search results current

### Business Risks
1. **Missing Required Drivers**: Household search may not find all
2. **False Criminal Matches**: Name-based matching issues
3. **Violation Data Quality**: DMV data may be incomplete

### Mitigation Strategies
- Implement robust deduplication logic
- Clear UI indicators for rule violations
- Manual override options with audit trail
- Progressive loading for large driver lists

## Dependencies
- Complete marital status rule specifications
- Criminal eligibility check service/rules
- DMV violation lookup capability
- Household search algorithm details
- Full list of removal reasons

## Recommendation

**Proceed with implementation** after clarifying:
1. Criminal eligibility check specifics
2. Complete business rules for marital status
3. Violation verification approach
4. Primary driver assignment logic

This is a complex requirement with significant business logic. Key considerations:

1. **Phased Approach**: Consider implementing basic driver management first, then add violations/SR-22
2. **Rule Engine**: Complex business rules may benefit from a rule engine pattern
3. **Performance**: Household search and eligibility checks need optimization
4. **User Experience**: Clear status indicators and validation messages critical

The three-section approach (Included/Excluded/Removed) provides good organization but requires careful state management.

## Next Steps Upon Approval
1. Design complete violation tracking system
2. Clarify all business rule edge cases
3. Create detailed UI mockups for driver cards
4. Plan integration test scenarios
5. Document state transition matrix