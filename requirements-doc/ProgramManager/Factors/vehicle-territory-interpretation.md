# Vehicle Territory Rate Factor Interpretation
## Aguila Dorada Texas Personal Auto Program

### Overview
This document provides the complete factual interpretation of the Vehicle Territory rate factor for the Aguila Dorada Texas Personal Auto insurance program. This serves as the definitive reference for understanding how territory factors with county adjustments impact premium calculations across all coverage types for each ZIP code in Texas.

## 1. Factor Identification

### Factor Details
- **Factor Name**: Vehicle Territory (Territory Factors w/County Adjustments)
- **Company**: Old American County Mutual
- **Managing General Agency**: Aguila Dorada
- **Program**: Private Passenger Auto
- **Effective Date**: New Business 07/15/2025, Renewal Business 08/15/2025
- **System**: New System Implementation
- **Total ZIP Codes**: 2,836 unique ZIP codes
- **Total Counties**: 254 Texas counties

## 2. Factor Structure and Definitions

### Territory Rating Structure
The Vehicle Territory factor applies specific rate multipliers based on the ZIP code and county where the vehicle is principally garaged. Each ZIP code receives unique territory factors for each coverage type based on localized risk patterns and claims experience.

### Coverage Types Affected
- **BI**: Bodily Injury Liability
- **PD**: Property Damage Liability
- **UMBI**: Uninsured Motorist Bodily Injury
- **UMPD**: Uninsured Motorist Property Damage
- **MED**: Medical Payments
- **PIP**: Personal Injury Protection
- **COMP**: Comprehensive
- **COLL**: Collision

### Factor Value Ranges
The territory factors exhibit wide variation across Texas ZIP codes:

#### Factor Range Analysis
- **Minimum Factors**: Range from 0.500 (floor) to various minimums by coverage
- **Maximum Factors**: Range up to 2.000 (cap) for some coverages
- **Neutral Factor**: 1.000 represents baseline territory risk

#### Coverage-Specific Ranges
- **BI (Bodily Injury)**: Ranges from approximately 0.517 to 1.500+
- **PD (Property Damage)**: Ranges from approximately 0.572 to 1.500+
- **UMBI**: Ranges from 0.500 to 1.500 (capped)
- **UMPD**: Ranges from 0.500 to 1.500 (capped)
- **MED/PIP**: Ranges from approximately 0.562 to 1.500 (capped)
- **COMP**: Ranges from approximately 0.810 to 2.000 (capped)
- **COLL**: Ranges from approximately 0.748 to 1.500+

## 3. Business Rules

### Territory Determination
- Territory is determined by the ZIP code of the vehicle's principal garaging location
- Each ZIP code has a unique set of territory factors for each coverage type
- County assignment is included for reference but ZIP code is the primary identifier
- All 254 Texas counties are represented in the territory structure

### Rate Application
- Territory factors are multiplicative adjustments applied to base rates
- Factors below 1.000 reduce premiums (favorable territories)
- Factors above 1.000 increase premiums (higher-risk territories)
- Factors are applied uniformly for all policy terms within the same ZIP code

### Factor Limitations
- UMBI and UMPD factors are capped at 1.500 maximum
- MED and PIP factors are capped at 1.500 maximum
- COMP factors are capped at 2.000 maximum
- Minimum factor floor of 0.500 applies to UMBI and UMPD
- Some factors show exactly matching values (MED and PIP always identical)

## 4. Premium Impact Examples

### Example 1: Low-Risk Rural Territory (ZIP 76380, Archer County)
**Base Annual Premium (before territory factors):**
- BI: $500
- PD: $300  
- UMBI: $200
- UMPD: $150
- COMP: $400
- COLL: $600

**After Territory Factors:**
- BI: $500 × 0.521 = $260.50 (savings of $239.50)
- PD: $300 × 0.587 = $176.10 (savings of $123.90)
- UMBI: $200 × 0.500 = $100.00 (savings of $100.00)
- UMPD: $150 × 0.500 = $75.00 (savings of $75.00)
- COMP: $400 × 2.000 = $800.00 (increase of $400.00)
- COLL: $600 × 0.754 = $452.40 (savings of $147.60)
- **Net Impact: $163.00 increase annually**

### Example 2: High-Risk Urban Territory (ZIP 77003, Harris County)
**Base Annual Premium (before territory factors):**
- BI: $500
- PD: $300
- UMBI: $200
- UMPD: $150
- COMP: $400
- COLL: $600

**After Territory Factors:**
- BI: $500 × 1.277 = $638.50 (increase of $138.50)
- PD: $300 × 1.264 = $379.20 (increase of $79.20)
- UMBI: $200 × 1.413 = $282.60 (increase of $82.60)
- UMPD: $150 × 1.500 = $225.00 (increase of $75.00)
- COMP: $400 × 1.000 = $400.00 (no change)
- COLL: $600 × 1.479 = $887.40 (increase of $287.40)
- **Total Annual Increase: $662.70**

### Example 3: Moderate Territory (ZIP 78026, Atascosa County)
**Base Annual Premium (before territory factors):**
- BI: $500
- PD: $300
- UMBI: $200
- UMPD: $150
- COMP: $400
- COLL: $600

**After Territory Factors:**
- BI: $500 × 0.875 = $437.50 (savings of $62.50)
- PD: $300 × 0.767 = $230.10 (savings of $69.90)
- UMBI: $200 × 0.800 = $160.00 (savings of $40.00)
- UMPD: $150 × 0.602 = $90.30 (savings of $59.70)
- COMP: $400 × 1.263 = $505.20 (increase of $105.20)
- COLL: $600 × 0.942 = $565.20 (savings of $34.80)
- **Total Annual Savings: $161.90**

## 5. System Implementation

### Rating Process
1. System identifies vehicle's principal garaging ZIP code
2. System retrieves corresponding territory factors for each coverage
3. Territory factors are applied to base rates before other adjustments
4. Final premium reflects combined impact of all territory adjustments

### Data Requirements
- Complete ZIP code database for all Texas ZIP codes (2,836 entries)
- County mapping for each ZIP code (254 counties)
- Coverage-specific factor values for each territory
- System validation to ensure proper factor application

### Processing Specifications
- Territory factors are applied as multiplicative adjustments
- Factor application occurs after base rate calculation
- System must handle factor caps and floors appropriately
- ZIP code validation ensures proper territory assignment

## 6. Quality Controls

### Data Validation
- All 2,836 ZIP codes must have complete factor sets for all 8 coverage types
- Factor values must fall within established ranges for each coverage
- County assignments must be accurate and consistent
- System must validate ZIP code format and existence

### Factor Reasonableness
- COMP factors showing values at the 2.000 cap require validation
- UMBI/UMPD factors at 1.500 cap require validation
- Factors at 0.500 floor require validation
- Large factor variations within counties require review

### System Controls
- ZIP code lookup validation prevents invalid territories
- Factor range checking prevents application of invalid multipliers
- County cross-reference validation ensures data consistency
- Territory factor updates require controlled deployment process

## 7. Customer Communication

### Rate Impact Disclosure
- Customers should understand that territory is based on garaging ZIP code
- Territory factors can significantly impact premiums across coverage types
- Some coverages may increase while others decrease in same territory
- Factor variations exist even within the same county

### Territory-Related Inquiries
- ZIP code determines territory assignment, not street address
- County information is provided for reference but ZIP code controls rating
- Territory factors reflect local risk characteristics and claims experience
- Factors may vary significantly between adjacent ZIP codes

### Rate Shopping Considerations
- Territory factors are unique to each ZIP code
- Moving to different ZIP code may significantly impact rates
- Comprehensive coverage shows highest factor variation (up to 2.000)
- Liability coverages generally show more moderate factor ranges

## 8. Cross-References

### Related Rate Factors
- **Vehicle County Modifier**: Additional county-based adjustments applied separately
- **Vehicle Base Rates**: Base rates to which territory factors are applied
- **Driver Assignment**: Driver location may differ from vehicle garaging location

### System Dependencies
- ZIP code validation system
- County assignment database
- Base rate calculation engine
- Premium calculation workflow

### Documentation References
- Territory factor source data: "2017 Tico Territory Model with County"
- Previous system effective: 02/1/2025
- New system implementation: 07/15/2025 (NB), 08/15/2025 (RB)

## 9. Validation Standards

### Data Integrity Checks
- All ZIP codes must have valid 5-digit format
- Each ZIP code must have complete factor set (8 coverage types)
- Factor values must be numeric and within acceptable ranges
- County assignments must be consistent and accurate

### Factor Validation Rules
- Minimum factor floor of 0.500 for UMBI and UMPD
- Maximum factor cap of 1.500 for UMBI, UMPD, MED, and PIP
- Maximum factor cap of 2.000 for COMP
- MED and PIP factors must be identical for each ZIP code

### System Testing Requirements
- Territory factor lookup accuracy testing
- Premium calculation verification across factor ranges
- ZIP code validation and error handling testing
- Performance testing with full 2,836 ZIP code dataset

## 10. Document Maintenance

### Update Procedures
- Territory factors derived from periodic territory model updates
- Factor changes require actuarial review and regulatory approval
- System deployment must maintain data integrity throughout update process
- Historical factor retention required for policy term consistency

### Version Control
- Current version: New System effective 07/15/2025 (NB), 08/15/2025 (RB)
- Previous version: Effective 02/1/2025
- Source model: 2017 Tico Territory Model with County
- Document version tracking for all territory factor updates

### Regulatory Compliance
- Territory factors must comply with Texas Department of Insurance requirements
- Factor development methodology must be actuarially sound
- Documentation must support regulatory filing requirements
- Factor changes require appropriate notice and approval processes

### Data Governance
- Territory factor data considered confidential and proprietary
- Access controls limit factor data to authorized personnel
- Change management procedures ensure proper approval workflow
- Audit trail maintains record of all factor updates and applications