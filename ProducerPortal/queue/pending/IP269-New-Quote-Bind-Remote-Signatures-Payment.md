# IP269 - Quotes: Bind - Step 3 & 4: Sign Remotely & Make Payment

## **A) WHY - Vision and Purpose**

This experience facilitates a **fully digital and seamless close-out** of the quoting process by enabling insureds to:

- **Electronically sign documents** required to activate their policy.
- **Make a payment** to bind the policy.
- **Complete their purchase remotely**, without agent or in-person steps.

This experience reduces friction, increases bind rates, and supports a modern, self-serve model for insurance.

---

## **B) WHAT - Core Requirements**

### 1. **Email or SMS Notification**

- Triggered when quote is finalized and ready for insured action.
- Includes:
    - CTA link to sign documents
    - Branded styling (for email)

### 2. **Sign Documents Flow**

- Web-based signing portal accessible from any device
- Displays:
    - Greeting and insurer name (e.g., “Finalize Your Auto Insurance”)
- User must enter their date of birth (YYYY/MM/DD) to proceed

### a. Signature Capture

- Prefill full name and initials, and generate signature
- Require user to adopt generated signature to proceed

### b. Document Review

- PDF viewer embedded with:
    - Sign here tags
    - Zoom and scroll
- Signing each document required to proceed
- If the user attempts to proceed without completing signatures, an error will be shown
- Upon completion, the user wil proceed to payment

### 3. **Payment Collection**

- Option to pay with:
    - Credit Card
    - E-Check
- Displays:
    - Total due today
    - Recurring payment details (if applicable)
    - Total premium + fees

### 4. **Status Feedback**

- If any information was skipped during the binding process (e.g. the insured must provide photos or documents), the user will be notified that this will require follow up to ensure the policy remains in force

### 5. **Mobile Compatibility**

- Responsive layout

---

## **C) HOW - Planning & Implementation**

---

## **D) User Experience (UX) & Flows**

### 1. **Flow: Email/SMS to Sign**

1. Insured receives email/text message
2. Clicks CTA → redirected to signing portal, enters Date of Birth
3. Creates/selects signature → confirms
4. Reviews and signs each document
5. Hits **Complete Signing**

### 2. **Flow: Payment**

1. After signing, user lands on payment page
2. Reviews quote/payment breakdown
3. Enters payment method
    1. For E-Check, must provide account number, routing number
    2. For Credit Card, must provide cardholder name, card number, CVV, expiration date, billing address
        1. The Insurance Company will have the option to enable a convenience fee for credit card payments; if this has been enabled, an additional alert will be shown when selecting the credit card as a payment option
            1. Within this alert, the dollar value is configurable
            2. If selected, the amount paid will update to include the fee value
4. Clicks **Submit Payment**
5. Confirmation screen with bind success

### 3. **Flow: Producer Experience**

1. After email or SMS is sent, the producer will see when it was sent and to what contact information
    1. They also have the ability to resend the link if needed
2. They will also see all documents and their signature status, as well as the suspenses that will be created due to missing information in the bind process
3. They will also see status of payment
4. Once all steps have been complete, the quote is closed and a policy is opened

### 4. **Flow: Errors & Remediation**

- If user attempts to proceed without completing required steps, an error will be shown

### 5. **Mobile**

- Mobile UX prioritizes:
    - Easy document scrolling
    - Signature drawing with finger
    - Large tap targets

---

## E) Master Schema Tables

---