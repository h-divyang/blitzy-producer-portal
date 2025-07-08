# Vehicle Mileage Ratio Rate Factor Interpretation
## Aguila Dorada Texas Personal Auto Program

### Overview
This document provides the complete factual interpretation of the Vehicle Mileage Ratio rate factor for the Aguila Dorada Texas Personal Auto insurance program. This serves as the definitive reference for understanding how vehicle mileage impacts premium calculations across all coverage types.

## 1. Factor Identification

### Factor Details
- **Factor Name**: Vehicle Mileage Ratio
- **Company**: Old American County Mutual
- **Managing General Agency**: Aguila Dorada
- **Program**: Private Passenger Auto
- **Effective Date**: New Business 07/15/2025, Renewal 08/15/2025
- **System**: New System Implementation

### Factor Application
- **Coverage Types Affected**: 
  - Bodily Injury (BI)
  - Property Damage (PD)
  - Other Than Collision (OTC)
  - Collision (COL)
- **Factor Type**: Multiplicative rating factor
- **Factor Range**: 0.650 to 5.696

## 2. Factor Structure and Definitions

### Mileage Ratio Calculation
The Vehicle Mileage Ratio is calculated using the following formula:
- **Mileage Ratio = Vehicle's Annual Mileage ÷ Average Mileage Base**
- **Rounded to**: Two decimal places

### Average Mileage Base by Vehicle Age
The Average Mileage Base varies by vehicle age:

| Vehicle Age | Average Mileage Base |
|-------------|---------------------|
| 1 year | 16,570 |
| 2 years | 16,470 |
| 3 years | 15,481 |
| 4 years | 15,258 |
| 5 years | 14,643 |
| 6 years | 14,062 |
| 7 years | 13,506 |
| 8 years | 13,141 |
| 9 years | 12,534 |
| 10 years | 12,001 |
| 11 years | 11,637 |
| 12 years | 11,279 |
| 13 years | 10,541 |
| 14 years | 10,027 |
| 15 years | 9,820 |
| 16 years | 9,633 |
| 17 years | 9,140 |
| 18 years | 8,882 |
| 19 years | 8,618 |
| 20 years | 8,324 |
| 21 years | 8,006 |
| 22 years | 7,833 |
| 23 years | 7,594 |
| 24 years | 7,343 |
| 25 years | 7,243 |
| 26 years | 7,143 |
| 27 years | 7,043 |
| 28 years | 6,943 |
| 29 years | 6,843 |
| 30 years | 6,743 |
| 31 years | 6,643 |
| 32 years | 6,613 |
| 33 years | 6,592 |
| 34 years | 6,542 |
| 35 years | 6,492 |
| 36 years | 6,442 |
| 37 years | 6,392 |
| 38 years | 6,342 |
| 39 years | 6,292 |
| 40+ years | 6,189 |

### Rate Factor Application
The calculated Mileage Ratio determines the rate factor applied to all coverages:

#### Low Mileage Factors (Ratio 0.00 - 0.99)
- **0.00**: 0.650 (35% discount)
- **0.01 - 0.49**: Factors range from 0.653 to 0.821
- **0.50 - 0.99**: Factors range from 0.825 to 0.996

#### Standard Mileage Factor (Ratio 1.00)
- **1.00**: 1.000 (base rate)

#### High Mileage Factors (Ratio 1.01 - 10.00)
- **1.01 - 2.00**: Factors range from 1.004 to 1.351
- **2.01 - 3.00**: Factors range from 1.354 to 1.728
- **3.01 - 4.00**: Factors range from 1.732 to 2.189
- **4.01 - 5.00**: Factors range from 2.195 to 2.777
- **5.01 - 6.00**: Factors range from 2.783 to 3.469
- **6.01 - 7.00**: Factors range from 3.477 to 4.291
- **7.01 - 8.00**: Factors range from 4.300 to 5.232
- **8.01 - 9.00**: Factors range from 5.243 to 5.531
- **9.01 - 10.00**: Factors range from 5.531 to 5.696

## 3. Business Rules

### Calculation Process
1. **Step 1**: Divide Vehicle's Annual mileage by the Average Mileage Base based on the Age of Vehicle
2. **Step 2**: Round to two decimal places
3. **Step 3**: Use this Mileage Ratio to determine the rating factor
4. **Step 4**: Only change factor when the mileage is adjusted (assumes annual mileage of vehicle changes proportionally to average mileage base, unless new mileage is run or provided by insured)

### Special Conditions
- **Vehicle Age 1**: Mileage Ratio shows as "NA" with factor 1.000
- **New Business**: Use Carfax data
- **Renewal**: Use Carfax to see if rates would improve

### Factor Stability
- Factor remains constant during policy term unless:
  - New mileage data is obtained (Carfax update)
  - Insured provides updated mileage information
  - Policy renewal triggers mileage review

## 4. Premium Impact Examples

### Low Mileage Scenarios
**Example 1: Very Low Usage**
- Vehicle Age: 5 years (Base: 14,643)
- Annual Mileage: 2,929 miles
- Mileage Ratio: 2,929 ÷ 14,643 = 0.20
- Factor: 0.719 (28.1% discount)

**Example 2: Below Average Usage**
- Vehicle Age: 10 years (Base: 12,001)
- Annual Mileage: 9,600 miles
- Mileage Ratio: 9,600 ÷ 12,001 = 0.80
- Factor: 0.930 (7.0% discount)

### Average Mileage Scenario
**Example 3: Standard Usage**
- Vehicle Age: 7 years (Base: 13,506)
- Annual Mileage: 13,506 miles
- Mileage Ratio: 13,506 ÷ 13,506 = 1.00
- Factor: 1.000 (no adjustment)

### High Mileage Scenarios
**Example 4: Above Average Usage**
- Vehicle Age: 15 years (Base: 9,820)
- Annual Mileage: 14,730 miles
- Mileage Ratio: 14,730 ÷ 9,820 = 1.50
- Factor: 1.175 (17.5% surcharge)

**Example 5: Very High Usage**
- Vehicle Age: 20 years (Base: 8,324)
- Annual Mileage: 24,972 miles
- Mileage Ratio: 24,972 ÷ 8,324 = 3.00
- Factor: 1.728 (72.8% surcharge)

## 5. System Implementation

### Data Sources
- **Primary Source**: Carfax vehicle history reports
- **Secondary Source**: Customer-provided odometer readings
- **Verification**: Annual mileage validation at renewal

### System Fields
- **VMR_VEHICLE_AGE**: Vehicle age in years
- **VMR_AVERAGE_BASE**: Average mileage base for vehicle age
- **VMR_ANNUAL_MILEAGE**: Vehicle's annual mileage
- **VMR_RATIO**: Calculated mileage ratio (rounded to 2 decimals)
- **VMR_FACTOR**: Applied rate factor

### Coverage Application
- **BI_VMR_FACTOR**: Applied to Bodily Injury premium
- **PD_VMR_FACTOR**: Applied to Property Damage premium
- **OTC_VMR_FACTOR**: Applied to Other Than Collision premium
- **COL_VMR_FACTOR**: Applied to Collision premium

## 6. Quality Controls

### Data Validation
- **Mileage Range**: Annual mileage must be positive integer
- **Ratio Range**: Calculated ratio must be between 0.00 and 10.00
- **Factor Lookup**: Must match exact ratio value in table
- **Age Validation**: Vehicle age must be 1 year or greater

### Audit Requirements
- **New Business**: Verify Carfax data retrieval
- **Renewal**: Compare current vs. prior mileage ratio
- **Factor Change**: Document reason for mileage update
- **Manual Override**: Requires underwriter approval

### Error Handling
- **Missing Carfax Data**: Request customer declaration
- **Invalid Mileage**: Flag for underwriter review
- **Ratio Out of Range**: Apply maximum factor (5.696)
- **System Failure**: Default to factor 1.000 pending review

## 7. Customer Communication

### Policy Documents
- **Declaration Page**: Show annual mileage and mileage factor
- **Rating Worksheet**: Display calculation details
- **Renewal Notice**: Highlight mileage-based rate changes

### Customer Messaging
**Low Mileage Discount**:
"Your vehicle qualifies for a low mileage discount based on annual usage of [X] miles."

**Standard Mileage**:
"Your vehicle mileage of [X] miles per year reflects average usage patterns."

**High Mileage Surcharge**:
"Your vehicle's higher annual mileage of [X] miles impacts your premium calculation."

### Update Process
- Customer can request mileage review at any time
- Updated mileage effective at next renewal
- Mid-term adjustments only for significant changes (>25% variance)

## 8. Cross-References

### Related Rating Factors
- **Vehicle Age**: Determines average mileage base
- **Vehicle Use**: Correlates with expected mileage patterns
- **Territory**: May impact average mileage expectations

### System Integration
- **Carfax Interface**: Automated mileage data retrieval
- **Rating Engine**: Real-time factor calculation
- **Policy Administration**: Mileage tracking and updates
- **Reporting System**: Mileage distribution analysis

### Regulatory Compliance
- **Filed Factor**: Approved by Texas Department of Insurance
- **Disclosure**: Mileage impact clearly shown on documents
- **Non-Discrimination**: Applied uniformly to all vehicles
- **Update Frequency**: Annual review at minimum

## 9. Validation Standards

### Calculation Verification
```
Example Validation:
Vehicle Age: 8 years
Average Base: 13,141 miles
Annual Mileage: 15,769 miles
Ratio Calculation: 15,769 ÷ 13,141 = 1.20
Factor Applied: 1.070
Premium Impact: +7.0%
```

### Testing Requirements
- **Unit Test**: Verify ratio calculation accuracy
- **Integration Test**: Confirm factor lookup functionality
- **Regression Test**: Validate all ratio ranges (0.00 to 10.00)
- **User Acceptance**: Verify customer communication clarity

### Performance Standards
- **Calculation Time**: < 100ms per vehicle
- **Data Retrieval**: < 2 seconds for Carfax lookup
- **Factor Application**: Real-time during quote/renewal
- **Update Processing**: < 5 seconds for mileage change

## 10. Document Maintenance

### Version Control
- **Current Version**: 1.0
- **Effective Date**: 07/15/2025
- **Last Review**: Document creation
- **Next Review**: 07/15/2026

### Update Triggers
- **Rate Filing Changes**: New factors or ratio ranges
- **System Enhancements**: Improved mileage data sources
- **Regulatory Requirements**: Compliance updates
- **Business Decisions**: Mileage strategy modifications

### Approval Requirements
- **Technical Review**: IT and Actuarial validation
- **Business Review**: Underwriting and Product approval
- **Compliance Review**: Legal and Regulatory sign-off
- **Final Approval**: Program Manager authorization

### Distribution List
- Underwriting Department
- IT Development Team
- Actuarial Department
- Customer Service Team
- Agency Partners
- Compliance Office

---

**Document Status**: Final
**Classification**: Internal Use
**Owner**: Aguila Dorada Program Management