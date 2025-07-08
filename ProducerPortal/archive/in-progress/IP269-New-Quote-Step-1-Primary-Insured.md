# IP269 - Quotes: New Quote - Step 1: Primary Insured

## **A) WHY – Vision and Purpose**

The purpose of this screen is to **initiate a new quote** by collecting and verifying the **primary insured’s identity**. It ensures a streamlined intake flow that is user-friendly, minimizes manual input, and leverages existing profile or search data to speed up the quoting process. The design supports **desktop and mobile form factors**, optimizing efficiency for agents in various working conditions.

---

## **B) WHAT – Core Requirements**

### **1. Effective Date & Program Selection**

- The agent must first select the effective date for this policy, which will determine which programs are available in the program drop-down
- An error will be surfaced if the effective date is 31 or more days in the future
- Block progression if rule is triggered

### **2. Search & Match Functionality**

- Ability to search existing records for a potential match using:
    - License Type
        - If ‘US License’ selected, Country field will become disabled and preset to the United States, and the following fields will be displayed:
            - Driver’s License Number
            - State Licensed
        - If an option other than ‘US License’ is selected, the following fields will be displayed:
            - Suffix (Optional)
            - First Name (Required)
            - Middle Name (Optional)
            - Last Name (Required)
            - Street Address (Required)
            - City (Required)
            - State (Required)
            - Country (Required)
            - ZIP Code (Required)
- Display matching results in a modal list with selection option

### **3. Search Results**

- Profile card containing:
    - Full name
    - Date of Birth
    - Address

### **4. Action Options**

- Yes - Information Correct: Selects matched record to prefill quote data
- No - Not a Match: Allows manual entry of insured details without a match
- No - Address Incorrect: Selects matched record to prefill quote data, but allows user to modify the address associated to the quote
- **Continue** button to proceed to the next step

---

## **C) HOW – Planning & Implementation**

---

## **D) User Experience (UX) & Flows**

### **1. Start a New Quote Flow**

1. User selects “Start New Quote”
2. Step 1: Form opens for **Primary Insured**
3. Enter the effective date and program
    1. The effective date must be within 30 days of the present date when the quote is started
4. Enter the insured’s information
    1. If US License holder, pre-populate Country field with United States and disable, and present Driver’s License Number and State Licensed field
    2. If any other License Type is selected, present Suffix, First Name, Middle Name, Last Name, Street Address, City, State, Country and ZIP Code fields
5. System auto-searches for matches
6. Match dialog appears
    - If match found → user selects and proceeds
    - If no match → enters details manually

### **2. Handling Match Found**

1. Yes - Information Correct: Selects matched record to prefill quote data
2. No - Not a Match: Allows manual entry of insured details without a match
3. No - Address Incorrect: Selects matched record to prefill quote data, but allows user to modify the address associated to the quote
4. **Continue** button to proceed to the next step

### **4. Mobile Flow**

- Stacked, scrollable layout
- Match results shown in full-screen modal
- All actions accessible via bottom CTA button

## E) Master Schema Tables