# IP269-New-Quote-Step-3-Vehicles - Updated Approach

## Cross-Requirement Decisions Incorporated

### ✅ **From Primary Insured Step:**
- **Quote Creation**: Quote entity already created in Step 1 (immediate creation)
- **DCS Integration**: DCS search patterns established (household drivers completed)
- **Producer Context**: Producer always attached to quote
- **Data Persistence**: Store and propagate results across models (no caching needed)
- **External Integration**: DCS-first approach established

### ✅ **From Named Insured Step:**
- **Data Review Pattern**: Focus on reviewing and enriching DCS-populated data
- **Read-only vs Manual**: Clear distinction between DCS data and manual entry
- **Verification Workflow**: Additional verification triggers established
- **Business Rules**: Cross-field validation patterns implemented

### ✅ **From Drivers Step:**
- **Driver Selection Complete**: Driver include/exclude status assigned
- **Driver Data Available**: Full driver roster for vehicle assignment
- **Manual Addition Patterns**: Established workflow for data not found in DCS
- **Business Rule Validation**: Marriage rule and criminal eligibility patterns

### 🔄 **Impact on Vehicles Step:**
- This step builds on **completed driver selection** from Step 2
- **Vehicle-to-driver assignment** leveraging existing driver data
- **DCS vehicle lookup** following established integration patterns
- **Owner verification** against existing driver roster
- **Enhanced data collection** for vehicles not found via household lookup

## Questions Requiring Clarification

### Business Logic Questions  
1. **Vehicle-Driver Assignment**:
   - How should primary driver assignment work with the completed driver roster from Step 2?
   - Can excluded drivers be assigned as vehicle owners?
   - What happens when vehicle owner is not in the driver roster?
2. **DCS Vehicle Integration**:
   - Should DCS vehicle lookup use the same household address patterns as Step 1 drivers?
   - How does vehicle lookup integrate with the established DCS workflow?
3. **Usage Types**:
   - What are all available vehicle usage types?
   - Do usage types affect coverage options or premiums?
   - Are certain usage types restricted by program?
4. **Garaging Address**:
   - Can garaging address differ from primary insured address?
   - Multiple garaging addresses for fleet policies?
   - Validation requirements for garaging ZIP?
5. **Vehicle Matching**:
   - When searching by Year/Make/Model, how close must the match be?
   - How are vehicle variants/trims handled?
6. **Owner Addition Workflow**:
   - When vehicle owner not found in driver roster, should this trigger Step 2 driver addition workflow?

### Technical Questions
1. **DCS Vehicle API Integration**:
   - Should DCS vehicle lookup follow the same patterns established in Step 1?
   - How does vehicle API integrate with existing DCS authentication?
2. **VIN Decoding Service**:
   - Which service for VIN decoding (DCS, third-party)?
   - What data points are returned?
   - Handling of invalid/undecodable VINs?
3. **License Plate Validation**:
   - Format validation by state?
   - Uniqueness checking?
4. **Performance**:
   - Expected volume of vehicles per household?
   - Pagination for large vehicle lists?
5. **Driver Integration**:
   - How to efficiently assign vehicles to drivers from Step 2 data?

### Edge Cases and Validation Rules
1. What if VIN decode conflicts with Year/Make/Model entry?
2. Maximum vehicles allowed per policy?
3. Commercial vs personal vehicle handling?
4. Salvage/rebuilt title restrictions?
5. Out-of-state vehicle registration?

## Updated Implementation Approach

### Overview (Based on Cross-Requirement Decisions)
Implement vehicle management that builds on completed driver selection (Step 2) and established DCS integration patterns (Step 1). The system leverages existing driver data for vehicle assignment and follows proven data persistence patterns.

### Entity Strategy
**Entities to reuse from infrastructure:**
- `vehicle` - Core vehicle information (existing model with modal typo)
- `address` - For garaging address  
- `map_quote_vehicle` - Quote-vehicle association
- All vehicle reference tables (make, model, usage_type)

**New entities needed:**
- `vehicle_owner` - Track registered owners  
- `map_vehicle_owner` - Vehicle-owner relationships
- `map_vehicle_driver` - Vehicle-to-driver assignments (primary driver)
- `vehicle_lookup_result` - Cache DCS lookup results

**Driver integration:**
- Link vehicles to drivers from Step 2 completed roster
- Vehicle ownership verification against existing drivers
- Primary driver assignment from available driver pool

### Integration Approach (Based on Established Patterns)
**External services building on Step 1:**
1. **DCS Vehicle Lookup** (GR-53)
   - Use same household address from Step 1 driver search
   - Follow established DCS authentication patterns
   - Return VIN, year, make, model with same data persistence approach
   
2. **VIN Decoder Service** 
   - Full vehicle specifications using Universal Entity Management (GR-52)
   - Validate VIN format
   - Cache results following no-caching clarification from stakeholders
   
3. **Vehicle History Service** (Optional)
   - Accident/claim history
   - Title status  
   - Ownership transfers

**Integration patterns consistent with previous steps:**
- DCS household lookup following Step 1 patterns
- Direct entity storage (no caching per stakeholder decision)
- Manual addition workflow when DCS doesn't find all vehicles

### Workflow Considerations (Cross-Step Integration)
**State transitions building on previous steps:**
1. From DRIVERS_COMPLETE (Step 2 completed with driver roster)
2. DCS vehicle lookup using Step 1 address data
3. Vehicle selection and driver assignment from Step 2 roster
4. Owner validation against existing drivers
5. Manual vehicle addition when DCS incomplete
6. Update to VEHICLES_COMPLETE

**Validation points:**
- VIN format and checksum
- Owner verification against Step 2 driver roster
- Primary driver assignment from available drivers
- Usage type required
- Garaging address validation
- License plate format by state

**Error handling consistent with previous steps:**
- Clear messages for invalid VINs
- Owner addition triggers Step 2-style driver addition workflow
- Graceful handling of DCS lookup failures with manual fallback

## Detailed Game Plan

### Step 1: Entity Analysis
1. Design vehicle owner tracking
2. Determine garaging address approach
3. Plan vehicle lookup caching
4. Review existing vehicle schema adequacy

### Step 2: Global Requirements Alignment
**Primary GRs to apply:**
- **GR-53**: DCS Integration (vehicle lookup)
- **GR-52**: Universal Entity Management (VIN decoder)
- **GR-04**: Validation & Data Handling (VIN/plate validation)
- **GR-18**: Workflow Requirements (owner addition flow)
- **GR-07**: Reusable Components (vehicle cards, search)
- **GR-20**: Application Business Logic (usage rules)

### Step 3: Backend Implementation (Section C)
1. **Vehicle Lookup API Building on Step 1**
   - GET /api/v1/quotes/{id}/vehicle-lookup
   - Use household address from Step 1 DCS data
   - Return vehicle list with DCS integration patterns
   
2. **VIN Decoder API**
   - POST /api/v1/vehicles/decode-vin
   - Full vehicle specifications
   - Direct storage (no caching per stakeholder decision)
   
3. **Add Vehicle APIs with Driver Integration**
   - POST /api/v1/quotes/{id}/vehicles
   - Support VIN and YMM entry
   - Validate ownership against Step 2 driver roster
   
4. **Driver Assignment Management**
   - PUT /api/v1/quotes/{id}/vehicles/{vehicleId}/driver
   - Assign primary driver from Step 2 roster
   - Trigger driver addition workflow if owner not found

### Step 4: Database Schema (Section E)
1. **New Tables:**
   - `vehicle_owner` with name, relationship  
   - `map_vehicle_owner` associations
   - `map_vehicle_driver` for primary driver assignments (links to Step 2 drivers)
   - `vehicle_lookup_result` for DCS results (direct storage per stakeholder)
   
2. **Modifications:**
   - Fix `modal` typo to `model` in existing vehicle table
   - Add usage_type_id to map_quote_vehicle
   - Add garaging_address_id
   - Ensure vehicle has all decode fields

3. **Driver Integration:**
   - Link vehicle ownership to Step 2 driver roster
   - Primary driver assignment from completed driver data
   - Owner validation against existing drivers

### Step 5: Quality Validation
1. Test VIN decoder accuracy
2. Verify owner validation flow
3. Test address-based lookup
4. Validate YMM search functionality
5. Confirm all usage types

## Risk Considerations

### Technical Risks
1. **VIN Decoder Reliability**: Multiple decoder sources recommended
2. **Address Matching Accuracy**: May miss vehicles at nearby addresses
3. **Owner Data Quality**: Registration data may be outdated

### Business Risks
1. **Missing Vehicles**: Household lookup incomplete
2. **Ownership Disputes**: Complex ownership scenarios
3. **Coverage Gaps**: Vehicles not properly added

### Mitigation Strategies
- Allow manual override for all lookups
- Clear ownership documentation requirements
- Comprehensive vehicle search options
- Audit trail for all additions

## Dependencies
- VIN decoder service selection
- Complete usage type definitions
- Owner validation rules
- Address matching algorithm
- License plate format rules by state

## Updated Recommendation

**Proceed with cross-step integrated vehicle management approach:**

1. **Build on completed driver selection** from Step 2 for vehicle assignments
2. **Use established DCS integration patterns** from Step 1 for vehicle lookup
3. **Implement driver-vehicle assignment workflow** leveraging existing driver data
4. **Create owner verification against driver roster** with addition workflow when needed
5. **Maintain data persistence pattern** established across previous steps

**Architecture Decision**: Leverage existing vehicle model (with typo fix) and Step 2 driver data while adding DCS vehicle lookup using Step 1 patterns. Integrate tightly with completed driver management for vehicle-driver assignments.

**Integration Strategy**: DCS household vehicle lookup using Step 1 address data, direct storage per stakeholder decision, manual addition workflow for vehicles not found, and seamless driver assignment from Step 2 roster.

**Cross-Step Consistency**: This step builds directly on quote creation (Step 1), DCS integration patterns (Step 1), and completed driver selection (Step 2) while preparing for coverage selection in later steps.

**Key architectural decisions based on cross-requirement analysis:**
1. **Driver Integration**: Use Step 2 completed driver roster for all vehicle assignments
2. **DCS Consistency**: Follow Step 1 DCS patterns for vehicle lookup
3. **Data Persistence**: Direct storage approach per stakeholder clarification
4. **Owner Workflow**: Trigger Step 2-style driver addition when owner not found

**Next Steps**: 
1. **Clarify vehicle-driver assignment workflow** with Step 2 integration
2. **Define DCS vehicle lookup scope** using Step 1 address patterns
3. **Specify owner addition workflow** integration with driver management
4. **Proceed to implementation** once vehicle management workflow clarified