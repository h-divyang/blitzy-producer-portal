# 41.0 Table Schema Requirements

To systematically outline and standardize table schema requirements, we must use a structured and hierarchical approach.

- Understand Table Relationships – Primary keys, foreign keys, and associations.
- Maintain Consistency – Follow a predefined naming convention.
- Ensure Completeness – Include required constraints, descriptions, and references.
- Follow Best Practices – Adhere to database standards (e.g., /MariaDB/ best practices).
1. Structured Table Schema Explanation
    1. General Table Structure
        1. Each table follows a consistent format, containing:
            - Primary Key (id) – Uniquely identifies each record.
            - Foreign Keys – References related entities.
            - Status Tracking – Uses a status_id column for record state management.
            - Audit Fields – Tracks user interactions (created_by, updated_by) and timestamps (created_at, updated_at).
            - Description Column (if applicable) – Stores additional details about the record.
    2. Naming Conventions
        - Tables: Use singular nouns (e.g., user, policy, producer).
        - Join Tables (Mapping Tables): Prefixed with map_ and use singular nouns (e.g., map_user_group, map_producer_signature).
        - Status Tables: Single status table and status_type table. status.id referred to in each map_ table.
        - Reference Tables: Use _type suffix (e.g., user_type, producer_type).
        - Timestamps: Always include created_at, updated_at for record lifecycle tracking.
    3. Standardized Table Types
        1. Each table falls into one of the following categories:
            - Core Entity Tables
                - Store primary records in the system.
                - Example: user, policy, producer.
            - Reference Tables
                - Define categories for entity classification.
                - Example: user_type, producer_type, document_type.
            - Mapping Tables (map_*)
                - Define many-to-many relationships.
                - Example: map_user_group, map_producer_user.
            - Status & Lifecycle Tracking
                - Track entity state (e.g., active, inactive).
                - Example: status, policy.status_id.
2. Conclusion
    - Tables use singular nouns.
    - Join tables use map_ prefix.
    - Each table includes a primary key (id), status tracking (status_id), and audit fields (created_by, updated_by, created_at, updated_at).
    - Use BIGINT for IDs, VARCHAR(255) for names, and TEXT for descriptions.
    - Ensure foreign keys reference appropriate parent tables.
    - Ensure timestamps track data changes.

## Policy Reinstatement Schema Extensions

### Status Values for Reinstatement (GR-64)
The following status values must be added to support policy reinstatement workflows:

```sql
-- Additional status values for policy reinstatement
INSERT INTO status (code, name, description, is_active) VALUES
('ELIGIBLE_FOR_REINSTATEMENT', 'Eligible for Reinstatement', 'Policy cancelled but within reinstatement window', true),
('REINSTATED', 'Reinstated', 'Policy successfully reinstated after cancellation', true),
('EXPIRED_REINSTATEMENT', 'Reinstatement Expired', 'Policy reinstatement window has expired', false);
```

### Reinstatement Calculation Table
```sql
CREATE TABLE reinstatement_calculation (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    policy_id BIGINT UNSIGNED NOT NULL,
    
    -- Calculation details
    cancellation_date DATE NOT NULL,
    reinstatement_date DATE NOT NULL,
    lapse_days INTEGER NOT NULL,
    
    -- Premium breakdown
    original_premium DECIMAL(10,2) NOT NULL,
    daily_premium_rate DECIMAL(8,4) NOT NULL,
    lapsed_premium DECIMAL(10,2) NOT NULL,
    adjusted_premium DECIMAL(10,2) NOT NULL,
    unpaid_premium DECIMAL(10,2) DEFAULT 0,
    reinstatement_fees DECIMAL(10,2) DEFAULT 0,
    total_due DECIMAL(10,2) NOT NULL,
    
    -- Standard audit fields
    status_id BIGINT UNSIGNED NOT NULL,
    created_by BIGINT UNSIGNED NOT NULL,
    updated_by BIGINT UNSIGNED NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    -- Foreign key constraints
    FOREIGN KEY (policy_id) REFERENCES policy(id),
    FOREIGN KEY (status_id) REFERENCES status(id),
    FOREIGN KEY (created_by) REFERENCES user(id),
    FOREIGN KEY (updated_by) REFERENCES user(id),
    
    -- Indexes for performance
    INDEX idx_policy (policy_id),
    INDEX idx_reinstatement_date (reinstatement_date),
    INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

## Cross-References

### Related Global Requirements
- **GR-64**: Policy Reinstatement with Lapse Process - Database schema requirements for reinstatement functionality
- **GR-02**: Database Design Principles - Audit field and constraint standards
- **GR-19**: Table Relationships - Mapping table and foreign key patterns