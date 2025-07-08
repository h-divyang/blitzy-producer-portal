# GR-64: Policy Reinstatement with Lapse Process

## Overview
This Global Requirement establishes comprehensive standards for policy reinstatement processes within insurance management systems. Policy reinstatement allows customers to restore canceled policies within a specified time window, subject to specific business rules and premium adjustments. This requirement provides a program-agnostic framework that can be implemented across multiple insurance programs while maintaining regulatory compliance and operational efficiency.

## 1. Reinstatement Eligibility and Business Rules

### Purpose and Definition
- **Policy Reinstatement**: Process of restoring a canceled insurance policy to active status within a specified timeframe
- **Lapse Period**: Time duration between policy cancellation and reinstatement during which no coverage exists
- **Regulatory Compliance**: Meets state-specific requirements for policy reinstatement procedures

### Eligibility Criteria Framework
- **Cancellation Reason Restriction**: Reinstatement eligibility based on reason for cancellation (program-configurable)
- **Time Window Limitation**: Configurable time limit for reinstatement eligibility (typically 30 days)
- **Policy Status Validation**: Must be in appropriate canceled status to be eligible for reinstatement
- **Payment Requirements**: All outstanding amounts must be paid to trigger reinstatement

### Core Business Rules
1. **No Coverage During Lapse**: No insurance coverage exists during the lapse period
2. **No Backdating**: Coverage effective date is reinstatement date, not cancellation date
3. **Premium Adjustment**: Premium must be recalculated to remove lapsed period premium
4. **Payment Triggered**: Reinstatement becomes effective upon successful payment
5. **Eligibility Expiration**: Policies become ineligible for reinstatement after time window expires

## 2. Reinstatement Workflow Architecture

### State Management Framework
Policy lifecycle states must support reinstatement workflow:

```
Active Policy States:
- active → cancelled (cancellation event)
- cancelled → eligible_for_reinstatement (if within time window and eligible reason)
- eligible_for_reinstatement → reinstated (upon successful payment)
- eligible_for_reinstatement → expired_reinstatement (after time window)
- reinstated → active (final state transition)
```

### Workflow Processing Requirements
1. **Cancellation Event Processing**:
   - Evaluate cancellation reason for reinstatement eligibility
   - Set reinstatement window expiration date
   - Suspend coverage and billing processes
   - Mark policy as eligible or ineligible for reinstatement

2. **Reinstatement Request Processing**:
   - Validate current eligibility status
   - Calculate reinstatement premium and fees
   - Present payment options to customer
   - Process payment and trigger reinstatement

3. **Reinstatement Completion**:
   - Update policy status to active
   - Restore coverage with new effective date
   - Restructure payment schedule if applicable
   - Generate confirmation documentation

### Time-Based Processing
- **Eligibility Window Monitoring**: System must track reinstatement eligibility expiration
- **Automated Status Updates**: Automatically expire reinstatement eligibility after time window
- **Real-time Calculations**: Premium calculations must account for exact timing of reinstatement

## 3. Premium Calculation Architecture

### Daily Rate Methodology
All reinstatement premium calculations based on daily rate approach:

```
Daily Premium Rate = Total Policy Premium ÷ Total Policy Term Days
Lapsed Premium = Daily Premium Rate × Lapse Days
Adjusted Total Premium = Original Premium - Lapsed Premium
```

### Reinstatement Amount Calculation
Complete reinstatement amount includes:

1. **Remaining Policy Premium**: Adjusted premium for remaining term
2. **Unpaid Prior Premium**: Any amounts due before cancellation
3. **Reinstatement Fees**: Program-specific fees for reinstatement processing
4. **Applicable Taxes**: Tax recalculation based on adjusted premium

### Payment Schedule Restructuring
- **Installment Recalculation**: Distribute remaining balance across remaining payment periods
- **Immediate Payment Rules**: Handle cases where next payment is due within specified days
- **Final Payment Handling**: Manage scenarios where only one payment remains
- **Payment Method Integration**: Support all standard payment methods

## 4. Integration Requirements

### Workflow System Integration (GR-18)
- **State Machine Integration**: Implement reinstatement states in workflow engine
- **Conditional Transitions**: Support program-specific eligibility rules
- **Time-Based Triggers**: Handle automatic state transitions based on time windows
- **Event Processing**: Generate appropriate events for reinstatement lifecycle

### Business Logic Integration (GR-20)
```php
interface ReinstatementServiceInterface
{
    public function validateEligibility(Policy $policy): ReinstatementEligibility;
    public function calculateReinstatementAmount(Policy $policy, Carbon $reinstatementDate): ReinstatementCalculation;
    public function processReinstatement(Policy $policy, Payment $payment): Policy;
    public function restructurePaymentSchedule(Policy $policy, Money $remainingBalance): PaymentSchedule;
}

class ReinstatementService implements ReinstatementServiceInterface
{
    public function validateEligibility(Policy $policy): ReinstatementEligibility
    {
        // Validate cancellation reason, time window, and current status
        // Return eligibility status with reasons if ineligible
    }
    
    public function calculateReinstatementAmount(Policy $policy, Carbon $reinstatementDate): ReinstatementCalculation
    {
        // Calculate daily rate, lapse period, adjusted premium
        // Include unpaid amounts and applicable fees
        // Return complete calculation breakdown
    }
    
    public function processReinstatement(Policy $policy, Payment $payment): Policy
    {
        // Validate payment amount matches calculation
        // Update policy status and effective dates
        // Trigger related system updates
        // Return updated policy
    }
}
```

### Action Tracking Integration (GR-37)
Required audit events for reinstatement processing:

```
Action Types:
- POLICY_REINSTATEMENT_ELIGIBILITY_EVALUATED
- POLICY_REINSTATEMENT_CALCULATION_PERFORMED
- POLICY_REINSTATEMENT_PAYMENT_RECEIVED
- POLICY_REINSTATEMENT_COMPLETED
- POLICY_REINSTATEMENT_FAILED
- POLICY_REINSTATEMENT_ELIGIBILITY_EXPIRED
```

### State Management Integration (GR-09)
Frontend state management patterns:

```javascript
// React hooks for reinstatement workflow
const useReinstatementEligibility = (policyId) => {
    // Real-time eligibility checking
    // Time window countdown
    // Eligibility status updates
};

const useReinstatementCalculation = (policy, reinstatementDate) => {
    // Dynamic premium calculation
    // Payment amount breakdown
    // Fee calculation display
};

const useReinstatementProcess = () => {
    // Reinstatement workflow management
    // Payment processing coordination
    // Status update handling
};
```

## 5. SR22 Filing Considerations (GR-10)

### SR22 Continuity During Reinstatement
- **Filing Continuation**: Active SR22 filings remain in effect during policy lapse periods
- **No New Filing Required**: Policy reinstatement does not trigger new SR22 filing
- **Status Maintenance**: SR22 status and state filing continue uninterrupted
- **Fee Integration**: SR22 fees included in premium calculations if applicable

### Business Rules for SR22 Policies
1. **Legal Requirement Continuity**: SR22 represents legal requirement independent of policy status
2. **State Notification**: No state notification required for policy reinstatement
3. **Premium Calculation**: Include SR22 fees in reinstatement amount calculations
4. **Documentation**: SR22 documents remain valid during lapse period

## 6. Database Schema Requirements (GR-41)

### Status Management Extensions
```sql
-- Additional status values for reinstatement workflow
INSERT INTO status (code, name, description, is_active) VALUES
('ELIGIBLE_FOR_REINSTATEMENT', 'Eligible for Reinstatement', 'Policy cancelled but within reinstatement window', true),
('REINSTATED', 'Reinstated', 'Policy successfully reinstated after cancellation', true),
('EXPIRED_REINSTATEMENT', 'Reinstatement Expired', 'Policy reinstatement window has expired', false);
```

### Reinstatement Tracking Schema
```sql
CREATE TABLE reinstatement_calculation (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    policy_id BIGINT UNSIGNED NOT NULL,
    
    -- Calculation details
    cancellation_date DATE NOT NULL,
    reinstatement_date DATE NOT NULL,
    lapse_days INTEGER NOT NULL,
    
    -- Premium breakdown
    original_premium DECIMAL(10,2) NOT NULL,
    daily_premium_rate DECIMAL(8,4) NOT NULL,
    lapsed_premium DECIMAL(10,2) NOT NULL,
    adjusted_premium DECIMAL(10,2) NOT NULL,
    unpaid_premium DECIMAL(10,2) DEFAULT 0,
    reinstatement_fees DECIMAL(10,2) DEFAULT 0,
    total_due DECIMAL(10,2) NOT NULL,
    
    -- Status and audit
    status_id BIGINT UNSIGNED NOT NULL,
    created_by BIGINT UNSIGNED NOT NULL,
    updated_by BIGINT UNSIGNED NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    -- Foreign key constraints
    FOREIGN KEY (policy_id) REFERENCES policy(id),
    FOREIGN KEY (status_id) REFERENCES status(id),
    FOREIGN KEY (created_by) REFERENCES user(id),
    FOREIGN KEY (updated_by) REFERENCES user(id),
    
    -- Indexes
    INDEX idx_policy (policy_id),
    INDEX idx_reinstatement_date (reinstatement_date),
    INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

## 7. Program-Specific Implementation

### Aguila Dorada Texas Personal Auto (GR-63)
Specific implementation requirements for the Aguila Dorada program:

- **Eligibility**: Only nonpayment cancellations eligible for reinstatement
- **Time Window**: 30 days from cancellation date (absolute limit)
- **Rewrite Requirement**: Policies canceled more than 30 days require new business
- **Fee Structure**: $25 reinstatement fee plus standard endorsement fees
- **Premium Calculation**: Daily rate methodology with exact day calculations
- **SR22 Integration**: Maintain SR22 status during reinstatement process

### Program Configuration Framework
```php
// Program-specific configuration
class ProgramReinstatementConfig
{
    public array $eligibleCancellationReasons = ['NONPAYMENT'];
    public int $reinstatementWindowDays = 30;
    public Money $reinstatementFee;
    public bool $allowBackdating = false;
    public bool $requireFullPayment = true;
}
```

## 8. Communication and Documentation (GR-44)

### Customer Communication Requirements
- **Cancellation Notice**: Must include reinstatement rights and timeline information
- **Reminder Communications**: Optional reminders during reinstatement window
- **Reinstatement Confirmation**: Automatic confirmation upon successful reinstatement
- **Eligibility Expiration**: Notification when reinstatement window expires

### Document Generation Requirements
- **Reinstatement Certificate**: Generated upon successful completion
- **Updated Policy Documents**: Reflect new effective dates and premium amounts
- **Payment Receipt**: Detailed breakdown of reinstatement payment
- **Installment Schedule**: New payment schedule if applicable

## 9. Performance and Monitoring Requirements

### Performance Standards
- **Eligibility Validation**: < 200ms for eligibility determination
- **Premium Calculation**: < 500ms for complete reinstatement calculation
- **Payment Processing**: < 2 seconds for payment validation and processing
- **Status Updates**: < 100ms for policy status transitions

### Monitoring and Alerting
- **Eligibility Expiration**: Monitor approaching expiration dates
- **Payment Failures**: Alert on failed reinstatement payments
- **Calculation Errors**: Monitor for premium calculation discrepancies
- **Processing Volumes**: Track reinstatement request volumes and success rates

## 10. Cross-References and Dependencies

### Related Global Requirements
- **GR-18**: Workflow Requirements - State machine and workflow patterns
- **GR-20**: Business Logic Standards - Service architecture and patterns
- **GR-37**: Action Tracking - Audit trail and logging requirements
- **GR-09**: State Management - Frontend state patterns
- **GR-41**: Table Schema - Database design standards
- **GR-17**: Functional Requirements - High-level functional architecture
- **GR-10**: SR22/SR26 Filing - Financial responsibility filing considerations
- **GR-63**: Aguila Dorada Program - Program-specific implementation

### External Dependencies
- **Payment Processing Systems**: Integration for reinstatement payments
- **Document Generation Systems**: Reinstatement document creation
- **Communication Systems**: Customer notification and correspondence
- **Billing Systems**: Payment schedule restructuring and management

### System Integration Points
- **Policy Management**: Core policy status and data management
- **Premium Calculation**: Rating engine integration for adjusted premiums
- **Payment Processing**: Financial transaction handling and validation
- **Customer Portal**: Reinstatement request and payment interfaces

## Implementation Guidelines

### Technical Implementation
- **Service Architecture**: Implement ReinstatementService following established patterns
- **API Design**: RESTful endpoints for reinstatement workflow management
- **Database Design**: Extend existing schema with reinstatement-specific tables
- **Security Controls**: Protect sensitive reinstatement data and calculations

### Business Implementation
- **Process Documentation**: Document detailed reinstatement procedures
- **Training Materials**: Develop training for customer service and agents
- **Quality Assurance**: Implement validation processes for calculations and workflows
- **Compliance Procedures**: Establish regulatory compliance verification

### Testing Requirements
- **Unit Testing**: Test individual reinstatement components and calculations
- **Integration Testing**: Test integration with policy and payment systems
- **Business Rule Testing**: Validate program-specific business rules
- **Performance Testing**: Verify response time and throughput requirements

## Compliance and Audit

### Regulatory Compliance
- **State Requirements**: Ensure compliance with state reinstatement regulations
- **Documentation Standards**: Maintain comprehensive audit trails
- **Consumer Protection**: Implement fair and transparent reinstatement practices
- **Financial Responsibility**: Accurate premium calculations and fee assessments

### Audit Requirements
- **Complete Audit Trail**: Track all reinstatement activities and decisions
- **Calculation Validation**: Maintain detailed records of premium calculations
- **Payment Tracking**: Comprehensive payment processing audit trails
- **Compliance Reporting**: Generate reports for regulatory compliance verification

This Global Requirement provides the foundation for implementing comprehensive policy reinstatement capabilities across multiple insurance programs while maintaining regulatory compliance, operational efficiency, and system performance standards.