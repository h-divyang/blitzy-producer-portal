# Vehicle Coverage Type Rate Factor Interpretation
## Aguila Dorada Texas Personal Auto Program

### Overview
This document provides the complete factual interpretation of the Vehicle Coverage Type rate factor for the Aguila Dorada Texas Personal Auto insurance program. This serves as the definitive reference for understanding how lienholder status and vehicle count impact premium calculations across all coverage types.

## 1. Factor Identification

### Factor Details
- **Factor Name**: Vehicle Coverage Type
- **Company**: Old American County Mutual
- **Managing General Agency**: Aguila Dorada
- **Program**: Private Passenger Auto
- **Effective Date**: New Business 07/15/2025, Renewal 08/15/2025
- **System**: New System Implementation

## 2. Coverage Type Factor Structure

### Factor Dimensions
**Two-Dimensional Factor Matrix:**
- **Lienholder Status**: Yes, No, LO (Liability Only), Non-Owner
- **Vehicle Count**: 1, 2, 3, 4+ vehicles
- **Coverage Application**: All coverage types (BI, PD, UMBI, UMPD, MED, PIP, OTC, COL)

### Lienholder Status Definitions
**Coverage Categories:**
- **Yes**: Vehicles with lienholder requiring comprehensive and collision coverage
- **No**: Vehicles without lienholder but with comprehensive and collision coverage
- **LO**: Liability Only coverage (no comprehensive/collision)
- **Non-Owner**: Non-owner policy coverage

## 3. Single Vehicle Factor Structure

### One Vehicle on Policy
**Lienholder Status vs. All Coverage Factors:**

#### Yes (With Lienholder) - 1 Vehicle
- **All Coverages Factor**: 1.000 (base rate)
- **Application**: Standard rate for financed vehicle with required physical damage coverage
- **Risk Profile**: Baseline risk due to lender requirements

#### No (Without Lienholder) - 1 Vehicle
- **All Coverages Factor**: 1.300 (30% surcharge)
- **Application**: Higher rate for unfinanced vehicle with comprehensive/collision
- **Risk Profile**: Higher risk due to optional physical damage coverage selection

#### LO (Liability Only) - 1 Vehicle
- **All Coverages Factor**: 0.800 (20% discount)
- **Application**: Discount for liability-only coverage
- **Risk Profile**: Lower exposure due to limited coverage scope

## 4. Multiple Vehicle Factor Structure

### Two Vehicles on Policy
**Lienholder Status Impact:**

#### Yes (With Lienholder) - 2 Vehicles
- **All Coverages Factor**: 1.000 (base rate)
- **Application**: Standard rate maintained for multiple financed vehicles
- **Risk Distribution**: Risk spread across multiple vehicles with lender oversight

#### No (Without Lienholder) - 2 Vehicles
- **All Coverages Factor**: 1.100 (10% surcharge)
- **Application**: Reduced surcharge for multiple unfinanced vehicles
- **Risk Improvement**: Lower surcharge than single vehicle due to fleet effect

#### LO (Liability Only) - 2 Vehicles
- **All Coverages Factor**: 0.800 (20% discount)
- **Application**: Same discount as single vehicle for liability-only
- **Consistent Pricing**: Liability-only discount remains constant regardless of vehicle count

### Three Vehicles on Policy
**Lienholder Status Impact:**

#### Yes (With Lienholder) - 3 Vehicles
- **All Coverages Factor**: 1.000 (base rate)
- **Application**: Base rate continues for three financed vehicles
- **Risk Management**: Lender requirements provide risk control

#### No (Without Lienholder) - 3 Vehicles
- **All Coverages Factor**: 1.100 (10% surcharge)
- **Application**: Same 10% surcharge as two-vehicle policies
- **Risk Plateau**: Surcharge stabilizes at multiple vehicle level

#### LO (Liability Only) - 3 Vehicles
- **All Coverages Factor**: 0.800 (20% discount)
- **Application**: Consistent liability-only discount
- **Coverage Consistency**: Same discount structure across vehicle counts

### Four or More Vehicles on Policy
**Lienholder Status Impact:**

#### Yes (With Lienholder) - 4+ Vehicles
- **All Coverages Factor**: 1.000 (base rate)
- **Application**: Base rate for large fleet with lender oversight
- **Fleet Benefits**: Lender requirements provide risk management

#### No (Without Lienholder) - 4+ Vehicles
- **All Coverages Factor**: 1.100 (10% surcharge)
- **Application**: Consistent 10% surcharge for large unfinanced fleet
- **Risk Management**: Surcharge recognizes increased risk without lender oversight

#### LO (Liability Only) - 4+ Vehicles
- **All Coverages Factor**: 0.800 (20% discount)
- **Application**: Maintained discount for large liability-only fleet
- **Risk Recognition**: Lower exposure due to limited coverage maintains discount

## 5. Non-Owner Policy Structure

### Non-Owner Coverage
**Special Coverage Type:**
- **All Coverages Factor**: 1.000 (base rate)
- **Application**: Base rate for non-owner policies
- **Coverage Nature**: No vehicle ownership, different risk profile
- **Risk Assessment**: Standard rating for non-owner liability coverage

## 6. Business Rules and Risk Logic

### Lienholder Impact on Risk
**Risk Assessment Factors:**
- **Lender Requirements**: Lienholders require comprehensive/collision coverage
- **Maintenance Standards**: Lenders often require higher maintenance standards
- **Risk Mitigation**: Lender oversight reduces certain risk factors
- **Coverage Consistency**: Required coverage eliminates coverage gaps

### Vehicle Count Impact on Risk
**Fleet Effect Analysis:**
- **Single Vehicle Risk**: Highest surcharge for unfinanced single vehicle (30%)
- **Multiple Vehicle Benefits**: Reduced surcharge for multiple vehicles (10%)
- **Risk Distribution**: Multiple vehicles spread risk exposure
- **Customer Stability**: Multiple vehicle customers show greater stability

### Liability Only Discount Logic
**Coverage Limitation Benefits:**
- **Reduced Exposure**: No physical damage coverage reduces claim exposure
- **Lower Severity**: Liability claims typically different severity profile
- **Risk Control**: Customer accepts higher retention of physical damage risk
- **Administrative Efficiency**: Simpler coverage structure

## 7. Special Business Rule

### Lienholder Rate Continuation
**OSIS System Note:**
- **Rule**: "When the lienholder drops off continue to use the Lienholder rate"
- **Application**: Customer continues to receive lienholder rate even after loan payoff
- **Benefit**: Rate reduction maintained for customer benefit
- **Implementation**: System must track historical lienholder status

### Rate Continuation Logic
**Business Rationale:**
- **Customer Retention**: Favorable rating continues after loan payoff
- **Risk Recognition**: Customer behavior established during lienholder period
- **Competitive Advantage**: Rate benefit encourages policy retention
- **System Implementation**: Requires tracking of historical lienholder status

## 8. Premium Impact Examples

### Sample Premium Calculations
**Base Premium (Before Coverage Type Factor): $1,200**

#### Single Vehicle Examples
**With Lienholder:**
- **Premium**: $1,200 × 1.000 = $1,200
- **Impact**: Base rate, no adjustment

**Without Lienholder:**
- **Premium**: $1,200 × 1.300 = $1,560
- **Surcharge**: $360 (30% increase)

**Liability Only:**
- **Premium**: $1,200 × 0.800 = $960
- **Discount**: $240 (20% reduction)

#### Two Vehicle Examples
**With Lienholder:**
- **Premium**: $1,200 × 1.000 = $1,200
- **Impact**: Base rate maintained

**Without Lienholder:**
- **Premium**: $1,200 × 1.100 = $1,320
- **Surcharge**: $120 (10% increase)

**Liability Only:**
- **Premium**: $1,200 × 0.800 = $960
- **Discount**: $240 (20% reduction, same as single vehicle)

## 9. System Implementation

### Data Requirements
- **Lienholder Status**: Tracking of current and historical lienholder information
- **Vehicle Count**: Accurate count of vehicles on policy
- **Coverage Elections**: Tracking of comprehensive/collision coverage elections
- **Rate Continuation**: System capability to maintain lienholder rates after payoff

### Processing Requirements
1. **Lienholder Verification**: Determine current lienholder status for each vehicle
2. **Vehicle Count Calculation**: Count total vehicles on policy
3. **Coverage Analysis**: Assess comprehensive/collision coverage elections
4. **Factor Application**: Apply appropriate coverage type factors
5. **Rate Continuation**: Implement lienholder rate continuation rule

## 10. Quality Controls

### Validation Procedures
- **Lienholder Accuracy**: Verify accurate lienholder status determination
- **Vehicle Count Validation**: Confirm accurate vehicle count calculation
- **Coverage Consistency**: Ensure coverage elections properly categorized
- **Factor Application**: Validate correct coverage type factors applied
- **Rate Continuation**: Verify proper application of lienholder rate continuation

### Exception Handling
- **Missing Lienholder Data**: Process for handling unclear lienholder status
- **Coverage Conflicts**: Resolution of coverage election inconsistencies
- **System Errors**: Error handling for coverage type factor calculation failures
- **Rate Continuation Issues**: Procedures for lienholder rate continuation problems

## 11. Customer Communication

### Coverage Type Benefits
- **Lienholder Advantages**: Explanation of lienholder rate benefits
- **Multi-Vehicle Discounts**: Communication of fleet pricing advantages
- **Liability Only Savings**: Clear explanation of liability-only discount benefits
- **Rate Continuation**: Notification of continued lienholder rate benefits

### Customer Education
- **Coverage Decisions**: Impact of coverage elections on pricing
- **Vehicle Count Benefits**: Advantages of multiple vehicle policies
- **Lienholder Requirements**: Understanding of lender coverage requirements
- **Rate Optimization**: Guidance on optimizing coverage type factors

## Cross-References
- **Lienholder Requirements**: See underwriting guidelines for lienholder coverage requirements
- **Coverage Elections**: See coverage documentation for comprehensive/collision options
- **Multi-Vehicle Policies**: See policy structure documentation for fleet policies
- **Rate Continuation**: See system procedures for maintaining historical rate benefits

## Validation Standards
This document serves as the authoritative source for:
- **Coverage Type Factors**: Definitive lienholder and vehicle count-based rating factors
- **Application Rules**: Coverage type factor selection and application methodology
- **Business Rules**: Lienholder rate continuation and coverage type determination procedures
- **System Requirements**: Technical coverage type factor implementation specifications

## Document Maintenance
- **Updates**: Changes to coverage type factors require document updates
- **Version Control**: Maintain version history for rating consistency
- **Approval**: All coverage type factor changes require actuarial approval
- **Distribution**: Updates communicated to all underwriting and customer service stakeholders