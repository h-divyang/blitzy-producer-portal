# IP269 - Quotes: New Quote - Step 2: Drivers

## **A) WHY – Vision and Purpose**

This step ensures that all relevant driver information is captured and associated with the quote. Proper driver data enables accurate risk assessment, pricing, and policy eligibility. The goal is to create a seamless interface for agents to:

- Add and manage drivers,
- Handle violations and license status,
- Ensure underwriting rules and validations are enforced.

---

## **B) WHAT – Core Requirements**

The purpose of this step is to expedite including, excluding, or removing drivers from the policy. All drivers returned are associated with the address identified in Step 1, indicating there is a relationship to the primary insured.

### **1. Driver List Management**

- Display all drivers with:
    - Name
    - Primary driver tag, if applicable
    - Date of Birth
    - State of Issue License
    - Identification Type Prefix
    - Identification Number
    - Included or Excluded Status
- Add new driver (modal flow)

### **2. Add a Driver Form**

- Fields:
    - Suffix (Optional)
    - First Name
    - Middle Name (Optional)
    - Last Name
    - Date of Birth
    - Gender
    - Marital Status
    - Relationship to Insured
    - Include or Exclude in Policy
    - Driver’s License Type
    - Country Licensed
    - Driver License Number
    - State Licensed
    - Employement Status
    - Occupation/Source of Income
    - Employer Name
    - SR-22 Required Status
    - Reason for SR-22
    - Violation Type
    - Violation Date

### **5. Business Rules & Validation**

- If an included driver has their Marital Status set to “Married”, there must be another driver on the policy with Marital Status set to Married, in either an included or excluded driver status
- If an included driver is deemed to be criminally ineligible to be included in a policy, they must be removed or excluded before proceeding

### **6. Save & Navigation**

- Continue button enabled only when:
    - All required fields are completed
- Progress automatically saved on each field update

---

## **C) HOW – Planning & Implementation**

### **Mobile Optimization**

- Responsive layout for Driver list and modal forms
- Sticky “Continue” CTA
- Mobile-first tabbing order

---

## **D) User Experience (UX) & Flows**

### **1. Add Driver Manually Flow**

1. Click “Add Driver”
2. Modal opens – user enters personal info
3. Continue to license section
4. Add violations if any
5. Save and return to driver list

### **2. Edit Driver Flow**

1. Click on a driver row
2. Modal opens, requiring additional information:
    1. Gather gender, marital status, and relationship to insured, and if the driver will be included or excluded on the policy
    2. If removing from the policy, status of driver will be set to removed, and the reason for removal must be included
3. Make changes and save

### **3. Driver List Presentation**

- Drivers will be presented in three sections: Included, Excluded, or Removed
    - If the system has been configured without the ability to remove drivers, this section will be removed and the CTA will shift up to eliminate unexpected white space
- The user can search the list using the search bar to expedite finding the relevant drivers
- The list of drivers will be paginated, and the user can click to show additional drivers
- The household search will run again when the user adds drivers manually, which may result in additional new drivers being returned
    - These drivers will be returned with a “New” tag to differentiate from other drivers in the list

## **E) Master Schema Tables**