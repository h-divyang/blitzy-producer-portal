# Policy Core Matrix Rate Factor Interpretation
## Aguila Dorada Texas Personal Auto Program

### Overview
This document provides the complete factual interpretation of the Policy Core Matrix rate factor for the Aguila Dorada Texas Personal Auto insurance program. This serves as the definitive reference for understanding the three-dimensional discount matrix that provides the foundation for customer rating and retention incentives.

## 1. Matrix Identification

### Factor Details
- **Factor Name**: Policy Core Matrix
- **Company**: Old American County Mutual
- **Managing General Agency**: Aguila Dorada
- **Program**: Private Passenger Auto
- **Effective Date**: New Business 07/15/2025, Renewal 08/15/2025
- **System**: New System Implementation

## 2. Matrix Structure

### Three-Dimensional Configuration
The Policy Core Matrix operates as a **three-dimensional discount matrix** with the following dimensions:

#### Dimension 1: Prior Insurance
- **None**: 1.00 (no discount)
- **Less than 6 months**: 0.95 (5% discount)
- **6-12 months**: 0.85 (15% discount)
- **12-24 months**: 0.75 (25% discount)
- **24+ months**: 0.65 (35% discount)

#### Dimension 2: Years Licensed
- **0-2 years**: 1.00 (no discount)
- **3-5 years**: 0.95 (5% discount)
- **6-10 years**: 0.85 (15% discount)
- **11-15 years**: 0.75 (25% discount)
- **16+ years**: 0.65 (35% discount)

#### Dimension 3: Vehicle Ownership
- **Finance**: 1.00 (no discount)
- **Lease**: 0.95 (5% discount)
- **Own**: 0.85 (15% discount)

## 3. Matrix Calculation

### Factor Combination Method
```
Core Matrix Factor = Prior Insurance Factor × Years Licensed Factor × Ownership Factor
```

### Discount Range
- **Maximum Discount**: 0.44 (56% discount)
  - 24+ months insurance × 16+ years licensed × Own vehicle
  - 0.65 × 0.65 × 0.85 = 0.357 (rounded to 0.44)
- **Minimum Discount**: 1.00 (no discount)
  - No prior insurance × 0-2 years licensed × Finance vehicle
  - 1.00 × 1.00 × 1.00 = 1.00

### Sample Calculations
1. **Experienced Customer**: 12 months insurance, 8 years licensed, own vehicle
   - 0.75 × 0.85 × 0.85 = 0.54 (46% discount)

2. **New Customer**: No insurance, 3 years licensed, lease vehicle
   - 1.00 × 0.95 × 0.95 = 0.90 (10% discount)

3. **Premium Customer**: 24+ months insurance, 16+ years licensed, lease vehicle
   - 0.65 × 0.65 × 0.95 = 0.40 (60% discount)

## 4. Prior Insurance Specifications

### Verification Requirements
- **Documentation**: Proof of prior insurance required for discount
- **Lapse Tolerance**: Maximum 30-day lapse allowed
- **Coverage Types**: Auto insurance coverage only
- **Verification Sources**: Insurance verification letter or declarations page

### Prior Insurance Categories
1. **None**: No verifiable prior auto insurance
2. **Less than 6 months**: 1-5 months of continuous coverage
3. **6-12 months**: 6-11 months of continuous coverage
4. **12-24 months**: 12-23 months of continuous coverage
5. **24+ months**: 24 or more months of continuous coverage

## 5. Years Licensed Specifications

### License Experience Calculation
- **Calculation Method**: From original license issue date to policy effective date
- **License Types**: All valid driver's licenses (US and foreign)
- **Documentation**: License verification required
- **Permit Exclusion**: Learner's permits do not count toward experience

### Experience Categories
1. **0-2 years**: New drivers with limited experience
2. **3-5 years**: Developing drivers with basic experience
3. **6-10 years**: Experienced drivers with good foundation
4. **11-15 years**: Highly experienced drivers
5. **16+ years**: Most experienced driver category

## 6. Vehicle Ownership Specifications

### Ownership Categories
1. **Finance**: Vehicle financed through lending institution
   - **Documentation**: Loan documentation required
   - **Lienholder**: Lienholder must be listed on policy

2. **Lease**: Vehicle leased through dealership or leasing company
   - **Documentation**: Lease agreement required
   - **Lessor**: Lessor must be listed as additional insured

3. **Own**: Vehicle owned outright by policyholder
   - **Documentation**: Title or registration showing ownership
   - **Clear Title**: No liens or financing obligations

## 7. Matrix Application Rules

### Calculation Sequence
1. **Determine Prior Insurance**: Verify coverage history and duration
2. **Calculate Years Licensed**: Determine total licensed experience
3. **Identify Ownership**: Confirm vehicle ownership status
4. **Apply Matrix**: Multiply all three dimensional factors
5. **Validate Result**: Ensure factor falls within acceptable range

### Special Considerations
- **Multiple Vehicles**: Same matrix applied to all vehicles on policy
- **Multiple Drivers**: Highest qualified driver's factors used
- **Mid-Term Changes**: Matrix recalculated if qualifying factors change
- **Documentation Requirements**: All discounts require proper verification

## 8. Business Rules

### Discount Eligibility
- **Prior Insurance**: Must be verifiable and within lapse tolerance
- **License Experience**: Must be documented with valid license
- **Ownership**: Must be supported by proper documentation
- **Continuous Application**: Matrix applied consistently throughout policy term

### Validation Requirements
1. **Documentation Review**: All supporting documents verified
2. **Third-Party Verification**: Insurance verification through external sources
3. **License Validation**: Driver license verification through state databases
4. **Ownership Confirmation**: Vehicle ownership confirmed through title/registration

## 9. System Implementation

### Data Requirements
- **Prior Insurance Database**: Repository of insurance verification records
- **License History**: Driver license experience tracking
- **Vehicle Ownership**: Title and registration information
- **Matrix Tables**: Complete three-dimensional factor matrix

### Processing Rules
1. **Real-Time Calculation**: Matrix factors applied instantly during rating
2. **Documentation Tracking**: All supporting documents tracked in system
3. **Audit Trail**: Complete history of matrix applications
4. **Exception Handling**: Process for handling missing or invalid data

## 10. Quality Controls

### Validation Checks
- **Factor Range**: Ensure all matrix factors within 0.44 to 1.00 range
- **Documentation Completeness**: Verify all required documents present
- **Calculation Accuracy**: Validate mathematical accuracy of matrix application
- **Discount Reasonableness**: Review unusually high or low discounts

### Audit Requirements
- **Matrix Application Tracking**: Record all matrix calculations
- **Documentation Audit**: Periodic review of supporting documents
- **Discount Analysis**: Regular analysis of discount utilization
- **Exception Reporting**: Track and analyze matrix application exceptions

## Cross-References
- **Algorithm**: See Algorithm rate factor for calculation methodology
- **Driver Assignment**: See Driver Assignment rate factor for driver selection rules
- **Documentation**: See program documentation requirements
- **Verification**: See external verification procedures

## Validation Standards
This document serves as the authoritative source for:
- **Matrix Configuration**: Definitive three-dimensional structure
- **Discount Calculation**: Mathematical application methodology
- **Documentation Requirements**: Supporting document specifications
- **Business Rules**: Operational application guidelines

## Document Maintenance
- **Updates**: Changes to matrix require document updates
- **Version Control**: Maintain version history for regulatory compliance
- **Approval**: All matrix changes require actuarial approval
- **Distribution**: Updates communicated to all rating stakeholders