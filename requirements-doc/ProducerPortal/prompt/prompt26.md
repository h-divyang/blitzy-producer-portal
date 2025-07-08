I have provided feedback on the following requirements.
* Gather the feedback and create a new file of each requirement with the updates.
  * Ensure to reference all supporting files outlined in README and CLAUDE files to ensure accuracy.
* Do not update or modify existing files in Aime/workspace/requirements/ProducerPortal/approved-requirements.

Aime/workspace/requirements/ProducerPortal/approved-requirements/IP269-New-Quote-Step-1-Primary-Insured.md
* producer_program
  * should be map_program_producer
* license_type
  * this should reference the license and license_type tables
  * license type should be kept simple
  * license should reference state_id, license_type_id, country_id, etc..
* map_quote_driver
  * is_primary_insured BOOLEAN DEFAULT FALSE,
    * this should already be on the driver record right? 
  * dcs_driver_id VARCHAR(100) NULL, 
  * dcs_correlation_id VARCHAR(100) NULL,
    * these should be on the driver record right?
* driver
  * ADD COLUMN license_type_id BIGINT UNSIGNED NULL;
    * this should just be the license record, right?

Aime/workspace/requirements/ProducerPortal/approved-requirements/IP269-New-Quote-Step-1-Named-Insured.md
* quote_discount_eligibility
  * I'm not too sure about the puropose of this table.
* housing_type
  * should be residence_type
* driver: Add gender_id, marital_status_id, housing_type_id references
  * residence_type_id
* prior_insurance
  * has_prior_insurance BOOLEAN DEFAULT FALSE,
    * this is redundant
  * policy_number VARCHAR(100) NULL,
    * should be policy_id
* discount_type
  * discount and discount_type table
  * type table for definition
  * dicount table for specficis

Aime/workspace/requirements/ProducerPortal/approved-requirements/IP269-New-Quote-Step-2-Drivers.md
* driver_violation
* violation_type
  * violation, violation_type, driver_violations
    * violation is all violations available to add to a driver
    * violation_type defines major, minor, DUI, etc.
    * map_entity_violation maps DCS soecific violation and system specific violations
    * driver_violations
      * maps the driver_id, violation_id, and other information.
* ALTER TABLE driver
  * ADD COLUMN occupation VARCHAR(200) NULL,
    * this should be occupation_id
    * occupation and occupation_type should be their own tables
* ALTER TABLE map_quote_driver
  * ADD COLUMN relationship_to_insured_id BIGINT UNSIGNED NULL,
    * this should be defined on the driver record
  * ADD COLUMN include_status ENUM('included', 'excluded', 'removed') DEFAULT 'included',
    * this should be in the driver_type table
  * ADD COLUMN removal_reason TEXT NULL,
    * this should be a table as they are predefined.
  * ADD COLUMN exclusion_reason TEXT NULL;
    * this is not needed right now I don't think.

Aime/workspace/requirements/ProducerPortal/approved-requirements/IP269-New-Quote-Step-3-Vehicles.md
* vehicle_owner
  * this should be stored as an entity with an entity_type as vehicle owner same with lienholder, lessee, etc.. and mapped to the vehicle as they do not always have this information
  * ownership_percentage DECIMAL(5,2) DEFAULT 100.00,
    * remove
  * external_source_id VARCHAR(100) NULL,
    * this should be source_entity_id
    * refer to the entity_id for DCS for exapmle.
* vehicle_usage_type 
  * commercial_use BOOLEAN DEFAULT FALSE,
    * this should be removed as it's going to be a type, itself.
* map_quote_vehicle
  * usage_type_id BIGINT UNSIGNED NOT NULL, 
  * garaging_address_id BIGINT UNSIGNED NULL, 
  * annual_mileage INT NULL,
    * these should all be on the vehicle record
* map_vehicle_owner
  * vehicle_owner_id BIGINT UNSIGNED NOT NULL,
    * this should reference the entity_id and be named vehicle_owner_entity_id
* ALTER TABLE vehicle
  * ADD COLUMN external_source_id VARCHAR(100) NULL,
    * this should be source_entity_id
    * refer to the entity_id for DCS for exapmle.