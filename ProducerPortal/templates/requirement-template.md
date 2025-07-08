# [Requirement ID] - [Requirement Name]

## Pre-Analysis Checklist

### Initial Review
- [ ] Read base requirement document completely
- [ ] Identify all UI elements and data fields mentioned
- [ ] Note workflow states and transitions described
- [ ] List relationships to existing entities

### Global Requirements Alignment
- [ ] Review applicable global requirements
- [ ] Note which GRs apply to this requirement
- [ ] Ensure patterns align with global standards
- [ ] Cross-reference with ProducerPortal standards

### Cross-Reference Check
- [ ] Review entity catalog for reusable entities
- [ ] Check architectural decisions for relevant patterns
- [ ] Search source code for similar functionality
- [ ] Review related requirements for shared entities

### Compliance Verification
- [ ] Verify alignment with CLAUDE.md standards
- [ ] Check naming convention compliance
- [ ] Validate reference table approach for ENUMs
- [ ] Ensure status_id usage instead of is_active

---

## Entity Analysis

### Entities Involved
| Entity Name | Type | Status | Notes |
|-------------|------|--------|--------|
| [entity] | Core/Reference/Map/Supporting | New/Existing/Modified | [usage notes] |

### New Tables Required
- **[table_name]**: [purpose and usage]
- **[reference_table]**: [lookup/type data]

### Modifications to Existing Tables
- **[table_name]**: [changes needed and impact]

### Relationships Identified
- [entity_a] → [relationship] → [entity_b]

---

## Field Mappings (Section C)

### Backend Mappings

#### [UI Section Name]

##### [Field Name]
- **Backend Mapping**: 
  ```
  get [entity].id from [table]
  -> get [related_entity] by [entity].[foreign_key]
  -> return [fields], [transformations]
  ```

##### [Another Field]
- **Backend Mapping**:
  ```
  [complete query path with all joins and conditions]
  ```

### Implementation Architecture
[Details about how the requirement is implemented, including patterns, services, and architectural decisions]

### Integration Specifications (if applicable)

#### External Service Integrations

##### [Service Name] Integration
**Entity Type**: [SERVICE_ENTITY_TYPE] (Universal Entity Management)  
**Provider**: [Provider Name]  
**Endpoint**: [Base URL/Endpoint]

**Circuit Breaker Configuration**:
```php
'[service_key]' => [
    'failure_threshold' => 5,
    'timeout_seconds' => 60,
    'recovery_timeout' => 300,
    'fallback_strategy' => '[fallback_approach]'
]
```

**Service Implementation**:
```php
class [ServiceName]Service implements [ServiceInterface]
{
    public function [primaryMethod]([parameters]): [ReturnType]
    {
        $correlationId = Str::uuid();
        
        try {
            // Log outbound communication
            $this->communicationService->logOutbound(
                '[source_type]', $this->[sourceId],
                'entity', $this->get[Service]EntityId(),
                '[communication_type]',
                $requestData,
                $correlationId
            );
            
            $response = $this->[service]Client->[method]([parameters]);
            
            // Cache successful response
            Cache::put("[cache_key]", $response, now()->add[Duration]());
            
            return new [ReturnType]([
                // Map response data
            ]);
            
        } catch ([ServiceException] $e) {
            if ($this->circuitBreaker->isOpen('[service_key]')) {
                return $this->handleFallback([parameters]);
            }
            throw $e;
        }
    }
    
    private function handleFallback([parameters]): [ReturnType]
    {
        Log::info('[Service] fallback triggered', [
            // Log relevant data (mask PII)
        ]);
        
        return new [ReturnType]([
            'status' => 'fallback_mode',
            'message' => '[Fallback message for users]'
        ]);
    }
}
```

**Request/Response Schema**:
```json
// Request
{
  "[field1]": "[value1]",
  "[field2]": "[value2]"
}

// Response
{
  "[response_field1]": "[value1]",
  "[response_field2]": {
    "[nested_field]": "[nested_value]"
  }
}
```

**Security & Privacy**:
- [Data encryption requirements]
- [PII masking in logs]
- [Correlation ID tracking]
- [Retention policies]

#### Communication Tracking (Following GR-44)

All external API calls logged using universal communication table:
```php
class CommunicationService
{
    public function logOutbound(
        string $sourceType, 
        int $sourceId,
        string $targetType, 
        int $targetId,
        string $communicationType,
        array $requestData,
        string $correlationId
    ): Communication {
        return Communication::create([
            'source_type' => $sourceType,
            'source_id' => $sourceId,
            'target_type' => $targetType,
            'target_id' => $targetId,
            'correlation_id' => $correlationId,
            'request_data' => $this->maskSensitiveData($requestData),
            'status_id' => CommunicationStatus::where('code', 'pending')->first()->id
        ]);
    }
    
    private function maskSensitiveData(array $data): array
    {
        // Mask [sensitive_fields], [pii_data], etc.
        return $data;
    }
}
```

#### Performance & Monitoring

**Response Time Targets**:
- [Operation 1]: < [time]ms
- [Operation 2]: < [time] seconds
- [External Service]: < [time] seconds (with fallback)

**Caching Strategy**:
- [Data Type 1]: [duration]
- [Data Type 2]: [duration]
- [External Results]: [duration]

**Rate Limiting**: [requests]/[time period] per [user/producer/tenant]

#### Error Handling

**Circuit Breaker Pattern**: [failure count] failures trigger protection, [timeout] timeout  
**Graceful Degradation**: [fallback behavior description]  
**Retry Logic**: [retry strategy description]

[Continue pattern for all UI sections and fields...]

---

## API Specifications

### Endpoints Required
```http
GET    /api/v1/[resource]              # List with pagination
GET    /api/v1/[resource]/{id}         # Single record
POST   /api/v1/[resource]              # Create
PUT    /api/v1/[resource]/{id}         # Update
DELETE /api/v1/[resource]/{id}         # Delete
```

### Real-time Updates
```javascript
// WebSocket channels
private-[entity].{id}                  # Specific entity updates
private-tenant.{tenant_id}.[entities]  # All entities for tenant
```

---

## Database Schema (Section E)

### New Core Tables

#### [table_name]
```sql
CREATE TABLE [table_name] (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  
  -- Business columns
  [column_name] [TYPE] [constraints],
  
  -- Foreign keys
  [related_table]_id BIGINT UNSIGNED NOT NULL,
  status_id BIGINT UNSIGNED NOT NULL,
  
  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY ([related_table]_id) REFERENCES [related_table](id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  -- Indexes
  INDEX idx_[related_table] ([related_table]_id),
  INDEX idx_status (status_id),
  INDEX idx_[business_field] ([business_field])
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### New Reference Tables

#### [reference_table]
```sql
CREATE TABLE [reference_table] (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  code VARCHAR(50) UNIQUE NOT NULL,
  name VARCHAR(100) NOT NULL,
  description TEXT NULL,
  status_id BIGINT UNSIGNED NOT NULL,
  
  -- Audit fields
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY (status_id) REFERENCES status(id),
  
  -- Indexes
  INDEX idx_code (code),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### New Relationship Tables

#### map_[entity_a]_[entity_b]
```sql
CREATE TABLE map_[entity_a]_[entity_b] (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  [entity_a]_id BIGINT UNSIGNED NOT NULL,
  [entity_b]_id BIGINT UNSIGNED NOT NULL,
  status_id BIGINT UNSIGNED NOT NULL,
  
  -- Audit fields
  created_by BIGINT UNSIGNED NOT NULL,
  updated_by BIGINT UNSIGNED NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  
  -- Foreign key constraints
  FOREIGN KEY ([entity_a]_id) REFERENCES [entity_a](id) ON DELETE CASCADE,
  FOREIGN KEY ([entity_b]_id) REFERENCES [entity_b](id),
  FOREIGN KEY (status_id) REFERENCES status(id),
  FOREIGN KEY (created_by) REFERENCES user(id),
  FOREIGN KEY (updated_by) REFERENCES user(id),
  
  -- Constraints
  UNIQUE KEY unique_[entity_a]_[entity_b] ([entity_a]_id, [entity_b]_id),
  
  -- Indexes
  INDEX idx_[entity_a] ([entity_a]_id),
  INDEX idx_[entity_b] ([entity_b]_id),
  INDEX idx_status (status_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### Modified Tables

#### ALTER TABLE [existing_table]
```sql
-- Add new columns
ALTER TABLE [existing_table] 
ADD COLUMN [new_column] [TYPE] [constraints];

-- Add foreign key
ALTER TABLE [existing_table]
ADD CONSTRAINT fk_[table]_[column] 
FOREIGN KEY ([column]_id) REFERENCES [other_table](id);

-- Add index
ALTER TABLE [existing_table]
ADD INDEX idx_[column] ([column]);
```

---

## Implementation Notes

### Dependencies
- [List any dependencies on other requirements]
- [Note integration points with existing systems]

### Migration Considerations
- [Data migration needs]
- [Backwards compatibility concerns]

### Performance Considerations
- [Query optimization notes]
- [Caching strategy]
- [Index strategy]

---

## Quality Checklist

### Pre-Implementation
- [ ] All UI fields mapped to database columns
- [ ] Existing entities reused where possible
- [ ] Reference tables created for all ENUMs
- [ ] Naming conventions followed consistently
- [ ] Relationships properly defined with foreign keys

### Post-Implementation
- [ ] All foreign keys have proper constraints
- [ ] Appropriate indexes for expected query patterns
- [ ] Audit fields included on all tables
- [ ] Status management consistent across tables
- [ ] Entity catalog updated with new entities
- [ ] Architectural decisions documented if new patterns

### Final Validation
- [ ] Backend mappings complete and accurate
- [ ] Database schema follows all standards
- [ ] No redundant tables or columns created
- [ ] Performance considerations addressed
- [ ] Documentation updated