# IP269 - Quotes: Bind - Step 3 & 4: Sign Documents In Person & Make Payment

## **A) WHY – Vision and Purpose**

This experience supports the **final steps of insurance policy activation**, streamlining the document signing and payment process in a single cohesive flow. It aims to:

- Enable seamless signing of all required legal documents.
- Provide flexibility in choosing a **payment method** (Producer E-Check, Insured E-Check, or Credit Card).
- Ensure compliant submission and real-time confirmation that the policy is bound and ready to activate.

The goal is to reduce friction at the moment of purchase, avoid drop-offs, and support both desktop and mobile signers in various contexts.

---

## **B) WHAT – Core Requirements**

If the user attempts to step backwards into the quoting flow, they will be presented a confirmation dialog indicating the policy will need to be re-rated, and any progress made towards binding the policy (uploading images, documents, signatures, etc.) will be purged and must be restarted.

The user will proceed into the in-person signing workflow after signature adoption in the previous step in the workflow.

### **1. Document Presentation & Signing**

- Display a list of all required documents (on both desktop and mobile).
- Support embedded document viewer with e-sign capability.
- Show signature prompt for each document.
- Track individual document statuses (e.g., signed, pending).
- Provide “Finish & Submit” button after all documents are signed.

### **2. Document Signing Confirmation**

- Display success message: **“You're all set!”** with confirmation checkmark.
- Route to payment and submission review step immediately after.

### **3. Payment Options**

- Present payment related information, including total premium pus fees, the minimum amount due today, the amount paid, the producer fee, who the fee was received by, and their remaining payments, both how many and how much each payment will be.
- Support three methods:
    - **Producer E-Check**
    - Insured E-Check
        - Required Fields: routing number, account number
    - **Credit Card**
        - Required fields: Cardholder Name, Card Number, Expiry, CVV, Billing Address

### **4. Review & Submit Section**

- Show full document list and allow review/download before submitting.
- Reconfirm total premium amount and any discounts.
- Upon submission for binding, the document package will be emailed to the user and the policy is now inforce

---

## **C) HOW – Planning & Implementation**

---

## **D) User Experience (UX) & Flows**

### **Flow 1: Full Signing + Payment (Desktop)**

1. User sees a document viewer with signature indicators where signatures or initialing are required in the document.
2. After all documents are signed:
    - Confirmation screen appears
    - Route to Review & Submit
3. User selects a **payment method**
    1. The user can select from Producer E-Check, Insured E-Check, or Credit Card
    2. The Insurance Company will have the option to enable a convenience fee for credit card payments; if this has been enabled, an additional alert will be shown when selecting the credit card as a payment option
        1. Within this alert, the dollar value is configurable
        2. If selected, the amount paid will update to include the fee value
4. User clicks **“Submit & Bind Policy”**

### Flow 2: User Addresses Suspenses

1. The user sees suspenses have been created due to missing documentation or photos earlier in the workflow.
2. The user can navigate using the “Upload Docs” CTA to return to the document upload step.
3. The user can then progress forward to the final binding stage, with all information previously entered retained.

---

## E) Master Schema Tables

---