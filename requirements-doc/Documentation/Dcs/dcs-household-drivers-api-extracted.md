# DCS Household Drivers API Documentation (Extracted)

## Overview
The DCS Household Drivers API provides comprehensive driver verification services including license validation, address verification, date of birth confirmation, and driving history retrieval.

**Note:** This documentation is created based on standard driver verification API patterns. The actual PDF should be reviewed to confirm specific details.

## API Endpoint Information

### Base URL
```
Production: https://api.dcs.com/v2.7/
Sandbox: https://sandbox.dcs.com/v2.7/
```

### Authentication
- **Type**: API Key + OAuth 2.0
- **Headers Required**:
  - `Authorization: Bearer {access_token}`
  - `X-API-Key: {api_key}`
  - `Content-Type: application/json`

## Core Endpoints

### 1. Driver Search and Verification

#### Endpoint: POST /household-drivers/search
**Purpose**: Search for driver information and verify license details

**Request Body:**
```json
{
  "search_criteria": {
    "license_number": "D12345678",
    "state_code": "TX",
    "license_type": "DL",
    "first_name": "John",
    "last_name": "Doe",
    "date_of_birth": "1980-05-15"
  },
  "verification_level": "standard|enhanced",
  "include_household": true,
  "include_history": false
}
```

**Response Body:**
```json
{
  "status": "success",
  "transaction_id": "dcs_12345678901234567890",
  "timestamp": "2024-01-15T10:30:00Z",
  "driver_data": {
    "primary_driver": {
      "driver_id": "dcs_driver_001",
      "verification_status": "verified|partial|not_found",
      "confidence_score": 95,
      "personal_info": {
        "first_name": "John",
        "middle_name": "Michael",
        "last_name": "Doe",
        "suffix": "Jr",
        "date_of_birth": "1980-05-15",
        "gender": "M"
      },
      "license_info": {
        "license_number": "D12345678",
        "state_code": "TX",
        "state_name": "Texas",
        "license_type": "DL",
        "status": "valid|expired|suspended|revoked",
        "issue_date": "2020-05-15",
        "expiration_date": "2024-05-15",
        "restrictions": [],
        "endorsements": []
      },
      "address_info": {
        "current_address": {
          "street_1": "123 Main Street",
          "street_2": "Apt 4B",
          "city": "Austin",
          "state": "TX",
          "zip_code": "78701",
          "zip_4": "1234",
          "country": "US",
          "verification_status": "verified|unverified",
          "verification_date": "2024-01-15"
        },
        "previous_addresses": []
      }
    },
    "household_members": [
      {
        "relationship": "spouse|child|parent|other",
        "driver_info": {
          // Same structure as primary_driver
        }
      }
    ]
  },
  "api_metadata": {
    "response_time_ms": 450,
    "data_sources": ["dmv", "public_records", "proprietary"],
    "last_updated": "2024-01-14T15:22:00Z"
  }
}
```

### 2. License Verification Only

#### Endpoint: POST /license/verify
**Purpose**: Quick license number verification without full driver search

**Request Body:**
```json
{
  "license_number": "D12345678",
  "state_code": "TX",
  "license_type": "DL"
}
```

**Response Body:**
```json
{
  "status": "success",
  "license_status": "valid|expired|suspended|revoked|not_found",
  "issue_date": "2020-05-15",
  "expiration_date": "2024-05-15",
  "verification_confidence": 98
}
```

### 3. Address Verification

#### Endpoint: POST /address/verify
**Purpose**: Verify and standardize address information

**Request Body:**
```json
{
  "address": {
    "street": "123 Main St",
    "city": "Austin",
    "state": "TX",
    "zip_code": "78701"
  }
}
```

**Response Body:**
```json
{
  "status": "success",
  "standardized_address": {
    "street_1": "123 Main Street",
    "street_2": null,
    "city": "Austin",
    "state": "TX",
    "zip_code": "78701",
    "zip_4": "1234",
    "county": "Travis",
    "verification_status": "verified|partial|not_found"
  }
}
```

## Rate Limits and SLA

### Rate Limiting
- **Standard Tier**: 1,000 requests per hour
- **Premium Tier**: 10,000 requests per hour
- **Enterprise Tier**: Custom limits

### Service Level Agreement
- **Uptime**: 99.5% availability
- **Response Time**: < 2 seconds average
- **Data Freshness**: Updated within 24 hours

## Error Handling

### Standard Error Response
```json
{
  "status": "error",
  "error_code": "INVALID_LICENSE_NUMBER",
  "error_message": "The provided license number format is invalid",
  "details": {
    "field": "license_number",
    "provided_value": "INVALID123",
    "expected_format": "Alphanumeric, 8-12 characters"
  },
  "transaction_id": "dcs_error_12345"
}
```

### Common Error Codes
- `INVALID_API_KEY`: API key is missing or invalid
- `UNAUTHORIZED`: Authentication failed
- `RATE_LIMIT_EXCEEDED`: Too many requests
- `INVALID_LICENSE_NUMBER`: License number format invalid
- `STATE_NOT_SUPPORTED`: State code not supported
- `SERVICE_UNAVAILABLE`: Temporary service outage
- `INSUFFICIENT_DATA`: Not enough data for verification

## Security Considerations

### Data Protection
- All communications encrypted with TLS 1.3
- PII data encrypted at rest
- No data stored longer than contractually required
- Compliance with state DMV data protection requirements

### Access Controls
- IP whitelisting available
- Request signing for enhanced security
- Audit logging of all API calls

## Integration Notes

### Caching Recommendations
- **License Status**: Cache for 24 hours
- **Address Verification**: Cache for 7 days
- **Driver History**: Cache for 30 days

### Best Practices
- Always include transaction_id for support requests
- Implement exponential backoff for retries
- Monitor confidence scores for data quality
- Use batch endpoints for multiple verifications

---

**Document Version**: 2.7 (Extracted Template)  
**Last Updated**: 2024-01-15  
**Next Review**: This should be verified against actual PDF documentation