# IP269-New-Quote-Step-3-Vehicles - Suggested Approach

## Questions Requiring Clarification

### Business Logic Questions
1. **Vehicle Lookup Scope**:
   - How does the system determine vehicles "associated with their address"?
     - DCS vehcile search
     - Aime/workspace/requirements/Documentation/Dcs/Household Vehicles API Documentation Version 2.3 conv.html
   - Does this include vehicles at nearby addresses or just exact matches?
     - See documentation above.
   - How recent must the association be?
     - Whatever is returned from DCS.
2. **Vehicle Ownership Requirements**:
   - Must all registered owners be on the policy?
     - Yes, if we can.
   - Can a vehicle have multiple owners?
     - Yes. they will be an an entity with an entity_type of vehicle owner.
   - What if owner information is unavailable?
     - We don't create the entity.
3. **Usage Types**:
   - What are all available vehicle usage types?
     - Will be defined in the Program Manager.
   - Do usage types affect coverage options or premiums?
     - Will be defined in the Program Manager.
   - Are certain usage types restricted by program?
     - Will be defined in the Program Manager.
4. **Garaging Address**:
   - Can garaging address differ from primary insured address?
     - Yes. Vehicle is rated on garaging address.
   - Multiple garaging addresses for fleet policies?
     - Not for now.
   - Validation requirements for garaging ZIP?
     - Will be defined in the Program Manager.
5. **Vehicle Matching**:
   - When searching by Year/Make/Model, how close must the match be?
     - We will be sending the vin and vehicle info to Verisk for VINMASTER Physical Damange Symbol information
     - This includes VIN, Trim, Year, Make, Model, Etc.
   - How are vehicle variants/trims handled?

### Technical Questions
1. **VIN Decoding Service**:
   - Which service for VIN decoding (DCS, third-party)?
     - Verisk VINMASTER
   - What data points are returned?
     - Not sure yet. Documentation pending.
   - Handling of invalid/undecodable VINs?
     - Not sure yet. Documentation pending.
2. **Vehicle History**:
   - Do we pull vehicle history (accidents, claims)?
     - Yes.
     - Some of this can be pulled from DCS vehicle history
     - We will also have Verisk LightSpeed info when documentation is supplied.
   - Integration with CARFAX or similar?
3. **License Plate Validation**:
   - Format validation by state?
     - Yes
   - Uniqueness checking?
     - Not now.
4. **Data Sources**:
   - Primary source for vehicle lookup by address?
     - DCS.
   - Fallback sources if primary fails?
     - None.
5. **Performance**:
   - Expected volume of vehicles per household?
     - Not sure.
   - Pagination for large vehicle lists?
     - Not sure.

### Edge Cases and Validation Rules
1. What if VIN decode conflicts with Year/Make/Model entry?
   2. Not sure yet. Documentation pending.
2. Maximum vehicles allowed per policy?
   3. Defined in the Program Manager
3. Commercial vs personal vehicle handling?
   4. personal.
4. Salvage/rebuilt title restrictions?
   5. Outlined in Program Manager
5. Out-of-state vehicle registration?
   6. skip.

## Suggested Implementation Approach

### Overview
Implement a flexible vehicle management system that prioritizes automated lookup while providing comprehensive manual entry options. The system must handle complex ownership scenarios and integrate with multiple data sources for vehicle information.

### Entity Strategy
**Entities to reuse:**
- `vehicle` - Core vehicle information
- `vehicle_registration` - License plate and registration
- `address` - For garaging address
- `map_quote_vehicle` - Quote-vehicle association
- All vehicle reference tables (make, model, usage_type)

**Potential new entities:**
- `vehicle_owner` - Track registered owners
- `map_vehicle_owner` - Vehicle-owner relationships
- `vehicle_lookup_result` - Cache lookup results
- `garaging_address` - If different from driver addresses

**Relationship modifications:**
- Enhance map_quote_vehicle with usage_type_id
- Add garaging_address_id to vehicle or map table
- Link vehicles to drivers (ownership)

### Integration Approach
**External services required:**
1. **DCS Vehicle Lookup** (GR-53)
   - Search vehicles by address association
   - Return VIN, year, make, model
   
2. **VIN Decoder Service**
   - Full vehicle specifications
   - Validate VIN format
   - Could be DCS or NHTSA
   
3. **Vehicle History Service** (Optional)
   - Accident/claim history
   - Title status
   - Ownership transfers

**Integration patterns to apply:**
- Batch processing for household vehicle lookup
- Real-time VIN decoding with caching
- Fallback chains for data sources

### Workflow Considerations
**State transitions:**
1. From DRIVERS_COMPLETE
2. Vehicle lookup and selection
3. Ownership validation
4. Update to VEHICLES_COMPLETE

**Validation points:**
- VIN format and checksum
- Owner must be on policy
- Usage type required
- Garaging address validation
- License plate format by state

**Error handling:**
- Clear messages for invalid VINs
- Owner addition workflow integration
- Graceful handling of lookup failures

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
1. **Vehicle Lookup API**
   - GET /api/v1/quotes/{id}/vehicle-lookup
   - Search by address association
   - Return vehicle list with details
   
2. **VIN Decoder API**
   - POST /api/v1/vehicles/decode-vin
   - Full vehicle specifications
   - Cache decoded results
   
3. **Add Vehicle APIs**
   - POST /api/v1/quotes/{id}/vehicles
   - Support VIN and YMM entry
   - Validate ownership
   
4. **Owner Management**
   - POST /api/v1/vehicles/{id}/owners
   - Trigger driver addition if needed
   - Link to policy workflow

### Step 4: Database Schema (Section E)
1. **New Tables:**
   - `vehicle_owner` with name, relationship
   - `map_vehicle_owner` associations
   - `vehicle_lookup_cache` for performance
   
2. **Modifications:**
   - Add usage_type_id to map_quote_vehicle
   - Add garaging_address_id
   - Ensure vehicle has all decode fields

3. **Indexes:**
   - VIN lookups
   - Address association queries
   - Owner relationship traversal

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

## Recommendation

**Proceed with implementation** with focus on:
1. Robust VIN decoding with fallbacks
2. Flexible ownership management
3. Clear UI for manual entry flows
4. Comprehensive validation

Key architectural decisions needed:
1. **Garaging Address**: Separate entity or use existing address?
2. **Owner Tracking**: How detailed should ownership records be?
3. **Lookup Caching**: How long to cache vehicle lookups?
4. **History Integration**: Include vehicle history in MVP?

The three-path approach (lookup/VIN/YMM) provides good flexibility but requires careful UI design to prevent confusion.

## Next Steps Upon Approval
1. Finalize VIN decoder service selection
2. Design complete owner management schema
3. Create UI mockups for three entry paths
4. Plan address association algorithm
5. Document validation rules matrix