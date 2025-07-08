# Vehicle County Modifier Rate Factor Interpretation
## Aguila Dorada Texas Personal Auto Program

### Overview
This document provides the complete factual interpretation of the Vehicle County Modifier rate factor for the Aguila Dorada Texas Personal Auto insurance program. This serves as the definitive reference for understanding how county of vehicle location impacts premium calculations across all coverage types.

## 1. Factor Identification

### Factor Details
- **Factor Name**: Vehicle County Modifier
- **Company**: Old American County Mutual
- **Managing General Agency**: Aguila Dorada
- **Program**: Private Passenger Auto
- **Effective Date**: New Business 07/15/2025, Renewal 08/15/2025
- **System**: New System Implementation
- **Previous System**: Effective 02/1/2025
- **Source Data**: 2017 Tico Territory Model with County

## 2. Factor Structure and Definitions

### County Modifier Classifications
The Vehicle County Modifier applies different rate adjustments based on the county where the vehicle is principally garaged. The factor includes all 254 Texas counties with specific modifier values for each coverage type.

### Coverage Types Affected
- **BI**: Bodily Injury Liability
- **PD**: Property Damage Liability
- **UMBI**: Uninsured Motorist Bodily Injury
- **UMPD**: Uninsured Motorist Property Damage
- **MED**: Medical Payments
- **PIP**: Personal Injury Protection
- **COMP**: Comprehensive
- **COLL**: Collision

### Modifier Categories

#### Standard Counties (Factor: 1.00)
Most Texas counties receive the baseline modifier of 1.00 (no adjustment) across all coverage types. These counties represent areas with standard risk profiles and include the majority of Texas counties such as:
- Andrews, Aransas, Archer, Armstrong
- Bailey, Bandera, Bastrop, Baylor
- Bee, Bell, Blanco, Borden
- And approximately 225 additional counties

#### High-Risk Counties (Factor: 0.90)
Selected counties receive a 0.90 modifier (10% reduction from base rate) for specific coverage types. This adjustment affects:

**Counties with BI/PD/COMP/COLL Reduction:**
- Anderson
- Angelina

**Coverage-Specific Application:**
- **BI (Bodily Injury)**: 0.90 factor
- **PD (Property Damage)**: 0.90 factor
- **UMBI**: 1.00 factor (no adjustment)
- **UMPD**: 1.00 factor (no adjustment)
- **MED**: 1.00 factor (no adjustment)
- **PIP**: 1.00 factor (no adjustment)
- **COMP (Comprehensive)**: 0.90 factor
- **COLL (Collision)**: 0.90 factor

## 3. Business Rules

### County Determination
- County is determined by the principal garaging location of the vehicle
- Each county has a specific set of modifier values for each coverage type
- Modifiers are applied uniformly within each county regardless of ZIP code variations

### Rate Application
- County modifiers are multiplicative factors applied to base rates
- A modifier of 1.00 represents no adjustment (neutral)
- A modifier of 0.90 represents a 10% reduction from base rates
- Modifiers are applied consistently across all policy terms

### System Changes
- New system implementation effective 07/15/2025 for new business
- Renewal business transitions effective 08/15/2025
- Previous system was effective 02/1/2025
- Change represents a reduction for specific counties in certain coverages

## 4. Premium Impact Examples

### Example 1: Anderson County Vehicle
**Base Annual Premium (before county modifier):**
- BI: $500
- PD: $300
- COMP: $400
- COLL: $600

**After County Modifier (0.90 for BI/PD/COMP/COLL):**
- BI: $500 × 0.90 = $450 (savings of $50)
- PD: $300 × 0.90 = $270 (savings of $30)
- COMP: $400 × 0.90 = $360 (savings of $40)
- COLL: $600 × 0.90 = $540 (savings of $60)
- **Total Annual Savings: $180**

### Example 2: Standard County Vehicle (e.g., Andrews County)
**Base Annual Premium (before county modifier):**
- BI: $500
- PD: $300
- COMP: $400
- COLL: $600

**After County Modifier (1.00 for all coverages):**
- BI: $500 × 1.00 = $500 (no change)
- PD: $300 × 1.00 = $300 (no change)
- COMP: $400 × 1.00 = $400 (no change)
- COLL: $600 × 1.00 = $600 (no change)
- **Total Premium Change: $0**

## 5. System Implementation

### Technical Specifications
- Factor is stored and applied at the county level
- System validates county assignment based on vehicle garaging address
- County modifiers are version-controlled with effective dates
- Automatic application during rating process

### Rating Process Integration
1. Vehicle garaging address determines county
2. County-specific modifiers are retrieved from rate table
3. Modifiers are applied to coverage-specific base premiums
4. Final premiums reflect county adjustments

### Data Sources
- Based on 2017 Tico Territory Model with County enhancements
- Incorporates regional risk assessment data
- Updated for new system implementation in 2025

## 6. Quality Controls

### Validation Standards
- County assignment must be valid Texas county
- All modifier values must be positive decimal numbers
- Version control ensures correct effective date application
- Rate table completeness verification for all 254 counties

### Audit Requirements
- Monthly validation of county modifier applications
- Quarterly review of county risk profiles
- Annual assessment of modifier effectiveness
- Documentation of any county reclassifications

### System Checks
- Automated validation of county code accuracy
- Cross-reference with ZIP code databases
- Exception reporting for invalid county assignments
- Rate table integrity verification

## 7. Customer Communication

### Disclosure Requirements
- County modifier impact disclosed at quote stage
- Premium breakdown shows county adjustment effects
- Policy documents reflect county-specific rates
- Renewal notices highlight any county modifier changes

### Customer Service Guidelines
- Explain county determination process clearly
- Provide examples of county modifier impacts
- Address questions about county boundaries
- Document any county correction requests

### Marketing Considerations
- Highlight favorable county modifiers where applicable
- Educate agents on county modifier variations
- Provide county comparison tools for prospects
- Emphasize geographic risk-based pricing rationale

## 8. Cross-References

### Related Rate Factors
- **Vehicle Territory**: Works in conjunction with county modifiers
- **ZIP Code Factors**: May overlay with county adjustments
- **Vehicle Usage**: Combined with county for comprehensive rating
- **Vehicle Base Rates**: Foundation rates before county adjustment

### Regulatory Compliance
- Texas Department of Insurance rate filing requirements
- Geographic rating factor approval documentation
- Actuarial support for county risk differentials
- Fair access to insurance considerations

### Documentation Links
- Rate filing exhibits for county modifier justification
- Actuarial memoranda supporting county classifications
- Geographic risk analysis reports
- County boundary reference materials

## 9. Validation Standards

### Data Accuracy Requirements
- County assignments must match official Texas county boundaries
- Modifier values must align with approved rate filings
- Effective dates must be consistent across all systems
- Historical modifier values retained for audit purposes

### Testing Protocols
- Pre-implementation testing of all county modifier applications
- Rate calculation verification for each county category
- System integration testing with related geographic factors
- User acceptance testing for agent and customer interfaces

### Exception Handling
- Process for addressing county assignment disputes
- Procedure for correcting county misclassifications
- Documentation requirements for county override situations
- Escalation process for complex county determination cases

## 10. Document Maintenance

### Update Schedule
- Annual review of county modifier effectiveness
- Semi-annual assessment of risk profile changes
- Quarterly validation of county boundary accuracy
- Monthly monitoring of system application consistency

### Version Control
- Document version tracking with change summaries
- Approval workflow for modifier updates
- Archive retention of previous county modifier versions
- Change notification process for stakeholders

### Responsible Parties
- **Actuarial Department**: County risk assessment and modifier development
- **Underwriting**: County classification validation and exception handling
- **IT Systems**: Technical implementation and system maintenance
- **Compliance**: Regulatory filing and approval coordination
- **Customer Service**: Agent and customer education and support

### Change Management
- Impact assessment for proposed county modifier changes
- Stakeholder notification process for updates
- Implementation timeline coordination
- Post-change monitoring and validation

---

*This document serves as the definitive reference for the Vehicle County Modifier rate factor within the Aguila Dorada Texas Personal Auto program. All rate applications, system implementations, and customer communications should align with the specifications outlined herein.*