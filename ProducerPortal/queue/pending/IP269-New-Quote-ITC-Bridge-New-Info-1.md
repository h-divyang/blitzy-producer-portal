# IP269 - Quotes: Bridged In - New Info Found Option 1

## **A) WHY – Vision and Purpose**

This version of the **Review Quote** screen is triggered when **external data enrichment (bridge)** identifies **new drivers** related to the quote that weren’t initially declared.

The purpose is to:

- Transparently notify the agent/user about auto-discovered information.
- Enable validation and editing of this data before proceeding.
- Preserve compliance and underwriting rules by ensuring all household members and vehicles are properly disclosed.

---

## **B) WHAT – Core Requirements**

### **1. Display Bridged-In Alerts**

- Sections for **Drivers** must support inline banner states
- All newly identified drivers will be automatically set to excluded drivers on the policy

### **2. Edit & Validation Flow**

- If the producer needs to modify this and set any of these drivers to included, the side panel will open for updating the driver, allowing them to set the driver to included
    - If set to included, the additional information as required earlier in the flow must be added and the premium calculation will be updated to reflect the additional driver

### **3. Premium Update Impact**

- Bridged-in items may affect premium:
    - Drivers rated vs excluded
    - Vehicles added may increase policy premium
- Updated premiums must be reflected in the summary.

---

## **C) HOW – Planning & Implementation**

---

## **D) User Experience (UX) & Flows**

### **1. Entry Flow**

- User lands on Review Quote after bridging data
- Drivers  section displays highlighted alerts

### **2. User Interactions**

- Can click **Edit Drivers** on the section, which will return the user to Step 2
- After update, user returns to this screen with changes reflected

## **E) Master Schema Tables**