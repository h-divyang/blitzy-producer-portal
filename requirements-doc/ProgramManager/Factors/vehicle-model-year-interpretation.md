# Vehicle Model Year Rate Factor Interpretation
## Aguila Dorada Texas Personal Auto Program

### Overview
This document provides the complete factual interpretation of the Vehicle Model Year rate factor for the Aguila Dorada Texas Personal Auto insurance program. This serves as the definitive reference for understanding how vehicle model year impacts premium calculations across liability and physical damage coverage types.

## 1. Factor Identification

### Factor Details
- **Factor Name**: Vehicle Model Year
- **Company**: Old American County Mutual
- **Managing General Agency**: Aguila Dorada
- **Program**: Private Passenger Auto
- **Effective Date**: New Business 07/15/2025, Renewal 08/15/2025
- **System**: New System Implementation

## 2. Factor Structure and Definitions

### Model Year Categories and Factors

**Liability Coverage (BI/PD):**
- **2023+**: 1.132 (13.2% surcharge)
- **2022**: 1.110 (11.0% surcharge)
- **2021**: 1.088 (8.8% surcharge)
- **2020**: 1.067 (6.7% surcharge)
- **2019**: 1.055 (5.5% surcharge)
- **2018**: 1.043 (4.3% surcharge)
- **2017**: 1.031 (3.1% surcharge)
- **2016**: 1.028 (2.8% surcharge)
- **2015**: 1.023 (2.3% surcharge)
- **2014**: 1.017 (1.7% surcharge)
- **2013**: 1.009 (0.9% surcharge)
- **2012**: 1.000 (base rate)
- **2011**: 0.991 (0.9% discount)
- **2010**: 0.981 (1.9% discount)
- **2009**: 0.969 (3.1% discount)
- **2008**: 0.957 (4.3% discount)
- **2007**: 0.945 (5.5% discount)
- **2006**: 0.932 (6.8% discount)
- **2005**: 0.917 (8.3% discount)
- **2004**: 0.901 (9.9% discount)
- **2003**: 0.884 (11.6% discount)
- **2002**: 0.865 (13.5% discount)
- **2001**: 0.845 (15.5% discount)
- **2000**: 0.824 (17.6% discount)
- **1999**: 0.801 (19.9% discount)
- **1998**: 0.784 (21.6% discount)
- **1997**: 0.763 (23.7% discount)
- **1996**: 0.741 (25.9% discount)
- **1995**: 0.719 (28.1% discount)
- **1994**: 0.695 (30.5% discount)
- **1993**: 0.685 (31.5% discount)
- **1992**: 0.675 (32.5% discount)
- **1991**: 0.665 (33.5% discount)
- **1990**: 0.655 (34.5% discount)
- **1989 and older**: 0.645 (35.5% discount)
- **Non-owner**: 1.000 (base rate)

**Physical Damage Coverage (COMP/COLL):**
- **2023+**: 1.468 (46.8% surcharge)
- **2022**: 1.411 (41.1% surcharge)
- **2021**: 1.357 (35.7% surcharge)
- **2020**: 1.305 (30.5% surcharge)
- **2019**: 1.255 (25.5% surcharge)
- **2018**: 1.207 (20.7% surcharge)
- **2017**: 1.161 (16.1% surcharge)
- **2016**: 1.116 (11.6% surcharge)
- **2015**: 1.058 (5.8% surcharge)
- **2014**: 1.000 (base rate)
- **2013**: 0.942 (5.8% discount)
- **2012**: 0.884 (11.6% discount)
- **2011**: 0.826 (17.4% discount)
- **2010**: 0.768 (23.2% discount)
- **2009**: 0.710 (29.0% discount)
- **2008**: 0.652 (34.8% discount)
- **2007**: 0.594 (40.6% discount)
- **2006**: 0.536 (46.4% discount)
- **2005**: 0.478 (52.2% discount)
- **2004**: 0.478 (52.2% discount)
- **2003**: 0.478 (52.2% discount)
- **2002**: 0.478 (52.2% discount)
- **2001**: 0.478 (52.2% discount)
- **2000**: 0.478 (52.2% discount)
- **1999**: 0.478 (52.2% discount)
- **1998**: 0.478 (52.2% discount)
- **1997**: 0.478 (52.2% discount)
- **1996**: 0.478 (52.2% discount)
- **1995**: 0.478 (52.2% discount)
- **1994**: 0.478 (52.2% discount)
- **1993**: 0.478 (52.2% discount)
- **1992**: 0.478 (52.2% discount)
- **1991**: 0.478 (52.2% discount)
- **1990**: 0.478 (52.2% discount)
- **1989 and older**: 1.000 (base rate)
- **Non-owner**: 1.000 (base rate)

## 3. Business Rules

### Model Year Assignment Rules
- **VIN Determination**: Model year determined from Vehicle Identification Number
- **Manufacturer Designation**: Based on manufacturer's assigned model year
- **Policy Effective Date**: Model year rating determined at policy inception
- **Stability**: Model year factor remains constant throughout policy term

### Coverage-Specific Applications
- **Liability Coverage**: BI and PD use identical model year factors
- **Physical Damage**: Comprehensive and Collision use identical model year factors
- **Non-Owner Policies**: Use base rate factor of 1.000 for all model years

### Model Year Determination Rules
1. **Primary Source**: VIN decoding for model year identification
2. **Secondary Source**: Vehicle registration documentation
3. **Tertiary Source**: Title or manufacturer documentation
4. **Validation**: Cross-reference multiple sources when available

## 4. Premium Impact Examples

### Liability Coverage Impact
**2023 Model Year Vehicle:**
- Base Premium: $1,000
- Model Year Factor: 1.132
- Adjusted Premium: $1,132 (13.2% increase)

**2000 Model Year Vehicle:**
- Base Premium: $1,000
- Model Year Factor: 0.824
- Adjusted Premium: $824 (17.6% decrease)

### Physical Damage Impact
**2023 Model Year Vehicle:**
- Base Premium: $1,000
- Model Year Factor: 1.468
- Adjusted Premium: $1,468 (46.8% increase)

**2005 Model Year Vehicle:**
- Base Premium: $1,000
- Model Year Factor: 0.478
- Adjusted Premium: $478 (52.2% decrease)

## 5. System Implementation

### Data Requirements
- **VIN Database**: Complete VIN decoding capability for model year determination
- **Model Year Matrix**: Full table of model year factors by coverage type
- **Validation Logic**: Model year reasonableness checks and validation
- **Exception Handling**: Process for VIN decode failures or discrepancies

### Processing Requirements
1. **Automatic Model Year Determination**: System extracts model year from VIN
2. **Factor Lookup**: System applies appropriate factor based on coverage type
3. **Validation Checks**: Verify model year is reasonable for vehicle age
4. **Manual Override**: Capability for manual model year entry when needed

### Rating Integration
- **Sequential Application**: Model year factor applied in rating sequence
- **Coverage Differentiation**: Different factors for liability vs. physical damage
- **Non-Owner Handling**: Special logic for non-owner policies

## 6. Quality Controls

### Validation Procedures
- **VIN Accuracy**: Verify VIN is valid and correctly decoded
- **Model Year Reasonableness**: Check model year against current date
- **Factor Application**: Confirm correct factors applied by coverage
- **Cross-Reference Validation**: Compare model year across documentation sources

### Exception Handling
- **VIN Decode Failures**: Process for manual model year determination
- **Conflicting Model Years**: Resolution hierarchy for discrepant sources
- **Missing VIN**: Alternative methods for model year determination
- **Future Model Years**: Handling of model years beyond current year

### Audit Requirements
- **Factor Accuracy**: Regular verification of model year factor application
- **Source Documentation**: Maintain record of model year determination source
- **Override Tracking**: Document all manual model year overrides
- **Discrepancy Resolution**: Track and resolve model year conflicts

## 7. Customer Communication

### Model Year Impact Disclosure
- **Rating Factor Notice**: Customers informed of model year rating impact
- **Coverage Differentiation**: Explanation of different impacts by coverage type
- **Premium Projection**: Multi-year premium impact as vehicle ages
- **Optimal Timing**: Guidance on vehicle replacement timing considerations

### Documentation Requirements
- **Model Year Verification**: Provide customers with model year determination
- **Factor Explanation**: Clear explanation of model year factor application
- **Alternative Options**: Information on coverage options affected by model year
- **Premium Impact**: Detailed breakdown of model year premium impact

## 8. Cross-References

### Related Rate Factors
- **Vehicle Age**: See Vehicle Age rate factor for age-based rating
- **Vehicle Use**: See Vehicle Usage rate factor for usage-based modifications
- **Vehicle Territory**: See Territory rate factor for location-based adjustments
- **Algorithm**: See Algorithm rate factor for overall calculation methodology

### System Documentation
- **VIN Processing**: See technical documentation for VIN decoding procedures
- **Rating Engine**: See rating system documentation for factor application
- **Validation Rules**: See system validation documentation for quality controls
- **Exception Processing**: See exception handling procedures for override processes

## 9. Validation Standards

### Actuarial Standards
- **Statistical Basis**: Model year factors based on actuarial analysis of loss experience
- **Credibility Standards**: Factors meet actuarial credibility requirements
- **Regulatory Compliance**: Factors comply with Texas Department of Insurance requirements
- **Documentation Requirements**: Complete actuarial justification for all factors

### Technical Standards
- **System Accuracy**: Model year determination must be 99.9% accurate
- **Processing Speed**: Model year lookup must complete within system performance standards
- **Data Integrity**: Model year data must be validated and consistent
- **Backup Procedures**: Alternative model year determination methods available

### Quality Assurance
- **Regular Review**: Quarterly review of model year factor accuracy
- **Exception Analysis**: Monthly analysis of model year determination exceptions
- **Customer Complaints**: Track and analyze model year-related customer issues
- **Continuous Improvement**: Ongoing enhancement of model year determination processes

## 10. Document Maintenance

### Update Requirements
- **Factor Changes**: Any changes to model year factors require document updates
- **System Changes**: Modifications to model year determination processes require documentation
- **Regulatory Changes**: Updates required for regulatory compliance changes
- **Business Rule Changes**: Documentation updates for business rule modifications

### Version Control
- **Change History**: Maintain complete history of all document changes
- **Approval Process**: All changes require appropriate business approval
- **Distribution**: Updated documentation distributed to all stakeholders
- **Training**: Staff training on any significant changes to model year processing

### Maintenance Schedule
- **Annual Review**: Complete annual review of model year factors and processes
- **Quarterly Updates**: Quarterly updates for system or process changes
- **As-Needed**: Immediate updates for urgent changes or corrections
- **Regulatory Compliance**: Updates as required for regulatory compliance

This document serves as the authoritative source for Vehicle Model Year rate factor implementation, application, and maintenance for the Aguila Dorada Texas Personal Auto program.