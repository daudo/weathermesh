# ADR-002: API Design and Versioning Strategy

## Status

Accepted

## Date

2025-09-23

## Context

weathermesh needs to provide a stable, well-designed REST API that can evolve over time without breaking existing clients. The API will serve diverse consumers including web frontends, mobile applications, IoT devices, and home automation systems. Each consumer type may have different requirements for data format, update frequency, and query complexity.

Key considerations:
- API must be intuitive and self-documenting
- Changes must not break existing clients
- Performance requirements vary (real-time vs. historical queries)
- Multiple data formats may be needed (JSON primarily, potentially MessagePack or Protocol Buffers)
- API will be publicly exposed requiring robust error handling
- Must support both simple and complex queries (single station vs. aggregations)

## Decisions

### 1. API Style: RESTful with OpenAPI 3.0

**Decision**: Implement a RESTful API following REST principles with OpenAPI 3.0 specification.

**Rationale**:
- **Wide understanding**: REST is the most widely understood API paradigm
- **Tooling ecosystem**: Extensive tooling for testing, documentation, and client generation
- **OpenAPI benefits**: Auto-generated documentation, client SDKs, validation
- **Caching friendly**: HTTP caching mechanisms work naturally with REST
- **Stateless**: Aligns with our scalability requirements
- **GraphQL consideration**: While GraphQL offers query flexibility, it adds complexity that isn't justified for our use case

**REST Principles to Follow**:
- Resources are nouns (e.g., `/stations`, `/measurements`)
- HTTP methods define actions (GET, POST, PUT, DELETE)
- Stateless requests (no server-side session state)
- HATEOAS where beneficial (linked resources)

### 2. Versioning Strategy: URL Path Versioning

**Decision**: Use URL path versioning with major version only (e.g., `/api/v1/stations`).

**Rationale**:
- **Clarity**: Version is immediately visible in the URL
- **Cache friendly**: Different versions have different URLs
- **Easy routing**: Simple to route different versions to different handlers
- **Wide adoption**: Most common approach in public APIs
- **Proxy friendly**: Works well with API gateways and reverse proxies

**Versioning Rules**:
- Major version only in URL (`v1`, `v2`)
- Minor versions handled through backwards-compatible additions
- Deprecation notices via headers (`Sunset` header)
- Minimum 6-month deprecation period for major versions

**Alternatives Considered**:
- Header versioning: Less visible, harder for debugging
- Query parameter versioning: Can be accidentally omitted
- Content negotiation: Complex, poor cache behavior

### 3. Resource Design

**Decision**: Design resources around domain concepts, not database tables.

**Resource Hierarchy**:
```
/api/v1/stations                          # List all stations
/api/v1/stations/{id}                     # Specific station
/api/v1/stations/{id}/current             # Current conditions
/api/v1/stations/{id}/measurements        # Historical data
/api/v1/measurements                      # Query across stations
/api/v1/aggregations                      # Computed aggregates
/api/v1/alerts                           # Alert configurations
/api/v1/alerts/{id}/events               # Alert trigger history
```

**Rationale**:
- **Domain-driven**: Matches how users think about weather data
- **Flexible querying**: Allows both station-specific and cross-station queries
- **Clear relationships**: Parent-child relationships are explicit
- **Future growth**: Easy to add new resource types

### 4. Query Parameter Standards

**Decision**: Use consistent query parameter patterns across all endpoints.

**Standard Parameters**:
```
# Temporal
?start=2024-01-01T00:00:00Z      # ISO 8601 timestamps
?end=2024-01-02T00:00:00Z
?duration=PT1H                   # ISO 8601 duration
?timezone=America/New_York       # IANA timezone

# Pagination
?limit=100                       # Max records (default: 100, max: 1000)
?offset=0                        # Skip records
?cursor=eyJpZCI6MTIzfQ==         # Cursor-based pagination for large sets

# Filtering
?fields=temperature,humidity     # Sparse fieldsets
?station_id=ws1,ws2              # Multiple values
?min_temp=0&max_temp=30          # Range queries

# Sorting
?sort=-timestamp,temperature     # - prefix for descending

# Format
?format=json                     # Response format (json default)
```

**Rationale**:
- **Consistency**: Same patterns across all endpoints
- **Standards-based**: ISO 8601 for times, IANA for timezones
- **Flexible**: Supports both offset and cursor pagination
- **Performance**: Field filtering reduces payload size

### 5. Response Format Standards

**Decision**: Use consistent JSON response envelopes with proper status codes.

**Success Response Structure**:
```json
{
  "data": {...},                 // or [...] for collections
  "meta": {
    "timestamp": "2024-01-01T12:00:00Z",
    "version": "1.0",
    "count": 100,
    "total": 1000
  },
  "links": {
    "self": "/api/v1/stations/ws1",
    "next": "/api/v1/stations/ws1?cursor=abc",
    "prev": "/api/v1/stations/ws1?cursor=xyz"
  }
}
```

**Error Response Structure**:
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid request parameters",
    "details": [
      {
        "field": "start_date",
        "issue": "Invalid ISO 8601 format"
      }
    ],
    "request_id": "req_abc123",
    "timestamp": "2024-01-01T12:00:00Z"
  }
}
```

**Rationale**:
- **Consistent structure**: Clients always know where to find data
- **Metadata support**: Additional context without polluting data
- **HATEOAS support**: Links for discoverability
- **Debugging support**: Request IDs for tracing issues
- **Machine-readable errors**: Structured error details for better client handling

### 6. HTTP Status Code Usage

**Decision**: Use semantic HTTP status codes consistently.

**Status Code Mapping**:
- `200 OK` - Successful GET, PUT
- `201 Created` - Successful POST creating resource
- `204 No Content` - Successful DELETE
- `400 Bad Request` - Client error (validation, malformed)
- `401 Unauthorized` - Missing or invalid authentication
- `403 Forbidden` - Authenticated but not authorized
- `404 Not Found` - Resource doesn't exist
- `409 Conflict` - State conflict (e.g., duplicate)
- `429 Too Many Requests` - Rate limit exceeded
- `500 Internal Server Error` - Server error
- `503 Service Unavailable` - Temporary unavailability

**Rationale**:
- **HTTP semantics**: Proper use of HTTP protocol
- **Client behavior**: Status codes drive client retry logic
- **Monitoring**: Easy to track error rates by status code

### 7. Content Negotiation

**Decision**: Support content negotiation via Accept headers with JSON as default.

**Supported Content Types**:
- `application/json` - Default, always supported
- `application/x-msgpack` - Binary format for bandwidth-sensitive clients
- `text/csv` - For data export use cases

**Implementation**:
```
Accept: application/json         # JSON response
Accept: application/x-msgpack    # MessagePack response
Accept: text/csv                 # CSV export
```

**Rationale**:
- **Flexibility**: Different clients have different needs
- **Performance**: MessagePack for IoT devices with limited bandwidth
- **Export support**: CSV for data analysis tools
- **Future extensibility**: Easy to add new formats

### 8. API Documentation

**Decision**: Generate documentation from OpenAPI specification.

**Documentation Requirements**:
- OpenAPI 3.0 specification as source of truth
- Auto-generated interactive documentation (Swagger UI)
- Code examples in multiple languages
- Versioned documentation
- Change logs for each version

**Rationale**:
- **Single source of truth**: Documentation stays synchronized with implementation
- **Interactive testing**: Developers can test API directly from documentation
- **Client generation**: OpenAPI enables client SDK generation
- **Industry standard**: Widely supported by tools and platforms

## Consequences

### Positive

- Clear, predictable API structure reduces learning curve
- Version in URL makes debugging and support easier
- Consistent patterns reduce client implementation complexity
- OpenAPI specification enables excellent tooling
- Standard HTTP semantics work with existing infrastructure
- Flexible query parameters support diverse use cases

### Negative

- URL versioning requires maintaining multiple endpoints
- JSON default may be verbose for some use cases
- REST limitations for complex queries (vs. GraphQL)
- Envelope format adds slight overhead to responses

### Risks

- Major version changes require client updates
- Complex aggregation queries may not fit REST paradigm well
- Rate limiting strategy needs careful tuning

## Notes

This ADR establishes the API design principles. Implementation details will be covered in:
- API authentication and authorization mechanisms
- Rate limiting and quota management
- WebSocket endpoint design for real-time updates
- Batch operation support if needed

## Examples

### Example API Calls

**Get current conditions:**
```
GET /api/v1/stations/ws1/current
```

**Query historical data with aggregation:**
```
GET /api/v1/measurements?
  stations=ws1,ws2&
  start=2024-01-01T00:00:00Z&
  end=2024-01-01T23:59:59Z&
  interval=1h&
  aggregation=avg&
  fields=temperature,humidity
```

**Create alert configuration:**
```
POST /api/v1/alerts
Content-Type: application/json

{
  "name": "Freeze Warning",
  "station_id": "ws1",
  "condition": "temperature < 0",
  "action": "mqtt_publish",
  "target": "weather/alerts/freeze"
}
```
