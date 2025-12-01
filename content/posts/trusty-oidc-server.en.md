---
title: "TRusTY: Building a Rust OIDC Server with FAPI 2.0"
date: 2025-11-30T10:00:00+02:00
lastmod: 2025-11-30T10:00:00+02:00
draft: true
weight: 1
tags: ["Rust", "OIDC", "OAuth2", "FAPI-2.0", "DPoP", "PAR", "Security", "Access Management", "Axum", "SQLite", "JWT"]
categories: ["Security", "Rust", "Access Management"]
author: "lostyzen"
showToc: true
TocOpen: false
hidemeta: false
comments: true
description: "Experience report on building a FAPI 2.0-compliant OpenID Connect server in Rust: DDD architecture, secure flow implementation, and comprehensive technical documentation."
canonicalURL: "https://lostyzen.github.io/en/posts/trusty-oidc-server/"
disableHLJS: false
disableShare: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "/images/trusty-oidc-og.png"
    alt: "TRusTY OIDC Server in Rust"
    caption: "Building a modern OIDC server with Rust"
    relative: false
    hidden: false
editPost:
    URL: "https://github.com/lostyzen/lostyzen.github.io/tree/main/content"
    Text: "Suggest changes"
    appendFilePath: true
keywords: ["rust", "oidc", "openid connect", "oauth2", "fapi 2.0", "dpop", "par", "security", "access management", "axum", "authentication", "authorization", "jwt", "private_key_jwt"]
schema:
  type: "BlogPosting"
  datePublished: "2025-11-30T10:00:00+02:00"
  dateModified: "2025-11-30T10:00:00+02:00"
  author: "lostyzen"
  publisher: "DevOps Blog"
images: ["/images/trusty-oidc-og.png"]
---

🔐 **TRusTY: Building a Rust OIDC Server with FAPI 2.0**

After exploring Rust for web development, I wanted to go further by tackling an ambitious technical challenge: **building a complete OpenID Connect (OIDC) provider**, compliant with the FAPI 2.0 security specifications. Not just an academic prototype - a real authentication server with the most demanding security standards from the financial sector.

## 🎯 **Why build an OIDC server in Rust?**

In the Access Management world, we typically use established solutions: **Keycloak**, **Auth0**, **Okta**, or **ForgeRock**. These tools do the job, but I've always been frustrated by certain aspects:

• **Configuration complexity**: Hundreds of parameters, heavy interfaces, steep learning curve  
• **Resource consumption**: 500 MB - 1 GB RAM for Keycloak, significant impact in cloud environments  
• **Flow opacity**: Hard to understand what happens under the hood when a flow fails  
• **Limited extensibility**: Customization via plugins/SPIs often complex and poorly documented  
• **Vendor lock-in**: Difficult migration between proprietary solutions  

The idea behind TRusTY? **Take back control** by building a modern, performant, and understandable OIDC server. Rust offers exactly what's needed: memory safety, native performance, and a mature web ecosystem with **Axum**.

## 🚀 **Architecture: DDD + Clean Architecture**

### **Layered separation**

Rather than piling code into monolithic controllers, I structured the project following **Domain-Driven Design** and **Clean Architecture** principles:

```
┌──────────────────────────────────────────────────┐
│           Presentation Layer (HTTP)              │
│  Handlers: /auth, /token, /userinfo, /par...    │
└─────────────────┬────────────────────────────────┘
                  │
┌─────────────────▼────────────────────────────────┐
│        Application Layer (Use Cases)             │
│  Services: TokenService, AuthCodeService...      │
└─────────────────┬────────────────────────────────┘
                  │
┌─────────────────▼────────────────────────────────┐
│        Domain Layer (Business Logic)             │
│  Entities: User, Client, AuthorizationCode       │
│  Value Objects: Claims, Scope, JwtRequest        │
└─────────────────┬────────────────────────────────┘
                  │
┌─────────────────▼────────────────────────────────┐
│      Infrastructure Layer (Technical)            │
│  Repositories (SQLite), JWT, DPoP, PAR...        │
└──────────────────────────────────────────────────┘
```

**Why this organization?**
✅ **Isolated tests**: Business logic testable without starting the server  
✅ **Maintainability**: Clear responsibilities, loose coupling  
✅ **Scalability**: Easy to add PostgreSQL, Redis, or other backends  
✅ **Understanding**: Architecture reflects OIDC business concepts  

### **Domain entities**

The system core relies on well-defined business entities:

- **`User`**: Represents a user with credentials (bcrypt) and claims
- **`Client`**: Registered client application (redirect_uris, JWKS, auth methods...)
- **`Session`**: User session with configurable lifetime
- **`AuthorizationCode`**: Ephemeral code (90s TTL) for Authorization Code Flow exchange
- **`AuthorizationSession`**: Authorization request context (state, nonce, PKCE...)

Each entity encapsulates **its own business logic** and **validation rules**. For example, `AuthorizationCode` automatically verifies:
- Expiration (90 seconds)
- Single use (no reuse)
- Binding to the correct client
- PKCE code_verifier validation

## 🔒 **FAPI 2.0: Financial-grade security**

### **Why FAPI 2.0?**

**FAPI** (Financial-grade API) is a security profile developed by the **OpenID Foundation** to meet banking and financial sector requirements. FAPI 2.0 (released 2023) goes even further with robust anti-theft and anti-replay mechanisms.

**Key implemented features**:

| Feature | RFC/Spec | Purpose |
|---------|----------|---------|
| **PKCE (S256)** | RFC 7636 | Prevent authorization code interception |
| **PAR** | RFC 9126 | Protect authorization parameters (server-side pre-registration) |
| **private_key_jwt** | RFC 7523 | Asymmetric client authentication (no shared secrets) |
| **DPoP** | RFC 9449 | Token binding to cryptographic key (anti-token theft) |
| **RP-Initiated Logout** | OIDC Logout | Client-initiated logout |
| **Token Revocation** | RFC 7009 | Explicit token revocation |
| **Token Introspection** | RFC 7662 | Token validation by Resource Servers |

### **DPoP in detail: the real innovation**

**The classic problem**: Bearer access tokens are like cash - **whoever has them can use them**. If an attacker intercepts the token (man-in-the-middle, malware, XSS...), they can use it freely until expiration.

**The DPoP solution (Demonstrating Proof-of-Possession)**:
1. The client generates an **ephemeral RSA key pair** at session start
2. With each token request, it signs a **cryptographic proof** (JWT) with its private key
3. The server binds the access token to the public key fingerprint (`cnf.jkt` in token)
4. To use the token, the client must **prove possession of the private key** with each API call

**Result**: Even if an attacker steals the token, they can't use it without the corresponding private key. The token becomes **unusable out of context**.

```rust
// Simplified DPoP validation (server-side)
pub async fn validate_dpop_proof(
    proof_jwt: &str,
    access_token: &str,
    http_method: &str,
    http_uri: &str,
) -> Result<Jwk, DpopError> {
    // 1. Parse and validate proof JWT signature
    let proof = decode_dpop_proof(proof_jwt)?;
    
    // 2. Verify htm (HTTP method) and htu (URI)
    if proof.htm != http_method || proof.htu != http_uri {
        return Err(DpopError::InvalidProof);
    }
    
    // 3. Verify token jkt matches proof public key
    let token_jkt = extract_jkt_from_token(access_token)?;
    let proof_jkt = calculate_jkt(&proof.jwk)?;
    
    if token_jkt != proof_jkt {
        return Err(DpopError::KeyMismatch);
    }
    
    Ok(proof.jwk)
}
```

### **PAR: Pushed Authorization Requests**

Another important innovation: **PAR** (RFC 9126) allows **pre-registering** authorization parameters server-side before redirecting the user.

**Classic flow (risky)**:
```
GET /auth?client_id=app&redirect_uri=https://evil.com&scope=admin&...
```
👉 Parameters visible in URL, modifiable by attacker

**PAR flow (secure)**:
```
1. POST /par (with all parameters + client signature)
   ← 201 Created { request_uri: "urn:ietf:params:oauth:request_uri:abc123" }

2. GET /auth?client_id=app&request_uri=urn:ietf:params:oauth:request_uri:abc123
```
👉 Parameters protected server-side, short and secure URL

**Advantages**:
- Guaranteed parameter integrity
- No URL length limit
- Automatic expiration (90 seconds)

## 🛠️ **Tech stack and implementation choices**

### **Why Axum for the server?**

After testing several Rust web frameworks (**Actix-web**, **Rocket**, **Warp**), I chose **Axum** for several reasons:

✅ **Ergonomics**: Elegant API with typed extractors, expressive routing  
✅ **Performance**: Built on **Tokio** and **Tower**, production-optimized  
✅ **Ecosystem**: Compatible with entire Tower ecosystem (middleware, rate limiting...)  
✅ **Type-safety**: Compilation guarantees route and handler consistency  
✅ **Maintenance**: Developed by Tokio team, long-term support assured  

### **SQLite: pragmatic choice**

For the MVP, I opted for **SQLite** over PostgreSQL/MySQL:

✅ **Zero-configuration**: No server to manage, embedded database  
✅ **Portable**: Single file, easy to backup and migrate  
✅ **Sufficient performance**: Up to 100k req/s reads, more than enough to start  
✅ **Dev ease**: Fast tests, no Docker required  

**PostgreSQL migration planned**: The Repository Pattern architecture allows easy switching to PostgreSQL for production (high availability, replication...).

### **Key dependencies**

```toml
[dependencies]
# Web framework
axum = "0.8"
tokio = { version = "1.0", features = ["full"] }
tower = "0.4"

# Database
sqlx = { version = "0.8", features = ["sqlite", "runtime-tokio"] }

# JWT and Crypto
jsonwebtoken = "9.0"
rsa = "0.9"
bcrypt = "0.15"

# Serialization
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"

# Templates and i18n
handlebars = "4.0"
fluent-templates = "0.11"

# Observability
tracing = "0.1"
opentelemetry = "0.24"
```

## 📊 **Implemented flows: full demonstration**

### **1. Standard OIDC Flow (Authorization Code + PKCE)**

The classic flow for a web application:

```
┌─────────┐              ┌──────────┐              ┌─────────┐
│ Client  │              │  TRusTY  │              │  User   │
└────┬────┘              └─────┬────┘              └────┬────┘
     │                         │                        │
     │  1. Redirect /auth      │                        │
     │────────────────────────>│                        │
     │  (client_id, PKCE...)   │                        │
     │                         │   2. Login page        │
     │                         │<───────────────────────│
     │                         │                        │
     │                         │   3. POST /login       │
     │                         │<───────────────────────│
     │                         │   (username, password) │
     │                         │                        │
     │  4. Callback with code  │                        │
     │<────────────────────────│                        │
     │                         │                        │
     │  5. POST /token         │                        │
     │────────────────────────>│                        │
     │  (code, code_verifier)  │                        │
     │                         │                        │
     │  6. access + id tokens  │                        │
     │<────────────────────────│                        │
```

### **2. FAPI 2.0 Flow with PAR + DPoP**

The secure flow for critical applications:

```
┌─────────┐              ┌──────────┐              
│ Client  │              │  TRusTY  │              
└────┬────┘              └─────┬────┘              
     │                         │                   
     │  1. POST /par           │                   
     │────────────────────────>│                   
     │  (client_assertion,     │                   
     │   PKCE, scope...)       │                   
     │                         │                   
     │  2. request_uri         │                   
     │<────────────────────────│                   
     │                         │                   
     │  3. Redirect /auth      │                   
     │────────────────────────>│                   
     │  (request_uri only)     │                   
     │                         │                   
     │  ... (login flow) ...   │                   
     │                         │                   
     │  4. POST /token         │                   
     │────────────────────────>│                   
     │  Header: DPoP <proof>   │                   
     │  Body: client_assertion,│                   
     │        code_verifier... │                   
     │                         │                   
     │  5. DPoP-bound tokens   │                   
     │<────────────────────────│                   
     │  (token_type: DPoP)     │                   
```

**Note**: Each API call with the token requires a **new signed DPoP proof**. Impossible to reuse an old proof (anti-replay protection via `jti` + temporal verification).

## 💡 **Demo client: testing in real conditions**

To validate the implementation, I developed **a single Python Flask client** offering **two authentication modes** selectable from the home page:

### **Standard OIDC Mode**
- Classic Authorization Code Flow
- PKCE S256
- `client_secret_post` authentication
- Flask session for state management
- Test account: standard user

### **Secure FAPI 2.0 Mode**
- Complete PAR flow
- `private_key_jwt` authentication with RSA keys
- DPoP proof generation on each request
- Ephemeral DPoP keys per session
- Test account: FAPI user

**🎯 Interactive demo available**: I deployed the server and client on **[Railway](https://railway.com)** - a great service offering native GitHub integration that allowed me to host the MVP for free for 1 month with their base plan. Perfect for getting started quickly without complex infrastructure!

**Available environments**:
- 🧪 **[DEV Demo](https://trusty-client-dev.up.railway.app)**: Development version with detailed logs
- 🚀 **[PROD Demo](https://trusty-client-prd.up.railway.app)**: Stable production version

**How to test**: Go to the home page, choose your mode (Standard or FAPI 2.0), and follow the complete flow. You'll see generated tokens, DPoP proofs (in FAPI mode), and can inspect JWT claims.

## 📚 **Comprehensive technical documentation**

The project includes detailed documentation. While waiting for the GitHub repository opening for v1.0 (H1 2026), technical documentation is **hosted directly on this blog**:

### **Architecture & Stack**
📄 **[Technical Documentation](/docs/trusty/technical-en/)**
- Complete DDD structure (Domain, Application, Infrastructure, Presentation)
- Technology stack with justifications
- SQLite database schemas
- Docker configuration and deployment

### **OIDC & FAPI 2.0 Flows**
📄 **[OIDC Flow Documentation](/docs/trusty/oidc-flow-en/)**
- Mermaid diagrams of all flows
- Request/response examples
- DPoP and PAR validation details
- Error handling and edge cases

### **Standards compliance**

| Specification | Status | Compliance level |
|--------------|--------|-----------------|
| **OpenID Connect Core 1.0** | ✅ Implemented | Complete Authorization Code Flow |
| **OAuth 2.0 PKCE (RFC 7636)** | ✅ Implemented | S256 only (plain rejected) |
| **PAR (RFC 9126)** | ✅ Implemented | 90s TTL, in-memory storage |
| **private_key_jwt (RFC 7523)** | ✅ Implemented | RS256, JWKS validation |
| **DPoP (RFC 9449)** | ✅ Implemented | jkt binding, htm/htu/ath validation |
| **FAPI 2.0 Security Profile** | ✅ Implemented | MVP subset (no Hybrid Flow) |
| **RP-Initiated Logout** | ✅ Implemented | With user confirmation |
| **Token Revocation (RFC 7009)** | ✅ Implemented | Access + Refresh tokens |
| **Token Introspection (RFC 7662)** | ✅ Implemented | Validation for Resource Servers |

## 🔮 **Roadmap: Next steps**

### **Version 0.9.0 (Q1 2026)**
- 🔄 **Refresh Token Flow**: Token renewal without re-authentication
- 🔑 **Full SAML Support**: Complete SAML 2.0 protocol implementation (IdP and SP)
- 🔐 **PostgreSQL Backend**: Migration for high availability
- 🧪 **Conformance Testing**: OpenID Certification test suite
- 📊 **Advanced Telemetry**: Prometheus metrics, distributed tracing

### **Version 1.0.0 (Q2-Q3 2026) - Ideas under consideration**

**Priority 1 - Certification and tooling**:
- ✅ **OpenID Certification**: Official compliance
- 🎛️ **Admin Console**: Management interface for OIDC provider settings and IdP/SP SAML configuration

**Priority 2 - Authentication journeys**:
- 🚶 **Authentication Journeys**: First baseline implementation with standard modules (to be determined - MFA, conditional access, step-up auth...)

**To explore based on priorities**:
- 🔑 **WebAuthn/FIDO2**: Passwordless authentication
- 📱 **CIBA (Client Initiated Backchannel Authentication)**: For mobile

⚠️ **Note**: These features are ideas under consideration and not yet fully committed. Development happens with my available resources and time (I have a day job! 😅), so the roadmap may evolve based on feedback and priorities.

### **Exploratory features**
- **Internal/External client distinction**: Differentiated management based on client type (not to be confused with Device Flow)
- **JAR (JWT Secured Authorization Request - RFC 9101)**: Encapsulation of authorization parameters in a signed JWT (alternative/complement to PAR)
- **mTLS (Mutual TLS)**: Mutual authentication at TLS level for highly secure clients
- **Rich Authorization Requests (RAR - RFC 9396)**: Allows requesting fine-grained and structured authorizations beyond simple scopes. Example: instead of `scope=read`, request `{"type":"account","actions":["read","transfer"],"limits":{"max_amount":1000}}`. Very useful for banking APIs where authorizations are complex.

## 🤝 **Need feedback from Access Management experts**

If you work in the **Identity & Access Management** field, your feedback would be valuable:

🔍 **Particular attention points**:
- **FAPI 2.0 flow compliance**: Are the DPoP implementations (proof generation/validation, jkt binding) and PAR (storage, TTL, client_assertion validation) compliant with specs? Are there uncovered edge cases?
- **Error handling and edge cases**: Are OAuth/OIDC error codes appropriate? How do you handle timing attacks, replays, expired codes in your implementations?
- **Token generation/storage security**: JWTs are RS256-signed with key rotation. Is authorization code and refresh token storage in SQLite secure enough for an MVP? What are your production practices?
- **Performance and scalability**: With SQLite and in-memory PAR storage, the system is limited to a single node. What are recommended patterns for migrating to distributed PostgreSQL + Redis without full refactoring?
- **Interoperability**: Have you tested integration with third-party Resource Servers (Spring Boot, .NET, Node.js)? What problems do you typically encounter with ID tokens and DPoP access tokens?

💬 **How to submit your feedback**:

For now, the project isn't publicly open yet (planned for v1.0 in H1 2026). Meanwhile, you can send me your feedback, questions, or recommendations by email:

📧 **Email**: [lostyzen@gmail.com](mailto:lostyzen@gmail.com)  
🔐 **PGP Key**: [Download from keys.openpgp.org](https://keys.openpgp.org/search?q=lostyzen%40gmail.com)  
📌 **Fingerprint**: `0205 9854 864D EE62 C2BB 455F D825 3521 EDD1 630B`

Feel free to:
- Test the demos (DEV/PROD) and report bugs/inconsistencies
- Challenge the technical documentation
- Propose specific business use cases from your sector
- Report deviations from OpenID/FAPI specifications

## 🔗 **Resources and useful links**

### **Official specifications**
- [OpenID Connect Core 1.0](https://openid.net/specs/openid-connect-core-1_0.html)
- [OAuth 2.0 PKCE (RFC 7636)](https://datatracker.ietf.org/doc/html/rfc7636)
- [Pushed Authorization Requests (RFC 9126)](https://datatracker.ietf.org/doc/html/rfc9126)
- [JWT Client Authentication (RFC 7523)](https://datatracker.ietf.org/doc/html/rfc7523)
- [DPoP (RFC 9449)](https://datatracker.ietf.org/doc/html/rfc9449)
- [FAPI 2.0 Security Profile](https://openid.net/specs/fapi-2_0-security-profile.html)

### **TRusTY Project**
- 🧪 **[DEV Demo](https://trusty-client-dev.up.railway.app)**: Development version with logs
- 🚀 **[PROD Demo](https://trusty-client-prd.up.railway.app)**: Stable production version
- 📖 **[Technical Documentation](/docs/trusty/technical-en/)**
- 📖 **[OIDC Flow Documentation](/docs/trusty/oidc-flow-en/)**
- 🏠 **GitHub Repository**: Opening planned with v1.0 (H1 2026)

## 🎉 **Conclusion**

Building an OIDC server from scratch in Rust is a serious technical challenge. But it's also an opportunity to **learn modern security mechanisms in depth** and **take back control** of your application authentication.

**TRusTY isn't production-ready yet** (it's an MVP!), but it already demonstrates that it's possible to build a performant, understandable OIDC server compliant with the most demanding standards.

**What you can do right now**:
- Test the demos (DEV/PROD) to see flows in action
- Browse the detailed technical documentation hosted on this blog
- Share your feedback and use cases by email

What's next? Continue implementing missing features, harden security, improve performance... and why not aim for **official OpenID certification** for v1.0!

**Questions? Suggestions?** Don't hesitate to contact me or comment below. Community exchanges are essential for evolving this kind of project.
