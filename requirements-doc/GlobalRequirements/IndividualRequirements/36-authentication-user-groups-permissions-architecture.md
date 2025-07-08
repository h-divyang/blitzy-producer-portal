# 36.0 Authentication, User Groups & Permissions Architecture

This architecture ensures role-based access control (RBAC) and feature-based permissions while allowing for flexibility in user and producer management. The system is designed to handle authentication, user classification, and fine-grained permissions for both internal users (e.g., CSRs, underwriters, claims handlers) and external users (e.g., producers, insureds).

### 36.1 Authentication & User Management

1. User Table
    1. Purpose: Stores user account details for authentication and identity management.
    2. Key Fields:
        - id – Unique identifier for the user.
        - name – Full name of the user.
        - email – User's email (used for login).
        - password – Encrypted password.
        - user_type_id – Links the user to a user type.
        - status – Active, inactive, or suspended.
        - audit – Tracks user creation and modifications.
2. User Type Table
    1. Purpose: Defines different types of users in the system (e.g., insured, producer, internal staff).
    2. Key Fields:
        - id – Unique identifier.
        - name – Name of the user type (e.g., Insured, Producer, Underwriter, Claims Adjuster).
        - description – Brief description of the user type.
3. User Group Table
    1. Purpose: Groups users based on their job function or level (e.g., CSR, manager, underwriter).
    2. Key Fields:
        - id – Unique identifier.
        - name – Name of the user group (e.g., CSR, Manager, Underwriter).
        - description – Brief explanation of the group's role in the system.
4. Map User to Group Table (map_user_group)
    1. Purpose: Maps users to multiple groups, allowing for flexible access control.
    2. Key Fields:
        - user_id – Links to a specific user.
        - user_group_id – Links to a user group.

### 36.2 Permissions & Feature Access

1. Feature Table
    1. Purpose: Defines system functionalities and modules that can have restricted access.
    2. Key Fields:
        - id – Unique identifier.
        - name – Name of the feature (e.g., Policy, Claims, Payments, Reports).
        - feature_type_id – Links to the feature type table.
2. Feature Type Table
    1. Purpose: Categorizes system features (e.g., System Module, Third-Party API).
    2. Key Fields:
        - id – Unique identifier.
        - name – Category of the feature (e.g., Core System, API Integration).
3. Permission Table
    1. Purpose: Defines actions users can perform on a feature (e.g., view, update, quote, bind).
    2. Key Fields:
        - id – Unique identifier.
        - name – Permission type (e.g., view, edit, delete, bind).
        - description – Explanation of the permission.
4. Map User Group to Feature Permissions (map_user_group_feature_permission)
    1. Purpose: Maps user groups to features and assigns permissions to them.
    2. Key Fields:
        - user_group_id – Links to a user group.
        - feature_id – Links to a system feature.
        - permission_id – Defines what actions can be performed on the feature.
    3. Example Use Case:
        - Underwriters (user group) may have view, update, and bind permissions for the policy module (feature).
        - CSRs (user group) may have view and update permissions but not bind permission.

### 36.3 Producer Management and Access Control

1. Producer Table
    1. Purpose: Stores producer-specific details and links them to their assigned users.
    2. Key Fields:
        - id – Unique identifier.
        - producer_code_id – Links to a specific producer code.
        - name – Producer's name.
        - type_id – Links to the producer type table.
2. Producer Code Table
    1. Purpose: Defines producer codes with their unique identifiers.
    2. Key Fields:
        - id – Unique identifier.
        - code – Unique producer code assigned by the system or carrier.
        - description – Additional details about the code.
        - status – Active or inactive.
        - audit – Tracks changes.
3. Producer Type Table
    1. Purpose: Defines different categories of producers (e.g., independent agency, direct writers).
    2. Key Fields:
        - id – Unique identifier.
        - name – Type of producer (e.g., Independent Agent, MGA, Direct).
4. Map User to Producer Table (map_user_producer)
    1. Purpose: Allows users to be linked to one or more producers, granting them access to producer-related data.
    2. Key Fields:
        - user_id – Links to a user.
        - producer_id – Links to a producer.
    3. Example Use Case:
        - A producer's assistant (user) can be mapped to multiple producer entities they manage.
        - A regional manager can be mapped to multiple independent agencies they oversee.
5. Producer Group Table
    1. Purpose: Organizes producers into groups for better management.
    2. Key Fields:
        - id – Unique identifier.
        - producer_group_type_id – Links to a producer group category.
        - code – Unique identifier for the group.
        - description – Summary of the group's purpose.
6. Producer Group Type Table
    1. Purpose: Defines the types of producer groups (e.g., regional agents, national producers).
    2. Key Fields:
        - id – Unique identifier.
        - name – Type of producer group (e.g., Regional, National).
7. Map Producer to Producer Group Table (map_producer_group)
    1. Purpose: Assigns producers to multiple producer groups.
    2. Key Fields:
        - producer_id – Links to a producer.
        - producer_group_id – Links to a producer group.
    3. Example Use Case:
        - A regional producer (producer_id) may belong to a national producers group (producer_group_id).
        - A national aggregator may oversee multiple regional producer groups.
8. Map Producer Group to Feature Permissions (map_producer_group_feature_permissions)
    1. Purpose: Defines which features producer groups can access and their permissions.
    2. Key Fields:
        - producer_group_id – Links to a producer group.
        - feature_id – Defines what part of the system is accessible.
        - permission_id – Determines the allowed actions (view, update, bind).
    3. Example Use Case:
        - Regional producers (producer group) may have view and quote access to policy records (feature).
        - National agencies may have bind and issue permissions in addition to view access.

### 36.4 How Authentication and Permissions Work Together

1. User Logs In
    - Authenticated via AuthenticationManager (OAuth2/JWT).
    - User type is identified via the user_type table.
2. User’s Role is Determined
    - User is linked to one or more user groups via map_user_group.
3. Feature Access is Evaluated
    - System checks map_user_group_feature_permission for allowed features and permissions.
    - If a user is associated with a producer, their access is further validated via map_user_producer.
4. Actions Are Controlled
    - Users can perform only the actions they have permissions for (view, update, bind).
    - Additional validation is performed for producers to ensure they operate within allowed producer groups.