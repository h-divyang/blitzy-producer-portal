# IP269 - Quotes: New Quote - Step 4: Underwriting Questions

## **A) WHY – Vision and Purpose**

The Underwriting Questions (UWQs) step serves as a **risk assessment checkpoint**. Its purpose is to:

- Identify eligibility for the quote based on carrier-specific guidelines.
- Flag high-risk profiles to be reviewed or blocked from quote continuation.
- Gather structured data for downstream policy binding and rating.

By enforcing UWQ completion, the system ensures only qualified prospects proceed, reducing underwriting friction and quote-to-bind errors.

---

## **B) WHAT – Core Requirements**

### **1. Questions Rendering**

- Dynamically display underwriting questions based on program
- Each question supports:
    - **Yes / No** radio options

### **2. Validation and Error Handling**

- Highlight any **unanswered required questions** in red with error message.
- If a **“disqualifying” answer** is selected (e.g., “Yes” to a high-risk factor), display:
    - Warning message
    - Optional override workflow
    - Disable “Continue” button until resolved

### **3. Eligibility Flags**

- Identify answers that result in:
    - **Hard stops** (carrier declines the risk)
    - **Warnings** (agent alert but can proceed)
- Trigger UI banners with messages:
    - e.g., “This answer may make the risk ineligible for some carriers.”

### **4. Data Handling**

- All responses should be saved per quote and persist on reload
- Submit to backend as part of quote payload
- Allow editing even after saving, before submission

### **5. Mobile Compatibility**

- Responsive layout with vertical stacking
- Sticky “Continue” button on mobile
- Scroll-to-error behavior on validation failure

---

## **C) HOW – Planning & Implementation**

---

## **D) User Experience (UX) & Flows**

### **1. Entry Flow**

- User navigates to “Underwriting Questions” step after Coverages
- Sees list of UWQs with radio buttons for each

### **2. Interaction Flow**

- User answers each question
- If disqualifying response is selected:
    - UI shows yellow warning banner at top
    - Specific question may be highlighted with explanation
    - "Continue" is still available if not a hard stop

### **3. Error Flow**

- If required question is left unanswered:
    - Red error message appears under it
    - “Continue” is disabled until all required answers are present

### **4. Submission Flow**

- On valid answers, user clicks “Continue”
- Backend validates eligibility
    - If accepted → proceeds to next step
    - If declined → displays error message with reason

### **5. Edge Case Handling**

- Questions may vary in number and content by state/carrier
- Disqualifying questions may:
    - Trigger automatic program exclusion
    - Require agent acknowledgement

## **E) Master Schema Tables**