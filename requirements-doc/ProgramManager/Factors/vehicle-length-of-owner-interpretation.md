# Vehicle Length of Owner Rate Factor Interpretation
## Aguila Dorada Texas Personal Auto Program

### Overview
This document provides the complete factual interpretation of the Vehicle Length of Owner rate factor for the Aguila Dorada Texas Personal Auto insurance program. This serves as the definitive reference for understanding how the duration of vehicle ownership impacts premium calculations across all coverage types.

## 1. Factor Identification

### Factor Details
- **Factor Name**: Vehicle Length of Owner
- **Company**: Old American County Mutual
- **Managing General Agency**: Aguila Dorada
- **Program**: Private Passenger Auto
- **Effective Date**: New Business 07/15/2025, Renewal 08/15/2025
- **System**: New System Implementation

## 2. Vehicle Length of Owner Factor Structure

### Ownership Duration Classifications and Factors
**All Coverage Types (BI, PD, OTC, COL):**

#### 0 to 30 Days
- **Factor**: 1.100 (10% surcharge)
- **Years_Owned**: 0 to 30 Days
- **Risk Level**: Highest surcharge for brand new ownership

#### 31 to 60 Days
- **Factor**: 1.070 (7% surcharge)
- **Years_Owned**: 31 to 60 Days
- **Risk Level**: High surcharge for very recent ownership

#### 60 to 183 Days
- **Factor**: 1.040 (4% surcharge)
- **Years_Owned**: 60 to 183 Days
- **Risk Level**: Moderate surcharge for recent ownership

#### 184 to 365 Days
- **Factor**: 1.020 (2% surcharge)
- **Years_Owned**: 184 to 365 Days
- **Risk Level**: Low surcharge approaching one year of ownership

#### 1 to 2 Years
- **Factor**: 1.000 (base rate)
- **Years_Owned**: 2
- **Risk Level**: Baseline risk after one year of ownership

#### 2 to 3 Years
- **Factor**: 0.980 (2% discount)
- **Years_Owned**: 3
- **Risk Level**: Beginning of ownership stability discounts

#### 3 to 4 Years
- **Factor**: 0.960 (4% discount)
- **Years_Owned**: 4
- **Risk Level**: Increasing discount for established ownership

#### 4 to 5 Years
- **Factor**: 0.940 (6% discount)
- **Years_Owned**: 5
- **Risk Level**: Significant discount for long-term ownership

#### 5 to 6 Years
- **Factor**: 0.920 (8% discount)
- **Years_Owned**: 6
- **Risk Level**: Higher discount for extended ownership

#### 6 to 7 Years
- **Factor**: 0.900 (10% discount)
- **Years_Owned**: 7
- **Risk Level**: Major discount for very long-term ownership

#### 7 to 8 Years
- **Factor**: 0.880 (12% discount)
- **Years_Owned**: 8
- **Risk Level**: Near-maximum discount for extended ownership

#### 8 to 9+ Years
- **Factor**: 0.860 (14% discount)
- **Years_Owned**: 9+
- **Risk Level**: Maximum discount for longest-term ownership

## 3. Business Rules

### OSIS Programming Implementation

#### New Business
- **Data Source**: Use CarFax data to determine ownership duration
- **Verification**: Automated lookup of vehicle ownership history
- **Application**: Apply appropriate factor based on current ownership duration

#### Endorsements
- **Adding New Vehicle**: Use "0 to 30 days" factor for newly added vehicles
- **Existing Vehicles**: Maintain current ownership duration tracking
- **Mid-Term Changes**: Apply factor changes immediately upon endorsement

#### Renewal
- **Age Progression**: Age the ownership from the starting point at inception
- **Automatic Updates**: System automatically progresses ownership duration
- **Factor Adjustment**: Apply new factor based on aged ownership duration

### Ownership Duration Determination
**Duration Calculation Rules:**
- **Start Date**: Ownership begins from date of purchase or acquisition
- **Day Count**: Precise calculation of days owned
- **Year Transitions**: Automatic progression through ownership tiers
- **Documentation**: CarFax or similar documentation required for verification

## 4. Premium Impact Analysis

### Risk-Based Factor Progression
**Ownership Duration by Risk Level:**

#### Highest Risk (1.100 to 1.020 Factors)
- **0-365 Days**: New ownership period with increased risk
- **Risk Rationale**: Unfamiliarity with vehicle, higher accident probability
- **Factor Range**: 10% to 2% surcharge

#### Base Risk (1.000 Factor)
- **1-2 Years**: Established ownership baseline
- **Risk Rationale**: Owner familiar with vehicle, average risk profile

#### Reduced Risk (0.980 to 0.860 Factors)
- **2-9+ Years**: Long-term ownership with reduced risk
- **Risk Rationale**: Owner expertise, vehicle maintenance, lower claims frequency
- **Factor Range**: 2% to 14% discount

### Ownership Duration Impact Logic
**Progressive Discount Structure:**
- **Initial Period**: Highest risk in first 30 days (10% surcharge)
- **Rapid Decline**: Risk decreases quickly in first year
- **Steady Progression**: Consistent 2% discount increases after year 2
- **Maximum Benefit**: 14% discount for 8+ years of ownership

## 5. Premium Impact Examples

### Sample Premium Calculations
**Base Premium (Before Length of Owner Factor): $1,000**

#### New Owner Impact (0-30 Days)
- **Premium**: $1,000 × 1.100 = $1,100
- **Surcharge**: $100 (10% increase)
- **Ownership**: Brand new owner

#### Six Month Owner Impact (184-365 Days)
- **Premium**: $1,000 × 1.020 = $1,020
- **Surcharge**: $20 (2% increase)
- **Ownership**: Nearly one year

#### Two Year Owner Impact
- **Premium**: $1,000 × 1.000 = $1,000
- **Impact**: No adjustment, base rate
- **Ownership**: Established owner

#### Five Year Owner Impact
- **Premium**: $1,000 × 0.940 = $940
- **Discount**: $60 (6% decrease)
- **Ownership**: Long-term owner

#### Nine Year+ Owner Impact
- **Premium**: $1,000 × 0.860 = $860
- **Discount**: $140 (14% decrease)
- **Ownership**: Maximum tenure owner

## 6. System Implementation

### Data Requirements
- **Ownership Start Date**: Date of vehicle acquisition
- **Current Date**: System date for duration calculation
- **CarFax Integration**: Automated ownership history retrieval
- **Manual Override**: Capability for documented ownership verification

### Processing Requirements
1. **Duration Calculation**: Calculate days/years between ownership start and current date
2. **Tier Assignment**: Assign appropriate ownership duration tier
3. **Factor Lookup**: Retrieve factor for assigned tier
4. **Premium Application**: Apply factor to all coverage types

### Renewal Processing
1. **Age Advancement**: Automatically age ownership duration at renewal
2. **Factor Update**: Apply new factor based on aged duration
3. **Premium Adjustment**: Recalculate premium with new factor
4. **Customer Notification**: Inform customer of ownership discount progression

## 7. Quality Controls

### Validation Procedures
- **Date Accuracy**: Verify ownership start date accuracy
- **Duration Calculation**: Validate correct duration computation
- **Factor Application**: Ensure correct factor applied based on duration
- **CarFax Verification**: Cross-check ownership data with external sources

### Exception Handling
- **Missing Data**: Process for handling missing ownership dates
- **Data Conflicts**: Resolution of conflicting ownership information
- **Manual Entry**: Procedures for manual ownership verification
- **System Errors**: Error handling for calculation or lookup failures

## 8. Customer Communication

### Ownership Duration Benefits
- **Discount Progression**: Clear explanation of how discounts increase over time
- **New Owner Surcharge**: Transparent communication about initial ownership surcharges
- **Long-Term Benefits**: Emphasis on rewards for vehicle retention
- **Documentation Requirements**: Explanation of ownership verification needs

### Premium Impact Transparency
- **Factor Display**: Show current ownership factor on policy documents
- **Future Discounts**: Project future discounts based on continued ownership
- **Comparison Tools**: Allow customers to see impact of ownership duration
- **Renewal Benefits**: Highlight increasing discounts at renewal

## 9. Cross-References
- **Vehicle Age**: See Vehicle Model Year for vehicle age-based rating
- **Driver Experience**: See Driver Class for driver experience factors
- **Usage Factors**: See Vehicle Usage for usage-based rating
- **Mileage Factors**: See Vehicle Mileage Ratio for mileage-based rating

## 10. Validation Standards

This document serves as the authoritative source for:
- **Ownership Factors**: Definitive vehicle ownership duration rating factors
- **Duration Rules**: Ownership duration calculation methodology
- **Business Rules**: New business, endorsement, and renewal procedures
- **System Requirements**: Technical ownership factor implementation specifications

## 11. Document Maintenance

### Update Procedures
- **Factor Changes**: Document updates required for any factor modifications
- **Rule Modifications**: Update business rules section for process changes
- **System Updates**: Reflect system implementation changes
- **Regulatory Compliance**: Ensure updates meet regulatory requirements

### Version Control
- **Change Tracking**: Maintain history of all factor and rule changes
- **Effective Dating**: Clear documentation of change effective dates
- **Approval Process**: Actuarial and regulatory approval requirements
- **Distribution**: Stakeholder notification of document updates

### Compliance Standards
- **Actuarial Justification**: Factors based on loss experience analysis
- **Non-Discrimination**: Ownership factors applied uniformly
- **Transparency**: Clear disclosure of ownership-based rating
- **Documentation**: Proper maintenance of ownership verification records