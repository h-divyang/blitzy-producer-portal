# IP269 - Quotes: Bind - Step 2: Document Upload

## **A) WHY – Vision and Purpose**

The purpose of the document upload experience is to:

- Enable users to **submit required documentation digitally** to support underwriting review (e.g., driver's license, proof of prior insurance).
- Reduce delays and friction in quote processing by streamlining the document intake process.
- Provide clear **visual confirmation** and status indicators.

---

## **B) WHAT – Core Requirements**

If the user attempts to step backwards into the quoting flow, they will be presented a confirmation dialog indicating the policy will need to be re-rated, and any progress made towards binding the policy (uploading images, documents, signatures, etc.) will be purged and must be restarted.

### **1. Upload Requirements Section**

- Display the list of required documents:
    - Proof of Prior Insurance
    - Driver’s License
    - Vehicle Registration
    - Additional Docs (if requested)
    - The list of documents will depend on the parameters of the policy written and will change dynamically based on policy changes
- Each section should contain:
    - Upload Button (with drag-and-drop support)
    - Document thumbnail after successful upload
    - Optional: "Remove" action for uploaded files using the ‘X’ CTA

### **2. Upload Methods**

- Drag & Drop functionality
- Click-to-browse and select file(s)
- File formats supported:
    - PDF, JPEG, PNG
    - Size limits (e.g., max 10MB/file)

### **3. Error Handling**

- Display error message on:
    - Invalid file format
    - File size exceeding limit

### **4. Upload Status Indicators**

- Display upload status dynamically with loading indicator

### **5. Suspense Creation**

- If the user continues to the next step without all required documents uploaded, present a warning indicating that a suspense will be created requiring this to be addressed later if not resolved now

---

## **C) HOW – Planning & Implementation**

---

## **D) User Experience (UX) & Flows**

### **Flow 1: Normal Upload Experience**

1. User lands on “Step 2: Upload Documents”
2. All required documents shown in cards with upload prompts
3. User uploads each file via click or drag/drop
4. Upon successful upload, the file card updates with file image preview
5. The user can then select the “Continue to Coverage” CTA
6. If the user does not provide all requested documents, an alert will indicate that a suspense has been created and the documents must be provided at a later date.

### **Flow 2: Returning User or Edit Mode**

1. User returns to in-progress quote
2. Previously uploaded documents are pre-filled with thumbnails
3. User can:
    - Replace or remove individual documents
    - Proceed if all requirements are satisfied

### **Visual Design Notes**

- Consistent layout between desktop (left panel) and mobile (right stack)

## **E) Master Schema Tables**