# Technical Documentation - TRusTY

> **Version**: 0.8.2-SNAPSHOT
> **Last update**: 2025-11-30
> **Architecture**: Domain-Driven Design (DDD) + Clean Architecture

TRusTY is an OpenID Connect (OIDC) provider implementing FAPI 2.0 security standards, built with Rust and Python.

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Server - Rust (Port 8081)](#server---rust-port-8081)
3. [Client - Python Flask (Port 5001/5002)](#client---python-flask-port-50015002)
4. [Technology Stack](#technology-stack)
5. [Database & Persistence](#database--persistence)
6. [Security Features](#security-features)
7. [Deployment](#deployment)

---

## Architecture Overview

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      TRusTY Ecosystem                        │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────────┐           ┌──────────────────┐        │
│  │  Python Client   │           │  Python Client   │        │
│  │  (Standard OIDC) │           │   (FAPI 2.0)     │        │
│  │  Port 5001       │           │   Port 5002      │        │
│  └────────┬─────────┘           └─────────┬────────┘        │
│           │                               │                  │
│           │    HTTP/HTTPS                 │                  │
│           └───────────────┬───────────────┘                  │
│                           │                                   │
│                  ┌────────▼────────┐                         │
│                  │  TRusTY Server  │                         │
│                  │  (Rust + Axum)  │                         │
│                  │  Port 8081      │                         │
│                  └────────┬────────┘                         │
│                           │                                   │
│                  ┌────────▼────────┐                         │
│                  │  SQLite         │                         │
│                  │  trusty.db      │                         │
│                  └─────────────────┘                         │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

### Design Principles

- **Domain-Driven Design (DDD)**: Clear separation of business logic and infrastructure
- **Clean Architecture**: Dependencies point inward (Domain ← Application ← Infrastructure ← Presentation)
- **SOLID Principles**: Single Responsibility, Dependency Inversion, etc.
- **Test-Driven Development**: Comprehensive unit and integration tests

---

## Server - Rust (Port 8081)

### Project Structure

```
server/
├── src/
│   ├── main.rs                    # Entry point, server configuration
│   ├── lib.rs                     # Library exports
│   ├── config.rs                  # Configuration management
│   └── oidc/                      # OIDC module (DDD layers)
│       ├── mod.rs
│       ├── domain/                # Domain Layer (business logic)
│       │   ├── entities/          # Aggregate roots & entities
│       │   │   ├── user.rs
│       │   │   ├── client.rs
│       │   │   ├── session.rs
│       │   │   ├── authorization_code.rs
│       │   │   └── authorization_session.rs
│       │   ├── repositories/      # Repository interfaces (traits)
│       │   │   ├── user_repository.rs
│       │   │   ├── client_repository.rs
│       │   │   ├── session_repository.rs
│       │   │   └── authorization_code_repository.rs
│       │   ├── services/          # Domain services
│       │   │   ├── password_service.rs
│       │   │   └── dpop_service.rs
│       │   └── value_objects/     # Value objects (immutable)
│       │       ├── claims.rs
│       │       ├── scope.rs
│       │       └── jwt_request.rs
│       ├── application/           # Application Layer (use cases)
│       │   ├── commands/          # Write operations
│       │   │   └── authorize.rs
│       │   ├── queries/           # Read operations
│       │   └── services/          # Application services
│       │       ├── token_service.rs
│       │       ├── user_service.rs
│       │       ├── authorization_code_service.rs
│       │       └── session_service.rs
│       ├── infrastructure/        # Infrastructure Layer (technical)
│       │   ├── repositories/      # Repository implementations
│       │   │   ├── sqlite/        # SQLite implementations
│       │   │   │   ├── user_repository.rs
│       │   │   │   ├── client_repository.rs
│       │   │   │   ├── session_repository.rs
│       │   │   │   └── mappers.rs
│       │   │   └── memory/        # In-memory (for tests)
│       │   │       └── ...
│       │   └── services/          # Infrastructure services
│       │       ├── jwt_service.rs
│       │       ├── jwks_service.rs
│       │       ├── dpop_validator.rs
│       │       ├── par_service.rs
│       │       ├── i18n_service.rs
│       │       └── audit_service.rs
│       └── presentation/          # Presentation Layer (HTTP/API)
│           ├── handlers/          # HTTP endpoint handlers
│           │   ├── auth.rs        # /auth
│           │   ├── token.rs       # /token
│           │   ├── userinfo.rs    # /userinfo
│           │   ├── par.rs         # /par
│           │   ├── login.rs       # /login
│           │   ├── logout.rs      # /logout
│           │   ├── revoke.rs      # /revoke
│           │   ├── introspect.rs  # /introspect
│           │   ├── discovery.rs   # /.well-known/openid-configuration
│           │   ├── jwks.rs        # /.well-known/jwks.json
│           │   ├── health.rs      # /health
│           │   └── observability.rs  # /observability/*
│           ├── middleware/        # HTTP middleware
│           │   └── tracing.rs
│           ├── responses/         # Response DTOs
│           └── validation/        # Input validation
├── migrations/                    # SQLite migrations
│   ├── 001_initial_schema.sql
│   ├── 002_demo_data.sql
│   ├── 003_audit_and_revoked_tokens.sql
│   ├── 004_add_token_endpoint_auth_method.sql
│   ├── 005_add_post_logout_redirect_uris.sql
│   └── 006_update_fapi_client_jwks.sql
├── templates/                     # Handlebars HTML templates
│   ├── login.hbs
│   ├── logout_confirmation.hbs
│   └── error.hbs
├── locales/                       # i18n translations (Fluent)
│   ├── en/
│   │   └── main.ftl
│   └── fr/
│       └── main.ftl
├── data/                          # SQLite database
│   └── trusty.db
├── logs/                          # Application logs
├── Cargo.toml                     # Rust dependencies
└── build.rs                       # Build script (version extraction)
```

### Domain Layer

**Entities** (Aggregate Roots):
- `User` - User accounts with credentials
- `Client` - OIDC clients (apps requesting authentication)
- `Session` - User sessions
- `AuthorizationCode` - Short-lived codes for token exchange
- `AuthorizationSession` - Authorization request tracking

**Value Objects**:
- `Claims` - JWT claims
- `Scope` - Authorization scopes
- `UserId`, `ClientId`, `SessionId` - Typed IDs

**Domain Services**:
- `PasswordService` - Password hashing/verification (bcrypt)
- `DpopService` - DPoP token binding logic

### Application Layer

**Services** (orchestrate use cases):
- `TokenService` - Token generation/validation
- `UserService` - User claims creation
- `AuthorizationCodeService` - Auth code lifecycle
- `SessionService` - Session management

### Infrastructure Layer

**Repositories** (data access):
- SQLite implementations (`sqlx` crate)
- In-memory implementations (for testing)
- Factory pattern for backend selection (config-driven)

**Services** (technical implementations):
- `JwtService` - JWT sign/verify (RS256)
- `JwksService` - Public key management
- `DpopValidator` - DPoP proof validation (RFC 9449)
- `ParService` - PAR request storage (in-memory, 90s TTL)
- `I18nService` - Internationalization (Fluent.rs)
- `AuditService` - Audit logging to SQLite

### Presentation Layer

**Handlers** (HTTP endpoints):
- RESTful API endpoints
- Form handling (login, logout)
- Error responses (JSON/HTML)

---

## Client - Python Flask (Port 5001/5002)

### Demo Client Types

**Standard OIDC Client** (`app.py` - Port 5001):
- Authorization Code Flow
- PKCE (S256)
- client_secret_post authentication
- Session-based state management

**FAPI 2.0 Client** (`fapi_client.py` - Port 5002):
- PAR (Pushed Authorization Request)
- PKCE (S256 - mandatory)
- private_key_jwt authentication
- DPoP token binding
- Ephemeral DPoP keys per session

### Client Structure

```
client/
├── app.py                     # Standard OIDC client (Flask)
├── fapi_client.py             # FAPI 2.0 client (Flask)
├── generate_fapi_keys.py      # RSA key generation for FAPI
├── export_fapi_key_env.py     # Export keys to env vars
├── templates/                 # Jinja2 HTML templates
│   ├── index.html
│   ├── profile.html
│   └── error.html
├── locales/                   # i18n translations (Flask-Babel)
│   ├── en/
│   └── fr/
├── static/                    # CSS, JS, images
├── requirements.txt           # Python dependencies
├── .env.example               # Configuration template
└── README.md                  # Client documentation
```

### Python Dependencies

```
Flask==3.1.0
Flask-Session==0.8.0
requests==2.32.3
PyJWT==2.9.0
cryptography==43.0.3
Authlib==1.3.2
python-dotenv==1.0.1
Flask-Babel==4.0.0
```

---

## Technology Stack

### Server (Rust)

| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| **Web Framework** | Axum | 0.8 | Async HTTP server |
| **Runtime** | Tokio | 1.0 | Async runtime |
| **Database** | SQLite | - | Embedded database |
| **ORM** | sqlx | 0.8 | Async SQL toolkit |
| **JWT** | jsonwebtoken | 9.0 | JWT sign/verify |
| **Crypto** | rsa, rand | 0.9 | RSA keys, random |
| **Password** | bcrypt | 0.15 | Secure hashing |
| **Serialization** | serde, serde_json | 1.0 | JSON/YAML parsing |
| **Templates** | handlebars | 4.0 | HTML rendering |
| **i18n** | fluent-templates | 0.11 | Translations |
| **Logging** | tracing | 0.1 | Structured logging |
| **Telemetry** | OpenTelemetry | 0.24 | Distributed tracing |
| **Sessions** | tower-sessions | 0.12 | Session management |
| **Config** | serde_yaml | 0.9 | YAML configuration |

### Client (Python)

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Web Framework** | Flask | HTTP server |
| **HTTP Client** | requests, Authlib | OIDC client |
| **JWT** | PyJWT | Token parsing |
| **Crypto** | cryptography | RSA keys, DPoP |
| **i18n** | Flask-Babel | Translations |
| **Sessions** | Flask-Session | Session storage |

---

## Database & Persistence

### SQLite Schema

**Tables**:
- `users` - User accounts (id UUID, username, email, password_hash, created_at)
- `clients` - OIDC clients (id, client_secret, redirect_uris, jwks, auth_method, etc.)
- `sessions` - User sessions (id UUID, user_id FK, created_at, expires_at, is_active)
- `authorization_codes` - Auth codes (code, user_id FK, client_id FK, redirect_uri, scope, code_challenge, created_at, expires_at, used)
- `par_requests` - PAR requests (request_uri, client_id, params JSON, created_at, expires_at)
- `audit_log` - Audit events (event_type, user_id, client_id, ip_address, user_agent, details JSON, timestamp)
- `revoked_tokens` - Revoked token JTIs (jti, token_type, revoked_at, expires_at)

### Migration System

- SQL migration files in `server/migrations/`
- Applied on server startup (idempotent)
- Version tracking in `_migrations` table (future)

### Backend Selection

Configurable via `config.yaml`:
```yaml
database:
  backend: Sqlite   # or InMemory (for tests)
  path: "server/data/trusty.db"
```

Repository factory pattern allows easy addition of PostgreSQL, MySQL, etc.

---

## Security Features

### Implemented Standards

| Feature | Specification | Status |
|---------|--------------|--------|
| **OIDC Core 1.0** | OpenID Connect Core | ✅ Implemented |
| **PKCE (S256)** | RFC 7636 | ✅ Mandatory |
| **PAR** | RFC 9126 | ✅ Implemented |
| **private_key_jwt** | RFC 7523 | ✅ Implemented |
| **DPoP** | RFC 9449 | ✅ Implemented |
| **FAPI 2.0** | FAPI 2.0 Security Profile | ✅ Implemented |
| **RP-Initiated Logout** | OIDC RP-Initiated Logout 1.0 | ✅ Implemented |
| **Token Revocation** | RFC 7009 | ✅ Implemented |
| **Token Introspection** | RFC 7662 | ✅ Implemented |

### Security Mechanisms

**Authentication**:
- bcrypt password hashing (cost 12)
- Asymmetric client authentication (RS256 JWT)
- Session-based authentication for UI

**Token Security**:
- RS256 signature (2048-bit keys)
- Short-lived tokens (access: 1h, auth code: 90s)
- Token binding with DPoP (jkt validation)
- Token revocation tracking

**Request Security**:
- PKCE (S256 only, plain rejected)
- State parameter (CSRF protection)
- Nonce (replay protection)
- PAR (parameter tampering protection)
- Signed JWT request objects

**Audit**:
- All authentication attempts logged
- Token operations tracked
- IP address + User-Agent captured

---

## Deployment

### Docker Support

**Dockerfile** (Rust server):
```dockerfile
FROM rust:1.83-alpine AS builder
# Build static binary
FROM alpine:latest
# FHS-compliant structure: /opt/trusty/
```

**docker-compose.yml**:
```yaml
services:
  trusty-server:
    build: ./server
    ports:
      - "8081:8081"
    volumes:
      - ./server/data:/opt/trusty/data
      - ./server/logs:/opt/trusty/logs
    environment:
      - RUST_LOG=info

  trusty-client:
    build: ./client
    ports:
      - "5001:5001"
    environment:
      - OIDC_ISSUER=http://trusty-server:8081
```

### Railway Deployment

**CI/CD with GitHub Actions**:
- `main` branch → Production environment
- `dev` branch → Development environment
- PR → Tests only (no deployment)

**Environment Variables**:
- `BUILD_VERSION` - Git tag or SNAPSHOT-{timestamp}
- `RUST_LOG` - Log level
- `DATABASE_PATH` - SQLite file path
- `ISSUER_URL` - OIDC issuer URL

### File Structure (FHS-compliant)

```
/opt/trusty/
├── bin/
│   ├── trusty              # Binary
│   └── start.sh            # Startup script
├── etc/
│   └── config.yaml         # Configuration
├── data/
│   └── trusty.db           # SQLite database (volume)
├── logs/                   # Logs (volume)
└── migrations/             # SQL migrations
```

---

## Configuration

### Server Config (`server/config.yaml`)

```yaml
server:
  host: "0.0.0.0"
  port: 8081
  issuer: "http://localhost:8081"

database:
  backend: Sqlite
  path: "server/data/trusty.db"

jwt:
  private_key_path: "server/keys/jwt_private_key.pem"
  public_key_path: "server/keys/jwt_public_key.pem"
  algorithm: "RS256"
  access_token_ttl: 3600      # 1 hour
  id_token_ttl: 3600          # 1 hour
  refresh_token_ttl: 2592000  # 30 days

logging:
  level: "info"
  directory: "server/logs"
  file_prefix: "trusty"

i18n:
  default_locale: "en"
  supported_locales: ["en", "fr"]
```

### Client Config (`client/.env`)

```bash
# OIDC Server
OIDC_ISSUER=http://localhost:8081
CLIENT_ID=demo_client
CLIENT_SECRET=demo_secret
REDIRECT_URI=http://localhost:5001/auth/callback

# FAPI Client (port 5002)
FAPI_CLIENT_ID=fapi_client
FAPI_REDIRECT_URI=http://localhost:5002/auth/callback
FAPI_PRIVATE_KEY_PATH=fapi_client_private_key.pem
FAPI_JWKS_PATH=fapi_client_public_jwks.json
```

---

## Development

### Prerequisites

**Server**:
- Rust 1.83+
- SQLite3
- OpenSSL

**Client**:
- Python 3.11+
- pip

### Setup

```bash
# Server
cd server
cargo build --release
cargo run

# Client (Standard OIDC)
cd client
python -m venv venv
source venv/bin/activate  # or venv\Scripts\activate on Windows
pip install -r requirements.txt
python app.py

# Client (FAPI 2.0)
python generate_fapi_keys.py
python fapi_client.py
```

### Testing

```bash
# Server tests
cd server
cargo test

# Integration tests
cargo test --test '*'

# Code coverage
cargo tarpaulin --out Html
```

---

## API Endpoints

See [OIDC_FLOW.en.md](OIDC_FLOW.en.md) for detailed endpoint documentation.

**Quick Reference**:
- `GET /.well-known/openid-configuration` - OIDC Discovery
- `GET /.well-known/jwks.json` - Public keys (JWKS)
- `GET /auth` - Authorization endpoint
- `POST /token` - Token endpoint
- `GET /userinfo` - UserInfo endpoint
- `POST /par` - Pushed Authorization Request
- `POST /revoke` - Token revocation
- `POST /introspect` - Token introspection
- `GET /logout` - RP-Initiated Logout
- `GET /health` - Health check

---

## Monitoring & Observability

### Logging

- Structured logging with `tracing`
- JSON format for production
- File rotation (daily logs)
- Log levels: TRACE, DEBUG, INFO, WARN, ERROR

### Metrics (Observability Endpoints)

- `GET /observability/sessions` - Active sessions count
- `GET /observability/tokens` - Token statistics
- `GET /observability/clients` - Client info

### OpenTelemetry (Future)

- Distributed tracing support
- OTLP exporter configuration
- Jaeger/Zipkin integration ready

---

## References

- [OIDC Flow Documentation](OIDC_FLOW.en.md)
- [Server README](../server/README.md)
- [Client README](../client/README.md)

---

**Document Version**: 2.0
**Status**: Production-ready
**Last Review**: 2025-11-30
