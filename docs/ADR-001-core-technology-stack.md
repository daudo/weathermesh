# Weathermesh - ADR-001: Core Technology Stack

## Status

Accepted

## Date

2025-09-11

## Context

We are building "weathermesh", a REST API to expose weather data collected by WeeWX. The current WeeWX architecture tightly couples data collection with presentation through its template/skin system, which violates modern software architecture principles of separation of concerns. 

The solution needs to:
- Provide a RESTful API for weather data access
- Support real-time data distribution via MQTT
- Enable multiple consumers (web frontends, IoT devices, home automation)
- Be maintainable and follow software engineering best practices
- Be fully open source for community distribution
- Support authentication for public internet exposure
- Scale from single-user home use to potentially multiple weather stations

## Decisions

### 1. Programming Language: Go

**Decision**: Use Go as the primary programming language for the API service.

**Rationale**:
- **Performance**: Go compiles to native code and offers performance comparable to C/C++, crucial for an API service
- **Deployment simplicity**: Produces single static binaries, ideal for containerization
- **Built-in concurrency**: Goroutines and channels make handling multiple MQTT subscriptions and HTTP requests straightforward
- **Memory efficiency**: Lower memory footprint compared to Python, important for resource-constrained environments
- **Strong typing**: Reduces runtime errors and improves code maintainability
- **Excellent standard library**: Native HTTP server, JSON handling, and database drivers

**Alternatives considered**:
- Python: Familiar to WeeWX community but slower performance and complex deployment (dependencies)
- Node.js: Rejected due to preference against JavaScript/TypeScript
- Rust: Better performance but steeper learning curve and smaller ecosystem

### 2. Cache Layer: Valkey

**Decision**: Use Valkey as the caching and pub/sub layer.

**Rationale**:
- **True open source**: Linux Foundation project, avoiding Redis licensing concerns
- **Drop-in Redis compatibility**: All Redis clients and tools work unchanged
- **Community backing**: Supported by major cloud providers (AWS, Google)
- **Proven technology**: Fork of battle-tested Redis 7.2.4
- **Multiple use cases**: Serves as cache, pub/sub broker, and session store

**Alternatives considered**:
- Redis: Licensing changed to source-available, not true open source
- KeyDB: Good performance but smaller community
- DragonflyDB: Excellent performance but BSL license could be problematic for OSS distribution

### 3. Message Broker: Eclipse Mosquitto

**Decision**: Use Mosquitto as the MQTT broker.

**Rationale**:
- **Lightweight**: Minimal resource usage, perfect for home servers
- **MQTT 5.0 support**: Latest protocol version with enhanced features
- **Battle-tested**: Widely deployed in IoT applications
- **Simple configuration**: Easy to set up and maintain
- **Bridge support**: Can federate with other MQTT brokers if needed
- **Active development**: Regular updates and security patches

**Alternatives considered**:
- EMQX: More features but significantly heavier resource usage
- RabbitMQ with MQTT plugin: Overkill for our use case
- VerneMQ: Good but unnecessary complexity for our needs

### 4. API Gateway: Traefik

**Decision**: Use Traefik as the reverse proxy and API gateway.

**Rationale**:
- **Container-native**: Designed for Docker/Kubernetes environments
- **Automatic TLS**: Let's Encrypt integration for free SSL certificates
- **Dynamic configuration**: Service discovery without restarts
- **Built-in middleware**: Rate limiting, authentication, CORS handling
- **Observability**: Native Prometheus metrics and tracing support
- **Modern architecture**: Built for microservices and API-first applications

**Alternatives considered**:
- Nginx: Would work but requires more manual configuration
- HAProxy: Excellent but lacks modern API gateway features
- Kong: Too complex for our needs

### 5. Metadata Storage: SQLite

**Decision**: Use a separate SQLite database for API-specific metadata.

**Rationale**:
- **Simplicity**: No additional database server to manage
- **Sufficient performance**: Handles thousands of reads/second, adequate for metadata
- **Easy backup**: Single file to backup/restore
- **Portability**: Database travels with the application
- **Low write volume**: API metadata changes infrequently
- **Migration path**: Easy to migrate to PostgreSQL if scaling requires it

**Alternatives considered**:
- PostgreSQL: Overkill for dozens of API keys and configurations
- Store in Valkey: Would work but less durable and harder to query
- Reuse WeeWX database: Would couple us to WeeWX internals

### 6. Container Orchestration: Docker/Podman Compose

**Decision**: Use Docker Compose (compatible with Podman) for container orchestration.

**Rationale**:
- **Simplicity**: Easy to understand and maintain
- **Sufficient for scale**: Handles our multi-service architecture well
- **Development friendly**: Same setup for development and production
- **Migration path**: Compose files can be converted to Kubernetes manifests
- **Podman compatibility**: Supports both Docker and Podman runtimes
- **Wide adoption**: Large community and extensive documentation

**Alternatives considered**:
- Kubernetes: Unnecessary complexity for our initial scale
- Docker Swarm: Abandoned by Docker Inc.
- Systemd units: Would work but less portable

### 7. Authentication Method: JWT with API Keys

**Decision**: Implement JWT-based authentication with API keys as the initial mechanism.

**Rationale**:
- **Stateless**: JWTs are self-contained, reducing server state
- **Standard**: Wide client library support
- **Flexible**: Can start with API keys, add OAuth2 later
- **Secure**: When properly implemented with refresh tokens
- **Microservices ready**: Works well in distributed systems
- **Migration path**: Can add full OAuth2 without breaking changes

**Alternatives considered**:
- Basic Auth: Too simple, requires server-side session state
- OAuth2 immediately: Adds complexity that may not be needed initially
- No authentication: Unacceptable for internet-exposed APIs

## Consequences

### Positive

- All components are truly open source, ensuring project can be freely distributed
- Architecture supports both simple home use and potential scaling
- Clean separation of concerns improves maintainability
- Technology choices are proven and have strong communities
- Container-based deployment simplifies installation for users
- Performance characteristics support real-time data distribution
- Clear migration paths exist for all components if scaling requires

### Negative

- Development team must learn Go and Traefik
- Multiple services increase operational complexity compared to monolithic WeeWX
- Initial setup more complex than traditional WeeWX skins
- Requires Docker/Podman knowledge for deployment
- More moving parts to monitor and debug

### Risks

- Go expertise needed for community contributions
- Valkey is relatively new (though based on proven Redis code)
- Multiple service coordination could introduce timing issues
- Container orchestration adds a layer of abstraction

## Notes

This ADR documents the foundational technology decisions. Subsequent ADRs will cover:
- ADR-002: API Design and Versioning Strategy
- ADR-003: Testing Strategy and TDD Approach
- ADR-004: MQTT Topic Structure and Event Schema
- ADR-005: Monitoring and Observability Strategy
