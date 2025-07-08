# Vehicle Age Rate Factor Interpretation
## Aguila Dorada Texas Personal Auto Program

### Overview
This document provides the complete factual interpretation of the Vehicle Age rate factor for the Aguila Dorada Texas Personal Auto insurance program. This serves as the definitive reference for understanding how vehicle age impacts premium calculations across all coverage types.

## 1. Factor Identification

### Factor Details
- **Factor Name**: Vehicle Age
- **Company**: Old American County Mutual
- **Managing General Agency**: Aguila Dorada
- **Program**: Private Passenger Auto
- **Effective Date**: New Business 07/15/2025, Renewal 08/15/2025
- **System**: New System Implementation

## 2. Age Classification Structure

### Age Categories and Factors
**All Coverages (BI, PD, UMBI, UMPD, MED, PIP, COMP, COLL):**
- **0 years old**: 1.10 (10% surcharge)
- **1 year old**: 1.05 (5% surcharge)
- **2 years old**: 1.00 (base rate)
- **3 years old**: 0.95 (5% discount)
- **4 years old**: 0.95 (5% discount)
- **5 years old**: 0.95 (5% discount)
- **6 years old**: 0.90 (10% discount)
- **7 years old**: 0.90 (10% discount)
- **8 years old**: 0.90 (10% discount)
- **9 years old**: 0.95 (5% discount)
- **10 years old**: 1.00 (base rate)
- **11 years old**: 1.05 (5% surcharge)
- **12 years old**: 1.05 (5% surcharge)
- **13 years old**: 1.10 (10% surcharge)
- **14 years old**: 1.15 (15% surcharge)
- **15 years old**: 1.20 (20% surcharge)

## 3. Age Determination Methodology

### Age Calculation Rules
- **Model Year Based**: Vehicle age determined by model year
- **Calendar Year Reference**: Age calculated as current year minus model year
- **Policy Effective Date**: Age determined as of policy effective date
- **Mid-Term Stability**: Age does not change during policy term

### Age Verification Sources
1. **Vehicle Identification Number (VIN)**: Primary source for model year determination
2. **Vehicle Registration**: Secondary verification source
3. **Title Documentation**: Supporting documentation for age verification
4. **Manufacturer Data**: Reference databases for VIN decoding

## 4. Risk Assessment by Age Groups

### New Vehicles (0-1 Years)
**Risk Characteristics:**
- Higher theft risk due to desirability
- Higher repair costs due to newer technology
- Lower mechanical failure rates
- **Rating Impact**: 5-10% surcharge

### Optimal Age Vehicles (2-8 Years)
**Risk Characteristics:**
- Balanced risk profile
- Reasonable repair costs
- Good safety features
- Optimal value retention
- **Rating Impact**: 10% discount to base rate

### Mature Vehicles (9-12 Years)
**Risk Characteristics:**
- Increasing mechanical issues
- Higher maintenance costs
- Declining safety technology
- **Rating Impact**: Base rate to 5% surcharge

### Older Vehicles (13+ Years)
**Risk Characteristics:**
- Higher mechanical failure rates
- Parts availability issues
- Outdated safety features
- Lower theft attractiveness
- **Rating Impact**: 10-20% surcharge

## 5. Coverage-Specific Applications

### Liability Coverage (BI/PD)
- **Age Impact**: Vehicle age affects liability risk through mechanical reliability
- **Factor Range**: 0.90 to 1.20
- **Risk Basis**: Older vehicles more likely to have mechanical failures causing accidents

### Uninsured Motorist (UMBI/UMPD)
- **Age Impact**: Similar to liability coverage
- **Factor Range**: 0.90 to 1.20
- **Risk Basis**: Consistent with liability risk patterns

### Physical Damage (COMP/COLL)
- **Age Impact**: Direct relationship between age and repair/replacement costs
- **Factor Range**: 0.90 to 1.20
- **Risk Basis**: Newer vehicles cost more to repair, older vehicles have higher claim frequency

### Medical Coverage (MED/PIP)
- **Age Impact**: Vehicle age affects injury severity through safety features
- **Factor Range**: 0.90 to 1.20
- **Risk Basis**: Newer vehicles have better safety features reducing injury severity

## 6. Business Rules

### Age Assignment Rules
- **Single Age**: All coverages use same vehicle age factor
- **Model Year Basis**: Age based on manufacturer's designated model year
- **Documentation Required**: VIN or registration required for age verification
- **System Lookup**: Automatic age determination from VIN decoding

### Eligibility Restrictions
**Age-Based Restrictions:**
- **Over 50 years old**: Not acceptable for any coverage
- **Over 20 years old**: Not acceptable for Comprehensive/Collision or UMPD
- **Classic/Antique**: Special consideration for vehicles over 25 years old

### Documentation Requirements
1. **VIN Verification**: VIN must be provided and verified
2. **Registration**: Current registration showing model year
3. **Title**: Title documentation for ownership verification
4. **Photos**: Vehicle photos may be required for age verification

## 7. System Implementation

### Data Requirements
- **VIN Database**: Comprehensive VIN decoding capability
- **Age Factors**: Complete matrix of age-based factors
- **Validation Rules**: Age calculation and verification logic
- **Exception Handling**: Process for VIN decoding failures

### Processing Requirements
1. **Automatic Age Calculation**: System calculates age from VIN or model year
2. **Factor Application**: Automatic application of age-appropriate factors
3. **Validation Checks**: Verify age falls within acceptable ranges
4. **Documentation Tracking**: Track all age verification documents

## 8. Quality Controls

### Validation Procedures
- **VIN Accuracy**: Verify VIN correctly decoded for model year
- **Age Calculation**: Validate mathematical accuracy of age calculation
- **Factor Application**: Confirm correct age factors applied
- **Eligibility Verification**: Ensure vehicle age meets program standards

### Exception Handling
- **VIN Decode Failures**: Process for manual age determination
- **Conflicting Information**: Resolution of age discrepancies
- **Missing Documentation**: Process for obtaining age verification
- **Classic Vehicle Exceptions**: Special handling for antique vehicles

## 9. Rate Impact Analysis

### Low-Risk Age Groups
**Ages 3-8 (Optimal Range):**
- Factor range: 0.90 to 0.95
- Premium impact: 5-10% discount
- Characteristics: Best balance of safety, reliability, and repair costs

### High-Risk Age Groups
**Ages 0-1 (New Vehicles):**
- Factor range: 1.05 to 1.10
- Premium impact: 5-10% surcharge
- Characteristics: High theft risk and repair costs

**Ages 13+ (Older Vehicles):**
- Factor range: 1.10 to 1.20
- Premium impact: 10-20% surcharge
- Characteristics: Mechanical reliability and safety concerns

### Age Transition Points
- **Age 2**: Base rate reference point
- **Ages 6-8**: Maximum discount period
- **Age 10**: Return to base rate
- **Age 13+**: Progressive surcharge increase

## 10. Market Considerations

### Vehicle Depreciation Impact
- **New Vehicle Premium**: Higher premiums for newest vehicles
- **Sweet Spot**: Ages 3-8 provide optimal premium rates
- **Older Vehicle Penalty**: Increasing premiums for vehicles over 13 years
- **Eligibility Cutoff**: Hard stop at 20 years for physical damage

### Customer Communication
- **Age Impact Disclosure**: Customers informed of age-based rating
- **Optimal Replacement Timing**: Guidance on vehicle replacement timing
- **Coverage Limitations**: Notification of age-based coverage restrictions
- **Premium Projections**: Multi-year premium impact projections

## Cross-References
- **Algorithm**: See Algorithm rate factor for calculation methodology
- **Vehicle Eligibility**: See program eligibility requirements for age restrictions
- **VIN Decoding**: See system documentation for VIN processing
- **Vehicle Use**: See Vehicle Use rate factor for usage-based factors

## Validation Standards
This document serves as the authoritative source for:
- **Age Factors**: Definitive vehicle age rating factors
- **Age Determination**: Methodology for vehicle age calculation
- **Eligibility Rules**: Age-based coverage restrictions
- **System Requirements**: Technical implementation specifications

## Document Maintenance
- **Updates**: Changes to age factors require document updates
- **Version Control**: Maintain version history for actuarial review
- **Approval**: All age factor changes require actuarial approval
- **Distribution**: Updates communicated to all rating stakeholders