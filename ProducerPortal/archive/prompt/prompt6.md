Consider
* Aime/workspace/requirements/ProducerPortal/integration-architecture-implementation-plan.md
* Aime/workspace/requirements/TpaManager/Dcs/Household Drivers API Documentation Version 2.7 conv.rtf

We don't need to outline any front-end information in these requirements unless it's specifically defined in our source files.
* an example is 
* icon VARCHAR(100), -- FontAwesome or custom icon 
* color VARCHAR(7), -- Hex color for UI theming 
* ui_component VARCHAR(100), -- React component name
* display_order INT DEFAULT 0,
* ui_config JSON, -- Display preferences, custom settings

features JSON, -- Available features for this type
* should we have this same conversation around feature and feature_type
  * I feel like anything defined as a feature should be defined and associated
    * We should be defining main parts of the system as system features and could surround those with configuration options as this will be a licensed system
  * what do you think?

is_active BOOLEAN DEFAULT TRUE,
* this should be deduced from status_id

-- Applicability rules
applies_to_entity_types JSON, -- Which entity types can use this config
applies_to_scopes JSON, -- Which scope types are valid
-- Flexible scope (system → program → producer → entity)
scope_type ENUM('system', 'entity', 'program', 'producer', 'user') NOT NULL,
scope_id BIGINT UNSIGNED NULL, -- ID of the scoped object
-- Channel support configuration
supported_channels JSON, -- ["api", "email", "phone", "mail"]
* I dont know that I understand these.

-- Configuration inheritance
parent_configuration_id BIGINT UNSIGNED NULL,
* is there a better way to do this?

-- Default behavior
default_retry_attempts INT DEFAULT 3,
default_timeout_seconds INT DEFAULT 30,
* should these be in the configuration table?

source_type ENUM('system', 'user', 'entity') NOT NULL,
target_type ENUM('system', 'user', 'entity') NOT NULL,
direction ENUM('inbound', 'outbound', 'bidirectional') NOT NULL,
* should these and other examples where types are being stored as strings be their own tables for scalablity? Or is that too much?

channel ENUM('api', 'email', 'sms', 'phone', 'mail', 'webhook', 'internal') NOT NULL,
* isnt this defined by communication type?

status ENUM('pending', 'processing', 'completed', 'failed', 'timeout') NOT NULL DEFAULT 'pending',
- should this be status_id

-- Distributed tracing
correlation_id VARCHAR(100) NOT NULL,
parent_communication_id BIGINT UNSIGNED NULL,
-- Business context links
quote_id BIGINT UNSIGNED NULL,
driver_id BIGINT UNSIGNED NULL,
policy_id BIGINT UNSIGNED NULL,
- is there a better way of handling these?

with these in mind, ask any additinoal questions if needed as well as output a revised version of Aime/workspace/requirements/ProducerPortal/integration-architecture-implementation-plan.md