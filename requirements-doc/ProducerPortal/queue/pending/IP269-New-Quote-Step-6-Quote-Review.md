# IP269 - Quotes: New Quote - Step 6: Review Quote

## **A) WHY – Vision and Purpose**

The **Review Quote** step consolidates all prior quote information into a single, editable summary. It gives the user or agent confidence before submission by:

- Allowing final review and minor corrections.
- Ensuring accuracy across personal details, vehicle, drivers, coverage, and premiums.
- Providing transparency into how the final premium was calculated.
  This step minimizes errors, improves trust, and helps ensure binding success.

---

## **B) WHAT – Core Requirements**

### **1. Summary Sections**

All major data blocks from the quote process must be displayed clearly:

- **Primary Insured Info**
- **Drivers** (with tags for included, or excluded)
- **Vehicles** (Year, Make, Model, VIN)
- **Coverage Details** (Policy-wide and per vehicle)
- **Applied Discounts** (e.g., Multi-Car, Homeowner)
- **Premium Summary** (including breakdown of payments)

### **2. Edit Functionality**

Each major section must have inline “Edit” links:

- Clicking navigates the user to that specific step of the quote process (e.g., clicking “Edit Coverages” goes to Step 5).
- Must persist state and not discard any prior entered data.

### **3. Premium Summary Calculation**

Display:

- **Total Premium (Full Term)**
- **Total Fees**
- **Total Due Today (Down Payment)**
- **Monthly Payment Amount** (if payment type is installment)

### **4. Responsive Views**

- Mobile stacked layout with collapsible sections
- “Continue” button must be sticky at the bottom on mobile

---

## **C) HOW – Planning & Implementation**

---

## **D) User Experience (UX) & Flows**

### **1. Entry Flow**

- Arrives from “Coverage Details” step
- Views all quote details in read-only mode
- Sees green checkmarks for completed steps in sidebar

### **2. Interaction Flow**

- User reviews each section
- Optionally clicks “Edit” to revise details
- Returns to this page with updated data

### **3. Premium Summary Clarity**

- Pricing appears at the bottom with clear separation:
    - **Breakdown** for Total, Fees, Discounts
    - **Final Total** prominently shown in bold
- Discount tags appear inline in the summary

### **4. Mobile Design**

- Each section collapsible for easier navigation
- Summary appears at the bottom with “Continue” sticky on scroll

### **5. Exit Flow**

- Click “Continue”
    - Validates quote completeness
    - Redirects to  **Bind** step

## **E) Master Schema Tables**