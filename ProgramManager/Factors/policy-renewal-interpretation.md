# Policy Renewal Rate Factor Interpretation
## Aguila Dorada Texas Personal Auto Program

### Overview
This document provides the complete factual interpretation of the Policy Renewal rate factor for the Aguila Dorada Texas Personal Auto insurance program. This serves as the definitive reference for understanding how renewal status and prior insurance history impact premium calculations across all coverage types.

## 1. Factor Identification

### Factor Details
- **Factor Name**: Policy Renewal
- **Company**: Old American County Mutual
- **Managing General Agency**: Aguila Dorada
- **Program**: Private Passenger Auto
- **Effective Date**: New Business 07/15/2025, Renewal 08/15/2025
- **System**: New System Implementation

## 2. Renewal Factor Structure

### Factor Dimensions
**Two-Dimensional Factor Matrix:**
- **Months of Prior Insurance**: 0, 6, 12, 18, 24, 30+ months
- **Prior Insurance Discount Eligible**: Y (Yes) or N (No)
- **Coverage Application**: All coverage types (BI, PD, UMBI, UMPD, MED, PIP, COMP, COLL)

### Prior Insurance Without Discount Eligibility (N)
**Months vs. All Coverage Factors:**

#### 0 Months Prior Insurance
- **All Coverages Factor**: 1.000 (base rate)
- **Application**: New customers with no prior auto insurance
- **Risk Profile**: Highest risk due to no insurance history

#### 6 Months Prior Insurance
- **All Coverages Factor**: 0.851 (14.9% discount)
- **Application**: Customers with minimal insurance history
- **Risk Profile**: Moderate improvement over uninsured customers

#### 12 Months Prior Insurance
- **All Coverages Factor**: 0.810 (19% discount)
- **Application**: Customers with one year insurance history
- **Risk Profile**: Established insurance behavior pattern

#### 18 Months Prior Insurance
- **All Coverages Factor**: 0.770 (23% discount)
- **Application**: Customers with extended insurance history
- **Risk Profile**: Strong insurance maintenance pattern

#### 24 Months Prior Insurance
- **All Coverages Factor**: 0.731 (26.9% discount)
- **Application**: Customers with substantial insurance history
- **Risk Profile**: Well-established insurance behavior

#### 30+ Months Prior Insurance
- **All Coverages Factor**: 0.701 (29.9% discount)
- **Application**: Customers with extensive insurance history
- **Risk Profile**: Maximum discount for long-term insurance history

### Prior Insurance With Discount Eligibility (Y)
**Enhanced Discount Structure:**

#### 0 Months Prior Insurance
- **All Coverages Factor**: 1.000 (base rate)
- **Application**: New customers but eligible for prior insurance discount
- **Note**: Same as non-discount eligible for zero months

#### 6 Months Prior Insurance
- **All Coverages Factor**: 0.925 (7.5% discount)
- **Application**: Short-term prior insurance with discount eligibility
- **Benefit**: Moderate discount improvement

#### 12 Months Prior Insurance
- **All Coverages Factor**: 0.900 (10% discount)
- **Application**: One year history with discount eligibility
- **Benefit**: Better than standard 12-month rate

#### 18 Months Prior Insurance
- **All Coverages Factor**: 0.875 (12.5% discount)
- **Application**: Extended history with discount eligibility
- **Benefit**: Moderate discount progression

#### 24 Months Prior Insurance
- **All Coverages Factor**: 0.850 (15% discount)
- **Application**: Substantial history with discount eligibility
- **Benefit**: Good discount for longer history

#### 30+ Months Prior Insurance
- **All Coverages Factor**: 0.825 (17.5% discount)
- **Application**: Extensive history with discount eligibility
- **Benefit**: Best discount for maximum history and eligibility

## 3. Discount Eligibility Criteria

### Prior Insurance Discount Qualification
**Eligibility Requirements:**
- **Definition**: Specific criteria determine prior insurance discount eligibility
- **Documentation**: Prior insurance verification required
- **Quality Standards**: Prior insurance must meet quality/continuity standards
- **Time Requirements**: Specific timeframes for qualifying prior insurance

### Discount vs. Non-Discount Comparison
**Factor Differences:**
- **Without Discount**: Better rates at longer durations (0.701 at 30+ months)
- **With Discount**: Consistent but lower discount rates (0.825 at 30+ months)
- **Crossover Point**: Non-discount becomes more favorable after extended periods
- **Business Logic**: Different risk assessment methodologies

## 4. Coverage Application Rules

### Universal Coverage Impact
- **All Coverage Types**: Same renewal factor applies to all coverage types
- **Consistent Application**: No coverage-specific variations in renewal factors
- **Multiplicative Effect**: Renewal factor multiplies all coverage premiums
- **Policy-Level Factor**: Single renewal factor per policy

### Prior Insurance History Determination
- **Documentation Required**: Verification of prior insurance coverage
- **Continuity Analysis**: Assessment of insurance coverage gaps
- **Carrier Verification**: Confirmation with previous insurance carriers
- **Time Calculation**: Accurate calculation of months of prior coverage

## 5. Business Rules

### Prior Insurance Verification
**Documentation Requirements:**
- **Insurance History**: Complete prior insurance coverage history
- **Coverage Periods**: Exact dates and duration of prior coverage
- **Coverage Types**: Types of coverage maintained with prior carriers
- **Lapse Documentation**: Any gaps in coverage and reasons

### Renewal Processing
**Renewal Considerations:**
- **Continuous Coverage**: Credit for maintaining continuous coverage
- **Policy History**: Internal policy performance and claims history
- **Payment History**: Customer payment performance with company
- **Risk Assessment**: Overall customer risk profile evaluation

### New Business vs. Renewal
**Application Differences:**
- **New Business**: Prior insurance history from external carriers
- **Renewal Business**: Internal policy history and performance
- **Factor Application**: Same factor structure applies to both
- **Risk Differentiation**: Different risk profiles for new vs. renewal customers

## 6. Premium Impact Examples

### Sample Premium Calculations
**Base Premium (Before Renewal Factor): $1,200**

#### No Prior Insurance (0 Months)
- **Premium**: $1,200 × 1.000 = $1,200
- **Impact**: No discount, full base premium
- **Risk Level**: Highest risk category

#### 6 Months Prior (No Discount Eligibility)
- **Premium**: $1,200 × 0.851 = $1,021.20
- **Savings**: $178.80 (14.9% discount)
- **Risk Level**: Moderate risk reduction

#### 12 Months Prior (No Discount Eligibility)
- **Premium**: $1,200 × 0.810 = $972
- **Savings**: $228 (19% discount)
- **Risk Level**: Established insurance behavior

#### 30+ Months Prior (No Discount Eligibility)
- **Premium**: $1,200 × 0.701 = $841.20
- **Savings**: $358.80 (29.9% discount)
- **Risk Level**: Maximum discount for long history

#### 30+ Months Prior (With Discount Eligibility)
- **Premium**: $1,200 × 0.825 = $990
- **Savings**: $210 (17.5% discount)
- **Comparison**: Less discount than non-eligible at same duration

## 7. Risk Assessment Logic

### Insurance History Risk Correlation
**Risk Factors:**
- **Uninsured Period**: Higher risk due to lack of financial responsibility
- **Short History**: Limited evidence of insurance commitment
- **Extended History**: Strong evidence of responsible insurance behavior
- **Continuous Coverage**: Lower risk due to consistent coverage maintenance

### Prior Insurance Discount Paradox
**Business Logic Analysis:**
- **Discount Eligibility**: May indicate different risk pool or qualification criteria
- **Rate Structure**: Different actuarial assumptions for discount-eligible customers
- **Risk Segmentation**: Separate risk assessment for different customer types
- **Competitive Positioning**: Market-driven discount structure considerations

## 8. System Implementation

### Data Requirements
- **Prior Insurance Database**: Complete prior insurance history tracking
- **Verification Systems**: Integration with insurance verification services
- **Documentation Storage**: Storage of prior insurance verification documents
- **Calculation Engine**: Automated renewal factor calculation and application

### Processing Requirements
1. **History Collection**: Gather complete prior insurance history
2. **Verification Process**: Verify prior insurance claims with carriers
3. **Factor Determination**: Determine appropriate renewal factors
4. **Application Process**: Apply renewal factors to all coverage types

## 9. Quality Controls

### Validation Procedures
- **History Accuracy**: Verify accuracy of prior insurance history
- **Factor Application**: Confirm correct renewal factors applied
- **Discount Eligibility**: Validate prior insurance discount eligibility determination
- **Coverage Consistency**: Ensure factor applied consistently across all coverages

### Exception Handling
- **Missing Documentation**: Process for handling incomplete prior insurance history
- **Verification Failures**: Procedures when prior insurance cannot be verified
- **Dispute Resolution**: Process for handling customer disputes over prior insurance credit
- **System Errors**: Error handling for renewal factor calculation failures

## 10. Customer Communication

### Prior Insurance Benefits
- **Discount Explanation**: Clear explanation of prior insurance discount benefits
- **Documentation Requirements**: Communication of required prior insurance documentation
- **Verification Process**: Explanation of prior insurance verification procedures
- **Factor Impact**: Transparency about how prior insurance affects premiums

### Customer Education
- **Insurance Continuity**: Importance of maintaining continuous auto insurance coverage
- **Documentation Retention**: Need to retain prior insurance documentation
- **Verification Cooperation**: Customer cooperation required for prior insurance verification
- **Discount Maximization**: How to qualify for maximum prior insurance discounts

## Cross-References
- **Underwriting Guidelines**: See underwriting documentation for prior insurance verification procedures
- **Customer Service Standards**: See customer service procedures for prior insurance documentation handling
- **Competitive Analysis**: See market analysis for prior insurance discount competitiveness
- **Actuarial Documentation**: See actuarial analysis supporting renewal factor development

## Validation Standards
This document serves as the authoritative source for:
- **Renewal Factors**: Definitive prior insurance-based rating factors
- **Application Rules**: Renewal factor selection and application methodology
- **Business Rules**: Prior insurance verification and discount eligibility procedures
- **System Requirements**: Technical renewal factor implementation specifications

## Document Maintenance
- **Updates**: Changes to renewal factors require document updates
- **Version Control**: Maintain version history for actuarial consistency
- **Approval**: All renewal factor changes require actuarial approval
- **Distribution**: Updates communicated to all underwriting and customer service stakeholders