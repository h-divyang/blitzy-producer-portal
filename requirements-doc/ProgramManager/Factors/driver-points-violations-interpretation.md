# Driver Points Violations Rate Factor Interpretation
## Aguila Dorada Texas Personal Auto Program

### Overview
This document provides the complete factual interpretation of the Driver Points Violations rate factor for the Aguila Dorada Texas Personal Auto insurance program. This factor applies specific point values to different violation types with escalating penalties for repeat offenses, serving as a key component in determining driver risk and premium calculation.

## 1. Factor Identification

### Factor Details
- **Factor Name**: Driver Points Violations
- **Company**: Old American County Mutual
- **Managing General Agency**: Aguila Dorada
- **Program**: Private Passenger Auto
- **Effective Date**: New Business 07/15/2025, Renewal 08/15/2025
- **System**: New System Implementation

## 2. Factor Structure and Definitions

### Violation Point System
The system assigns specific point values based on violation type and occurrence frequency:

#### Column Structure
- **Description**: Violation type description
- **1ST**: Points assigned for first occurrence
- **2ND**: Points assigned for second occurrence  
- **3RD**: Points assigned for third occurrence
- **Rate Chart**: Reference number (999 for all violations)
- **Violation Number**: Unique identifier for each violation type
- **Group Number**: Categorical grouping (1=Major Moving, 7=Alcohol Related, 8=Accident At-Fault)
- **Months**: Duration of point application (36 months for all violations)
- **Fourth/Fifth+ Occ Points**: Points for subsequent occenses (20 points each)

### Violation Categories

#### Accident-Related Violations (Group 8)
**Point Assignment: 3-4-6 (1st-2nd-3rd)**
- **ACCIDENT AT-FAULT** (Violation #177)
  - 1st: 3 points, 2nd: 4 points, 3rd: 6 points
  - Subsequent: 20 points each
- **ACCIDENT WITH PEDESTRIAN** (Violation #178)
  - Same point structure as at-fault accidents
  - Grouped under "ACCIDENT AT-FAULT" category

#### Alcohol-Related Violations (Group 7)  
**Point Assignment: 6-6-6 (all occurrences)**
- **DUI** (Violation #11)
- **DUI - ALCOHOL/LIQUOR** (Violation #12)
- **DUI - DRUGS/OPIATES** (Violation #172)
- **DWI** (Violation #14)
- **DWAI** (Violation #13)
- **OWI** (Violation #192)
- **CONSUMING ALCOHOL WHILE DRIVING** (Violation #38)
- **DRIVING SCHOOL BUS WHILE INTOXICATED** (Violation #9)
- **POSSESSION OF CONTROLLED SUBSTANCE** (Violation #174)
- **VIOLATION OF LIQUOR LAW** (Violation #84)

All alcohol-related violations receive **6 points for any occurrence**, escalating to **20 points for fourth and subsequent offenses**.

#### Major Moving Violations (Group 1)
**Point Assignment: 3-5-5 (1st-2nd-3rd)**

**Traffic Safety Violations:**
- **CARELESS AND IMPRUDENT DRIVING** (Violation #5)
- **RECKLESS DRIVING** (Violation #33)
- **NEGLIGENT DRIVING** (Violation #28)
- **EXCESSIVE SPEEDING** (Violation #59)
- **UNSAFE SPEED** (Violation #130)

**Highway Safety Violations:**
- **DRIVING ON WRONG SIDE OF ROAD** (Violation #45)
- **DRIVING THE WRONG WAY** (Violation #47)
- **DRIVING WRONG WAY ON ONE-WAY** (Violation #56)
- **DRIVING ON SIDEWALK OR PARKWAY** (Violation #44)

**Specialized Violations:**
- **RACING** (Violation #32)
- **SPEED CONTEST** (Violation #35)
- **HIT AND RUN** (Violation #179)
- **IMPROPER PASSING OF A SCHOOL BUS** (Violation #72)
- **SPEEDING IN A CONSTRUCTION ZONE** (Violation #123)
- **SPEEDING IN A SCHOOL ZONE** (Violation #124)

**Serious Criminal Violations:**
- **INVOLUNTARY MANSLAUGHTER** (Violation #186)
- **VEHICULAR INJURY** (Violation #36)
- **OBSTRUCTING POLICE** (Violation #30)

## 3. Business Rules

### Point Calculation Rules
- **Lookback Period**: 36 months from violation date
- **Progressive Penalties**: Point values increase with repeat offenses
- **Maximum Penalty**: 20 points for fourth and subsequent offenses across all violation types
- **Group Classification**: Violations grouped by severity and type

### Application Rules
- **Per Violation Basis**: Each violation receives individual point assignment
- **Occurrence Tracking**: System tracks violation sequence (1st, 2nd, 3rd, etc.)
- **Date-Based Calculation**: Points applied from conviction date
- **Cumulative Effect**: Multiple violations result in cumulative point totals

### Special Provisions
- **Alcohol Violations**: Consistent 6-point assignment regardless of occurrence number (until 4th+)
- **Accident Violations**: Escalating point structure (3-4-6)
- **Major Moving**: Mid-level escalation (3-5-5)
- **System Integration**: Expectation of additional violation mappings from Oakwood system

## 4. Premium Impact Examples

### Alcohol-Related Violation Impact
- **First DUI**: 6 points applied
- **Second DUI**: 6 points applied
- **Third DUI**: 6 points applied
- **Fourth DUI**: 20 points applied

### Accident Progression
- **First At-Fault Accident**: 3 points
- **Second At-Fault Accident**: 4 points  
- **Third At-Fault Accident**: 6 points
- **Fourth At-Fault Accident**: 20 points

### Major Moving Violation Series
- **First Reckless Driving**: 3 points
- **Second Reckless Driving**: 5 points
- **Third Reckless Driving**: 5 points
- **Fourth Reckless Driving**: 20 points

## 5. System Implementation

### Data Requirements
- **Violation Tracking**: System must track violation sequence and dates
- **Point Calculation**: Automatic point assignment based on violation type and occurrence
- **Group Classification**: Ability to categorize violations by group number
- **Duration Management**: 36-month rolling window for all violations

### Integration Points
- **RTR System**: Expected integration with Oakwood for additional violation mappings
- **Conviction Data**: Real-time access to conviction records
- **Point Application**: Automatic factor application based on total points
- **Exception Handling**: Process for disputed or unclear violations

### Technical Specifications
- **Rate Chart Reference**: Universal "999" reference number
- **Violation Numbers**: Unique identifier system for each violation type
- **Group Numbers**: Categorical classification system
- **Duration Standard**: 36-month application period across all violation types

## 6. Quality Controls

### Validation Procedures
- **Violation Identification**: Verify correct violation code assignment
- **Point Calculation**: Validate point assignment based on occurrence sequence
- **Date Verification**: Confirm violation dates within 36-month lookback
- **Group Classification**: Ensure proper categorical assignment

### Audit Requirements
- **Violation Documentation**: Maintain records of all violation assignments
- **Point Calculation Audit**: Track all point calculation decisions
- **Occurrence Tracking**: Document violation sequence determinations
- **Exception Documentation**: Record manual adjustments and rationale

## 7. Customer Communication

### Disclosure Requirements
- **Point Assignment**: Clear explanation of point values per violation type
- **Escalation Structure**: Documentation of increasing penalties for repeat offenses
- **Duration Information**: 36-month application period for all violations
- **Appeal Process**: Procedures for challenging violation assignments

### Educational Materials
- **Violation Categories**: Explanation of different violation groups
- **Point Impact**: How points affect premium calculations
- **Prevention Information**: Resources for avoiding future violations
- **Rehabilitation Options**: Information on point reduction programs if available

## 8. Cross-References

### Related Rate Factors
- **Driver Points**: See Driver Points rate factor for overall point system multipliers
- **Driver Criminal History**: See Criminal History rate factor for felony-related violations
- **Algorithm**: See Algorithm rate factor for calculation methodology
- **Driver Class**: Integration with base driver classification system

### External Systems
- **Oakwood RTR**: Expected delivery of additional violation mappings
- **Court Systems**: Integration with conviction record systems
- **State DMV**: Motor vehicle record access and validation
- **Verisk Data**: Third-party violation and conviction data sources

## 9. Validation Standards

### Data Accuracy Standards
- **Violation Codes**: 100% accuracy in violation type identification
- **Point Assignment**: Correct point values based on occurrence sequence
- **Date Validation**: Accurate conviction date recording and calculation
- **Group Assignment**: Proper categorical classification per violation type

### System Performance Standards
- **Real-Time Processing**: Immediate point calculation upon violation entry
- **Data Integrity**: Consistent violation tracking across policy periods
- **Exception Handling**: Systematic resolution of disputed violations
- **Audit Trail**: Complete documentation of all point assignments and changes

## 10. Document Maintenance

### Update Requirements
- **Violation Additions**: Process for adding new violation types from Oakwood
- **Point Modifications**: Actuarial approval required for point value changes
- **Group Restructuring**: System updates for violation category changes
- **Integration Updates**: Coordination with external system modifications

### Version Control
- **Document Updates**: Track all changes to violation point assignments
- **Effective Dates**: Coordinate updates with system implementation dates
- **Approval Process**: Actuarial and underwriting approval for all modifications
- **Distribution**: Communication of changes to all relevant stakeholders

### Regulatory Compliance
- **State Requirements**: Ensure compliance with Texas insurance regulations
- **Filing Updates**: Submit rate factor changes to regulatory authorities
- **Documentation Standards**: Maintain records per regulatory requirements
- **Audit Preparation**: Ready documentation for regulatory examinations