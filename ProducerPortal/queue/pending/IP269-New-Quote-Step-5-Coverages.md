# IP269 - Quotes: New Quote - Step 5: Coverage Selection

## **A) WHY – Vision and Purpose**

The Coverage step is designed to allow users (agents or producers) to select insurance coverage options tailored to the customer's needs and risk profile. It ensures:

- Accurate premium generation,
- Compliance with underwriting rules and state mandates,
- Flexibility in policy customization,
- Clear comparisons across carrier programs.

This step is pivotal in balancing coverage adequacy and affordability, enabling competitive quoting.

---

## **B) WHAT – Core Requirements**

### **1. Coverage Overview**

- The policy will include policy-wide coverages, and coverages per vehicle

### **2. Coverage Selection**

Each coverage type displays:

- Dropdown for selecting limit or deductible, per coverage type (Bodily Injury, Property Damage, Medical Payments, PIP, etc.)
- Checkbox for selecting coverage, if applicable
- Additional equipment coverage, which will require type and value, if selected

### **3. Validation Rules**

- Deductibles must meet state or underwriting minimums
- Block progress until all required coverages are selected

### **5. Premium Calculation**

- Premium recalculates automatically on selection changes
- Total Premium indicates premium plus fees
- The producer will select if the user will pay in installments, set the percentage down, as well as when their first installment would be paid to inform the total premium calculation

### **6. Save & Navigation**

- **"Continue"** button will take the user to review the quote
- Option to **save quote** at any time

---

## **C) HOW – Planning & Implementation**

---

## **D) User Experience (UX) & Flows**

### **1. Select Coverage Flow**

1. Click on a coverage row (e.g., “Bodily Injury”)
2. Select dropdown for limits
3. User selects limit
4. Premium recalculates

### **2. Mobile Considerations**

- Carrier rows stack vertically
- Expandable accordion for coverage items
- Sticky Continue CTA on scroll

## **E) Master Schema Tables**