# IP269 - Quotes: New Quote - Step 3: Vehicle Information

## **A) WHY – Vision and Purpose**

The goal of this step is to ensure that all vehicles to be covered under the policy are correctly added, verified, and associated with the policyholder. This ensures:

- Proper premium calculation
- Accurate risk assessment
- Full coverage details for underwriting and legal purposes

It allows for both **lookup-based vehicle matching** from third-party data sources and **manual entry** when lookup is insufficient or data is unavailable.

---

## **B) WHAT – Core Requirements**

### 1. **Vehicle Lookup**

- Vehicle look-up occurs against the primary insured’s address, for any vehicles associated with their address as provided
- This pulls available vehicle data from third party-sources, and displays returned results in a list format for review before confirmation
- Vehicles are then presented in a list format, where the user can add or remove vehicles from the policy

### 2. **Manual Entry Flow**

- User may add a vehicle not returned in the search results by manually entering vehicle information

---

## **C) HOW – Planning & Implementation**

---

## **D) User Experience (UX) & Flows**

**Adding Returned Vehicle**

1. For vehicles returned in the vehicle look-up, they can be added by clicking the “Add” CTA.
2. This will open the side panel, where the user will include the usage type, before adding the vehicle.
3. The vehicle will then show in the “All Vehicles in the Household” section of the vehicle list.

**Add Vehicle Manually - By VIN**

1. For vehicles not returned in the vehicle look-up, they can be added manually by clicking the “Manually Add” CTA.
2. If the user has the VIN available, they can populate the VIN, Usage Type, and Garaging ZIP Code, before initiating search.
3. If the vehicle information is returned, the vehicle will be added to the policy and listed in the “All Vehicles in the Household” section of the vehicle list.

**Add Vehicle Manually - By Year/Make/Model**

1. For vehicles not returned in the vehicle look-up, they can be added manually by clicking the “Manually Add” CTA.
2. If the user does not have the VIN available, they can look the vehicle up by Year, Make, and Model. They must also provide the usage type, and the garaging address for the vehicle, before intiating search.
3. If the vehicle information matches an existing vehicle, they will select the vehicle from the lsit of options, and provide the license plate number and license plate state.
4. Upon completion, the vehicle will be added to the policy and listed in the “All Vehicles in the Household” section of the vehicle list.

**Add Vehicle Owners to Policy**

1. If the user attempts to add a vehicle whose registered owner is not on the policy, the user must add the registered owner to the policy before the vehicle can be added.
2. To complete this workflow, the user must confirm if the owner will be included or excluded on the policy.
    1. If excluded, populate the gender, marital status, and relationship to the insured.
    2. If included, populate the gender, marital status, employment information, SR-22, and violations, if applicable.

## **E) Master Schema Tables**