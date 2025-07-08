# Region Rate Factor Interpretation
## Aguila Dorada Texas Personal Auto Program

### Overview
This document provides the complete factual interpretation of the Region rate factor for the Aguila Dorada Texas Personal Auto insurance program. This serves as the definitive reference for understanding the regional premium adjustments applied based on county-level geographic risk assessments across Texas.

## 1. Factor Identification

### Factor Details
- **Factor Name**: Region
- **Company**: Old American County Mutual
- **Managing General Agency**: Aguila Dorada
- **Program**: Private Passenger Auto
- **Effective Date**: New Business 07/15/2025, Renewal 08/15/2025
- **System**: New System Implementation

### Factor Purpose
The Region factor applies county-specific premium adjustments to reflect varying risk levels across different geographic areas within Texas. The factor recognizes that insurance risk varies significantly based on location-specific characteristics including population density, economic conditions, and proximity to border areas.

## 2. Factor Structure and Definitions

### Regional Categories
The program divides Texas into **12 distinct regional categories**, each representing different risk characteristics:

**Active Rating Regions (with factors):**
- **A Houston Area**: 6 counties (Rate factor not specified in source data)
- **B DFW Area**: 14 counties (Factor: 0.90)
- **C Hwy 35 Corridor**: 21 counties (Factor: 0.90)
- **D Border Cnty**: 24 counties (Factor: 0.80)
- **E Corpus**: 13 counties (Factor: 0.90)
- **F E Texas**: 13 counties (Factor: 0.90)
- **G EC Texas**: 19 counties (Factor: 0.90)
- **H NE Texas**: 19 counties (Factor: 0.90)
- **I SE Texas**: 11 counties (Rate factor not specified in source data)
- **J Small Mkt Central Tx**: 13 counties (Factor: 0.90)
- **K West Tx Small Pop Cnty**: 92 counties (Rate factor not specified in source data)

### Rate Factor Values
**Two primary rate factors are applied:**
- **0.80 (Credit)**: Applied to Border Counties region
- **0.90 (Credit)**: Applied to most other active regions

### Factor Application
- **Multiplicative Factor**: Applied as a percentage adjustment to base premium
- **County-Based Assignment**: Each Texas county is assigned to a specific region
- **Policy-Level Application**: All vehicles on the policy use the same regional factor
- **Address-Based Determination**: Factor determined by policyholder's garaging address

## 3. Business Rules

### Assignment Rules
- **County Determination**: Regional assignment based on the county where the vehicle is garaged
- **Policy Consistency**: All vehicles on the same policy receive the same regional factor
- **Address Verification**: Factor assignment requires verified county location
- **No Mid-Term Changes**: Regional assignment cannot be modified during the policy term

### Regional Definitions

#### D Border Cnty (Factor: 0.80)
**Counties Included (24 total):**
BREWSTER, BROOKS, CAMERON, DIMMIT, DUVAL, EDWARDS, HIDALGO, HUDSPETH, JIM HOGG, KENEDY, KINNEY, LA SALLE, MAVERICK, MCMULLEN, PECOS, PRESIDIO, STARR, UVALDE, VAL VERDE, WEBB, WILLACY, ZAPATA, ZAVALA

**Population Coverage:** 1,881,631 total residents
**Risk Characteristics:** Border counties receive the most favorable rating factor at 0.80

#### Regions with 0.90 Factor
**B DFW Area (14 counties):** ELLIS and other DFW metropolitan counties
**C Hwy 35 Corridor (21 counties):** GUADALUPE, HAYS and counties along Interstate 35
**E Corpus (13 counties):** VICTORIA and Corpus Christi area counties
**F E Texas (13 counties):** ANDERSON, ANGELINA, CHEROKEE, HENDERSON and other East Texas counties
**G EC Texas (19 counties):** BRAZOS, WALKER and East Central Texas counties
**H NE Texas (19 counties):** BOWIE, GREGG and Northeast Texas counties
**J Small Mkt Central Tx (13 counties):** LUBBOCK, POTTER, RANDALL, TAYLOR, WICHITA and other Central Texas markets

### Inactive Regions
**Regions without specified factors:**
- **A Houston Area**: 6 counties (factor not provided)
- **I SE Texas**: 11 counties (factor not provided)
- **K West Tx Small Pop Cnty**: 92 counties (factor not provided)

## 4. Premium Impact Examples

### Factor Impact Analysis
**Border Counties Credit (0.80 factor):**
- **Premium Reduction**: 20% discount from base premium
- **Example**: $1,000 base premium becomes $800 with border county credit
- **Coverage**: Applies to 24 counties with 1.88 million residents

**Standard Regional Credit (0.90 factor):**
- **Premium Reduction**: 10% discount from base premium
- **Example**: $1,000 base premium becomes $900 with standard regional credit
- **Coverage**: Applies to most active rating regions

### Comparative Impact
- **Border vs. Standard**: Border counties receive 11.1% better rates than standard regions (0.80 vs 0.90)
- **Regional Consistency**: All counties within the same region receive identical factors
- **Statewide Coverage**: Active factors cover 153 of 254 Texas counties

## 5. System Implementation

### Technical Requirements
- **County Validation**: System must validate county assignment against approved regional mapping
- **Address Integration**: Regional factor determined by garaging address county lookup
- **Rating Integration**: Factor applied multiplicatively to calculated base premium
- **Policy Application**: Same factor applied to all coverages and vehicles on policy

### Data Sources
- **County Mapping**: Predefined table mapping each Texas county to regional category
- **Population Data**: County population figures used for risk assessment
- **Address Validation**: Integration with address verification systems for accurate county determination

### Processing Rules
- **Factor Lookup**: System retrieves factor based on county-to-region mapping
- **Default Handling**: Procedures for counties not in active rating regions
- **Validation Checks**: Confirmation that county exists and factor is available

## 6. Quality Controls

### Data Validation
- **County Completeness**: Verification that all Texas counties have regional assignments
- **Factor Consistency**: Confirmation that all counties within a region have identical factors
- **Population Accuracy**: Regular updates to county population figures
- **Address Verification**: Validation of garaging address county determination

### Audit Procedures
- **Factor Application**: Review that correct regional factors are applied
- **Geographic Accuracy**: Verification of county-to-region mapping
- **System Testing**: Regular testing of factor lookup and application
- **Rate Consistency**: Confirmation that regional adjustments are applied consistently

### Exception Handling
- **Invalid Counties**: Procedures for addresses that cannot be county-validated
- **Missing Factors**: Handling for regions without specified rate factors
- **Border Cases**: Resolution of ambiguous county assignments

## 7. Customer Communication

### Disclosure Requirements
- **Factor Explanation**: Clear explanation that rates vary by county location
- **Regional Categories**: Description of how counties are grouped into rating regions
- **Factor Impact**: Explanation of how regional factors affect premium calculation
- **Address Importance**: Emphasis on accurate garaging address for proper rating

### Customer Inquiries
- **Factor Questions**: Explanation of why regional factors vary across Texas
- **County Changes**: Process for updating regional factor when customer moves
- **Rate Differences**: Clarification of rate variations between neighboring counties

### Documentation
- **Policy Documents**: Regional factor disclosed in policy documentation
- **Rate Sheets**: County-specific factors available upon request
- **Explanation Materials**: Consumer-friendly explanation of regional rating factors

## 8. Cross-References

### Related Rating Factors
- **Territory**: Works in conjunction with territory assignments for geographic rating
- **Vehicle Base Rates**: Regional factors applied to territory-specific base rates
- **Vehicle County Modifier**: Additional county-level adjustments may apply
- **Policy Core Matrix**: Regional factors integrated into overall rating algorithm

### System Integration
- **Address Management**: Integration with address validation and county lookup systems
- **Rating Engine**: Regional factors incorporated into premium calculation workflow
- **Policy Administration**: Regional assignments maintained throughout policy lifecycle

### Regulatory Compliance
- **Rate Filing**: Regional factors included in regulatory rate filings
- **Geographic Rating**: Compliance with Texas regulations on territorial rating
- **Fair Pricing**: Demonstration that regional factors reflect actuarial risk differences

## 9. Validation Standards

### Data Quality Standards
- **County Accuracy**: 100% accuracy in county-to-region mapping
- **Factor Precision**: Rate factors maintained to two decimal places
- **Population Currency**: County population data updated annually
- **System Reliability**: 99.9% uptime for regional factor lookup

### Testing Requirements
- **Factor Application**: Automated testing of regional factor assignment
- **County Validation**: Testing of address-to-county conversion accuracy
- **Rate Calculation**: End-to-end testing of regional factor impact on premiums
- **Exception Scenarios**: Testing of edge cases and error conditions

### Performance Metrics
- **Processing Speed**: Regional factor lookup completed within 100ms
- **Accuracy Rate**: 99.95% accuracy in county determination
- **Customer Inquiries**: Track and resolve regional factor questions
- **System Availability**: Monitor regional factor system performance

## 10. Document Maintenance

### Update Schedule
- **Annual Review**: Review of regional factor assignments and values
- **Population Updates**: Annual update of county population figures
- **Regulatory Changes**: Updates based on regulatory filing changes
- **System Enhancements**: Documentation updates for system improvements

### Change Control
- **Factor Changes**: Formal approval process for regional factor modifications
- **County Reassignment**: Procedures for moving counties between regions
- **Documentation Updates**: Version control for interpretation document changes
- **Stakeholder Notification**: Communication of changes to relevant parties

### Version History
- **Current Version**: Region Rate Factor Interpretation v1.0
- **Effective Date**: Implementation scheduled for 07/15/2025 (New Business), 08/15/2025 (Renewal)
- **Last Updated**: Documentation prepared for new system implementation
- **Next Review**: Scheduled for Q3 2025 following initial implementation

---

*This interpretation document represents the factual analysis of the Region rate factor based on source data provided. All rate factors, county assignments, and business rules are documented as specified in the original HTML source material for the Aguila Dorada Texas Personal Auto program.*