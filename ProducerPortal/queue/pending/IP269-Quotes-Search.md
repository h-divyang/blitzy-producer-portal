# IP269 - Quotes: Quotes Search

## **A) WHY - Vision and Purpose**

The **Quotes Search & Flyout** experience empowers users to efficiently manage, review, and act on insurance quotes. It provides a **centralized location to view quote activity**, quickly access **detailed quote breakdowns**, and take action through **intuitive mobile and desktop views**. This enables faster decision-making, clearer communication, and streamlined operations across devices.

---

## **B) WHAT - Core Requirements**

### **1. Quotes Dashboard (Desktop & Mobile)**

- **List of quotes** with key details in table view (desktop) and expandable list (mobile):
    - Quote Status, with color indicators
    - Date & Time Submitted
    - Effective Date
    - Insured Name
    - Agent Number
    - Vehicles
    - Drivers
    - Requoted From
        - This will include a **policy number** and link to previous policy, if applicable
- **Search & Filter Capabilities**
    - Filter by date submitted, offices, or status
    - Sort by selecting the column headers
    - Text search by name or quote number
- Start **New Quote** button

### **2. Quote Flyout Panel (Expanded View)**

Upon selection of a quote, the side panel flyout will present, showing a summary view of the quote.

- **Header Section**:
    - Insured name
    - Quote status
    - Requoted From, if applicable
    - Submitted On
    - Effective date
- **See Full Details:**
    - This button will take the user into the quoting experience where they left off for quotes in progress
- **Insured Details**:
    - Insured Name
    - Primary Phone Number
    - Alternate Phone Number
    - Email
    - Notification Preference
    - Address
- **Suspenses:**
    - This will present any suspenses that are currently open on this quote that require action by the producer to resolve
- **Vehicles:**
    - Year
    - Make
    - Model
    - VIN
    - Plate
- **Drivers:**
    - Name
    - Primary Driver Tag
    - Included vs Excluded Status
    - Date of Birth
    - State of License Issue
    - Identification Prefix
    - Identification Number
- **Documents Section**:
    - This includes documents attached to the quote (e.g. Proof of Prior Insurance, Driver’s License Image, Vehicle Photos, etc.)
- **Coverage Breakdown**
    - Individual policy coverages (BI, PD, MedPay, Comp, Collision, etc.) with limits and deductibles
- **Rate Information**
    - Prior Policy - if they held insurance previously, this will indicate ‘Yes’
    - Rate Chart - the rate chart used for their quote
    - Company - the insurance carrier associated with the program they were quoted on

---

## **C) HOW - Planning & Implementation**

Front End Mapping of Fields

---

## **D) User Experience (UX) & Flows**

### **1. Viewing Quotes (Desktop)**

1. User lands on the **Quotes Home Dashboard**
2. Scrolls or searches for a specific quote
3. Clicks a row to open the **Quote Flyout Panel** on the right
4. To open the quote, the user can click “Open Quote” to enter the Quoting workflow where they left off

### **2. Viewing Quotes (Mobile)**

1. User opens mobile menu > Quotes
2. Taps on a quote row from the list
3. Flyout expands full screen with quote details
4. To open the quote, the user can click “Open Quote” to enter the Quoting workflow where they left off

### **3. Starting a New Quote**

1. Clicks the **“+ Start New Quote”** button
2. Redirected to quote intake form

## **E) Master Schema Tables**