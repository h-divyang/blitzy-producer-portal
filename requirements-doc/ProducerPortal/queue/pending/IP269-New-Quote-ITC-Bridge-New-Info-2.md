# IP269 - Quotes: Bridged In - New Info Found Option 2

## **A) WHY – Vision and Purpose**

This screen supports a critical step in the quoting journey: informing the user that **third-party data (bridge)** has found **undisclosed drivers**  tied to the household or policy.

The purpose is to:

- Ensure accuracy and completeness of underwriting data.
- Prevent quote submission with missing or invalid data.
- Encourage user intervention to verify or exclude discovered entities.

Option 2 emphasizes visibility with **bolder alert styling**, indicating **urgency or validation requirements** before quote finalization. This version requires the producer to address the newly identified drivers before proceeding to bind.

---

## **B) WHAT – Core Requirements**

### **1. Bridged-In Alert Card**

- A card in the **Drivers section** that:
    - Describes the nature of the discrepancy.
    - Uses a **warning-style** background (yellow/orange) with border.
    - Contains a clear “Exclude All” CTA
    - Includes explanation like:

      > “We found new driver information associated to your household. Please review and choose whether to include or exclude them from the policy.”
>

### **2. Badge Status Indicators**

- Each newly discovered person/vehicle includes a **status tag** indicating Missing Info

### **3. Interaction Lock**

- The system **blocks advancement** (`Continue` button) until:
    - All bridged-in entities are reviewed and given a status.
    - This includes either confirming or excluding them explicitly.
    - This can be done by selecting the driver, opening the right side panel to manage inclusion or exclusion, as well as the additional information required for the driver

### **4. Premium Update**

- Any action (e.g., including a new driver) must:
    - Recalculate and display updated premium values.
    - Update the “Premium Summary” section dynamically.

---

## **C) HOW – Planning & Implementation**

---

## **D) User Experience (UX) & Flows**

### **1. On Arrival**

- User lands on the Review Quote screen.
- Drivers section contains:
    - Yellow alert card warning them of found data.
    - A list of drivers, with badges indicating status.

### **2. Interaction Flow**

- User clicks on a driver with the relevant “Missing Info’’ status
- The driver side panel will open, allowing the user to populate the gender, marital status, relationship to insured, and if the driver is included or excluded from the policy
- The user can also bulk exclude all identified drivers using the “Exclude All” CTA
- Upon completion:
    - Tag is updated, indicating the driver is included or excluded
    - Banner disappears if all bridged entities are resolved

### **3. Premium Impact**

- Once a bridged-in driver is confirmed:
    - Premium adjusts in real-time
    - All impacted coverage or fees update dynamically

## **E) Master Schema Tables**