# #1 - Policy Summary Overview

## **A) WHY - Vision and Purpose**

The **Policy Summary Overview** provides users with a **comprehensive view of their insurance policy details, payment status, and applicable discounts**. This feature ensures **clarity, transparency, and accessibility**, allowing users to quickly retrieve essential policy-related information and take necessary actions, such as making a payment or contacting their agent.

## **B) WHAT - Core Requirements**

### **1. Policy Information Display**

- Display key **policy details**, including:
    - **Policy Number**
        - Policy Number → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `policy` by [`policy.id`](http://policy.id/) -> get `policy_prefix` by `policy.policy_prefix_id` -> concat `policy_prefix.value` and `policy.number`
    - **Policyholder Name**
        - Named Insured Name → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `map_policy_named_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by `map_policy_named_driver.driver_id` -> get `driver` by [`driver.id](http://driver.id/)` -> get `name` by `driver.name_id`
    - **Effective Date & Expiration Date**
        - Effective Date → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by `user.id` -> get policy by [`policy.id`](http://policy.id/) -> `policy.effective_date`
        - Expiration Date → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `policy` by [`policy.id`](http://policy.id/) -> `policy.expiration_date`
    - **Insurance Type & Coverage Summary**
        - Coverage Type → get `map_policy_driver_vehicle` by [`policy.id`](http://policy.id/) -> get `map_policy_coverage_limit` by [`policy.id`](http://policy.id/) -> get `map_vehicle_coverage_limit` by `map_policy_driver_vehicle.vehicle_id` -> get `map_driver_coverage_limit` by `map_policy_driver_vehicle.driver_id` -> get [`coverage.id`](http://coverage.id/) by `map_policy_coverage_limit.coverage_id` and `map_vehicle_coverage_limit.coverage_id` and `map_driver_coverage_limit.coverage_id` -> get `coverage` by [`coverage.id`](http://coverage.id/) -> get `coverage_type.id` by `coverage.coverage_type_id` -> get `coverage_type.name` by `coverage_type.id`
        - Coverage Limit → get `map_policy_driver_vehicle` by [`policy.id`](http://policy.id/) -> get `map_policy_coverage_limit` by [`policy.id`](http://policy.id/) -> get `map_vehicle_coverage_limit` by `map_policy_driver_vehicle.vehicle_id` -> get [`coverage.id`](http://coverage.id/) by `map_policy_coverage_limit.coverage_id` and `map_vehicle_coverage_limit.coverage_id` -> get [`limit.id`](http://limit.id/) by `map_policy_coverage_limit` and `map_vehicle_coverage_limit`
        - Coverage Premium → get `map_policy_driver_vehicle` by [`policy.id`](http://policy.id/) -> get `map_policy_coverage_limit` by [`policy.id`](http://policy.id/) -> get `map_vehicle_coverage_limit` by `map_policy_driver_vehicle.vehicle_id` -> get `map_driver_coverage_limit` by `map_policy_driver_vehicle.driver_id` -> get [`coverage.id`](http://coverage.id/) by `map_policy_coverage_limit.coverage_id` and `map_vehicle_coverage_limit.coverage_id` and `map_driver_coverage_limit.coverage_id` -> get `coverage` by [`coverage.id`](http://coverage.id/) -> get `map_coverage_premium` by [`coverage.id`](http://coverage.id/) -> get [`premium.id`](http://premium.id/) by `map_coverage_premium.premium_id` -> get `premium` by [`premium.id`](http://premium.id/) -> get `sum` of `premium.credit` and `premium.debit`

### **2. Payment Information**

- Show the **total premium amount** and **remaining balance**.
    - Total Policy Premium
        - Retrieve the policy record:
            - Use `map_user_policy_driver` by `user.id` to obtain `policy.id`.
            - Retrieve the policy record from `policy` to access policy details.
        - Retrieve premium records:
            - Use `map_policy_premium_transaction` where `policy_id` = `policy.id` to fetch all associated premium records.
        - Calculate Base Premium:
            - For each premium record retrieved from premium, compute:
                - `premium_net` = `premium.credit` – `premium.debit`.
        - Compute Total Term Premium:
            - Sum all `premium_net` values from the premium records to obtain the Total Term Premium Due for the entire term.
    - Remaining Premium Balance
        - get `policy.id` from `map_user_policy_driver` by `user.id` -> get `policy` by `policy.id` -> `map_policy_transaction` -> get all `transaction_id`'s -> foreach `transaction.id` -> sum of `transaction.credit` and `transaction.debit` filtered by `transaction_type_id`'s to include
- Display the **next payment amount and due date**.
    - Next Payment Amount
        - Retrieve the policy record:
            - Use `map_user_policy_driver` by `user.id` to obtain `policy.id`.
            - Retrieve the policy record from policy to access `installment_due_day` and `term_months`.
        - Aggregate premium data:
            - Use `map_policy_premium_transaction` where `policy_id` = `policy.id`.
            - For each premium record from premium, calculate:

              `premium_net` = `premium.credit` – `premium.debit`.

            - Sum all `premium_net` values to obtain Total Premium.
        - Aggregate fee data:
            - Use `map_policy_fee_transaction` where `policy_id` = `policy.id`.
            - For each fee record from fee, calculate:

              `fee_net` = `fee.debit` – `fee.credit`.

            - Sum all `fee_net` values to obtain Total Fees.
        - Aggregate payment transactions:
            - Use `map_policy_installment_transaction` where `policy_id` = `policy.id`.
            - For each transaction from transaction, sum the debit amounts (payments are recorded as debits).
            - Sum these amounts to get Total Payments Applied.
        - Compute net premium remaining:
            - Net Premium Remaining = (Total Premium + Total Fees) – (Total Payments Applied).
        - Determine the remaining installment count:
            - Count the number of installment records for the policy via `map_policy_installment_transaction`.
            - Remaining Installments = `policy.term_months` – (count of installment records).
        - Calculate Next Payment Amount:
            - If an installment record exists (with status "Pending" or "Partially Paid"):
                - Next Payment Amount = `installment.total` – `installment.paid_to_date`.
            - If no installment record exists:
                - Next Installment Amount = Net Premium Remaining / Remaining Installments.
    - Due Date
        - Retrieve the policy record:
            - Use `map_user_policy_driver` by `user.id` to obtain `policy.id`.
            - Retrieve the policy record from `policy` to access `installment_due_day`.
        - Check for outstanding items:
            - Retrieve any installment records for the policy via `map_policy_installment_transaction` with status "Pending" or "Partially Paid" and extract their `due_date` values.
            - Retrieve fee records via `map_policy_fee_transaction` (if fees have due dates and outstanding balances) and extract their `due_date` values.
            - Identify the earliest due date among these outstanding installments/fees.
        - Calculate the next billing date dynamically:
            - If today’s date is before the `installment_due_day`, then set the calculated next due date to the current month’s `installment_due_day`.
                - Example: If today is the 10th and `installment_due_day` is 15, then next due date = the 15th of the current month.
            - If today’s date is on or after the `installment_due_day`, then set the calculated next due date to the `installment_due_day` in the next month.
                - Example: If today is the 16th and `installment_due_day` is 15, then next due date = the 15th of the next month.
        - Determine the effective due date:
            - Compare the dynamically calculated due date with the earliest due date from outstanding installments/fees (if any exist).
            - The Next Payment Due Date is the earliest of:
                - The dynamically calculated next due date based on `installment_due_day`, or
                - The earliest `due_date` among outstanding installment/fee records.
        - Set the due date in the new installment record:
            - When generating a new installment record (via a cronjob or event trigger), assign its `due_date` field to the determined Next Payment Due Date.
        - This `due_date` becomes the Due Date of the Next Payment.


### **3. User Information & Contact Details**

- Show **policyholder’s address, email, and phone number**.
    - Named Insured Address → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `map_policy_named_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by `map_policy_named_driver.driver_id` -> get `map_driver_address` by [`driver.id`](http://driver.id/) -> get [`address.id`](http://address.id/) by `map_driver_address.address_id` -> get `address` by `address_id`
    - Named Insured Email → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `map_policy_named_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by `map_policy_named_driver.driver_id` -> get `map_driver_email_address` by [`driver.id`](http://driver.id/) -> get `email_address.id` by `map_driver_email_address.email_address_id` -> get `email_address` by `email_address_id`
    - Named Insured Primary Phone Number → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `map_policy_named_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by `map_policy_named_driver.driver_id` -> get `map_driver_contact_number` by [`driver.id`](http://driver.id/) -> get `contact_number.id` by `map_driver_contact_number.contact_number_id` -> get `contact_number` by `contact_number_id` and `contact_number.contact_number_type` for primary
    - Named Insured Alternate Phone Number → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `map_policy_named_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by `map_policy_named_driver.driver_id` -> `get map_driver_contact_number` by [driver.id](http://driver.id/) -> get `contact_number.id` by `map_driver_contact_number.contact_number_id` -> get `contact_number` by `contact_number_id` and `contact_number.contact_number_type` for alternate
- Display **insurance agent contact details** for quick assistance.
    - Producer Phone Number → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `map_producer_policy` by [`policy.id`](http://policy.id/) -> get [`producer.id`](http://producer.id/) by `map_producer_policy.producer_id` -> get `map_producer_contact_number` by [`producer.id`](http://producer.id/) -> get `contact_number` by `map_producer_contact_number.contact_number_id`
    - Producer Name → get `policy.id` from `map_user_policy_driver` by `user.id` -> get `map_producer_policy` by `policy.id` -> get `producer.id` by `map_producer_policy.producer_id` -> get `map_producer_name` by `producer.id`
    - Producer Address → get `policy.id` from `map_user_policy_driver` by `user.id` -> get `map_producer_policy` by `policy.id` -> get `producer.id` by `map_producer_policy.producer_id` -> get `address.id` from `map_producer_address` by `producer.id` -> get address by `address.id`

### **4. Interactive Features**

- Provide a **“Make Payment” button** for seamless bill payment.
- Tooltips to explain key policy terms (e.g., expiration date, remaining balance).

## **C) HOW - Planning & Implementation**

**Full Mapping of Front-End Fields to Database Fields**

- Producer Label Configuration → Stored in the appropriate application side configuration file.
- Policy Number → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `policy` by [`policy.id`](http://policy.id/) -> get `policy_prefix` by `policy.policy_prefix_id` -> concat `policy_prefix.value` and `policy.number`
- Policy Status → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `policy` by [`policy.id`](http://policy.id/) -> `policy.status_id`
- Named Insured Name → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `map_policy_named_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by m`ap_policy_named_driver.driver_id` -> get `driver` by [`driver.id](http://driver.id/)` -> get `name` by `driver.name_id`
- Next Payment Amount
    - Retrieve the policy record:
        - Use `map_user_policy_driver` by `user.id` to obtain `policy.id`.
        - Retrieve the policy record from policy to access `installment_due_day` and `term_months`.
    - Aggregate premium data:
        - Use `map_policy_premium_transaction` where `policy_id` = `policy.id`.
        - For each premium record from premium, calculate:

          `premium_net` = `premium.credit` – `premium.debit`.

        - Sum all `premium_net` values to obtain Total Premium.
    - Aggregate fee data:
        - Use `map_policy_fee_transaction` where `policy_id` = `policy.id`.
        - For each fee record from fee, calculate:

          `fee_net` = `fee.debit` – `fee.credit`.

        - Sum all `fee_net` values to obtain Total Fees.
    - Aggregate payment transactions:
        - Use `map_policy_installment_transaction` where `policy_id` = `policy.id`.
        - For each transaction from transaction, sum the debit amounts (payments are recorded as debits).
        - Sum these amounts to get Total Payments Applied.
    - Compute net premium remaining:
        - Net Premium Remaining = (Total Premium + Total Fees) – (Total Payments Applied).
    - Determine the remaining installment count:
        - Count the number of installment records for the policy via `map_policy_installment_transaction`.
        - Remaining Installments = `policy.term_months` – (count of installment records).
    - Calculate Next Payment Amount:
        - If an installment record exists (with status "Pending" or "Partially Paid"):
            - Next Payment Amount = `installment.total` – `installment.paid_to_date`.
        - If no installment record exists:
            - Next Installment Amount = Net Premium Remaining / Remaining Installments.
    - Table Summary:
        - `policy` (`id`, `installment_due_day`, `term_months`)
        - `map_user_policy_driver` (`user.id`, `policy.id`)
        - `map_policy_premium_transaction` (`policy_id`, `premium_id`)
        - `premium` (`credit`, `debit`)
        - `map_policy_fee_transaction` (`policy_id`, `fee_id`)
        - `fee` (`debit`, `credit`)
        - `map_policy_installment_transaction` (`policy_id`, `installment_id`)
        - `transaction` (`debit`, `processed_date`)
        - `installment` (`total`, `paid_to_date`, `due_date`, `status_id`)
- Due Date of Next Payment
    - Retrieve the policy record:
        - Use `map_user_policy_driver` by `user.id` to obtain `policy.id`.
        - Retrieve the policy record from `policy` to access `installment_due_day`.
    - Check for outstanding items:
        - Retrieve any installment records for the policy via `map_policy_installment_transaction` with status "Pending" or "Partially Paid" and extract their `due_date` values.
        - Retrieve fee records via `map_policy_fee_transaction` (if fees have due dates and outstanding balances) and extract their `due_date` values.
        - Identify the earliest due date among these outstanding installments/fees.
    - Calculate the next billing date dynamically:
        - If today’s date is before the `installment_due_day`, then set the calculated next due date to the current month’s `installment_due_day`.
            - Example: If today is the 10th and `installment_due_day` is 15, then next due date = the 15th of the current month.
        - If today’s date is on or after the `installment_due_day`, then set the calculated next due date to the `installment_due_day` in the next month.
            - Example: If today is the 16th and `installment_due_day` is 15, then next due date = the 15th of the next month.
    - Determine the effective due date:
        - Compare the dynamically calculated due date with the earliest due date from outstanding installments/fees (if any exist).
        - The Next Payment Due Date is the earliest of:
            - The dynamically calculated next due date based on `installment_due_day`, or
            - The earliest `due_date` among outstanding installment/fee records.
    - Set the due date in the new installment record:
        - When generating a new installment record (via a cronjob or event trigger), assign its `due_date` field to the determined Next Payment Due Date.
    - This `due_date` becomes the Due Date of the Next Payment.
    - Table Summary:
        - `policy` (`id`, `installment_due_day`)
        - `map_user_policy_driver` (`user.id`, `policy.id`)
        - `map_policy_installment_transaction` (`policy_id`, `installment_id`) and `installment` (`due_date`, `status_id`)
        - `map_policy_fee_transaction` (`policy_id`, `fee_id`)
        - `fee` (`due_date`)
- Named Insured Address → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `map_policy_named_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by `map_policy_named_driver.driver_id` -> get `map_driver_address` by [`driver.id`](http://driver.id/) -> get [`address.id`](http://address.id/) by `map_driver_address.address_id` -> get `address` by `address_id`
- Named Insured Email → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `map_policy_named_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by `map_policy_named_driver.driver_id` -> get `map_driver_email_address` by [`driver.id`](http://driver.id/) -> get `email_address.id` by `map_driver_email_address.email_address_id` -> get `email_address` by `email_address_id`
- Named Insured Primary Phone Number → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `map_policy_named_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by `map_policy_named_driver.driver_id` -> get `map_driver_contact_number` by [`driver.id`](http://driver.id/) -> get `contact_number.id` by `map_driver_contact_number.contact_number_id` -> get `contact_number` by `contact_number_id` and `contact_number.contact_number_type` for primary
- Named Insured Alternate Phone Number → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `map_policy_named_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by `map_policy_named_driver.driver_id` -> `get map_driver_contact_number` by [driver.id](http://driver.id/) -> get `contact_number.id` by `map_driver_contact_number.contact_number_id` -> get `contact_number` by `contact_number_id` and `contact_number.contact_number_type` for alternate
- Effective Date → get [`policy.id`](http://policy.id/) from map_user_policy_driver by [user.id](http://user.id/) -> get policy by [`policy.id`](http://policy.id/) -> `policy.effective_date`
- Inception Date → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `policy` by [`policy.id`](http://policy.id/) -> `policy.inception_date`
- Expiration Date → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `policy` by [`policy.id`](http://policy.id/) -> `policy.expiration_date`
- Total Policy Premium → get [`policy.id`](http://`policy.id`/) from `map_user_policy_driver` by [`user.id`](http://`user.id`/) -> get `policy` by [`policy.id`](http://`policy.id`/) -> `map_policy_transaction` -> get all `transaction_id`'s -> foreach [`transaction.id`](http://`transaction.id`/) -> sum of `transaction.credit` and `transaction.debit` filtered by `transaction_type_id`'s to include
- Remaining Premium Balance → get [`policy.id`](http://`policy.id`/) from `map_user_policy_driver` by [`user.id`](http://`user.id`/) -> get `policy` by [`policy.id`](http://`policy.id`/) -> `map_policy_transaction` -> get all `transaction_id`'s -> foreach [`transaction.id`](http://`transaction.id`/) -> sum of `transaction.credit` and `transaction.debit` filtered by `transaction_type_id`'s to include
- Policy Term → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `policy` by [`policy.id`](http://policy.id/) -> `policy.term_id`
- Payment Method → get [`driver.id`](http://driver.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `driver` by [`driver.id`](http://driver.id/) -> `driver.payment_type_id`
- Applied Discounts → get `map_policy_driver_vehicle` by `policy.id` -> get `driver.id` and `vehicle.id` by `map_policy_driver_vehicle.driver_id` and `map_policy_driver_vehicle.vehicle_id` -> get `map_policy_discount` and get `map_vehicle_discount` and get `map_driver_discount` by `driver.id` and `vehicle.id` and `policy.id` -> get `discount.id` by `map_driver_discount.discount_id` and `map_vehicle_discount.discount_id` -> sum `discount.credit` and `discount.debit`
- Producer Phone Number → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `map_producer_policy` by [`policy.id`](http://policy.id/) -> get [`producer.id`](http://producer.id/) by `map_producer_policy.producer_id` -> get `map_producer_contact_number` by [`producer.id`](http://producer.id/) -> get `contact_number` by `map_producer_contact_number.contact_number_id`
- Producer Name → get `policy.id` from `map_user_policy_driver` by `user.id` -> get `map_producer_policy` by `policy.id` -> get `producer.id` by `map_producer_policy.producer_id` -> get `map_producer_name` by `producer.id`
- Producer Address → get `policy.id` from `map_user_policy_driver` by `user.id` -> get `map_producer_policy` by `policy.id` -> get `producer.id` by `map_producer_policy.producer_id` -> get `address.id` from `map_producer_address` by `producer.id` -> get address by `address.id`
- Driver Name → get `map_policy_driver` by `policy.id` -> get `driver.id` by `map_policy_driver.driver_id` -> get driver by `driver.id` -> get `name.id` by `driver.name_id` -> get `name`
- Primary Driver Tag → get `map_policy_named_driver` by `policy.id` -> get `driver.id` by `map_policy_named_driver.driver_id` -> named driver is the primary driver
- Driver Status (Included vs Excluded) → get `map_policy_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by `map_policy_driver.driver_id` -> get `driver` by [`driver.id`](http://driver.id/) -> get `driver_type.id` by `driver.driver_type_id` -> driver types defined for included and excluded
- Driver Date of Birth → get `map_policy_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by `map_policy_driver.driver_id` -> get `driver` by [`driver.id`](http://driver.id/) -> get `driver.date_of_birth`
- Driver’s License Number → get `map_policy_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by `map_policy_driver.driver_id` -> get `driver` by [`driver.id`](http://driver.id/) -> get [`license.id`](http://license.id/) by `driver.license_id` -> get `license` by [`license.id`](http://license.id/)
- Issuing State for License → get `map_policy_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by `map_policy_driver.driver_id` -> get `driver` by [`driver.id`](http://driver.id/) -> get [`license.id`](http://license.id/) by `driver.license_id` -> get `license` by [`license.id`](http://license.id/) -> get [`state.id`](http://state.id/) by `license.state_id` -> get `state` by [`state.id`](http://state.id/)
- Vehicle Year → get `map_policy_driver_vehicle` by [`policy.id`](http://policy.id/) -> get [`vehicle.id`](http://vehicle.id/) by `map_policy_driver_vehicle.vehicle_id` -> get `vehicle_year.id` by `vehicle.vehicle_year_id` -> get `vehicle_year` by `vehicle_year.id` -> get `vehicle_year.value`
- Vehicle Make → get `map_policy_driver_vehicle` by [`policy.id`](http://policy.id/) -> get [`vehicle.id`](http://vehicle.id/) by `map_policy_driver_vehicle.vehicle_id` -> get `vehicle_make.id` by `vehicle.vehicle_make_id` -> get `vehicle_make` by `vehicle_make.id` -> get `vehicle_make.name`
- Vehicle Model → get `map_policy_driver_vehicle` by [policy.id](http://policy.id/) -> get [`vehicle.id`](http://vehicle.id/) by `map_policy_driver_vehicle.vehicle_id` -> get `vehicle_model.id` by `vehicle.vehicle_model_id` -> get `vehicle_model` by `vehicle_model.id` and `vehicle.vehicle_make_id` -> get `vehicle_model.name`
- Vehicle VIN → get `map_policy_driver_vehicle` by [`policy.id`](http://policy.id/) -> get [`vehicle.id`](http://vehicle.id/) by `map_policy_driver_vehicle.vehicle_id` -> get `vehicle` by [`vehicle.id`](http://vehicle.id/) -> get `vehicle.vin`
- Vehicle License Plate Number → get `map_policy_driver_vehicle` by [`policy.id`](http://policy.id/) -> get [`vehicle.id`](http://vehicle.id/) by `map_policy_driver_vehicle.vehicle_id` -> get `vehicle` by [`vehicle.id`](http://vehicle.id/) -> get `license_plate` by `vehicle.license_plate_id` -> get `license_plate.number`
- Coverage Type → get `map_policy_driver_vehicle` by [`policy.id`](http://policy.id/) -> get `map_policy_coverage_limit` by [`policy.id`](http://policy.id/) -> get `map_vehicle_coverage_limit` by `map_policy_driver_vehicle.vehicle_id` -> get `map_driver_coverage_limit` by `map_policy_driver_vehicle.driver_id` -> get [`coverage.id`](http://coverage.id/) by `map_policy_coverage_limit.coverage_id` and `map_vehicle_coverage_limit.coverage_id` and `map_driver_coverage_limit.coverage_id` -> get `coverage` by [`coverage.id`](http://coverage.id/) -> get `coverage_type.id` by `coverage.coverage_type_id` -> get `coverage_type.name` by `coverage_type.id`
- Coverage Limit → get `map_policy_driver_vehicle` by [`policy.id`](http://policy.id/) -> get `map_policy_coverage_limit` by [`policy.id`](http://policy.id/) -> get `map_vehicle_coverage_limit` by `map_policy_driver_vehicle.vehicle_id` -> get [`coverage.id`](http://coverage.id/) by `map_policy_coverage_limit.coverage_id` and `map_vehicle_coverage_limit.coverage_id` -> get [`limit.id`](http://limit.id/) by `map_policy_coverage_limit` and `map_vehicle_coverage_limit`
- Coverage Premium → get `map_policy_driver_vehicle` by [`policy.id`](http://policy.id/) -> get `map_policy_coverage_limit` by [`policy.id`](http://policy.id/) -> get `map_vehicle_coverage_limit` by `map_policy_driver_vehicle.vehicle_id` -> get `map_driver_coverage_limit` by `map_policy_driver_vehicle.driver_id` -> get [`coverage.id`](http://coverage.id/) by `map_policy_coverage_limit.coverage_id` and `map_vehicle_coverage_limit.coverage_id` and `map_driver_coverage_limit.coverage_id` -> get `coverage` by [`coverage.id`](http://coverage.id/) -> get `map_coverage_premium` by [`coverage.id`](http://coverage.id/) -> get [`premium.id`](http://premium.id/) by `map_coverage_premium.premium_id` -> get `premium` by [`premium.id`](http://premium.id/) -> get `sum` of `premium.credit` and `premium.debit`
- Total Policy Premium
    - Retrieve the policy record:
        - Use `map_user_policy_driver` by `user.id` to obtain `policy.id`.
        - Retrieve the policy record from `policy` to access policy details.
    - Retrieve premium records:
        - Use `map_policy_premium_transaction` where `policy_id` = `policy.id` to fetch all associated premium records.
    - Calculate Base Premium:
        - For each premium record retrieved from premium, compute:
            - `premium_net` = `premium.credit` – `premium.debit`.
    - Compute Total Term Premium:
        - Sum all `premium_net` values from the premium records to obtain the Total Term Premium Due for the entire term.
    - Table Summary:
        - `policy` `(id`)
        - `map_user_policy_driver` (`user.id`, `policy.id`)
        - `map_policy_premium_transaction` (`policy_id`, `premium_id`)
        - `premium` (`credit`, `debit`)

## **D) User Experience (UX) & Flows**

### **1. Viewing Policy Summary**

1. User navigates to **Policy Summary**.
2. The system displays **key policy details, payment status, and discounts**.
3. User can view **tooltip explanations** for complex terms.

### **2. Making a Payment**

1. User clicks **“Make Payment”** button.
2. System redirects to the **payment module**.
3. User completes the transaction and gets confirmation.

### **3. Contacting an Agent**

1. User views agent details under **“Your Agent’s Information”**.
2. Contact options include **phone number and office address**.

## **E) Master Schema Tables**

`address`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for each address. |
| address_type_id | BIGINT | NOT NULL, FOREIGN KEY to address_type(id) | References the type of address (e.g., residential, billing). |
| county_id | BIGINT | NOT NULL, FOREIGN KEY to county(id) | References the county associated with the address. |
| city_id | BIGINT | NOT NULL, FOREIGN KEY to city(id) | The name of the city. |
| state_id | BIGINT | NOT NULL, FOREIGN KEY to state(id) | References the state associated with the address. |
| zip_code_id | BIGINT | NOT NULL, FOREIGN KEY to zip_code(id) | References the ZIP code associated with the address. |
| description | TEXT |  | Additional details about the address. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status of the address. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the address record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the address record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`contact_number`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the contact number. |
| contact_number_type_id | BIGINT | NOT NULL, FOREIGN KEY to contact_number_type(id) | References the type of contact number. |
| value | VARCHAR(20) | NOT NULL, UNIQUE | The contact number (e.g., phone, fax). |
| description | TEXT |  | Additional details about the contact number. |
| status_id | BIGINT | FOREIGN KEY to status(id) | Status of the contact number record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the contact number record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the contact number record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update. |

`coverage`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for each coverage record. |
| coverage_type_id | BIGINT | NOT NULL, FOREIGN KEY to coverage_type(id) | References the type of coverage. |
| name | VARCHAR(50) | NOT NULL, UNIQUE | Name of the coverage. |
| description | TEXT |  | Additional details about the coverage. |
| effective_date | DATE |  | The date when the coverage becomes effective. |
| expiration_date | DATE |  | The date when the coverage expires. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | References the status of the coverage. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`coverage_type`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the coverage type. |
| name | VARCHAR(50) | NOT NULL, UNIQUE | Name of the coverage type (e.g., 'Standard'). |
| description | TEXT |  | Additional details about the coverage type. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the coverage type. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the coverage type record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the coverage type. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`discount`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for each discount record. |
| discount_type_id | BIGINT | NOT NULL, FOREIGN KEY to discount_type(id) | References the type of discount (e.g., Good Driver, Safe Vehicle). |
| name | VARCHAR(255) | NOT NULL, UNIQUE | Name of the discount. |
| description | TEXT |  | Additional details about the discount. |
| credit | DECIMAL(12,2) | CHECK (credit >= 0) | Amount credited due to the discount. |
| debit | DECIMAL(12,2) | CHECK (debit >= 0) | Amount debited, if applicable, in special scenarios. |
| effective_date | DATE | NOT NULL | Date the discount becomes effective. |
| expiration_date | DATE | NOT NULL | Date the discount expires. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | References the current status of the discount. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the discount record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the discount record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`driver`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for each driver record. |
| date_of_birth | DATE | NOT NULL | Driver's date of birth. |
| name_id | BIGINT | NOT NULL, FOREIGN KEY to name(id) | References the driver's name record. |
| license_id | BIGINT | NOT NULL, FOREIGN KEY to license(id) | References the driver's license record. |
| driver_type_id | BIGINT | NOT NULL, FOREIGN KEY to driver_type(id) | References the type of driver (e.g., Included). |
| payment_type_id | BIGINT | FOREIGN KEY to payment_type(id) | References the type of payment associated with the driver, if applicable. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | References the status of the driver. |
| description | TEXT |  | Additional details about the driver. |
| signature_required | BOOLEAN | NOT NULL, DEFAULT FALSE | Specifies if a signature is required for the driver. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`driver_type`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for each driver type. |
| name | VARCHAR(255) | NOT NULL, UNIQUE | Name of the driver type (e.g., 'Included'). |
| description | TEXT |  | Additional details about the driver type. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the driver type record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`email_address`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT UNSIGNED | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the email address record. |
| email_address_type_id | BIGINT UNSIGNED | NOT NULL, FOREIGN KEY to email_address_type(id) | References the type of email address. |
| value | VARCHAR(255) | NOT NULL, UNIQUE | The actual email address. |
| email_verified_at | TIMESTAMP | NULL | Timestamp indicating when the email was verified. |
| description | TEXT |  | Additional details about the email address. |
| status_id | BIGINT UNSIGNED | FOREIGN KEY to status(id) | References the status of the email address. |
| created_by | BIGINT UNSIGNED | NOT NULL, FOREIGN KEY to user(id) | User who created the email address record. |
| updated_by | BIGINT UNSIGNED | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`fee`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the fee record. |
| fee_type_id | BIGINT | NOT NULL, FOREIGN KEY to fee_type(id) | References the type of fee. |
| name | VARCHAR(255) | NOT NULL, UNIQUE | Name of the fee record. |
| description | TEXT |  | Additional details about the fee. |
| credit | DECIMAL(18,2) | DEFAULT 0.00 | Credit amount associated with the fee. |
| debit | DECIMAL(18,2) | DEFAULT 0.00 | Debit amount associated with the fee. |
| effective_date | DATE |  | Date when the fee becomes effective. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status of the fee record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the fee record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the fee record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`license`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for each license record. |
| license_type_id | BIGINT | NOT NULL, FOREIGN KEY to license_type(id) | References the type of license. |
| state_id | BIGINT | NOT NULL, FOREIGN KEY to state(id) | References the state where the license was issued. |
| country_id | BIGINT | NOT NULL, FOREIGN KEY to country(id) | References the country where the license was issued. |
| number | VARCHAR(255) | NOT NULL | Unique driver’s license number. |
| issue_date | DATE | NULLABLE | Date the license was issued. |
| expiration_date | DATE | NOT NULL | Date the license expires. |
| description | TEXT |  | Additional details about the license. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the license record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |
| Unique Constraint |  | UNIQUE(license_type_id, state_id, country_id, number) | Ensures uniqueness for the combination of license fields. |

`license_plate`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the license plate. |
| number | VARCHAR(50) | NOT NULL, UNIQUE | License plate number. |
| state_id | BIGINT | NOT NULL, FOREIGN KEY to state(id) | State that issued the license plate. |
| license_plate_type_id | BIGINT | FOREIGN KEY to license_plate_type(id) | Type of the license plate. |
| description | TEXT |  | Additional details about the license plate. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the license plate. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the license plate record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the license plate record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the license plate record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the license plate record was last updated. |

`limit`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the limit record. |
| name | VARCHAR(100) | NOT NULL, UNIQUE | Name of the limit (e.g., "BI Per Person", "PD Per Occurrence"). |
| limit_type_id | BIGINT | NOT NULL, FOREIGN KEY to limit_type(id) | References the type/category of limit. |
| minimum | DECIMAL(15,2) | NOT NULL | The minimum limit amount. |
| maximum | DECIMAL(15,2) | NOT NULL | The maximum limit amount. |
| description | TEXT |  | Additional details about the limit. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the limit. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`map_coverage_premium`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping. |
| coverage_id | BIGINT | NOT NULL, FOREIGN KEY to coverage(id) | References the associated coverage. |
| premium_id | BIGINT | NOT NULL, FOREIGN KEY to premium(id) | References the associated premium. |
| description | TEXT |  | Additional notes about this mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | References the status of the record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp of record creation. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of last record update. |

`map_driver_address`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| driver_id | BIGINT | NOT NULL, FOREIGN KEY to driver(id) | References the driver associated with the address. |
| address_id | BIGINT | NOT NULL, FOREIGN KEY to address(id) | References the address associated with the driver. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the mapping record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the mapping record was last updated. |

`map_driver_contact_number`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping. |
| driver_id | BIGINT | NOT NULL, FOREIGN KEY to driver(id) | References the driver. |
| contact_number_id | BIGINT | NOT NULL, FOREIGN KEY to contact_number(id) | References the contact number. |
| description | TEXT |  | Additional notes about the mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp of creation. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update. |

`map_driver_coverage_limit`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the driver-to-coverage-limit mapping record. |
| driver_id | BIGINT | NOT NULL, FOREIGN KEY to driver(id) | References the driver associated with the coverage limit. |
| coverage_id | BIGINT | NOT NULL, FOREIGN KEY to coverage(id) | References the coverage associated with the driver. |
| limit_id | BIGINT | NOT NULL, FOREIGN KEY to limit(id) | References the limit applied to the coverage. |
| description | TEXT |  | Additional details about the mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`map_driver_discount`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping. |
| driver_id | BIGINT | NOT NULL, FOREIGN KEY to driver(id) | References the driver. |
| discount_id | BIGINT | NOT NULL, FOREIGN KEY to discount(id) | References the discount. |
| description | TEXT |  | Additional details about the mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update. |

`map_driver_email_address`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| driver_id | BIGINT | NOT NULL, FOREIGN KEY to driver(id) | References the driver associated with the email address. |
| email_address_id | BIGINT | NOT NULL, FOREIGN KEY to email_address(id) | References the email address associated with the driver. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the mapping record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the mapping record was last updated. |

`map_driver_premium`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| driver_id | BIGINT | NOT NULL, FOREIGN KEY to driver(id) | References the associated driver. |
| premium_id | BIGINT | NOT NULL, FOREIGN KEY to premium(id) | References the associated premium. |
| description | TEXT |  | Additional notes about this mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update. |

`map_policy_coverage_limit`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the record. |
| policy_id | BIGINT | NOT NULL, FOREIGN KEY to policy(id) | References the policy. |
| coverage_id | BIGINT | NOT NULL, FOREIGN KEY to coverage(id) | References the coverage. |
| limit_id | BIGINT | NOT NULL, FOREIGN KEY to limit(id) | References the limit. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`map_policy_discount`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping. |
| policy_id | BIGINT | NOT NULL, FOREIGN KEY to policy(id) | References the policy. |
| discount_id | BIGINT | NOT NULL, FOREIGN KEY to discount(id) | References the discount. |
| description | TEXT |  | Additional details about the mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update. |

`map_policy_driver`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for each mapping. |
| policy_id | BIGINT | NOT NULL, FOREIGN KEY to policy(id) | References the associated policy. |
| driver_id | BIGINT | NOT NULL, FOREIGN KEY to driver(id) | References the associated driver. |
| description | TEXT |  | Additional details about the policy-driver relationship. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping (e.g., active, inactive). |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |
| Unique Constraint |  | UNIQUE (policy_id, driver_id) | Ensures no duplicate policy-driver mapping. |

`map_policy_driver_vehicle`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping |
| policy_id | BIGINT | NOT NULL, FOREIGN KEY to policy(id) | References the policy table |
| driver_id | BIGINT | NOT NULL, FOREIGN KEY to driver(id) | References the driver table |
| vehicle_id | BIGINT | NOT NULL, FOREIGN KEY to vehicle(id) | References the vehicle table |
| description | TEXT |  | Additional details about the mapping |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | References the status table |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the mapping was created |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the mapping was last updated |

`map_policy_fee`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| policy_id | BIGINT | NOT NULL, FOREIGN KEY to policy(id) | References the associated policy. |
| fee_id | BIGINT | NOT NULL, FOREIGN KEY to fee(id) | References the associated fee. |
| description | TEXT |  | Additional notes about this mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update. |

`map_policy_named_driver`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| policy_id | BIGINT | NOT NULL, FOREIGN KEY to policy(id) | References the policy record. |
| driver_id | BIGINT | NOT NULL, FOREIGN KEY to driver(id) | References the named driver associated with the policy. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the mapping record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the mapping record was last updated. |

`map_policy_premium`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping. |
| policy_id | BIGINT | NOT NULL, FOREIGN KEY to policy(id) | References the associated policy. |
| premium_id | BIGINT | NOT NULL, FOREIGN KEY to premium(id) | References the associated premium. |
| description | TEXT |  | Additional notes about this mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | References the status of the record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp of record creation. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of last record update. |

`map_policy_transaction`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the map record. |
| policy_id | BIGINT | NOT NULL, FOREIGN KEY to policy(id) | References the associated policy. |
| transaction_id | BIGINT | NOT NULL, FOREIGN KEY to transaction(id) | References the associated transaction. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status of the map record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the map record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the map record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`map_producer_address`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| producer_id | BIGINT | NOT NULL, FOREIGN KEY to producer(id) | References the producer. |
| address_id | BIGINT | NOT NULL, FOREIGN KEY to address(id) | References the address. |
| description | TEXT |  | Additional details about the mapping. |
| status_id | BIGINT | FOREIGN KEY to status(id) | Status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`map_producer_contact_number`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| producer_id | BIGINT | NOT NULL, FOREIGN KEY to producer(id) | References the producer. |
| contact_number_id | BIGINT | NOT NULL, FOREIGN KEY to contact_number(id) | References the contact number. |
| description | TEXT |  | Additional details about the mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update. |

`map_producer_name`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT UNSIGNED | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| producer_id | BIGINT UNSIGNED | NOT NULL, FOREIGN KEY to producer(id) | References the producer record. |
| first_name_id | BIGINT UNSIGNED | NOT NULL, FOREIGN KEY to name(id) | References the first name of the producer. |
| middle_name_id | BIGINT UNSIGNED | FOREIGN KEY to name(id) | References the middle name of the producer. |
| last_name_id | BIGINT UNSIGNED | NOT NULL, FOREIGN KEY to name(id) | References the last name of the producer. |
| surname_name_id | BIGINT UNSIGNED | FOREIGN KEY to name(id) | References the surname of the producer. |
| description | TEXT |  | Additional details about the mapping. |
| status_id | BIGINT UNSIGNED | FOREIGN KEY to status(id) | References the status of the mapping. |
| created_by | BIGINT UNSIGNED | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT UNSIGNED | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`map_producer_policy`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| producer_id | BIGINT | NOT NULL, FOREIGN KEY to producer(id) | References the producer associated with the policy. |
| policy_id | BIGINT | NOT NULL, FOREIGN KEY to policy(id) | References the policy related to the producer. |
| description | TEXT |  | Additional details about the producer-policy relationship. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`map_transaction_payment`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the map record. |
| transaction_id | BIGINT | NOT NULL, FOREIGN KEY to transaction(id) | References the associated transaction. |
| payment_id | BIGINT | NOT NULL, FOREIGN KEY to payment(id) | References the associated payment. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status of the map record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the map record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the map record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`map_user_policy_driver`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| user_id | BIGINT | NOT NULL, FOREIGN KEY to user(id) | References the user associated with the policy and driver. |
| policy_id | BIGINT | NOT NULL, FOREIGN KEY to policy(id) | References the policy associated with the user and driver. |
| driver_id | BIGINT | NOT NULL, FOREIGN KEY to driver(id) | References the driver associated with the user and policy. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the mapping record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the mapping record was last updated. |

`map_vehicle_coverage_limit`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the vehicle-to-coverage-limit mapping record. |
| vehicle_id | BIGINT | NOT NULL, FOREIGN KEY to vehicle(id) | References the vehicle associated with the coverage limit. |
| coverage_id | BIGINT | NOT NULL, FOREIGN KEY to coverage(id) | References the coverage associated with the vehicle. |
| limit_id | BIGINT | NOT NULL, FOREIGN KEY to limit(id) | References the limit applied to the coverage. |
| description | TEXT |  | Additional details about the mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`map_vehicle_discount`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping. |
| vehicle_id | BIGINT | NOT NULL, FOREIGN KEY to vehicle(id) | References the vehicle. |
| discount_id | BIGINT | NOT NULL, FOREIGN KEY to discount(id) | References the discount. |
| description | TEXT |  | Additional details about the mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update. |

`map_vehicle_premium`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| vehicle_id | BIGINT | NOT NULL, FOREIGN KEY to vehicle(id) | References the associated vehicle. |
| premium_id | BIGINT | NOT NULL, FOREIGN KEY to premium(id) | References the associated premium. |
| description | TEXT |  | Additional notes about this mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update. |

`name`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT UNSIGNED | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the name record. |
| name_type_id | BIGINT UNSIGNED | NOT NULL, FOREIGN KEY to name_type(id) | References the type of the name. |
| value | VARCHAR(255) | NOT NULL | The actual name value (e.g., 'John', 'Smith'). |
| description | TEXT |  | Additional details about the name. |
| status_id | BIGINT UNSIGNED | FOREIGN KEY to status(id) | References the status of the name record. |
| created_by | BIGINT UNSIGNED | NOT NULL, FOREIGN KEY to user(id) | User who created the name record. |
| updated_by | BIGINT UNSIGNED | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update. |

`policy`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the policy record. |
| policy_prefix_id | BIGINT | NULLABLE, FOREIGN KEY to policy_prefix(id) | References the policy prefix. |
| number | VARCHAR(50) | NULLABLE, UNIQUE(policy_prefix_id, number) | Unique policy identifier within a prefix scope. |
| policy_type_id | BIGINT | NOT NULL, FOREIGN KEY to policy_type(id) | References the type of policy. |
| effective_date | DATE | NULLABLE | The start date of the policy. |
| inception_date | DATE | NULLABLE | The inception date of the policy (first effective date). |
| expiration_date | DATE | NOT NULL | The end date of the policy. |
| renewal_date | DATE | NULLABLE | Date when the policy is set for renewal. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | References the status of the policy. |
| term_id | BIGINT | NOT NULL, FOREIGN KEY to term(id) | References the term of the policy. |
| description | TEXT | NULLABLE | Additional details about the policy. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`policy_prefix`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for each policy prefix. |
| name | VARCHAR(50) | NOT NULL, UNIQUE | Unique policy prefix (e.g., 'PLCY-', 'AUTO-'). |
| description | TEXT |  | Additional details about the policy prefix. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | References the status of the policy prefix. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`premium`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the premium record. |
| premium_type_id | BIGINT | NOT NULL, FOREIGN KEY to premium_type(id) | Type of premium associated with the record. |
| name | VARCHAR(255) | NOT NULL, UNIQUE | Name of the premium record. |
| description | TEXT |  | Additional details about the premium. |
| credit | DECIMAL(12,2) | CHECK (credit >= 0) | Credit value associated with the premium. |
| debit | DECIMAL(12,2) | CHECK (debit >= 0) | Debit value associated with the premium. |
| effective_date | DATE |  | Date when the premium becomes effective. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the premium record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the premium record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the premium record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the premium record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the premium record was last updated. |

`producer`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for each producer/agency. |
| producer_code_id | BIGINT | NOT NULL, FOREIGN KEY to producer_code(id), UNIQUE | Unique producer code for identification. |
| number | VARCHAR(50) | NOT NULL, UNIQUE | Unique number assigned to the producer. |
| name | VARCHAR(255) | NOT NULL, UNIQUE | Name of the producer/agency. |
| description | TEXT |  | Additional details about the producer. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status of the producer. |
| producer_type_id | BIGINT | NOT NULL, FOREIGN KEY to producer_type(id) | References the type of producer associated with the program. |
| signature_required | BOOLEAN | NOT NULL, DEFAULT FALSE | Specifies if a signature is required for the producer. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the producer record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the producer record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`state`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for each state. |
| name | VARCHAR(255) | NOT NULL | The full name of the state (e.g., Texas, California). |
| code | VARCHAR(10) | NOT NULL | Abbreviated state code (e.g., TX, CA). |
| description | TEXT |  | Additional details or notes about the state. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the state record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |
| Unique Constraint |  | UNIQUE (name, code) | Ensures no duplicate entries for state name and code. |

`status`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the status record. |
| name | VARCHAR(100) | NOT NULL, UNIQUE | Name of the status (e.g., 'Active', 'Pending'). |
| description | TEXT |  | Additional details about the status. |
| created_by | BIGINT | FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`transaction`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the transaction. |
| transaction_type_id | BIGINT | NOT NULL, FOREIGN KEY to transaction_type(id) | References the type of transaction. |
| name | VARCHAR(255) | NOT NULL | Name of the transaction. |
| description | TEXT |  | Additional details about the transaction, including notes like "all current and future transactions (like installments)." |
| credit | DECIMAL(12,2) | CHECK (credit >= 0) | Total credit amount for the transaction. |
| debit | DECIMAL(12,2) | CHECK (debit >= 0) | Total debit amount for the transaction. |
| confirmation_code | VARCHAR(255) |  | Confirmation code associated with the transaction. |
| due_date | DATE |  | The date when the transaction is due. |
| processed_date | TIMESTAMP |  | The date when the transaction was processed. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the transaction. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the transaction. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the transaction. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the transaction was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update to the transaction. |

`user`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for each user. |
| user_type_id | BIGINT | NOT NULL, FOREIGN KEY to user_type(id) | References the type of user (e.g., insured, producer, claims, system). |
| user_group_id | BIGINT | NOT NULL, FOREIGN KEY to user_group(id) | References the user group (e.g., manager, csr, underwriting). |
| username | VARCHAR(255) | UNIQUE, NOT NULL | Username |
| password | VARCHAR(255) | NOT NULL | User's hashed password. |
| description | TEXT |  | Additional details about the user. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the user's status (e.g., Active, Inactive). |
| created_by | BIGINT | FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`vehicle`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the vehicle. |
| vehicle_type_id | BIGINT | NOT NULL, FOREIGN KEY to vehicle_type(id) | References the vehicle type table. |
| vehicle_year_id | BIGINT | NOT NULL, FOREIGN KEY to vehicle_year(id) | References the vehicle year table. |
| vehicle_make_id | BIGINT | NOT NULL, FOREIGN KEY to vehicle_make(id) | References the vehicle make table. |
| vehicle_model_id | BIGINT | NOT NULL, FOREIGN KEY to vehicle_model(id) | References the vehicle model table. |
| license_plate_id | BIGINT | FOREIGN KEY to license_plate(id) | References the license plate assigned to the vehicle. |
| vin | VARCHAR(255) | NOT NULL, UNIQUE | Vehicle Identification Number (VIN). |
| mileage | BIGINT |  | Mileage of the vehicle. |
| description | TEXT |  | Additional details about the vehicle. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the vehicle. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the vehicle record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the vehicle record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the vehicle record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the vehicle record was last updated. |

`vehicle_make`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the vehicle make |
| name | VARCHAR(255) | NOT NULL, UNIQUE | Name of the vehicle make (e.g., 'Toyota', 'Ford') |
| description | TEXT |  | Additional details about the vehicle make |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | References the status table |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated |

`vehicle_model`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the vehicle model |
| vehicle_make_id | BIGINT | NOT NULL, FOREIGN KEY to vehicle_make(id) | References the vehicle make |
| name | VARCHAR(255) | NOT NULL | Name of the vehicle model (e.g., 'Camry', 'F-150') |
| description | TEXT |  | Additional details about the vehicle model |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | References the status table |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated |

`vehicle_year`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the vehicle year |
| value | INT | NOT NULL, UNIQUE | Year of the vehicle (e.g., 2020, 2021) |
| description | TEXT |  | Additional details about the vehicle year |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | References the status table |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated |

# #2 - Drivers and Vehicles

## **A) WHY - Vision and Purpose**

The **Drivers & Vehicles** section in the **Policy Summary** provides users with a **clear and structured view of all insured drivers and vehicles** associated with their insurance policy. This feature ensures **policy transparency** and allows users to verify their coverage details, track active and excluded drivers, and confirm the insured vehicles linked to their policy.

## **B) WHAT - Core Requirements**

### **1. Drivers Section**

- Display **all drivers** associated with the policy.
- Show key driver details:
    - **Name**
        - Driver Name → get `map_policy_driver` by `policy.id` -> get `driver.id` by `map_policy_driver.driver_id` -> get driver by `driver.id` -> get `name.id` by `driver.name_id` -> get `name`
    - **Date of Birth**
        - Driver Date of Birth → get `map_policy_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by `map_policy_driver.driver_id` -> get `driver` by [`driver.id`](http://driver.id/) -> get `driver.date_of_birth`
    - **License Number**
        - Driver’s License Number → get `map_policy_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by `map_policy_driver.driver_id` -> get `driver` by [`driver.id`](http://driver.id/) -> get [`license.id`](http://license.id/) by `driver.license_id` -> get `license` by [`license.id`](http://license.id/)
    - **Status (e.g., “Included” or “Excluded”)**
        - Driver Status (Included vs Excluded) → get `map_policy_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by `map_policy_driver.driver_id` -> get `driver` by [`driver.id`](http://driver.id/) -> get `driver_type.id` by `driver.driver_type_id` -> driver types defined for included and excluded

### **2. Vehicles Section**

- Display **all insured vehicles** in the policy.
- Show key vehicle details:
    - **Make, Model, and Year**
        - Vehicle Year → get `map_policy_driver_vehicle` by [`policy.id`](http://policy.id/) -> get [`vehicle.id`](http://vehicle.id/) by `map_policy_driver_vehicle.vehicle_id` -> get `vehicle_year.id` by `vehicle.vehicle_year_id` -> get `vehicle_year` by `vehicle_year.id` -> get `vehicle_year.value`
        - Vehicle Make → get `map_policy_driver_vehicle` by [`policy.id`](http://policy.id/) -> get [`vehicle.id`](http://vehicle.id/) by `map_policy_driver_vehicle.vehicle_id` -> get `vehicle_make.id` by `vehicle.vehicle_make_id` -> get `vehicle_make` by `vehicle_make.id` -> get `vehicle_make.name`
        - Vehicle Model → get `map_policy_driver_vehicle` by `policy.id` -> get [`vehicle.id`](http://vehicle.id/) by `map_policy_driver_vehicle.vehicle_id` -> get `vehicle_model.id` by `vehicle.vehicle_model_id` -> get `vehicle_model` by `vehicle_model.id` and `vehicle.vehicle_make_id` -> get `vehicle_model.name`
        - Vehicle License Plate Number → get `map_policy_driver_vehicle` by [`policy.id`](http://policy.id/) -> get [`vehicle.id`](http://vehicle.id/) by `map_policy_driver_vehicle.vehicle_id` -> get `vehicle` by [`vehicle.id`](http://vehicle.id/) -> get `license_plate` by `vehicle.license_plate_id` -> get `license_plate.number`
    - **VIN**
        - Vehicle VIN → get `map_policy_driver_vehicle` by [`policy.id`](http://policy.id/) -> get [`vehicle.id`](http://vehicle.id/) by `map_policy_driver_vehicle.vehicle_id` -> get `vehicle` by [`vehicle.id`](http://vehicle.id/) -> get `vehicle.vin`

### **3. Interactive Features**

- Expand/collapse sections for **Drivers & Vehicles** to optimize readability.
- Ability to quickly **switch between policy summary tabs** (Overview, Drivers & Vehicles, Coverage).

## **C) HOW - Planning & Implementation**

**Full Mapping of Front-End Fields to Database Fields**

- Producer Label Configuration → Stored in the appropriate application side configuration file.
- Policy Number → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `policy` by [`policy.id`](http://policy.id/) -> get `policy_prefix` by `policy.policy_prefix_id` -> concat `policy_prefix.value` and `policy.number`
- Policy Status → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `policy` by [`policy.id`](http://policy.id/) -> `policy.status_id`
- Named Insured Name → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `map_policy_named_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by m`ap_policy_named_driver.driver_id` -> get `driver` by [`driver.id](http://driver.id/)` -> get `name` by `driver.name_id`
- Next Payment Amount
    - Retrieve the policy record:
        - Use `map_user_policy_driver` by `user.id` to obtain `policy.id`.
        - Retrieve the policy record from policy to access `installment_due_day` and `term_months`.
    - Aggregate premium data:
        - Use `map_policy_premium_transaction` where `policy_id` = `policy.id`.
        - For each premium record from premium, calculate:

          `premium_net` = `premium.credit` – `premium.debit`.

        - Sum all `premium_net` values to obtain Total Premium.
    - Aggregate fee data:
        - Use `map_policy_fee_transaction` where `policy_id` = `policy.id`.
        - For each fee record from fee, calculate:

          `fee_net` = `fee.debit` – `fee.credit`.

        - Sum all `fee_net` values to obtain Total Fees.
    - Aggregate payment transactions:
        - Use `map_policy_installment_transaction` where `policy_id` = `policy.id`.
        - For each transaction from transaction, sum the debit amounts (payments are recorded as debits).
        - Sum these amounts to get Total Payments Applied.
    - Compute net premium remaining:
        - Net Premium Remaining = (Total Premium + Total Fees) – (Total Payments Applied).
    - Determine the remaining installment count:
        - Count the number of installment records for the policy via `map_policy_installment_transaction`.
        - Remaining Installments = `policy.term_months` – (count of installment records).
    - Calculate Next Payment Amount:
        - If an installment record exists (with status "Pending" or "Partially Paid"):
            - Next Payment Amount = `installment.total` – `installment.paid_to_date`.
        - If no installment record exists:
            - Next Installment Amount = Net Premium Remaining / Remaining Installments.
    - Table Summary:
        - `policy` (`id`, `installment_due_day`, `term_months`)
        - `map_user_policy_driver` (`user.id`, `policy.id`)
        - `map_policy_premium_transaction` (`policy_id`, `premium_id`)
        - `premium` (`credit`, `debit`)
        - `map_policy_fee_transaction` (`policy_id`, `fee_id`)
        - `fee` (`debit`, `credit`)
        - `map_policy_installment_transaction` (`policy_id`, `installment_id`)
        - `transaction` (`debit`, `processed_date`)
        - `installment` (`total`, `paid_to_date`, `due_date`, `status_id`)
- Due Date of Next Payment
    - Retrieve the policy record:
        - Use `map_user_policy_driver` by `user.id` to obtain `policy.id`.
        - Retrieve the policy record from `policy` to access `installment_due_day`.
    - Check for outstanding items:
        - Retrieve any installment records for the policy via `map_policy_installment_transaction` with status "Pending" or "Partially Paid" and extract their `due_date` values.
        - Retrieve fee records via `map_policy_fee_transaction` (if fees have due dates and outstanding balances) and extract their `due_date` values.
        - Identify the earliest due date among these outstanding installments/fees.
    - Calculate the next billing date dynamically:
        - If today’s date is before the `installment_due_day`, then set the calculated next due date to the current month’s `installment_due_day`.
            - Example: If today is the 10th and `installment_due_day` is 15, then next due date = the 15th of the current month.
        - If today’s date is on or after the `installment_due_day`, then set the calculated next due date to the `installment_due_day` in the next month.
            - Example: If today is the 16th and `installment_due_day` is 15, then next due date = the 15th of the next month.
    - Determine the effective due date:
        - Compare the dynamically calculated due date with the earliest due date from outstanding installments/fees (if any exist).
        - The Next Payment Due Date is the earliest of:
            - The dynamically calculated next due date based on `installment_due_day`, or
            - The earliest `due_date` among outstanding installment/fee records.
    - Set the due date in the new installment record:
        - When generating a new installment record (via a cronjob or event trigger), assign its `due_date` field to the determined Next Payment Due Date.
    - This `due_date` becomes the Due Date of the Next Payment.
    - Table Summary:
        - `policy` (`id`, `installment_due_day`)
        - `map_user_policy_driver` (`user.id`, `policy.id`)
        - `map_policy_installment_transaction` (`policy_id`, `installment_id`) and `installment` (`due_date`, `status_id`)
        - `map_policy_fee_transaction` (`policy_id`, `fee_id`)
        - `fee` (`due_date`)
- Named Insured Address → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `map_policy_named_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by `map_policy_named_driver.driver_id` -> get `map_driver_address` by [`driver.id`](http://driver.id/) -> get [`address.id`](http://address.id/) by `map_driver_address.address_id` -> get `address` by `address_id`
- Named Insured Email → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `map_policy_named_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by `map_policy_named_driver.driver_id` -> get `map_driver_email_address` by [`driver.id`](http://driver.id/) -> get `email_address.id` by `map_driver_email_address.email_address_id` -> get `email_address` by `email_address_id`
- Named Insured Primary Phone Number → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `map_policy_named_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by `map_policy_named_driver.driver_id` -> get `map_driver_contact_number` by [`driver.id`](http://driver.id/) -> get `contact_number.id` by `map_driver_contact_number.contact_number_id` -> get `contact_number` by `contact_number_id` and `contact_number.contact_number_type` for primary
- Named Insured Alternate Phone Number → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `map_policy_named_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by `map_policy_named_driver.driver_id` -> `get map_driver_contact_number` by [driver.id](http://driver.id/) -> get `contact_number.id` by `map_driver_contact_number.contact_number_id` -> get `contact_number` by `contact_number_id` and `contact_number.contact_number_type` for alternate
- Effective Date → get [`policy.id`](http://policy.id/) from map_user_policy_driver by [user.id](http://user.id/) -> get policy by [`policy.id`](http://policy.id/) -> `policy.effective_date`
- Inception Date → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `policy` by [`policy.id`](http://policy.id/) -> `policy.inception_date`
- Expiration Date → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `policy` by [`policy.id`](http://policy.id/) -> `policy.expiration_date`
- Total Policy Premium → get [`policy.id`](http://`policy.id`/) from `map_user_policy_driver` by [`user.id`](http://`user.id`/) -> get `policy` by [`policy.id`](http://`policy.id`/) -> `map_policy_transaction` -> get all `transaction_id`'s -> foreach [`transaction.id`](http://`transaction.id`/) -> sum of `transaction.credit` and `transaction.debit` filtered by `transaction_type_id`'s to include
- Remaining Premium Balance → get [`policy.id`](http://`policy.id`/) from `map_user_policy_driver` by [`user.id`](http://`user.id`/) -> get `policy` by [`policy.id`](http://`policy.id`/) -> `map_policy_transaction` -> get all `transaction_id`'s -> foreach [`transaction.id`](http://`transaction.id`/) -> sum of `transaction.credit` and `transaction.debit` filtered by `transaction_type_id`'s to include
- Policy Term → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `policy` by [`policy.id`](http://policy.id/) -> `policy.term_id`
- Payment Method → get [`driver.id`](http://driver.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `driver` by [`driver.id`](http://driver.id/) -> `driver.payment_type_id`
- Applied Discounts → get `map_policy_driver_vehicle` by `policy.id` -> get `driver.id` and `vehicle.id` by `map_policy_driver_vehicle.driver_id` and `map_policy_driver_vehicle.vehicle_id` -> get `map_policy_discount` and get `map_vehicle_discount` and get `map_driver_discount` by `driver.id` and `vehicle.id` and `policy.id` -> get `discount.id` by `map_driver_discount.discount_id` and `map_vehicle_discount.discount_id` -> sum `discount.credit` and `discount.debit`
- Producer Phone Number → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `map_producer_policy` by [`policy.id`](http://policy.id/) -> get [`producer.id`](http://producer.id/) by `map_producer_policy.producer_id` -> get `map_producer_contact_number` by [`producer.id`](http://producer.id/) -> get `contact_number` by `map_producer_contact_number.contact_number_id`
- Producer Name → get `policy.id` from `map_user_policy_driver` by `user.id` -> get `map_producer_policy` by `policy.id` -> get `producer.id` by `map_producer_policy.producer_id` -> get `map_producer_name` by `producer.id`
- Producer Address → get `policy.id` from `map_user_policy_driver` by `user.id` -> get `map_producer_policy` by `policy.id` -> get `producer.id` by `map_producer_policy.producer_id` -> get `address.id` from `map_producer_address` by `producer.id` -> get address by `address.id`
- Driver Name → get `map_policy_driver` by `policy.id` -> get `driver.id` by `map_policy_driver.driver_id` -> get driver by `driver.id` -> get `name.id` by `driver.name_id` -> get `name`
- Primary Driver Tag → get `map_policy_named_driver` by `policy.id` -> get `driver.id` by `map_policy_named_driver.driver_id` -> named driver is the primary driver
- Driver Status (Included vs Excluded) → get `map_policy_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by `map_policy_driver.driver_id` -> get `driver` by [`driver.id`](http://driver.id/) -> get `driver_type.id` by `driver.driver_type_id` -> driver types defined for included and excluded
- Driver Date of Birth → get `map_policy_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by `map_policy_driver.driver_id` -> get `driver` by [`driver.id`](http://driver.id/) -> get `driver.date_of_birth`
- Driver’s License Number → get `map_policy_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by `map_policy_driver.driver_id` -> get `driver` by [`driver.id`](http://driver.id/) -> get [`license.id`](http://license.id/) by `driver.license_id` -> get `license` by [`license.id`](http://license.id/)
- Issuing State for License → get `map_policy_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by `map_policy_driver.driver_id` -> get `driver` by [`driver.id`](http://driver.id/) -> get [`license.id`](http://license.id/) by `driver.license_id` -> get `license` by [`license.id`](http://license.id/) -> get [`state.id`](http://state.id/) by `license.state_id` -> get `state` by [`state.id`](http://state.id/)
- Vehicle Year → get `map_policy_driver_vehicle` by [`policy.id`](http://policy.id/) -> get [`vehicle.id`](http://vehicle.id/) by `map_policy_driver_vehicle.vehicle_id` -> get `vehicle_year.id` by `vehicle.vehicle_year_id` -> get `vehicle_year` by `vehicle_year.id` -> get `vehicle_year.value`
- Vehicle Make → get `map_policy_driver_vehicle` by [`policy.id`](http://policy.id/) -> get [`vehicle.id`](http://vehicle.id/) by `map_policy_driver_vehicle.vehicle_id` -> get `vehicle_make.id` by `vehicle.vehicle_make_id` -> get `vehicle_make` by `vehicle_make.id` -> get `vehicle_make.name`
- Vehicle Model → get `map_policy_driver_vehicle` by [policy.id](http://policy.id/) -> get [`vehicle.id`](http://vehicle.id/) by `map_policy_driver_vehicle.vehicle_id` -> get `vehicle_model.id` by `vehicle.vehicle_model_id` -> get `vehicle_model` by `vehicle_model.id` and `vehicle.vehicle_make_id` -> get `vehicle_model.name`
- Vehicle VIN → get `map_policy_driver_vehicle` by [`policy.id`](http://policy.id/) -> get [`vehicle.id`](http://vehicle.id/) by `map_policy_driver_vehicle.vehicle_id` -> get `vehicle` by [`vehicle.id`](http://vehicle.id/) -> get `vehicle.vin`
- Vehicle License Plate Number → get `map_policy_driver_vehicle` by [`policy.id`](http://policy.id/) -> get [`vehicle.id`](http://vehicle.id/) by `map_policy_driver_vehicle.vehicle_id` -> get `vehicle` by [`vehicle.id`](http://vehicle.id/) -> get `license_plate` by `vehicle.license_plate_id` -> get `license_plate.number`
- Coverage Type → get `map_policy_driver_vehicle` by [`policy.id`](http://policy.id/) -> get `map_policy_coverage_limit` by [`policy.id`](http://policy.id/) -> get `map_vehicle_coverage_limit` by `map_policy_driver_vehicle.vehicle_id` -> get `map_driver_coverage_limit` by `map_policy_driver_vehicle.driver_id` -> get [`coverage.id`](http://coverage.id/) by `map_policy_coverage_limit.coverage_id` and `map_vehicle_coverage_limit.coverage_id` and `map_driver_coverage_limit.coverage_id` -> get `coverage` by [`coverage.id`](http://coverage.id/) -> get `coverage_type.id` by `coverage.coverage_type_id` -> get `coverage_type.name` by `coverage_type.id`
- Coverage Limit → get `map_policy_driver_vehicle` by [`policy.id`](http://policy.id/) -> get `map_policy_coverage_limit` by [`policy.id`](http://policy.id/) -> get `map_vehicle_coverage_limit` by `map_policy_driver_vehicle.vehicle_id` -> get [`coverage.id`](http://coverage.id/) by `map_policy_coverage_limit.coverage_id` and `map_vehicle_coverage_limit.coverage_id` -> get [`limit.id`](http://limit.id/) by `map_policy_coverage_limit` and `map_vehicle_coverage_limit`
- Coverage Premium → get `map_policy_driver_vehicle` by [`policy.id`](http://policy.id/) -> get `map_policy_coverage_limit` by [`policy.id`](http://policy.id/) -> get `map_vehicle_coverage_limit` by `map_policy_driver_vehicle.vehicle_id` -> get `map_driver_coverage_limit` by `map_policy_driver_vehicle.driver_id` -> get [`coverage.id`](http://coverage.id/) by `map_policy_coverage_limit.coverage_id` and `map_vehicle_coverage_limit.coverage_id` and `map_driver_coverage_limit.coverage_id` -> get `coverage` by [`coverage.id`](http://coverage.id/) -> get `map_coverage_premium` by [`coverage.id`](http://coverage.id/) -> get [`premium.id`](http://premium.id/) by `map_coverage_premium.premium_id` -> get `premium` by [`premium.id`](http://premium.id/) -> get `sum` of `premium.credit` and `premium.debit`
- Total Policy Premium
    - Retrieve the policy record:
        - Use `map_user_policy_driver` by `user.id` to obtain `policy.id`.
        - Retrieve the policy record from `policy` to access policy details.
    - Retrieve premium records:
        - Use `map_policy_premium_transaction` where `policy_id` = `policy.id` to fetch all associated premium records.
    - Calculate Base Premium:
        - For each premium record retrieved from premium, compute:
            - `premium_net` = `premium.credit` – `premium.debit`.
    - Compute Total Term Premium:
        - Sum all `premium_net` values from the premium records to obtain the Total Term Premium Due for the entire term.
    - Table Summary:
        - `policy` `(id`)
        - `map_user_policy_driver` (`user.id`, `policy.id`)
        - `map_policy_premium_transaction` (`policy_id`, `premium_id`)
        - `premium` (`credit`, `debit`)

## **D) User Experience (UX) & Flows**

### **1. Viewing Drivers & Vehicles**

1. User navigates to **Policy Summary → Drivers & Vehicles** tab.
2. The system displays **two sections: Drivers & Vehicles**.
3. Users can **expand/collapse sections** for better navigation.

### **2. Verifying Driver Status**

1. Users see a list of **drivers with their status (Included/Excluded)**.
2. Users can quickly verify if a driver is covered under the policy.

### **3. Checking Insured Vehicles**

1. Users see a list of **vehicles covered under the policy**.
2. Each vehicle’s **Make, Model, Year, and VIN** is displayed for verification.

## **E) Master Schema Tables**

`address`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for each address. |
| address_type_id | BIGINT | NOT NULL, FOREIGN KEY to address_type(id) | References the type of address (e.g., residential, billing). |
| county_id | BIGINT | NOT NULL, FOREIGN KEY to county(id) | References the county associated with the address. |
| city_id | BIGINT | NOT NULL, FOREIGN KEY to city(id) | The name of the city. |
| state_id | BIGINT | NOT NULL, FOREIGN KEY to state(id) | References the state associated with the address. |
| zip_code_id | BIGINT | NOT NULL, FOREIGN KEY to zip_code(id) | References the ZIP code associated with the address. |
| description | TEXT |  | Additional details about the address. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status of the address. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the address record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the address record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`contact_number`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the contact number. |
| contact_number_type_id | BIGINT | NOT NULL, FOREIGN KEY to contact_number_type(id) | References the type of contact number. |
| value | VARCHAR(20) | NOT NULL, UNIQUE | The contact number (e.g., phone, fax). |
| description | TEXT |  | Additional details about the contact number. |
| status_id | BIGINT | FOREIGN KEY to status(id) | Status of the contact number record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the contact number record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the contact number record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update. |

`coverage`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for each coverage record. |
| coverage_type_id | BIGINT | NOT NULL, FOREIGN KEY to coverage_type(id) | References the type of coverage. |
| name | VARCHAR(50) | NOT NULL, UNIQUE | Name of the coverage. |
| description | TEXT |  | Additional details about the coverage. |
| effective_date | DATE |  | The date when the coverage becomes effective. |
| expiration_date | DATE |  | The date when the coverage expires. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | References the status of the coverage. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`coverage_type`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the coverage type. |
| name | VARCHAR(50) | NOT NULL, UNIQUE | Name of the coverage type (e.g., 'Standard'). |
| description | TEXT |  | Additional details about the coverage type. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the coverage type. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the coverage type record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the coverage type. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`discount`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for each discount record. |
| discount_type_id | BIGINT | NOT NULL, FOREIGN KEY to discount_type(id) | References the type of discount (e.g., Good Driver, Safe Vehicle). |
| name | VARCHAR(255) | NOT NULL, UNIQUE | Name of the discount. |
| description | TEXT |  | Additional details about the discount. |
| credit | DECIMAL(12,2) | CHECK (credit >= 0) | Amount credited due to the discount. |
| debit | DECIMAL(12,2) | CHECK (debit >= 0) | Amount debited, if applicable, in special scenarios. |
| effective_date | DATE | NOT NULL | Date the discount becomes effective. |
| expiration_date | DATE | NOT NULL | Date the discount expires. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | References the current status of the discount. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the discount record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the discount record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`driver`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for each driver record. |
| date_of_birth | DATE | NOT NULL | Driver's date of birth. |
| name_id | BIGINT | NOT NULL, FOREIGN KEY to name(id) | References the driver's name record. |
| license_id | BIGINT | NOT NULL, FOREIGN KEY to license(id) | References the driver's license record. |
| driver_type_id | BIGINT | NOT NULL, FOREIGN KEY to driver_type(id) | References the type of driver (e.g., Included). |
| payment_type_id | BIGINT | FOREIGN KEY to payment_type(id) | References the type of payment associated with the driver, if applicable. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | References the status of the driver. |
| description | TEXT |  | Additional details about the driver. |
| signature_required | BOOLEAN | NOT NULL, DEFAULT FALSE | Specifies if a signature is required for the driver. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`driver_type`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for each driver type. |
| name | VARCHAR(255) | NOT NULL, UNIQUE | Name of the driver type (e.g., 'Included'). |
| description | TEXT |  | Additional details about the driver type. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the driver type record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`email_address`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT UNSIGNED | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the email address record. |
| email_address_type_id | BIGINT UNSIGNED | NOT NULL, FOREIGN KEY to email_address_type(id) | References the type of email address. |
| value | VARCHAR(255) | NOT NULL, UNIQUE | The actual email address. |
| email_verified_at | TIMESTAMP | NULL | Timestamp indicating when the email was verified. |
| description | TEXT |  | Additional details about the email address. |
| status_id | BIGINT UNSIGNED | FOREIGN KEY to status(id) | References the status of the email address. |
| created_by | BIGINT UNSIGNED | NOT NULL, FOREIGN KEY to user(id) | User who created the email address record. |
| updated_by | BIGINT UNSIGNED | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`fee`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the fee record. |
| fee_type_id | BIGINT | NOT NULL, FOREIGN KEY to fee_type(id) | References the type of fee. |
| name | VARCHAR(255) | NOT NULL, UNIQUE | Name of the fee record. |
| description | TEXT |  | Additional details about the fee. |
| credit | DECIMAL(18,2) | DEFAULT 0.00 | Credit amount associated with the fee. |
| debit | DECIMAL(18,2) | DEFAULT 0.00 | Debit amount associated with the fee. |
| effective_date | DATE |  | Date when the fee becomes effective. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status of the fee record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the fee record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the fee record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`license`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for each license record. |
| license_type_id | BIGINT | NOT NULL, FOREIGN KEY to license_type(id) | References the type of license. |
| state_id | BIGINT | NOT NULL, FOREIGN KEY to state(id) | References the state where the license was issued. |
| country_id | BIGINT | NOT NULL, FOREIGN KEY to country(id) | References the country where the license was issued. |
| number | VARCHAR(255) | NOT NULL | Unique driver’s license number. |
| issue_date | DATE | NULLABLE | Date the license was issued. |
| expiration_date | DATE | NOT NULL | Date the license expires. |
| description | TEXT |  | Additional details about the license. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the license record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |
| Unique Constraint |  | UNIQUE(license_type_id, state_id, country_id, number) | Ensures uniqueness for the combination of license fields. |

`license_plate`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the license plate. |
| number | VARCHAR(50) | NOT NULL, UNIQUE | License plate number. |
| state_id | BIGINT | NOT NULL, FOREIGN KEY to state(id) | State that issued the license plate. |
| license_plate_type_id | BIGINT | FOREIGN KEY to license_plate_type(id) | Type of the license plate. |
| description | TEXT |  | Additional details about the license plate. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the license plate. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the license plate record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the license plate record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the license plate record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the license plate record was last updated. |

`limit`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the limit record. |
| name | VARCHAR(100) | NOT NULL, UNIQUE | Name of the limit (e.g., "BI Per Person", "PD Per Occurrence"). |
| limit_type_id | BIGINT | NOT NULL, FOREIGN KEY to limit_type(id) | References the type/category of limit. |
| minimum | DECIMAL(15,2) | NOT NULL | The minimum limit amount. |
| maximum | DECIMAL(15,2) | NOT NULL | The maximum limit amount. |
| description | TEXT |  | Additional details about the limit. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the limit. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`map_coverage_premium`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping. |
| coverage_id | BIGINT | NOT NULL, FOREIGN KEY to coverage(id) | References the associated coverage. |
| premium_id | BIGINT | NOT NULL, FOREIGN KEY to premium(id) | References the associated premium. |
| description | TEXT |  | Additional notes about this mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | References the status of the record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp of record creation. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of last record update. |

`map_driver_address`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| driver_id | BIGINT | NOT NULL, FOREIGN KEY to driver(id) | References the driver associated with the address. |
| address_id | BIGINT | NOT NULL, FOREIGN KEY to address(id) | References the address associated with the driver. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the mapping record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the mapping record was last updated. |

`map_driver_contact_number`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping. |
| driver_id | BIGINT | NOT NULL, FOREIGN KEY to driver(id) | References the driver. |
| contact_number_id | BIGINT | NOT NULL, FOREIGN KEY to contact_number(id) | References the contact number. |
| description | TEXT |  | Additional notes about the mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp of creation. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update. |

`map_driver_coverage_limit`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the driver-to-coverage-limit mapping record. |
| driver_id | BIGINT | NOT NULL, FOREIGN KEY to driver(id) | References the driver associated with the coverage limit. |
| coverage_id | BIGINT | NOT NULL, FOREIGN KEY to coverage(id) | References the coverage associated with the driver. |
| limit_id | BIGINT | NOT NULL, FOREIGN KEY to limit(id) | References the limit applied to the coverage. |
| description | TEXT |  | Additional details about the mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`map_driver_discount`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping. |
| driver_id | BIGINT | NOT NULL, FOREIGN KEY to driver(id) | References the driver. |
| discount_id | BIGINT | NOT NULL, FOREIGN KEY to discount(id) | References the discount. |
| description | TEXT |  | Additional details about the mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update. |

`map_driver_email_address`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| driver_id | BIGINT | NOT NULL, FOREIGN KEY to driver(id) | References the driver associated with the email address. |
| email_address_id | BIGINT | NOT NULL, FOREIGN KEY to email_address(id) | References the email address associated with the driver. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the mapping record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the mapping record was last updated. |

`map_driver_premium`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| driver_id | BIGINT | NOT NULL, FOREIGN KEY to driver(id) | References the associated driver. |
| premium_id | BIGINT | NOT NULL, FOREIGN KEY to premium(id) | References the associated premium. |
| description | TEXT |  | Additional notes about this mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update. |

`map_policy_coverage_limit`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the record. |
| policy_id | BIGINT | NOT NULL, FOREIGN KEY to policy(id) | References the policy. |
| coverage_id | BIGINT | NOT NULL, FOREIGN KEY to coverage(id) | References the coverage. |
| limit_id | BIGINT | NOT NULL, FOREIGN KEY to limit(id) | References the limit. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`map_policy_discount`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping. |
| policy_id | BIGINT | NOT NULL, FOREIGN KEY to policy(id) | References the policy. |
| discount_id | BIGINT | NOT NULL, FOREIGN KEY to discount(id) | References the discount. |
| description | TEXT |  | Additional details about the mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update. |

`map_policy_driver`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for each mapping. |
| policy_id | BIGINT | NOT NULL, FOREIGN KEY to policy(id) | References the associated policy. |
| driver_id | BIGINT | NOT NULL, FOREIGN KEY to driver(id) | References the associated driver. |
| description | TEXT |  | Additional details about the policy-driver relationship. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping (e.g., active, inactive). |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |
| Unique Constraint |  | UNIQUE (policy_id, driver_id) | Ensures no duplicate policy-driver mapping. |

`map_policy_driver_vehicle`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping |
| policy_id | BIGINT | NOT NULL, FOREIGN KEY to policy(id) | References the policy table |
| driver_id | BIGINT | NOT NULL, FOREIGN KEY to driver(id) | References the driver table |
| vehicle_id | BIGINT | NOT NULL, FOREIGN KEY to vehicle(id) | References the vehicle table |
| description | TEXT |  | Additional details about the mapping |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | References the status table |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the mapping was created |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the mapping was last updated |

`map_policy_fee`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| policy_id | BIGINT | NOT NULL, FOREIGN KEY to policy(id) | References the associated policy. |
| fee_id | BIGINT | NOT NULL, FOREIGN KEY to fee(id) | References the associated fee. |
| description | TEXT |  | Additional notes about this mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update. |

`map_policy_named_driver`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| policy_id | BIGINT | NOT NULL, FOREIGN KEY to policy(id) | References the policy record. |
| driver_id | BIGINT | NOT NULL, FOREIGN KEY to driver(id) | References the named driver associated with the policy. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the mapping record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the mapping record was last updated. |

`map_policy_premium`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping. |
| policy_id | BIGINT | NOT NULL, FOREIGN KEY to policy(id) | References the associated policy. |
| premium_id | BIGINT | NOT NULL, FOREIGN KEY to premium(id) | References the associated premium. |
| description | TEXT |  | Additional notes about this mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | References the status of the record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp of record creation. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of last record update. |

`map_policy_transaction`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the map record. |
| policy_id | BIGINT | NOT NULL, FOREIGN KEY to policy(id) | References the associated policy. |
| transaction_id | BIGINT | NOT NULL, FOREIGN KEY to transaction(id) | References the associated transaction. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status of the map record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the map record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the map record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`map_producer_address`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| producer_id | BIGINT | NOT NULL, FOREIGN KEY to producer(id) | References the producer. |
| address_id | BIGINT | NOT NULL, FOREIGN KEY to address(id) | References the address. |
| description | TEXT |  | Additional details about the mapping. |
| status_id | BIGINT | FOREIGN KEY to status(id) | Status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`map_producer_contact_number`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| producer_id | BIGINT | NOT NULL, FOREIGN KEY to producer(id) | References the producer. |
| contact_number_id | BIGINT | NOT NULL, FOREIGN KEY to contact_number(id) | References the contact number. |
| description | TEXT |  | Additional details about the mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update. |

`map_producer_name`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT UNSIGNED | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| producer_id | BIGINT UNSIGNED | NOT NULL, FOREIGN KEY to producer(id) | References the producer record. |
| first_name_id | BIGINT UNSIGNED | NOT NULL, FOREIGN KEY to name(id) | References the first name of the producer. |
| middle_name_id | BIGINT UNSIGNED | FOREIGN KEY to name(id) | References the middle name of the producer. |
| last_name_id | BIGINT UNSIGNED | NOT NULL, FOREIGN KEY to name(id) | References the last name of the producer. |
| surname_name_id | BIGINT UNSIGNED | FOREIGN KEY to name(id) | References the surname of the producer. |
| description | TEXT |  | Additional details about the mapping. |
| status_id | BIGINT UNSIGNED | FOREIGN KEY to status(id) | References the status of the mapping. |
| created_by | BIGINT UNSIGNED | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT UNSIGNED | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`map_producer_policy`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| producer_id | BIGINT | NOT NULL, FOREIGN KEY to producer(id) | References the producer associated with the policy. |
| policy_id | BIGINT | NOT NULL, FOREIGN KEY to policy(id) | References the policy related to the producer. |
| description | TEXT |  | Additional details about the producer-policy relationship. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`map_transaction_payment`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the map record. |
| transaction_id | BIGINT | NOT NULL, FOREIGN KEY to transaction(id) | References the associated transaction. |
| payment_id | BIGINT | NOT NULL, FOREIGN KEY to payment(id) | References the associated payment. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status of the map record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the map record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the map record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`map_user_policy_driver`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| user_id | BIGINT | NOT NULL, FOREIGN KEY to user(id) | References the user associated with the policy and driver. |
| policy_id | BIGINT | NOT NULL, FOREIGN KEY to policy(id) | References the policy associated with the user and driver. |
| driver_id | BIGINT | NOT NULL, FOREIGN KEY to driver(id) | References the driver associated with the user and policy. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the mapping record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the mapping record was last updated. |

`map_vehicle_coverage_limit`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the vehicle-to-coverage-limit mapping record. |
| vehicle_id | BIGINT | NOT NULL, FOREIGN KEY to vehicle(id) | References the vehicle associated with the coverage limit. |
| coverage_id | BIGINT | NOT NULL, FOREIGN KEY to coverage(id) | References the coverage associated with the vehicle. |
| limit_id | BIGINT | NOT NULL, FOREIGN KEY to limit(id) | References the limit applied to the coverage. |
| description | TEXT |  | Additional details about the mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`map_vehicle_discount`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping. |
| vehicle_id | BIGINT | NOT NULL, FOREIGN KEY to vehicle(id) | References the vehicle. |
| discount_id | BIGINT | NOT NULL, FOREIGN KEY to discount(id) | References the discount. |
| description | TEXT |  | Additional details about the mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update. |

`map_vehicle_premium`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| vehicle_id | BIGINT | NOT NULL, FOREIGN KEY to vehicle(id) | References the associated vehicle. |
| premium_id | BIGINT | NOT NULL, FOREIGN KEY to premium(id) | References the associated premium. |
| description | TEXT |  | Additional notes about this mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update. |

`name`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT UNSIGNED | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the name record. |
| name_type_id | BIGINT UNSIGNED | NOT NULL, FOREIGN KEY to name_type(id) | References the type of the name. |
| value | VARCHAR(255) | NOT NULL | The actual name value (e.g., 'John', 'Smith'). |
| description | TEXT |  | Additional details about the name. |
| status_id | BIGINT UNSIGNED | FOREIGN KEY to status(id) | References the status of the name record. |
| created_by | BIGINT UNSIGNED | NOT NULL, FOREIGN KEY to user(id) | User who created the name record. |
| updated_by | BIGINT UNSIGNED | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update. |

`policy`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the policy record. |
| policy_prefix_id | BIGINT | NULLABLE, FOREIGN KEY to policy_prefix(id) | References the policy prefix. |
| number | VARCHAR(50) | NULLABLE, UNIQUE(policy_prefix_id, number) | Unique policy identifier within a prefix scope. |
| policy_type_id | BIGINT | NOT NULL, FOREIGN KEY to policy_type(id) | References the type of policy. |
| effective_date | DATE | NULLABLE | The start date of the policy. |
| inception_date | DATE | NULLABLE | The inception date of the policy (first effective date). |
| expiration_date | DATE | NOT NULL | The end date of the policy. |
| renewal_date | DATE | NULLABLE | Date when the policy is set for renewal. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | References the status of the policy. |
| term_id | BIGINT | NOT NULL, FOREIGN KEY to term(id) | References the term of the policy. |
| description | TEXT | NULLABLE | Additional details about the policy. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`policy_prefix`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for each policy prefix. |
| name | VARCHAR(50) | NOT NULL, UNIQUE | Unique policy prefix (e.g., 'PLCY-', 'AUTO-'). |
| description | TEXT |  | Additional details about the policy prefix. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | References the status of the policy prefix. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`premium`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the premium record. |
| premium_type_id | BIGINT | NOT NULL, FOREIGN KEY to premium_type(id) | Type of premium associated with the record. |
| name | VARCHAR(255) | NOT NULL, UNIQUE | Name of the premium record. |
| description | TEXT |  | Additional details about the premium. |
| credit | DECIMAL(12,2) | CHECK (credit >= 0) | Credit value associated with the premium. |
| debit | DECIMAL(12,2) | CHECK (debit >= 0) | Debit value associated with the premium. |
| effective_date | DATE |  | Date when the premium becomes effective. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the premium record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the premium record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the premium record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the premium record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the premium record was last updated. |

`producer`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for each producer/agency. |
| producer_code_id | BIGINT | NOT NULL, FOREIGN KEY to producer_code(id), UNIQUE | Unique producer code for identification. |
| number | VARCHAR(50) | NOT NULL, UNIQUE | Unique number assigned to the producer. |
| name | VARCHAR(255) | NOT NULL, UNIQUE | Name of the producer/agency. |
| description | TEXT |  | Additional details about the producer. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status of the producer. |
| producer_type_id | BIGINT | NOT NULL, FOREIGN KEY to producer_type(id) | References the type of producer associated with the program. |
| signature_required | BOOLEAN | NOT NULL, DEFAULT FALSE | Specifies if a signature is required for the producer. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the producer record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the producer record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`state`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for each state. |
| name | VARCHAR(255) | NOT NULL | The full name of the state (e.g., Texas, California). |
| code | VARCHAR(10) | NOT NULL | Abbreviated state code (e.g., TX, CA). |
| description | TEXT |  | Additional details or notes about the state. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the state record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |
| Unique Constraint |  | UNIQUE (name, code) | Ensures no duplicate entries for state name and code. |

`status`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the status record. |
| name | VARCHAR(100) | NOT NULL, UNIQUE | Name of the status (e.g., 'Active', 'Pending'). |
| description | TEXT |  | Additional details about the status. |
| created_by | BIGINT | FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`transaction`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the transaction. |
| transaction_type_id | BIGINT | NOT NULL, FOREIGN KEY to transaction_type(id) | References the type of transaction. |
| name | VARCHAR(255) | NOT NULL | Name of the transaction. |
| description | TEXT |  | Additional details about the transaction, including notes like "all current and future transactions (like installments)." |
| credit | DECIMAL(12,2) | CHECK (credit >= 0) | Total credit amount for the transaction. |
| debit | DECIMAL(12,2) | CHECK (debit >= 0) | Total debit amount for the transaction. |
| confirmation_code | VARCHAR(255) |  | Confirmation code associated with the transaction. |
| due_date | DATE |  | The date when the transaction is due. |
| processed_date | TIMESTAMP |  | The date when the transaction was processed. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the transaction. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the transaction. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the transaction. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the transaction was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update to the transaction. |

`user`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for each user. |
| user_type_id | BIGINT | NOT NULL, FOREIGN KEY to user_type(id) | References the type of user (e.g., insured, producer, claims, system). |
| user_group_id | BIGINT | NOT NULL, FOREIGN KEY to user_group(id) | References the user group (e.g., manager, csr, underwriting). |
| username | VARCHAR(255) | UNIQUE, NOT NULL | Username |
| password | VARCHAR(255) | NOT NULL | User's hashed password. |
| description | TEXT |  | Additional details about the user. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the user's status (e.g., Active, Inactive). |
| created_by | BIGINT | FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`vehicle`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the vehicle. |
| vehicle_type_id | BIGINT | NOT NULL, FOREIGN KEY to vehicle_type(id) | References the vehicle type table. |
| vehicle_year_id | BIGINT | NOT NULL, FOREIGN KEY to vehicle_year(id) | References the vehicle year table. |
| vehicle_make_id | BIGINT | NOT NULL, FOREIGN KEY to vehicle_make(id) | References the vehicle make table. |
| vehicle_model_id | BIGINT | NOT NULL, FOREIGN KEY to vehicle_model(id) | References the vehicle model table. |
| license_plate_id | BIGINT | FOREIGN KEY to license_plate(id) | References the license plate assigned to the vehicle. |
| vin | VARCHAR(255) | NOT NULL, UNIQUE | Vehicle Identification Number (VIN). |
| mileage | BIGINT |  | Mileage of the vehicle. |
| description | TEXT |  | Additional details about the vehicle. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the vehicle. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the vehicle record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the vehicle record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the vehicle record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the vehicle record was last updated. |

`vehicle_make`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the vehicle make |
| name | VARCHAR(255) | NOT NULL, UNIQUE | Name of the vehicle make (e.g., 'Toyota', 'Ford') |
| description | TEXT |  | Additional details about the vehicle make |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | References the status table |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated |

`vehicle_model`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the vehicle model |
| vehicle_make_id | BIGINT | NOT NULL, FOREIGN KEY to vehicle_make(id) | References the vehicle make |
| name | VARCHAR(255) | NOT NULL | Name of the vehicle model (e.g., 'Camry', 'F-150') |
| description | TEXT |  | Additional details about the vehicle model |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | References the status table |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated |

`vehicle_year`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the vehicle year |
| value | INT | NOT NULL, UNIQUE | Year of the vehicle (e.g., 2020, 2021) |
| description | TEXT |  | Additional details about the vehicle year |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | References the status table |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated |

# #3 - Coverage

## **A) WHY - Vision and Purpose**

The **Coverage** section in the **Policy Summary** provides users with a **clear, structured breakdown of their insurance coverage details**. It ensures **policy transparency** by showing coverage types, limits, and associated premiums. Users can **easily review what is covered** under their policy, ensuring they have adequate protection and avoiding potential gaps in coverage.

## **B) WHAT - Core Requirements**

### **1. Coverage Breakdown**

- Display **each coverage type** included in the policy.
- Show key details for each coverage:
    - **Coverage Type** (e.g., Bodily Injury, Property Damage)
        - `map_policy_coverage_limit` by [`policy.id`](http://policy.id/) -> get `map_vehicle_coverage_limit` by `map_policy_driver_vehicle.vehicle_id` -> get `map_driver_coverage_limit` by `map_policy_driver_vehicle.driver_id` -> get [`coverage.id`](http://coverage.id/) by `map_policy_coverage_limit.coverage_id` and `map_vehicle_coverage_limit.coverage_id` and `map_driver_coverage_limit.coverage_id` -> get `coverage` by [`coverage.id`](http://coverage.id/) -> get `coverage_type.id` by `coverage.coverage_type_id` -> get `coverage_type.name` by `coverage_type.id`
    - **Coverage Limits** (e.g., $50,000/person - $100,000/accident)
        - Coverage Limit → get `map_policy_driver_vehicle` by [`policy.id`](http://policy.id/) -> get `map_policy_coverage_limit` by [`policy.id`](http://policy.id/) -> get `map_vehicle_coverage_limit` by `map_policy_driver_vehicle.vehicle_id` -> get [`coverage.id`](http://coverage.id/) by `map_policy_coverage_limit.coverage_id` and `map_vehicle_coverage_limit.coverage_id` -> get [`limit.id`](http://limit.id/) by `map_policy_coverage_limit` and `map_vehicle_coverage_limit`
    - **Premium Amount** (e.g., $210.00 for Bodily Injury)
        - Coverage Premium → get `map_policy_driver_vehicle` by [`policy.id`](http://policy.id/) -> get `map_policy_coverage_limit` by [`policy.id`](http://policy.id/) -> get `map_vehicle_coverage_limit` by `map_policy_driver_vehicle.vehicle_id` -> get `map_driver_coverage_limit` by `map_policy_driver_vehicle.driver_id` -> get [`coverage.id`](http://coverage.id/) by `map_policy_coverage_limit.coverage_id` and `map_vehicle_coverage_limit.coverage_id` and `map_driver_coverage_limit.coverage_id` -> get `coverage` by [`coverage.id`](http://coverage.id/) -> get `map_coverage_premium` by [`coverage.id`](http://coverage.id/) -> get [`premium.id`](http://premium.id/) by `map_coverage_premium.premium_id` -> get `premium` by [`premium.id`](http://premium.id/) -> get `sum` of `premium.credit` and `premium.debit`

### **2. Total Policy Cost**

- Display a **total premium amount** summing up all coverage types.
    - Total Policy Premium
        - Retrieve the policy record:
            - Use `map_user_policy_driver` by `user.id` to obtain `policy.id`.
            - Retrieve the policy record from `policy` to access policy details.
        - Retrieve premium records:
            - Use `map_policy_premium_transaction` where `policy_id` = `policy.id` to fetch all associated premium records.
        - Calculate Base Premium:
            - For each premium record retrieved from premium, compute:
                - `premium_net` = `premium.credit` – `premium.debit`.
        - Compute Total Term Premium:
            - Sum all `premium_net` values from the premium records to obtain the Total Term Premium Due for the entire term.

### **3. Interactive Features**

- Ability to **expand/collapse coverage details** for better readability.
- Quick navigation between **Overview, Drivers & Vehicles, and Coverage** tabs.

## **C) HOW - Planning & Implementation**

**Full Mapping of Front-End Fields to Database Fields**

- Producer Label Configuration → Stored in the appropriate application side configuration file.
- Policy Number → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `policy` by [`policy.id`](http://policy.id/) -> get `policy_prefix` by `policy.policy_prefix_id` -> concat `policy_prefix.value` and `policy.number`
- Policy Status → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `policy` by [`policy.id`](http://policy.id/) -> `policy.status_id`
- Named Insured Name → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `map_policy_named_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by m`ap_policy_named_driver.driver_id` -> get `driver` by [`driver.id](http://driver.id/)` -> get `name` by `driver.name_id`
- Next Payment Amount
    - Retrieve the policy record:
        - Use `map_user_policy_driver` by `user.id` to obtain `policy.id`.
        - Retrieve the policy record from policy to access `installment_due_day` and `term_months`.
    - Aggregate premium data:
        - Use `map_policy_premium_transaction` where `policy_id` = `policy.id`.
        - For each premium record from premium, calculate:

          `premium_net` = `premium.credit` – `premium.debit`.

        - Sum all `premium_net` values to obtain Total Premium.
    - Aggregate fee data:
        - Use `map_policy_fee_transaction` where `policy_id` = `policy.id`.
        - For each fee record from fee, calculate:

          `fee_net` = `fee.debit` – `fee.credit`.

        - Sum all `fee_net` values to obtain Total Fees.
    - Aggregate payment transactions:
        - Use `map_policy_installment_transaction` where `policy_id` = `policy.id`.
        - For each transaction from transaction, sum the debit amounts (payments are recorded as debits).
        - Sum these amounts to get Total Payments Applied.
    - Compute net premium remaining:
        - Net Premium Remaining = (Total Premium + Total Fees) – (Total Payments Applied).
    - Determine the remaining installment count:
        - Count the number of installment records for the policy via `map_policy_installment_transaction`.
        - Remaining Installments = `policy.term_months` – (count of installment records).
    - Calculate Next Payment Amount:
        - If an installment record exists (with status "Pending" or "Partially Paid"):
            - Next Payment Amount = `installment.total` – `installment.paid_to_date`.
        - If no installment record exists:
            - Next Installment Amount = Net Premium Remaining / Remaining Installments.
    - Table Summary:
        - `policy` (`id`, `installment_due_day`, `term_months`)
        - `map_user_policy_driver` (`user.id`, `policy.id`)
        - `map_policy_premium_transaction` (`policy_id`, `premium_id`)
        - `premium` (`credit`, `debit`)
        - `map_policy_fee_transaction` (`policy_id`, `fee_id`)
        - `fee` (`debit`, `credit`)
        - `map_policy_installment_transaction` (`policy_id`, `installment_id`)
        - `transaction` (`debit`, `processed_date`)
        - `installment` (`total`, `paid_to_date`, `due_date`, `status_id`)
- Due Date of Next Payment
    - Retrieve the policy record:
        - Use `map_user_policy_driver` by `user.id` to obtain `policy.id`.
        - Retrieve the policy record from `policy` to access `installment_due_day`.
    - Check for outstanding items:
        - Retrieve any installment records for the policy via `map_policy_installment_transaction` with status "Pending" or "Partially Paid" and extract their `due_date` values.
        - Retrieve fee records via `map_policy_fee_transaction` (if fees have due dates and outstanding balances) and extract their `due_date` values.
        - Identify the earliest due date among these outstanding installments/fees.
    - Calculate the next billing date dynamically:
        - If today’s date is before the `installment_due_day`, then set the calculated next due date to the current month’s `installment_due_day`.
            - Example: If today is the 10th and `installment_due_day` is 15, then next due date = the 15th of the current month.
        - If today’s date is on or after the `installment_due_day`, then set the calculated next due date to the `installment_due_day` in the next month.
            - Example: If today is the 16th and `installment_due_day` is 15, then next due date = the 15th of the next month.
    - Determine the effective due date:
        - Compare the dynamically calculated due date with the earliest due date from outstanding installments/fees (if any exist).
        - The Next Payment Due Date is the earliest of:
            - The dynamically calculated next due date based on `installment_due_day`, or
            - The earliest `due_date` among outstanding installment/fee records.
    - Set the due date in the new installment record:
        - When generating a new installment record (via a cronjob or event trigger), assign its `due_date` field to the determined Next Payment Due Date.
    - This `due_date` becomes the Due Date of the Next Payment.
    - Table Summary:
        - `policy` (`id`, `installment_due_day`)
        - `map_user_policy_driver` (`user.id`, `policy.id`)
        - `map_policy_installment_transaction` (`policy_id`, `installment_id`) and `installment` (`due_date`, `status_id`)
        - `map_policy_fee_transaction` (`policy_id`, `fee_id`)
        - `fee` (`due_date`)
- Named Insured Address → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `map_policy_named_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by `map_policy_named_driver.driver_id` -> get `map_driver_address` by [`driver.id`](http://driver.id/) -> get [`address.id`](http://address.id/) by `map_driver_address.address_id` -> get `address` by `address_id`
- Named Insured Email → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `map_policy_named_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by `map_policy_named_driver.driver_id` -> get `map_driver_email_address` by [`driver.id`](http://driver.id/) -> get `email_address.id` by `map_driver_email_address.email_address_id` -> get `email_address` by `email_address_id`
- Named Insured Primary Phone Number → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `map_policy_named_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by `map_policy_named_driver.driver_id` -> get `map_driver_contact_number` by [`driver.id`](http://driver.id/) -> get `contact_number.id` by `map_driver_contact_number.contact_number_id` -> get `contact_number` by `contact_number_id` and `contact_number.contact_number_type` for primary
- Named Insured Alternate Phone Number → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `map_policy_named_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by `map_policy_named_driver.driver_id` -> `get map_driver_contact_number` by [driver.id](http://driver.id/) -> get `contact_number.id` by `map_driver_contact_number.contact_number_id` -> get `contact_number` by `contact_number_id` and `contact_number.contact_number_type` for alternate
- Effective Date → get [`policy.id`](http://policy.id/) from map_user_policy_driver by [user.id](http://user.id/) -> get policy by [`policy.id`](http://policy.id/) -> `policy.effective_date`
- Inception Date → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `policy` by [`policy.id`](http://policy.id/) -> `policy.inception_date`
- Expiration Date → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `policy` by [`policy.id`](http://policy.id/) -> `policy.expiration_date`
- Total Policy Premium → get [`policy.id`](http://`policy.id`/) from `map_user_policy_driver` by [`user.id`](http://`user.id`/) -> get `policy` by [`policy.id`](http://`policy.id`/) -> `map_policy_transaction` -> get all `transaction_id`'s -> foreach [`transaction.id`](http://`transaction.id`/) -> sum of `transaction.credit` and `transaction.debit` filtered by `transaction_type_id`'s to include
- Remaining Premium Balance → get [`policy.id`](http://`policy.id`/) from `map_user_policy_driver` by [`user.id`](http://`user.id`/) -> get `policy` by [`policy.id`](http://`policy.id`/) -> `map_policy_transaction` -> get all `transaction_id`'s -> foreach [`transaction.id`](http://`transaction.id`/) -> sum of `transaction.credit` and `transaction.debit` filtered by `transaction_type_id`'s to include
- Policy Term → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `policy` by [`policy.id`](http://policy.id/) -> `policy.term_id`
- Payment Method → get [`driver.id`](http://driver.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `driver` by [`driver.id`](http://driver.id/) -> `driver.payment_type_id`
- Applied Discounts → get `map_policy_driver_vehicle` by `policy.id` -> get `driver.id` and `vehicle.id` by `map_policy_driver_vehicle.driver_id` and `map_policy_driver_vehicle.vehicle_id` -> get `map_policy_discount` and get `map_vehicle_discount` and get `map_driver_discount` by `driver.id` and `vehicle.id` and `policy.id` -> get `discount.id` by `map_driver_discount.discount_id` and `map_vehicle_discount.discount_id` -> sum `discount.credit` and `discount.debit`
- Producer Phone Number → get [`policy.id`](http://policy.id/) from `map_user_policy_driver` by [`user.id`](http://user.id/) -> get `map_producer_policy` by [`policy.id`](http://policy.id/) -> get [`producer.id`](http://producer.id/) by `map_producer_policy.producer_id` -> get `map_producer_contact_number` by [`producer.id`](http://producer.id/) -> get `contact_number` by `map_producer_contact_number.contact_number_id`
- Producer Name → get `policy.id` from `map_user_policy_driver` by `user.id` -> get `map_producer_policy` by `policy.id` -> get `producer.id` by `map_producer_policy.producer_id` -> get `map_producer_name` by `producer.id`
- Producer Address → get `policy.id` from `map_user_policy_driver` by `user.id` -> get `map_producer_policy` by `policy.id` -> get `producer.id` by `map_producer_policy.producer_id` -> get `address.id` from `map_producer_address` by `producer.id` -> get address by `address.id`
- Driver Name → get `map_policy_driver` by `policy.id` -> get `driver.id` by `map_policy_driver.driver_id` -> get driver by `driver.id` -> get `name.id` by `driver.name_id` -> get `name`
- Primary Driver Tag → get `map_policy_named_driver` by `policy.id` -> get `driver.id` by `map_policy_named_driver.driver_id` -> named driver is the primary driver
- Driver Status (Included vs Excluded) → get `map_policy_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by `map_policy_driver.driver_id` -> get `driver` by [`driver.id`](http://driver.id/) -> get `driver_type.id` by `driver.driver_type_id` -> driver types defined for included and excluded
- Driver Date of Birth → get `map_policy_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by `map_policy_driver.driver_id` -> get `driver` by [`driver.id`](http://driver.id/) -> get `driver.date_of_birth`
- Driver’s License Number → get `map_policy_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by `map_policy_driver.driver_id` -> get `driver` by [`driver.id`](http://driver.id/) -> get [`license.id`](http://license.id/) by `driver.license_id` -> get `license` by [`license.id`](http://license.id/)
- Issuing State for License → get `map_policy_driver` by [`policy.id`](http://policy.id/) -> get [`driver.id`](http://driver.id/) by `map_policy_driver.driver_id` -> get `driver` by [`driver.id`](http://driver.id/) -> get [`license.id`](http://license.id/) by `driver.license_id` -> get `license` by [`license.id`](http://license.id/) -> get [`state.id`](http://state.id/) by `license.state_id` -> get `state` by [`state.id`](http://state.id/)
- Vehicle Year → get `map_policy_driver_vehicle` by [`policy.id`](http://policy.id/) -> get [`vehicle.id`](http://vehicle.id/) by `map_policy_driver_vehicle.vehicle_id` -> get `vehicle_year.id` by `vehicle.vehicle_year_id` -> get `vehicle_year` by `vehicle_year.id` -> get `vehicle_year.value`
- Vehicle Make → get `map_policy_driver_vehicle` by [`policy.id`](http://policy.id/) -> get [`vehicle.id`](http://vehicle.id/) by `map_policy_driver_vehicle.vehicle_id` -> get `vehicle_make.id` by `vehicle.vehicle_make_id` -> get `vehicle_make` by `vehicle_make.id` -> get `vehicle_make.name`
- Vehicle Model → get `map_policy_driver_vehicle` by [policy.id](http://policy.id/) -> get [`vehicle.id`](http://vehicle.id/) by `map_policy_driver_vehicle.vehicle_id` -> get `vehicle_model.id` by `vehicle.vehicle_model_id` -> get `vehicle_model` by `vehicle_model.id` and `vehicle.vehicle_make_id` -> get `vehicle_model.name`
- Vehicle VIN → get `map_policy_driver_vehicle` by [`policy.id`](http://policy.id/) -> get [`vehicle.id`](http://vehicle.id/) by `map_policy_driver_vehicle.vehicle_id` -> get `vehicle` by [`vehicle.id`](http://vehicle.id/) -> get `vehicle.vin`
- Vehicle License Plate Number → get `map_policy_driver_vehicle` by [`policy.id`](http://policy.id/) -> get [`vehicle.id`](http://vehicle.id/) by `map_policy_driver_vehicle.vehicle_id` -> get `vehicle` by [`vehicle.id`](http://vehicle.id/) -> get `license_plate` by `vehicle.license_plate_id` -> get `license_plate.number`
- Coverage Type → get `map_policy_driver_vehicle` by [`policy.id`](http://policy.id/) -> get `map_policy_coverage_limit` by [`policy.id`](http://policy.id/) -> get `map_vehicle_coverage_limit` by `map_policy_driver_vehicle.vehicle_id` -> get `map_driver_coverage_limit` by `map_policy_driver_vehicle.driver_id` -> get [`coverage.id`](http://coverage.id/) by `map_policy_coverage_limit.coverage_id` and `map_vehicle_coverage_limit.coverage_id` and `map_driver_coverage_limit.coverage_id` -> get `coverage` by [`coverage.id`](http://coverage.id/) -> get `coverage_type.id` by `coverage.coverage_type_id` -> get `coverage_type.name` by `coverage_type.id`
- Coverage Limit → get `map_policy_driver_vehicle` by [`policy.id`](http://policy.id/) -> get `map_policy_coverage_limit` by [`policy.id`](http://policy.id/) -> get `map_vehicle_coverage_limit` by `map_policy_driver_vehicle.vehicle_id` -> get [`coverage.id`](http://coverage.id/) by `map_policy_coverage_limit.coverage_id` and `map_vehicle_coverage_limit.coverage_id` -> get [`limit.id`](http://limit.id/) by `map_policy_coverage_limit` and `map_vehicle_coverage_limit`
- Coverage Premium → get `map_policy_driver_vehicle` by [`policy.id`](http://policy.id/) -> get `map_policy_coverage_limit` by [`policy.id`](http://policy.id/) -> get `map_vehicle_coverage_limit` by `map_policy_driver_vehicle.vehicle_id` -> get `map_driver_coverage_limit` by `map_policy_driver_vehicle.driver_id` -> get [`coverage.id`](http://coverage.id/) by `map_policy_coverage_limit.coverage_id` and `map_vehicle_coverage_limit.coverage_id` and `map_driver_coverage_limit.coverage_id` -> get `coverage` by [`coverage.id`](http://coverage.id/) -> get `map_coverage_premium` by [`coverage.id`](http://coverage.id/) -> get [`premium.id`](http://premium.id/) by `map_coverage_premium.premium_id` -> get `premium` by [`premium.id`](http://premium.id/) -> get `sum` of `premium.credit` and `premium.debit`
- Total Policy Premium
    - Retrieve the policy record:
        - Use `map_user_policy_driver` by `user.id` to obtain `policy.id`.
        - Retrieve the policy record from `policy` to access policy details.
    - Retrieve premium records:
        - Use `map_policy_premium_transaction` where `policy_id` = `policy.id` to fetch all associated premium records.
    - Calculate Base Premium:
        - For each premium record retrieved from premium, compute:
            - `premium_net` = `premium.credit` – `premium.debit`.
    - Compute Total Term Premium:
        - Sum all `premium_net` values from the premium records to obtain the Total Term Premium Due for the entire term.
    - Table Summary:
        - `policy` `(id`)
        - `map_user_policy_driver` (`user.id`, `policy.id`)
        - `map_policy_premium_transaction` (`policy_id`, `premium_id`)
        - `premium` (`credit`, `debit`)

## **D) User Experience (UX) & Flows**

### **1. Viewing Policy Coverage**

1. User navigates to **Policy Summary → Coverage** tab.
2. The system displays a **list of all coverage types, limits, and associated premiums**.
3. Users can **expand/collapse sections** to improve navigation.

### **2. Understanding Coverage Costs**

1. Users see a **breakdown of coverage premiums**.
2. The system displays a **total premium amount** at the bottom.

## **E) Master Schema Tables**

`address`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for each address. |
| address_type_id | BIGINT | NOT NULL, FOREIGN KEY to address_type(id) | References the type of address (e.g., residential, billing). |
| county_id | BIGINT | NOT NULL, FOREIGN KEY to county(id) | References the county associated with the address. |
| city_id | BIGINT | NOT NULL, FOREIGN KEY to city(id) | The name of the city. |
| state_id | BIGINT | NOT NULL, FOREIGN KEY to state(id) | References the state associated with the address. |
| zip_code_id | BIGINT | NOT NULL, FOREIGN KEY to zip_code(id) | References the ZIP code associated with the address. |
| description | TEXT |  | Additional details about the address. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status of the address. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the address record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the address record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`contact_number`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the contact number. |
| contact_number_type_id | BIGINT | NOT NULL, FOREIGN KEY to contact_number_type(id) | References the type of contact number. |
| value | VARCHAR(20) | NOT NULL, UNIQUE | The contact number (e.g., phone, fax). |
| description | TEXT |  | Additional details about the contact number. |
| status_id | BIGINT | FOREIGN KEY to status(id) | Status of the contact number record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the contact number record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the contact number record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update. |

`coverage`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for each coverage record. |
| coverage_type_id | BIGINT | NOT NULL, FOREIGN KEY to coverage_type(id) | References the type of coverage. |
| name | VARCHAR(50) | NOT NULL, UNIQUE | Name of the coverage. |
| description | TEXT |  | Additional details about the coverage. |
| effective_date | DATE |  | The date when the coverage becomes effective. |
| expiration_date | DATE |  | The date when the coverage expires. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | References the status of the coverage. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`coverage_type`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the coverage type. |
| name | VARCHAR(50) | NOT NULL, UNIQUE | Name of the coverage type (e.g., 'Standard'). |
| description | TEXT |  | Additional details about the coverage type. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the coverage type. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the coverage type record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the coverage type. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`discount`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for each discount record. |
| discount_type_id | BIGINT | NOT NULL, FOREIGN KEY to discount_type(id) | References the type of discount (e.g., Good Driver, Safe Vehicle). |
| name | VARCHAR(255) | NOT NULL, UNIQUE | Name of the discount. |
| description | TEXT |  | Additional details about the discount. |
| credit | DECIMAL(12,2) | CHECK (credit >= 0) | Amount credited due to the discount. |
| debit | DECIMAL(12,2) | CHECK (debit >= 0) | Amount debited, if applicable, in special scenarios. |
| effective_date | DATE | NOT NULL | Date the discount becomes effective. |
| expiration_date | DATE | NOT NULL | Date the discount expires. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | References the current status of the discount. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the discount record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the discount record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`driver`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for each driver record. |
| date_of_birth | DATE | NOT NULL | Driver's date of birth. |
| name_id | BIGINT | NOT NULL, FOREIGN KEY to name(id) | References the driver's name record. |
| license_id | BIGINT | NOT NULL, FOREIGN KEY to license(id) | References the driver's license record. |
| driver_type_id | BIGINT | NOT NULL, FOREIGN KEY to driver_type(id) | References the type of driver (e.g., Included). |
| payment_type_id | BIGINT | FOREIGN KEY to payment_type(id) | References the type of payment associated with the driver, if applicable. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | References the status of the driver. |
| description | TEXT |  | Additional details about the driver. |
| signature_required | BOOLEAN | NOT NULL, DEFAULT FALSE | Specifies if a signature is required for the driver. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`driver_type`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for each driver type. |
| name | VARCHAR(255) | NOT NULL, UNIQUE | Name of the driver type (e.g., 'Included'). |
| description | TEXT |  | Additional details about the driver type. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the driver type record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`email_address`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT UNSIGNED | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the email address record. |
| email_address_type_id | BIGINT UNSIGNED | NOT NULL, FOREIGN KEY to email_address_type(id) | References the type of email address. |
| value | VARCHAR(255) | NOT NULL, UNIQUE | The actual email address. |
| email_verified_at | TIMESTAMP | NULL | Timestamp indicating when the email was verified. |
| description | TEXT |  | Additional details about the email address. |
| status_id | BIGINT UNSIGNED | FOREIGN KEY to status(id) | References the status of the email address. |
| created_by | BIGINT UNSIGNED | NOT NULL, FOREIGN KEY to user(id) | User who created the email address record. |
| updated_by | BIGINT UNSIGNED | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`fee`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the fee record. |
| fee_type_id | BIGINT | NOT NULL, FOREIGN KEY to fee_type(id) | References the type of fee. |
| name | VARCHAR(255) | NOT NULL, UNIQUE | Name of the fee record. |
| description | TEXT |  | Additional details about the fee. |
| credit | DECIMAL(18,2) | DEFAULT 0.00 | Credit amount associated with the fee. |
| debit | DECIMAL(18,2) | DEFAULT 0.00 | Debit amount associated with the fee. |
| effective_date | DATE |  | Date when the fee becomes effective. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status of the fee record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the fee record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the fee record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`license`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for each license record. |
| license_type_id | BIGINT | NOT NULL, FOREIGN KEY to license_type(id) | References the type of license. |
| state_id | BIGINT | NOT NULL, FOREIGN KEY to state(id) | References the state where the license was issued. |
| country_id | BIGINT | NOT NULL, FOREIGN KEY to country(id) | References the country where the license was issued. |
| number | VARCHAR(255) | NOT NULL | Unique driver’s license number. |
| issue_date | DATE | NULLABLE | Date the license was issued. |
| expiration_date | DATE | NOT NULL | Date the license expires. |
| description | TEXT |  | Additional details about the license. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the license record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |
| Unique Constraint |  | UNIQUE(license_type_id, state_id, country_id, number) | Ensures uniqueness for the combination of license fields. |

`license_plate`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the license plate. |
| number | VARCHAR(50) | NOT NULL, UNIQUE | License plate number. |
| state_id | BIGINT | NOT NULL, FOREIGN KEY to state(id) | State that issued the license plate. |
| license_plate_type_id | BIGINT | FOREIGN KEY to license_plate_type(id) | Type of the license plate. |
| description | TEXT |  | Additional details about the license plate. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the license plate. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the license plate record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the license plate record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the license plate record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the license plate record was last updated. |

`limit`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the limit record. |
| name | VARCHAR(100) | NOT NULL, UNIQUE | Name of the limit (e.g., "BI Per Person", "PD Per Occurrence"). |
| limit_type_id | BIGINT | NOT NULL, FOREIGN KEY to limit_type(id) | References the type/category of limit. |
| minimum | DECIMAL(15,2) | NOT NULL | The minimum limit amount. |
| maximum | DECIMAL(15,2) | NOT NULL | The maximum limit amount. |
| description | TEXT |  | Additional details about the limit. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the limit. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`map_coverage_premium`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping. |
| coverage_id | BIGINT | NOT NULL, FOREIGN KEY to coverage(id) | References the associated coverage. |
| premium_id | BIGINT | NOT NULL, FOREIGN KEY to premium(id) | References the associated premium. |
| description | TEXT |  | Additional notes about this mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | References the status of the record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp of record creation. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of last record update. |

`map_driver_address`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| driver_id | BIGINT | NOT NULL, FOREIGN KEY to driver(id) | References the driver associated with the address. |
| address_id | BIGINT | NOT NULL, FOREIGN KEY to address(id) | References the address associated with the driver. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the mapping record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the mapping record was last updated. |

`map_driver_contact_number`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping. |
| driver_id | BIGINT | NOT NULL, FOREIGN KEY to driver(id) | References the driver. |
| contact_number_id | BIGINT | NOT NULL, FOREIGN KEY to contact_number(id) | References the contact number. |
| description | TEXT |  | Additional notes about the mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp of creation. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update. |

`map_driver_coverage_limit`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the driver-to-coverage-limit mapping record. |
| driver_id | BIGINT | NOT NULL, FOREIGN KEY to driver(id) | References the driver associated with the coverage limit. |
| coverage_id | BIGINT | NOT NULL, FOREIGN KEY to coverage(id) | References the coverage associated with the driver. |
| limit_id | BIGINT | NOT NULL, FOREIGN KEY to limit(id) | References the limit applied to the coverage. |
| description | TEXT |  | Additional details about the mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`map_driver_discount`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping. |
| driver_id | BIGINT | NOT NULL, FOREIGN KEY to driver(id) | References the driver. |
| discount_id | BIGINT | NOT NULL, FOREIGN KEY to discount(id) | References the discount. |
| description | TEXT |  | Additional details about the mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update. |

`map_driver_email_address`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| driver_id | BIGINT | NOT NULL, FOREIGN KEY to driver(id) | References the driver associated with the email address. |
| email_address_id | BIGINT | NOT NULL, FOREIGN KEY to email_address(id) | References the email address associated with the driver. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the mapping record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the mapping record was last updated. |

`map_driver_premium`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| driver_id | BIGINT | NOT NULL, FOREIGN KEY to driver(id) | References the associated driver. |
| premium_id | BIGINT | NOT NULL, FOREIGN KEY to premium(id) | References the associated premium. |
| description | TEXT |  | Additional notes about this mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update. |

`map_policy_coverage_limit`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the record. |
| policy_id | BIGINT | NOT NULL, FOREIGN KEY to policy(id) | References the policy. |
| coverage_id | BIGINT | NOT NULL, FOREIGN KEY to coverage(id) | References the coverage. |
| limit_id | BIGINT | NOT NULL, FOREIGN KEY to limit(id) | References the limit. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`map_policy_discount`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping. |
| policy_id | BIGINT | NOT NULL, FOREIGN KEY to policy(id) | References the policy. |
| discount_id | BIGINT | NOT NULL, FOREIGN KEY to discount(id) | References the discount. |
| description | TEXT |  | Additional details about the mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update. |

`map_policy_driver`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for each mapping. |
| policy_id | BIGINT | NOT NULL, FOREIGN KEY to policy(id) | References the associated policy. |
| driver_id | BIGINT | NOT NULL, FOREIGN KEY to driver(id) | References the associated driver. |
| description | TEXT |  | Additional details about the policy-driver relationship. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping (e.g., active, inactive). |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |
| Unique Constraint |  | UNIQUE (policy_id, driver_id) | Ensures no duplicate policy-driver mapping. |

`map_policy_driver_vehicle`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping |
| policy_id | BIGINT | NOT NULL, FOREIGN KEY to policy(id) | References the policy table |
| driver_id | BIGINT | NOT NULL, FOREIGN KEY to driver(id) | References the driver table |
| vehicle_id | BIGINT | NOT NULL, FOREIGN KEY to vehicle(id) | References the vehicle table |
| description | TEXT |  | Additional details about the mapping |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | References the status table |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the mapping was created |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the mapping was last updated |

`map_policy_fee`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| policy_id | BIGINT | NOT NULL, FOREIGN KEY to policy(id) | References the associated policy. |
| fee_id | BIGINT | NOT NULL, FOREIGN KEY to fee(id) | References the associated fee. |
| description | TEXT |  | Additional notes about this mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update. |

`map_policy_named_driver`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| policy_id | BIGINT | NOT NULL, FOREIGN KEY to policy(id) | References the policy record. |
| driver_id | BIGINT | NOT NULL, FOREIGN KEY to driver(id) | References the named driver associated with the policy. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the mapping record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the mapping record was last updated. |

`map_policy_premium`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping. |
| policy_id | BIGINT | NOT NULL, FOREIGN KEY to policy(id) | References the associated policy. |
| premium_id | BIGINT | NOT NULL, FOREIGN KEY to premium(id) | References the associated premium. |
| description | TEXT |  | Additional notes about this mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | References the status of the record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp of record creation. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of last record update. |

`map_policy_transaction`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the map record. |
| policy_id | BIGINT | NOT NULL, FOREIGN KEY to policy(id) | References the associated policy. |
| transaction_id | BIGINT | NOT NULL, FOREIGN KEY to transaction(id) | References the associated transaction. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status of the map record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the map record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the map record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`map_producer_address`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| producer_id | BIGINT | NOT NULL, FOREIGN KEY to producer(id) | References the producer. |
| address_id | BIGINT | NOT NULL, FOREIGN KEY to address(id) | References the address. |
| description | TEXT |  | Additional details about the mapping. |
| status_id | BIGINT | FOREIGN KEY to status(id) | Status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`map_producer_contact_number`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| producer_id | BIGINT | NOT NULL, FOREIGN KEY to producer(id) | References the producer. |
| contact_number_id | BIGINT | NOT NULL, FOREIGN KEY to contact_number(id) | References the contact number. |
| description | TEXT |  | Additional details about the mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update. |

`map_producer_name`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT UNSIGNED | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| producer_id | BIGINT UNSIGNED | NOT NULL, FOREIGN KEY to producer(id) | References the producer record. |
| first_name_id | BIGINT UNSIGNED | NOT NULL, FOREIGN KEY to name(id) | References the first name of the producer. |
| middle_name_id | BIGINT UNSIGNED | FOREIGN KEY to name(id) | References the middle name of the producer. |
| last_name_id | BIGINT UNSIGNED | NOT NULL, FOREIGN KEY to name(id) | References the last name of the producer. |
| surname_name_id | BIGINT UNSIGNED | FOREIGN KEY to name(id) | References the surname of the producer. |
| description | TEXT |  | Additional details about the mapping. |
| status_id | BIGINT UNSIGNED | FOREIGN KEY to status(id) | References the status of the mapping. |
| created_by | BIGINT UNSIGNED | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT UNSIGNED | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`map_producer_policy`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| producer_id | BIGINT | NOT NULL, FOREIGN KEY to producer(id) | References the producer associated with the policy. |
| policy_id | BIGINT | NOT NULL, FOREIGN KEY to policy(id) | References the policy related to the producer. |
| description | TEXT |  | Additional details about the producer-policy relationship. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`map_transaction_payment`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the map record. |
| transaction_id | BIGINT | NOT NULL, FOREIGN KEY to transaction(id) | References the associated transaction. |
| payment_id | BIGINT | NOT NULL, FOREIGN KEY to payment(id) | References the associated payment. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status of the map record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the map record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the map record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`map_user_policy_driver`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| user_id | BIGINT | NOT NULL, FOREIGN KEY to user(id) | References the user associated with the policy and driver. |
| policy_id | BIGINT | NOT NULL, FOREIGN KEY to policy(id) | References the policy associated with the user and driver. |
| driver_id | BIGINT | NOT NULL, FOREIGN KEY to driver(id) | References the driver associated with the user and policy. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the mapping record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the mapping record was last updated. |

`map_vehicle_coverage_limit`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the vehicle-to-coverage-limit mapping record. |
| vehicle_id | BIGINT | NOT NULL, FOREIGN KEY to vehicle(id) | References the vehicle associated with the coverage limit. |
| coverage_id | BIGINT | NOT NULL, FOREIGN KEY to coverage(id) | References the coverage associated with the vehicle. |
| limit_id | BIGINT | NOT NULL, FOREIGN KEY to limit(id) | References the limit applied to the coverage. |
| description | TEXT |  | Additional details about the mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`map_vehicle_discount`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping. |
| vehicle_id | BIGINT | NOT NULL, FOREIGN KEY to vehicle(id) | References the vehicle. |
| discount_id | BIGINT | NOT NULL, FOREIGN KEY to discount(id) | References the discount. |
| description | TEXT |  | Additional details about the mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the mapping. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the mapping. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update. |

`map_vehicle_premium`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the mapping record. |
| vehicle_id | BIGINT | NOT NULL, FOREIGN KEY to vehicle(id) | References the associated vehicle. |
| premium_id | BIGINT | NOT NULL, FOREIGN KEY to premium(id) | References the associated premium. |
| description | TEXT |  | Additional notes about this mapping. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the mapping record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update. |

`name`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT UNSIGNED | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the name record. |
| name_type_id | BIGINT UNSIGNED | NOT NULL, FOREIGN KEY to name_type(id) | References the type of the name. |
| value | VARCHAR(255) | NOT NULL | The actual name value (e.g., 'John', 'Smith'). |
| description | TEXT |  | Additional details about the name. |
| status_id | BIGINT UNSIGNED | FOREIGN KEY to status(id) | References the status of the name record. |
| created_by | BIGINT UNSIGNED | NOT NULL, FOREIGN KEY to user(id) | User who created the name record. |
| updated_by | BIGINT UNSIGNED | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update. |

`policy`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the policy record. |
| policy_prefix_id | BIGINT | NULLABLE, FOREIGN KEY to policy_prefix(id) | References the policy prefix. |
| number | VARCHAR(50) | NULLABLE, UNIQUE(policy_prefix_id, number) | Unique policy identifier within a prefix scope. |
| policy_type_id | BIGINT | NOT NULL, FOREIGN KEY to policy_type(id) | References the type of policy. |
| effective_date | DATE | NULLABLE | The start date of the policy. |
| inception_date | DATE | NULLABLE | The inception date of the policy (first effective date). |
| expiration_date | DATE | NOT NULL | The end date of the policy. |
| renewal_date | DATE | NULLABLE | Date when the policy is set for renewal. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | References the status of the policy. |
| term_id | BIGINT | NOT NULL, FOREIGN KEY to term(id) | References the term of the policy. |
| description | TEXT | NULLABLE | Additional details about the policy. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`policy_prefix`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for each policy prefix. |
| name | VARCHAR(50) | NOT NULL, UNIQUE | Unique policy prefix (e.g., 'PLCY-', 'AUTO-'). |
| description | TEXT |  | Additional details about the policy prefix. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | References the status of the policy prefix. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`premium`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the premium record. |
| premium_type_id | BIGINT | NOT NULL, FOREIGN KEY to premium_type(id) | Type of premium associated with the record. |
| name | VARCHAR(255) | NOT NULL, UNIQUE | Name of the premium record. |
| description | TEXT |  | Additional details about the premium. |
| credit | DECIMAL(12,2) | CHECK (credit >= 0) | Credit value associated with the premium. |
| debit | DECIMAL(12,2) | CHECK (debit >= 0) | Debit value associated with the premium. |
| effective_date | DATE |  | Date when the premium becomes effective. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the premium record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the premium record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the premium record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the premium record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the premium record was last updated. |

`producer`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for each producer/agency. |
| producer_code_id | BIGINT | NOT NULL, FOREIGN KEY to producer_code(id), UNIQUE | Unique producer code for identification. |
| number | VARCHAR(50) | NOT NULL, UNIQUE | Unique number assigned to the producer. |
| name | VARCHAR(255) | NOT NULL, UNIQUE | Name of the producer/agency. |
| description | TEXT |  | Additional details about the producer. |
| status_id | BIGINT | FOREIGN KEY to status(id) | References the status of the producer. |
| producer_type_id | BIGINT | NOT NULL, FOREIGN KEY to producer_type(id) | References the type of producer associated with the program. |
| signature_required | BOOLEAN | NOT NULL, DEFAULT FALSE | Specifies if a signature is required for the producer. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the producer record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the producer record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`state`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for each state. |
| name | VARCHAR(255) | NOT NULL | The full name of the state (e.g., Texas, California). |
| code | VARCHAR(10) | NOT NULL | Abbreviated state code (e.g., TX, CA). |
| description | TEXT |  | Additional details or notes about the state. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the state record. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |
| Unique Constraint |  | UNIQUE (name, code) | Ensures no duplicate entries for state name and code. |

`status`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the status record. |
| name | VARCHAR(100) | NOT NULL, UNIQUE | Name of the status (e.g., 'Active', 'Pending'). |
| description | TEXT |  | Additional details about the status. |
| created_by | BIGINT | FOREIGN KEY to user(id) | User who created the record. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the record. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the record was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp when the record was last updated. |

`transaction`

| **Column Name** | **Data Type** | **Constraints** | **Description** |
| --- | --- | --- | --- |
| id | BIGINT | AUTO_INCREMENT, PRIMARY KEY | Unique identifier for the transaction. |
| transaction_type_id | BIGINT | NOT NULL, FOREIGN KEY to transaction_type(id) | References the type of transaction. |
| name | VARCHAR(255) | NOT NULL | Name of the transaction. |
| description | TEXT |  | Additional details about the transaction, including notes like "all current and future transactions (like installments)." |
| credit | DECIMAL(12,2) | CHECK (credit >= 0) | Total credit amount for the transaction. |
| debit | DECIMAL(12,2) | CHECK (debit >= 0) | Total debit amount for the transaction. |
| confirmation_code | VARCHAR(255) |  | Confirmation code associated with the transaction. |
| due_date | DATE |  | The date when the transaction is due. |
| processed_date | TIMESTAMP |  | The date when the transaction was processed. |
| status_id | BIGINT | NOT NULL, FOREIGN KEY to status(id) | Status of the transaction. |
| created_by | BIGINT | NOT NULL, FOREIGN KEY to user(id) | User who created the transaction. |
| updated_by | BIGINT | FOREIGN KEY to user(id) | User who last updated the transaction. |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | Timestamp when the transaction was created. |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | Timestamp of the last update to the transaction. |