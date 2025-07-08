# Vehicle Severe Problem Rate Factor Interpretation
## Aguila Dorada Texas Personal Auto Program

### Overview
This document provides the complete factual interpretation of the Vehicle Severe Problem rate factor for the Aguila Dorada Texas Personal Auto insurance program. This factor applies premium adjustments based on branded titles and severe accident indicators identified through DCS (Data Collection System) integration, affecting both liability and physical damage coverages.

## 1. Factor Identification

### Factor Details
- **Factor Name**: Vehicle Severe Problem (Branded Title / Severe Problem Reports)
- **Company**: Old American County Mutual
- **Managing General Agency**: Aguila Dorada
- **Program**: Private Passenger Auto
- **Effective Date**: New Business 07/15/2025, Renewal 08/15/2025
- **System**: New System Implementation
- **Data Source**: DCS (Data Collection System) integration

## 2. Factor Structure and Definitions

### Coverage Application
The factor applies different multipliers across all coverage types:
- **BI** (Bodily Injury Liability)
- **PD** (Property Damage Liability)  
- **UMBI** (Uninsured Motorist Bodily Injury)
- **UMPD** (Uninsured Motorist Property Damage)
- **MED** (Medical Payments)
- **PIP** (Personal Injury Protection)
- **COMP** (Comprehensive)
- **COLL** (Collision)

### Risk Categories

#### Clean Vehicles (No Hit)
**Description**: Vehicles with no branded title or severe problem indicators
**Factor Application**:
- All Coverages: 1.00 (no adjustment)
- Represents baseline risk with no premium modification

#### Title Brands Category
**Factor Application**:
- Liability Coverages (BI, PD, UMBI, UMPD, MED, PIP): 1.15 (15% surcharge)
- Physical Damage Coverages (COMP, COLL): 1.25 (25% surcharge)

**Specific Title Brand Types**:
- **Salvage**: Vehicle declared total loss by insurance company
- **Junk**: Vehicle designated for parts or scrap only
- **Rebuilt/Reconstructed**: Vehicle rebuilt after total loss
- **Dismantled**: Vehicle taken apart for parts
- **Flood**: Vehicle damaged by flood or water
- **Fire**: Vehicle damaged by fire
- **Hail**: Vehicle damaged by hail
- **Canadian Total Loss**: Total loss designation from Canadian insurer
- **Lemon**: Vehicle with recurring defects under lemon law
- **Other**: Any other branded title designation

#### Severe Accident Indicators
**Factor Application**:
- Liability Coverages: 1.15 (15% surcharge)
- Physical Damage Coverages: 1.25 (25% surcharge)

**Indicator Types**:
- **Police Report Severe Damage**: Official police documentation of severe accident damage
- **Crash Test Vehicle**: Vehicle used in crash testing programs
- **Salvage Auction**: Vehicle sold through salvage auction process

## 3. Business Rules

### DCS Data Integration
The system processes data from specific DCS fields to determine vehicle status:

#### Primary Data Fields
- **DPSStolenFlag**: Indicates stolen vehicle status
- **FloodDamageFlag**: Identifies flood damage history
- **LemonLawFlag**: Marks lemon law vehicles
- **BondedTitleCode**: Identifies bonded title situations
- **ReconditionedCode**: Indicates reconditioned vehicles
- **ReconstructedFlag**: Marks reconstructed vehicles
- **TitleRevokedFlag**: Identifies revoked title status
- **JunkFlag**: Marks junk vehicles
- **Exported**: Indicates exported vehicle status
- **RegistrationInvalidFlag**: Identifies invalid registration

#### Processing Rules
1. **DCS Data Collection**: Automatic retrieval of vehicle data from DCS system
2. **Flag Analysis**: Evaluation of all relevant flags for each vehicle
3. **Category Assignment**: Classification into Clean, Title Brands, or Severe Accident categories
4. **Factor Application**: Application of appropriate multipliers by coverage type
5. **Multiple Indicators**: "Other" designation used when multiple title indicators exist

### Hard Stop Provisions
**Critical Finding**: Vehicles with data in certain fields trigger automatic decline
- System implements "Hard Stop for vehicle with data in one of these fields"
- Specific trigger fields not detailed in source documentation
- Requires manual underwriting review for vehicles with severe problem indicators

## 4. Premium Impact Examples

### Title Brand Impact Analysis
**Salvage Title Vehicle**:
- Bodily Injury Liability: 15% surcharge (1.15 factor)
- Collision Coverage: 25% surcharge (1.25 factor)
- Comprehensive Coverage: 25% surcharge (1.25 factor)

**Flood Damage Vehicle**:
- Property Damage Liability: 15% surcharge
- Physical Damage Coverages: 25% surcharge
- Higher physical damage surcharge reflects increased claim risk

### Multiple Brand Situation
**Vehicle with Multiple Title Brands**:
- Classified under "Other" category
- Same surcharge structure: 15% liability, 25% physical damage
- No compounding of multiple brand penalties

### Clean Vehicle Baseline
**No Title Issues**:
- All coverages: No surcharge (1.00 factor)
- Represents standard risk profile
- Baseline for comparing increased risk vehicles

## 5. System Implementation

### DCS Integration Architecture
- **Real-Time Data Access**: Direct integration with DCS database
- **Automated Processing**: Systematic evaluation of all DCS data fields
- **Flag-Based Logic**: Decision tree based on specific data field values
- **Exception Handling**: Process for handling incomplete or conflicting data

### Data Processing Workflow
1. **Vehicle Identification**: VIN-based lookup in DCS system
2. **Data Field Evaluation**: Analysis of all relevant flags and codes
3. **Risk Classification**: Assignment to appropriate risk category
4. **Factor Determination**: Selection of coverage-specific multipliers
5. **System Application**: Automatic application to policy rating

### Technical Requirements
- **DCS Connectivity**: Reliable connection to external data source
- **Real-Time Processing**: Immediate factor application during quoting
- **Data Validation**: Verification of DCS data accuracy and currency
- **Audit Trail**: Complete documentation of factor application decisions

## 6. Quality Controls

### Data Validation Procedures
- **DCS Data Accuracy**: Regular validation of source data quality
- **Field Mapping Verification**: Ensure correct interpretation of DCS flags
- **Factor Application Audit**: Verify appropriate multipliers applied
- **Exception Review**: Manual validation of edge cases and unusual situations

### Processing Controls
- **Automated Validation**: System checks for data completeness and accuracy
- **Manual Review Triggers**: Identification of cases requiring underwriter attention
- **Hard Stop Enforcement**: Proper implementation of automatic decline rules
- **Documentation Standards**: Complete record-keeping for all decisions

## 7. Customer Communication

### Disclosure Requirements
- **Factor Explanation**: Clear description of how vehicle history affects premiums
- **Data Source Transparency**: Information about DCS data usage
- **Surcharge Justification**: Explanation of increased risk factors
- **Appeal Process**: Procedures for challenging vehicle classifications

### Educational Materials
- **Title Brand Definitions**: Explanation of different branded title types
- **Risk Factors**: How vehicle history correlates with claim risk
- **Prevention Information**: Guidance on avoiding problem vehicles
- **Purchase Recommendations**: Advice for buying used vehicles

## 8. Cross-References

### Related Rate Factors
- **Vehicle VIN Master**: Integration with vehicle identification and classification
- **Vehicle Territory**: Geographic risk factors
- **Vehicle Model Year**: Age-related risk considerations
- **Physical Damage Symbols**: Base physical damage rating

### External Data Sources
- **DCS System**: Primary data source for vehicle history
- **State Title Databases**: Verification of title brand information
- **Insurance Industry Databases**: Cross-validation of vehicle history
- **Salvage Auction Records**: Verification of auction history

### System Integration Points
- **Rating Engine**: Factor application in premium calculation
- **Underwriting System**: Hard stop implementation and exception handling
- **Policy Administration**: Real-time factor updates
- **Claims System**: Coordination with loss history data

## 9. Validation Standards

### Data Quality Standards
- **DCS Integration**: 99.5% uptime and data accuracy
- **Field Interpretation**: 100% accuracy in flag evaluation
- **Factor Application**: Correct multipliers for all coverage types
- **Hard Stop Implementation**: Proper decline triggers for severe cases

### System Performance Standards
- **Real-Time Processing**: Sub-second response for vehicle lookups
- **Data Currency**: Daily updates from DCS source system
- **Exception Resolution**: 24-hour resolution of data discrepancies
- **Audit Compliance**: Complete documentation of all factor applications

### Risk Assessment Validation
- **Actuarial Analysis**: Regular review of factor effectiveness
- **Loss Experience Tracking**: Monitoring of claim experience by category
- **Competitive Analysis**: Benchmarking against industry standards
- **Regulatory Compliance**: Adherence to state insurance requirements

## 10. Document Maintenance

### Update Procedures
- **DCS Field Changes**: Process for handling new or modified data fields
- **Factor Adjustments**: Actuarial approval for multiplier modifications
- **Hard Stop Updates**: Review and modification of automatic decline triggers
- **System Enhancements**: Documentation updates for technical improvements

### Version Control
- **Factor Modifications**: Track all changes to multiplier values
- **System Updates**: Coordinate with DCS integration changes
- **Documentation Standards**: Maintain comprehensive change history
- **Approval Process**: Required approvals for all factor modifications

### Stakeholder Communication
- **Actuarial Team**: Factor effectiveness and loss experience analysis
- **Underwriting**: Exception handling and hard stop implementation
- **IT Systems**: DCS integration maintenance and updates
- **Regulatory Affairs**: Compliance monitoring and filing requirements

### Future Enhancements
- **Enhanced Data Sources**: Integration with additional vehicle history databases
- **Predictive Modeling**: Development of more sophisticated risk scoring
- **Real-Time Updates**: Implementation of immediate factor adjustments
- **Customer Portal**: Self-service vehicle history information access

### Regulatory Considerations
- **State Compliance**: Adherence to Texas insurance regulations
- **Fair Pricing**: Actuarial justification for all surcharge levels
- **Data Privacy**: Protection of vehicle history information
- **Filing Requirements**: Regulatory approval for factor modifications