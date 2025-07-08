# 45.0 Paperless Workflow Enrollment Process Requirements

The Paperless Workflow Enrollment Process automates the transition of policyholders into a digital, paperless environment by applying a paperless discount—if eligible—rerating their policies and updating their communication preferences. The discount is available only if it is defined in the map_policy_discount table and linked to the policy via the map_program_policy table. This process ensures that only eligible policies receive the discount and that all related changes are accurately recorded and auditable.

**Core Functional Requirements**

**Discount Eligibility and Application**

- Eligibility Check:
    - Upon policy binding or update, the system verifies if the current program offers a paperless discount by checking the map_policy_discount table.
    - The discount is applied only if it is defined for the policy's program.
- Discount Mapping to Policy:
    - If eligible, the system creates or updates a mapping record in the policy-to-discount mapping table, linking the policy (via policy_id) with the discount (via discount_id).
    - Essential columns in this mapping include status indicators and audit metadata.
- Policy Re-Rating:
    - Once the discount is applied, the policy is rerated to reflect the reduced premium.
    - The re-rating ensures that the financial benefits of paperless enrollment are accurately calculated.

**Paperless Enrollment and Communication Preferences**

- Enrollment Trigger:
    - Successful discount application and policy rerating trigger the paperless enrollment process.
- Communication Preference Update:
    - The system updates the driver’s communication preference using the map_driver_communication_preference table.
    - This mapping confirms that the policyholder will receive all future communications digitally (e.g., "Receive Email Updates").
- Status Update:
    - Policy and driver records are updated to reflect the paperless enrollment status.

**Audit and Logging**

- Audit Logging:
    - Every step—from eligibility evaluation and discount mapping to policy rerating and communication preference updates—is logged for full traceability.
    - Mapping records include detailed audit metadata such as created_by, updated_by, timestamps, and status indicators

**Workflow Details and Data Mapping**

**Step-by-Step Process**

- Policy Trigger and Eligibility Check:
    - When a policy is bound or updated, the system checks the map_policy_discount table to determine if a paperless discount is available for the policy's program.
- Discount Mapping via map_program_policy:
    - If eligible, discount details are retrieved from the discount table and applied by creating or updating a mapping record that links the policy to the discount.
- Policy Re-Rating:
    - After the discount is applied, the policy is rerated so that the updated premium reflects the discount.
- Paperless Enrollment:
    - The system updates the driver’s communication preferences by creating or updating a record in the map_driver_communication_preference table.
    - This confirms enrollment in the paperless workflow, ensuring that all future communications are delivered digitally.
- Notification and User Feedback:
    - Users receive notifications via digital channels confirming the application of the paperless discount and enrollment status.
    - The user interface is updated to reflect the new paperless status.
- Audit Logging:
    - Every action taken throughout the process is recorded in the audit system, ensuring complete traceability of changes.

**Key Tables and Columns**

- Discount Table:
    - Stores discount records with fields such as id, discount_type_id, name, credit, effective_date, and expiration_date.
- Policy-to-Discount Mapping Table:
    - Maps policies to discounts using columns like policy_id, discount_id, and status/audit metadata.
- map_policy_discount:
    - Defines which discounts are available for each policy.
- map_program_policy:
    - Links programs to specific policies to ensure that only eligible policies receive the paperless discount.
- map_driver_communication_preference:
    - Maps drivers to their selected communication preferences, confirming enrollment in digital communications.

**Integration with Other Systems**

- Policy and Rating Services:
    - Integration with PolicyManager and RateManager ensures consistent application of discount benefits and policy rerating.
- Communication Services:
    - Integration with CommunicationManager supports sending notifications regarding paperless enrollment.
- Audit and Logging:
    - All process steps are integrated with centralized logging and monitoring systems for compliance and performance tracking.