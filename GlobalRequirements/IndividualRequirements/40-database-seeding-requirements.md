# 40.0 Database Seeding Requirements

Laravel’s database seeders must be utilized to efficiently populate databases with initial data and test datasets for development, testing, and staging environments. This ensures consistent application behavior across different environments while allowing automated data provisioning.

1. Standardized Seeding Implementation
    - All seeders must extend Laravel’s Seeder class and be located in the database/seeders directory.
    - Seeders must be structured to allow modular execution, enabling the execution of individual seeders without requiring a full database refresh.
    - Factories should be used whenever possible to generate large datasets efficiently.
2. Types of Seeders
    1. System Seeders (Initial Data for Core Setup)
        - These seeders must contain essential system data that is required for the application to function.
        - Examples:
            - User Roles & Permissions – Populates roles and permissions tables.
            - System Configurations – Inserts default configuration settings.
            - Lookup Tables – Populates dropdowns, statuses, and static reference data.
    2. Testing Seeders (Development & QA Datasets)
        - Testing seeders should populate the database with mock records for functional and UI testing.
        - Examples:
            - Fake Users – Populates users with randomized names and emails.
            - Sample Policies & Claims – Generates realistic insurance policy and claim records for testing.
            - Transaction History – Populates financial transactions to simulate real-world usage.
    3. Demo Data Seeders (For Stading Environments)
        - These seeders must insert non-sensitive but meaningful data for demonstration purposes.
        - Used in client-facing demo environments to showcase application functionality.
        - Examples:
            - Sample insured and producer accounts.
            - Pre-filled quotes, policies, and claims for walkthroughs.
3. Seeder Execution Requirements
    1. Running Seeders
        - The system must allow seeders to be executed individually or in groups as needed.
        - Laravel’s DatabaseSeeder class must orchestrate execution when seeding the entire system.
    2. Reset & Reseed Workflow
        - Developers and testers must be able to wipe and reseed the database when needed for fresh test runs.
        - The reseeding process should drop all tables, recreate the schema, and repopulate with seed data in a single command.
4. Best Practices for Seeders
    1. Ensure Idempotency
        - Seeders should be re-runnable without causing duplicate data.
        - Existing records should be checked before inserting new data to prevent duplication.
    2. Use Factories for Bulk Data Generation
        - Factories must be used to generate large datasets efficiently for testing and demo environments.
        - Factories allow for scalable data generation with minimal code repetition.
    3. Environment-Specific Seeding
        - Seeders must be able to dynamically adjust based on environment settings to prevent accidental data insertion in production.
        - Testing data must never be seeded into production environments.
5. Security & Performance Considerations
    - Real customer data must never be used in testing or demo seeders.
    - Large seed operations should be handled in batches to prevent memory exhaustion.
    - Production deployments must not automatically execute seeders, except for system seeders necessary for setup.