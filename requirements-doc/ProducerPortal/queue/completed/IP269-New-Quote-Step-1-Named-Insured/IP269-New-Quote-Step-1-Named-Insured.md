# IP269 - Quotes: New Quote - Step 1: Named Insured Information

## **A) WHY – Vision and Purpose**

This section is designed to review **search results for the chosen primary insured** and supplement their profile with any **additional verified or missing customer information**. The purpose is to ensure that all relevant personal data is:

- Confirmed as accurate
- Correctly tied to the selected policy
- Enriched using third-party data sources when needed

This improves data integrity for quoting, binding, and compliance, and supports the insurance agent's ability to confidently move forward with the policy.

---

## **B) WHAT – Core Requirements**

### 1. **Primary Insured Search Results**

- Display returned data for the given individual, or allow the user to key all information manually if no result is returned
- In the case where an individual is returned, the following fields will be presented as read-only: Suffix, First Name, Middle Name, Last Name, Date of Birth, License Number, State Issuer for License, and Country

### 2. **Customer Info Panels**

- Additional sections to capture:
    - Address
    - Gender
    - Marital Status
    - Primary Phone Number
    - Alternate Phone Number
    - Email Address
    - Notification Preference
    - Prior Insurance Status
        - Has the primary insured had prior insurance? If yes, show:
            - Prior Insurance Company
            - Prior Insurance Expiration Date
            - Number of Months Insured with Prior Insurance
    - Eligible Discounts
    - Housing

### 4. **Modals & Overlays**

- **“Are you sure?” confirmation modal** when switching search results or redoing a search.
- Error/warning messaging when required fields are incomplete or data mismatches occur.
    - For example, if the user selects ‘Enroll in Paperless’ discount without a valid email address. The email address fields will be outlined in red and must be populated before proceeding.

### 5. **Navigation**

- On mobile, form fields become stacked sections.

---

## **C) HOW – Planning & Implementation**

---

## **D) User Experience (UX) & Flows**

**Record Found**

1. Agent will review the pre-populated address returned with the record and confirm it is correct before continuing. If incorrect, the agent will modify the information.
2. The agent will populate the additional details section, including their gender, marital status, primary and alternate phone numebrs, email address, notification preferences, as well as prior insurance information and eligible discounts.

**No Record Found**

1. If no record is found, the agent must populate all information on the user, including their name, and date of birth, along with all fields defined in the record found workflow.

## **E) Master Schema Tables**