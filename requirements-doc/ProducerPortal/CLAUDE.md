# Producer Portal Specific Standards

## Reference Architecture Integration

### Global Requirements Foundation
All ProducerPortal requirements build upon Global Requirements:
`/app/workspace/requirements/GlobalRequirements/IndividualRequirements/`

### Approved Requirements Library
Established patterns available at:
`/app/workspace/requirements/ProducerPortal/approved-requirements/`

Use approved requirements for:
- Entity pattern validation
- API design consistency  
- Database schema standards
- Integration approach verification

### Infrastructure Consistency
Cross-reference with existing codebase:
`/app/workspace/blitzy-requirements/` (staging branch)

Mandatory checks:
- Entity model alignment
- Service layer patterns
- API endpoint consistency
- Database migration impacts

### Git Integration Workflow
Before requirement analysis:
1. Update local repository before analysis
2. Review existing implementations
3. Document integration points
4. Validate against current architecture

### Laravel Application Patterns
From blitzy-requirements infrastructure analysis:

#### Backend Architecture Standards
- **Framework**: Laravel 10.x with PHP 8.4+
- **Authentication**: Laravel Sanctum with API tokens
- **Database**: MariaDB with Eloquent ORM
- **Service Layer**: Business logic in dedicated Service classes

#### API Endpoint Standards
**Route Organization:**
- `routes/api.php` - General authentication and user endpoints
- `routes/portal_api.php` - Insured Portal specific endpoints
- Route prefixes for logical grouping (`/api/v1/`, `/portal/`)

**Controller Patterns:**
```php
// Standard CRUD endpoints
Route::prefix('policies')->group(function () {
    Route::get('/', [PolicyController::class, 'index']);
    Route::get('/{id}', [PolicyController::class, 'show']);
    Route::post('/', [PolicyController::class, 'store']);
    Route::put('/{id}', [PolicyController::class, 'update']);
    Route::delete('/{id}', [PolicyController::class, 'destroy']);
});
```

#### Model Relationship Patterns
**Established Patterns from Infrastructure:**
- `Policy` → `User` via `MapUserPolicyDriver`
- `Document` → `Policy` via `map_policy_document`
- `Payment` → `Policy` via `map_policy_transaction`
- All models use `status_id` for state management
- SoftDeletes trait for audit trail maintenance

#### Service Layer Architecture
**Service Class Patterns:**
- `PolicyService` - Policy validation and business logic
- `PaymentService` - Payment processing and history
- `DocumentManager` - Document processing and storage
- `UserPreferencesService` - User settings management
- Services inject repositories and handle business rules

## Domain Context

### System Architecture
- **Multi-tenant architecture** without explicit tenant_id in tables
- **Producer-based access control** through user → producer relationships
- **Quote → Policy → Loss lifecycle** with distinct states
- **Rate, Quote, Bind (RQB)** workflow pattern

### Core Business Concepts
- **Named Insured**: Primary person on the policy (driver with `is_named_insured = true`)
- **Primary Driver**: Main driver of a vehicle (separate from named insured)
- **Suspenses**: Tasks/requirements that must be resolved
- **Requotes**: New quotes based on existing policies

## Established Entity Patterns

### Driver Management
- `driver` table contains all driver information
- `is_named_insured` boolean on driver table (not in map table)
- `driver_type_id` references `driver_type` table for included/excluded/other
- `relationship_to_insured_id` for family/other relationships

### Vehicle Management
- `vehicle` table contains core vehicle data (VIN, year, make, model)
- `is_primary_vehicle` boolean on vehicle table
- `vehicle_use_type_id` for usage classification
- Separate `vehicle_registration` table for plate/registration tracking

### Contact Information
- Separate tables for `phone`, `email`, `address`
- Type references: `phone_type_id`, `email_type_id`, `address_type_id`
- Verification tracking on the entity tables
- Map tables for driver associations: `map_driver_phone`, etc.

### Suspense System
- Central `suspense` table for all suspense types
- Map table approach for associations:
  - `map_quote_suspense`
  - `map_policy_suspense`
- No polymorphic relationships - use explicit map tables

### Document Management
- Central `document` table with `document_type_id`
- Map tables for associations: `map_quote_document`, etc.
- File storage paths, not binary data


## Common Query Patterns

### Finding Named Insured
```sql
SELECT d.*, n.* 
FROM driver d
JOIN name n ON d.name_id = n.id
WHERE d.is_named_insured = TRUE
AND d.status_id = :active_status_id
```

### Active Record Filtering
Always include status check:
```sql
WHERE status_id = :active_status_id
```

### Primary Record Selection
```sql
WHERE is_primary = TRUE
LIMIT 1
```

## Integration Points

### With Global Requirements
- Follow IAM patterns from `01-identity-access-management-iam-updated.md`
- Use workflow patterns from `18-workflow-requirements-updated.md`
- Apply caching strategies from `33-data-services-databases-caching-streaming-updated.md`

### Technology Stack
- **Laravel 12.x** with PHP 8.4+
- **MariaDB 12.x LTS** with read replicas
- **Redis 7.x** for caching
- **Elasticsearch** (future) via Laravel Scout
- **Laravel Echo + Pusher** for real-time updates
- **Kong API Gateway** for API management
- **Apache Camel** for third-party integration routing and transformation
- **HashiCorp Vault** for secure credential management in integrations

## API Design Patterns

### RESTful Endpoints
```
GET    /api/v1/{resource}           # List with pagination
GET    /api/v1/{resource}/{id}      # Single record
POST   /api/v1/{resource}           # Create
PUT    /api/v1/{resource}/{id}      # Update
DELETE /api/v1/{resource}/{id}      # Delete

# Nested resources
GET    /api/v1/{resource}/{id}/{nested}
POST   /api/v1/{resource}/{id}/{nested}

# Integration endpoints
GET    /api/v1/integrations/{code}/config
POST   /api/v1/integrations/third-party-request
GET    /api/v1/integrations/{id}/nodes
POST   /api/v1/quotes/check-duplicates
```

### Standard Response Format
```json
{
  "data": {...},
  "meta": {
    "current_page": 1,
    "total_pages": 10,
    "total_count": 250,
    "per_page": 25
  }
}
```

### Search/Filter Parameters
```
?search=term
?filters[status_ids][]=1&filters[status_ids][]=2
?sort_by=created_at&sort_order=desc
?page=1&per_page=25
```

## Decisions Made During IP269-Quotes-Search

1. **No tenant_id in quote tables** - Multi-tenancy handled at application layer
2. **submitted_at removed** - Use created_at for submission timestamp
3. **Financial fields removed from quote** - Calculated dynamically
4. **Status colors in frontend only** - Not stored in database
5. **All ENUMs converted to reference tables** - For flexibility
6. **Characteristics moved to entity tables** - Not in map tables
7. **Map tables simplified** - Only contain relationships and status

## Universal Entity Management for Producer Portal

Based on **GR-52 Universal Entity Management Architecture**, all external entities (APIs, attorneys, body shops, vendors) use a unified pattern requiring zero code changes for new entity types.

### Core Architecture Components
- **entity_category**: Categorizes entity types (INTEGRATION, PARTNER, VENDOR, SYSTEM)
- **entity_type**: Defines schemas and validation rules with JSON metadata
- **entity**: Stores all external entity instances with flexible metadata

### Benefits (GR-52 Standards)
- **90% faster development** for new external entity types
- **Zero code changes** to add new entity types
- **Consistent CRUD operations** across all entities
- **Automatic UI support** through metadata schemas
- **Sub-second performance** for 10,000+ entity queries

### Standard Entity Categories
- **INTEGRATION**: Third-party API integrations (DCS, SendGrid, Twilio)
- **PARTNER**: Business partners (attorneys, body shops, medical providers)
- **VENDOR**: Service vendors and suppliers
- **SYSTEM**: Internal system entities and configurations

### Implemented Entity Types
#### DCS Integration APIs (GR-53)
- **DCS_HOUSEHOLD_DRIVERS**: Driver verification and household lookup
- **DCS_HOUSEHOLD_VEHICLES**: Vehicle data and VIN decoding
- **DCS_CRIMINAL**: Criminal background verification

#### Communication Services (GR-44)
- **SENDGRID_EMAIL**: Email delivery service integration
- **TWILIO_SMS**: SMS and voice communication service

#### Business Partners (Future Implementation)
- **ATTORNEY**: Legal counsel partners with bar number tracking
- **BODY_SHOP**: Vehicle repair facilities with certification tracking
- **VENDOR**: General service providers with capability tracking

### Integration Patterns (GR-48 Compliance)
- All API calls route through **Apache Camel** integration platform
- DCS integrations use standard entity communication patterns
- Field mappings handled through entity metadata JSON schemas
- Response transformations defined in entity configuration
- Circuit breaker protection with configurable failure thresholds

### Component-Based Security (GR-52)
- **system_component**: Associates backend functionality with frontend and security
- **system_component_permission**: Granular permissions by security group
- Permission codes control access (read/write/delete/admin)
- Component namespaces separate frontend/backend concerns
- No complex licensing - focus on core functionality

### Configuration Management (GR-52)
- **Three-level hierarchy**: entity → program → system (most specific wins)
- JSON-based configuration with schema validation
- Runtime configuration changes through UI
- Performance target: <100ms configuration resolution
- Support for encrypted credential storage via HashiCorp Vault

### Universal Communication Patterns (GR-44)
- **Polymorphic source/target**: source_type/source_id, target_type/target_id
- **Communication types**: API calls, emails, SMS, phone calls
- **Communication channels**: Real-time vs batch processing
- **Correlation IDs** for workflow tracking and distributed tracing
- **Ultra-simple design** focusing on essential fields only

### Performance Standards (GR-52)
- Entity queries: <500ms for 10,000+ entities
- Communication queries: <200ms with correlation ID indexing
- Configuration resolution: <100ms across hierarchy
- Metadata validation: <50ms per entity type
- DCS API calls: Driver <5s, Vehicle <3s, Criminal <10s

### Compliance Requirements (GR-52)
- **Data retention**: 7 years for insurance regulatory compliance
- **Audit logging**: All external entity interactions with PII masking
- **Encryption**: All credentials and sensitive data at rest
- **Access control**: Component-based permissions with role separation
- **Privacy**: Support for consumer data rights and deletion requests

### Global Requirements Integration
- **GR-52**: Universal Entity Management - Core architecture foundation
- **GR-53**: DCS Integration Architecture - Specific API implementations
- **GR-44**: Communication Architecture - SendGrid/Twilio patterns
- **GR-48**: External Integrations Catalog - Apache Camel routing
- **GR-36**: Authentication & Permissions - Security component patterns
- **GR-33**: Data Services & Caching - Performance optimization

## Communication Architecture Patterns

Based on **GR-44 Comprehensive Communication Architecture**, the system implements unified multi-channel communication with SendGrid for email and Twilio for SMS/voice services.

### Multi-Channel Communication Framework
- **Unified CommunicationService**: Handles all email, SMS, and voice communications
- **Template Engine**: Dynamic content replacement with insurance-specific helpers
- **Audit Integration**: Complete communication tracking with correlation IDs
- **Failure Handling**: Circuit breakers and retry mechanisms for service reliability

### SendGrid Email Integration (GR-44)
- **Transactional emails**: Policy confirmations, claim updates, premium notices
- **Template management**: Tenant-specific customization with fallback to system defaults
- **Delivery tracking**: Webhook integration for open/click/bounce tracking
- **Security**: Vault-managed API keys with automatic rotation

### Twilio SMS and Voice Integration (GR-44)
- **SMS notifications**: Premium due reminders, claim status updates, verification codes
- **Voice calls**: Critical notifications with TwiML-generated messages
- **Phone verification**: Multi-factor authentication support
- **Cost controls**: Rate limiting and maximum price protection

### Insurance-Specific Communication Types
Based on **GR-44 template categories**:

#### Policy Notifications
- `policy_bound`: Policy binding confirmation
- `policy_renewal`: Policy renewal notice
- `policy_cancellation`: Policy cancellation notice
- `premium_due`: Premium payment due reminder
- `coverage_change`: Coverage modification notice

#### Claims Communications
- `claim_acknowledged`: Claim acknowledgment
- `claim_status_update`: Claim status update
- `claim_settlement`: Claim settlement notice
- `claim_denial`: Claim denial notification
- `additional_info_needed`: Additional information request

#### Security & Verification
- `email_verification`: Email address verification
- `phone_verification`: Phone number verification
- `password_reset`: Password reset request
- `security_alert`: Security alert notification
- `login_verification`: Login verification code

### Template Management Patterns
- **Dynamic compilation**: Blade-style variable replacement with insurance helpers
- **Currency formatting**: `@currency(premium_amount)` helper
- **Date formatting**: `@date(effective_date, 'M j, Y')` helper
- **Policy formatting**: `@policy(policy_number)` for consistent formatting
- **Tenant customization**: Override system templates with tenant-specific versions

### Performance and Reliability Standards
- **SendGrid rate limits**: 600 requests/minute per tenant
- **Twilio rate limits**: 1000 requests/minute per tenant
- **Template caching**: Redis-cached compiled templates for performance
- **Circuit breakers**: 5 failures trigger protection, 60-second timeout
- **Retry policies**: Exponential backoff with maximum attempt limits

### Communication Audit and Tracking
- **Universal logging**: All communications logged with correlation IDs
- **PII masking**: Automatic masking of sensitive data in audit logs
- **Delivery status**: Real-time status updates via webhooks
- **Correlation tracking**: Link related communications across workflows
- **Retention policies**: 7-year retention for insurance compliance

## Policy Reinstatement Patterns (GR-64)

### Reinstatement Workflow
- **30-day eligibility window** for nonpayment cancellations
- **Premium recalculation** based on lapse period using daily rate methodology
- **Payment-triggered reinstatement** processing with real-time effective dates
- **Installment schedule restructuring** for remaining payment periods
- **No backdating** - coverage effective from reinstatement date only

### Service Layer Integration
- **ReinstatementService** for business logic following GR-20 patterns
- **Integration with existing PolicyService** for consistency
- **Payment processing coordination** using established PaymentService
- **Audit trail requirements** following GR-37 action tracking
- **SR22 status continuity** during reinstatement (GR-10 integration)

### State Management Patterns
- **Real-time eligibility checking** with countdown timers
- **Dynamic premium calculation** based on reinstatement date
- **Form state management** with validation and acknowledgments
- **Cache invalidation** for policy status and calculation updates

## Anti-Patterns to Avoid

### DON'T
- Put business logic fields in map tables
- Use ENUM columns (use reference tables instead)
- Store calculated values (compute dynamically)
- Use `is_active` boolean (use `status_id`)
- Create duplicate entities (reuse existing)
- Store frontend display logic in database
- **Create entity-specific tables when universal pattern applies** (GR-52)
- **Skip global requirements review** before implementing new patterns
- **Ignore approved requirements patterns** without justification
- **Skip infrastructure consistency checks** against blitzy-requirements
- **Use separate communication tables per entity type** (use universal communication)
- **Hardcode API configurations** (use entity metadata and configuration hierarchy)
- **Create manual field mappings** (use JSON schemas and dynamic mapping)

### DO
- Keep map tables simple (just relationships)
- Use reference tables for all types
- Calculate values in queries/application
- Use consistent status management
- Check entity catalog before creating new
- Keep frontend/backend concerns separate
- **Review approved requirements patterns first** before implementing
- **Cross-reference with infrastructure** (blitzy-requirements)
- **Apply universal entity management for ALL external entities** (APIs, attorneys, body shops, vendors)
- **Follow Global Requirements patterns** (GR-52, GR-44, GR-48, GR-53)
- **Use polymorphic communication tracking** with correlation IDs
- **Implement JSON metadata schemas** for automatic UI generation
- **Use three-level configuration hierarchy** (entity → program → system)
- **Check GR-52 entity categories** before creating new entity types
- **Validate against existing codebase patterns** using infrastructure

