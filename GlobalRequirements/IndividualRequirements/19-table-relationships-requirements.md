# 19.0 Table Relationships & Requirements

1. action_type
    - Stores the names of actions like create, read, update, delete, trashed, processed, etc.
    - Does not specify whether the action is manual or automated; that’s determined by the user performing the action.
    - A status_id and user references (created_by, updated_by) may track the lifecycle of each action type definition.
2. action
    - References action_type_id to identify which operation was performed (CRUD, trash, process, etc.).
    - Stores description with details of what exactly changed.
    - Has user references (created_by, updated_by) to link the action to a specific person (manual user) or automated account (system, scheduler).
    - A status_id indicates the action’s current validity or completion state.
3. map_ENTITY_action (e.g., map_document_action)
    1. Entity reference: e.g., document_id for a document entity.
    - action_id: Links back to the action just recorded.
    - Optionally includes a separate description or status_id for the mapping itself, if needing to track the mapping’s lifecycle.
    - Also records created_by, updated_by to capture who linked the action with the entity (often the same user who performed the action).