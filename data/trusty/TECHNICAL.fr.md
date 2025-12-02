# Documentation Technique - TRusTY

> **Version** : 0.8.2-SNAPSHOT
> **Dernière mise à jour** : 2025-11-30
> **Architecture** : Domain-Driven Design (DDD) + Clean Architecture

TRusTY est un fournisseur OpenID Connect (OIDC) implémentant les standards de sécurité FAPI 2.0, développé en Rust et Python.

---

## Table des Matières

1. [Vue d'Ensemble de l'Architecture](#vue-densemble-de-larchitecture)
2. [Serveur - Rust (Port 8081)](#serveur---rust-port-8081)
3. [Client - Python Flask (Port 5001/5002)](#client---python-flask-port-50015002)
4. [Stack Technologique](#stack-technologique)
5. [Base de Données & Persistance](#base-de-données--persistance)
6. [Fonctionnalités de Sécurité](#fonctionnalités-de-sécurité)
7. [Déploiement](#déploiement)

---

## Vue d'Ensemble de l'Architecture

### Architecture Haut Niveau

```
┌─────────────────────────────────────────────────────────────┐
│                    Écosystème TRusTY                         │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────────┐           ┌──────────────────┐        │
│  │  Client Python   │           │  Client Python   │        │
│  │  (OIDC Standard) │           │   (FAPI 2.0)     │        │
│  │  Port 5001       │           │   Port 5002      │        │
│  └────────┬─────────┘           └─────────┬────────┘        │
│           │                               │                  │
│           │    HTTP/HTTPS                 │                  │
│           └───────────────┬───────────────┘                  │
│                           │                                   │
│                  ┌────────▼────────┐                         │
│                  │ Serveur TRusTY  │                         │
│                  │  (Rust + Axum)  │                         │
│                  │  Port 8081      │                         │
│                  └────────┬────────┘                         │
│                           │                                   │
│                  ┌────────▼────────┐                         │
│                  │     SQLite      │                         │
│                  │   trusty.db     │                         │
│                  └─────────────────┘                         │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

### Principes de Conception

- **Domain-Driven Design (DDD)** : Séparation claire de la logique métier et de l'infrastructure
- **Clean Architecture** : Dépendances pointant vers l'intérieur (Domain ← Application ← Infrastructure ← Presentation)
- **Principes SOLID** : Responsabilité unique, Inversion de dépendances, etc.
- **Test-Driven Development** : Tests unitaires et d'intégration complets

---

## Serveur - Rust (Port 8081)

### Structure du Projet

```
server/
├── src/
│   ├── main.rs                    # Point d'entrée, configuration serveur
│   ├── lib.rs                     # Exports de bibliothèque
│   ├── config.rs                  # Gestion de la configuration
│   └── oidc/                      # Module OIDC (couches DDD)
│       ├── mod.rs
│       ├── domain/                # Couche Domaine (logique métier)
│       │   ├── entities/          # Racines d'agrégat & entités
│       │   │   ├── user.rs
│       │   │   ├── client.rs
│       │   │   ├── session.rs
│       │   │   ├── authorization_code.rs
│       │   │   └── authorization_session.rs
│       │   ├── repositories/      # Interfaces repository (traits)
│       │   │   ├── user_repository.rs
│       │   │   ├── client_repository.rs
│       │   │   ├── session_repository.rs
│       │   │   └── authorization_code_repository.rs
│       │   ├── services/          # Services domaine
│       │   │   ├── password_service.rs
│       │   │   └── dpop_service.rs
│       │   └── value_objects/     # Objets valeur (immuables)
│       │       ├── claims.rs
│       │       ├── scope.rs
│       │       └── jwt_request.rs
│       ├── application/           # Couche Application (cas d'usage)
│       │   ├── commands/          # Opérations d'écriture
│       │   │   └── authorize.rs
│       │   ├── queries/           # Opérations de lecture
│       │   └── services/          # Services application
│       │       ├── token_service.rs
│       │       ├── user_service.rs
│       │       ├── authorization_code_service.rs
│       │       └── session_service.rs
│       ├── infrastructure/        # Couche Infrastructure (technique)
│       │   ├── repositories/      # Implémentations repository
│       │   │   ├── sqlite/        # Implémentations SQLite
│       │   │   │   ├── user_repository.rs
│       │   │   │   ├── client_repository.rs
│       │   │   │   ├── session_repository.rs
│       │   │   │   └── mappers.rs
│       │   │   └── memory/        # En mémoire (pour tests)
│       │   │       └── ...
│       │   └── services/          # Services infrastructure
│       │       ├── jwt_service.rs
│       │       ├── jwks_service.rs
│       │       ├── dpop_validator.rs
│       │       ├── par_service.rs
│       │       ├── i18n_service.rs
│       │       └── audit_service.rs
│       └── presentation/          # Couche Présentation (HTTP/API)
│           ├── handlers/          # Gestionnaires d'endpoints HTTP
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
│           ├── middleware/        # Middleware HTTP
│           │   └── tracing.rs
│           ├── responses/         # DTOs de réponse
│           └── validation/        # Validation des entrées
├── migrations/                    # Migrations SQLite
│   ├── 001_initial_schema.sql
│   ├── 002_demo_data.sql
│   ├── 003_audit_and_revoked_tokens.sql
│   ├── 004_add_token_endpoint_auth_method.sql
│   ├── 005_add_post_logout_redirect_uris.sql
│   └── 006_update_fapi_client_jwks.sql
├── templates/                     # Templates HTML Handlebars
│   ├── login.hbs
│   ├── logout_confirmation.hbs
│   └── error.hbs
├── locales/                       # Traductions i18n (Fluent)
│   ├── en/
│   │   └── main.ftl
│   └── fr/
│       └── main.ftl
├── data/                          # Base de données SQLite
│   └── trusty.db
├── logs/                          # Logs d'application
├── Cargo.toml                     # Dépendances Rust
└── build.rs                       # Script de build (extraction version)
```

### Couche Domaine

**Entités** (Racines d'agrégat) :
- `User` - Comptes utilisateurs avec identifiants
- `Client` - Clients OIDC (applications demandant l'authentification)
- `Session` - Sessions utilisateur
- `AuthorizationCode` - Codes de courte durée pour échange de tokens
- `AuthorizationSession` - Suivi des requêtes d'autorisation

**Objets Valeur** :
- `Claims` - Claims JWT
- `Scope` - Scopes d'autorisation
- `UserId`, `ClientId`, `SessionId` - IDs typés

**Services Domaine** :
- `PasswordService` - Hachage/vérification mot de passe (bcrypt)
- `DpopService` - Logique de liaison de token DPoP

### Couche Application

**Services** (orchestrent les cas d'usage) :
- `TokenService` - Génération/validation de tokens
- `UserService` - Création de claims utilisateur
- `AuthorizationCodeService` - Cycle de vie du code d'autorisation
- `SessionService` - Gestion des sessions

### Couche Infrastructure

**Repositories** (accès aux données) :
- Implémentations SQLite (crate `sqlx`)
- Implémentations en mémoire (pour tests)
- Pattern Factory pour sélection du backend (configurable)

**Services** (implémentations techniques) :
- `JwtService` - Signature/vérification JWT (RS256)
- `JwksService` - Gestion des clés publiques
- `DpopValidator` - Validation des preuves DPoP (RFC 9449)
- `ParService` - Stockage des requêtes PAR (en mémoire, TTL 90s)
- `I18nService` - Internationalisation (Fluent.rs)
- `AuditService` - Journalisation d'audit vers SQLite

### Couche Présentation

**Handlers** (endpoints HTTP) :
- Endpoints API RESTful
- Gestion de formulaires (login, logout)
- Réponses d'erreur (JSON/HTML)

---

## Client - Python Flask (Port 5001/5002)

### Types de Clients Démo

**Client OIDC Standard** (`app.py` - Port 5001) :
- Authorization Code Flow
- PKCE (S256)
- Authentification client_secret_post
- Gestion d'état basée sur session

**Client FAPI 2.0** (`fapi_client.py` - Port 5002) :
- PAR (Pushed Authorization Request)
- PKCE (S256 - obligatoire)
- Authentification private_key_jwt
- Liaison de token DPoP
- Clés DPoP éphémères par session

### Structure du Client

```
client/
├── app.py                     # Client OIDC standard (Flask)
├── fapi_client.py             # Client FAPI 2.0 (Flask)
├── generate_fapi_keys.py      # Génération clés RSA pour FAPI
├── export_fapi_key_env.py     # Export clés vers variables d'env
├── templates/                 # Templates HTML Jinja2
│   ├── index.html
│   ├── profile.html
│   └── error.html
├── locales/                   # Traductions i18n (Flask-Babel)
│   ├── en/
│   └── fr/
├── static/                    # CSS, JS, images
├── requirements.txt           # Dépendances Python
├── .env.example               # Template de configuration
└── README.md                  # Documentation client
```

### Dépendances Python

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

## Stack Technologique

### Serveur (Rust)

| Composant | Technologie | Version | Objectif |
|-----------|------------|---------|----------|
| **Framework Web** | Axum | 0.8 | Serveur HTTP async |
| **Runtime** | Tokio | 1.0 | Runtime async |
| **Base de Données** | SQLite | - | Base de données embarquée |
| **ORM** | sqlx | 0.8 | Toolkit SQL async |
| **JWT** | jsonwebtoken | 9.0 | Signature/vérif JWT |
| **Crypto** | rsa, rand | 0.9 | Clés RSA, aléatoire |
| **Mots de passe** | bcrypt | 0.15 | Hachage sécurisé |
| **Sérialisation** | serde, serde_json | 1.0 | Parsing JSON/YAML |
| **Templates** | handlebars | 4.0 | Rendu HTML |
| **i18n** | fluent-templates | 0.11 | Traductions |
| **Logging** | tracing | 0.1 | Logging structuré |
| **Télémétrie** | OpenTelemetry | 0.24 | Tracing distribué |
| **Sessions** | tower-sessions | 0.12 | Gestion sessions |
| **Config** | serde_yaml | 0.9 | Configuration YAML |

### Client (Python)

| Composant | Technologie | Objectif |
|-----------|------------|----------|
| **Framework Web** | Flask | Serveur HTTP |
| **Client HTTP** | requests, Authlib | Client OIDC |
| **JWT** | PyJWT | Parsing de tokens |
| **Crypto** | cryptography | Clés RSA, DPoP |
| **i18n** | Flask-Babel | Traductions |
| **Sessions** | Flask-Session | Stockage session |

---

## Base de Données & Persistance

### Schéma SQLite

**Tables** :
- `users` - Comptes utilisateur (id UUID, username, email, password_hash, created_at)
- `clients` - Clients OIDC (id, client_secret, redirect_uris, jwks, auth_method, etc.)
- `sessions` - Sessions utilisateur (id UUID, user_id FK, created_at, expires_at, is_active)
- `authorization_codes` - Codes d'autorisation (code, user_id FK, client_id FK, redirect_uri, scope, code_challenge, created_at, expires_at, used)
- `par_requests` - Requêtes PAR (request_uri, client_id, params JSON, created_at, expires_at)
- `audit_log` - Événements d'audit (event_type, user_id, client_id, ip_address, user_agent, details JSON, timestamp)
- `revoked_tokens` - Tokens révoqués (jti, token_type, revoked_at, expires_at)

### Système de Migration

- Fichiers de migration SQL dans `server/migrations/`
- Appliqués au démarrage du serveur (idempotents)
- Suivi de version dans table `_migrations` (futur)

### Sélection du Backend

Configurable via `config.yaml` :
```yaml
database:
  backend: Sqlite   # ou InMemory (pour tests)
  path: "server/data/trusty.db"
```

Le pattern Repository Factory permet l'ajout facile de PostgreSQL, MySQL, etc.

---

## Fonctionnalités de Sécurité

### Standards Implémentés

| Fonctionnalité | Spécification | Statut |
|----------------|--------------|--------|
| **OIDC Core 1.0** | OpenID Connect Core | ✅ Implémenté |
| **PKCE (S256)** | RFC 7636 | ✅ Obligatoire |
| **PAR** | RFC 9126 | ✅ Implémenté |
| **private_key_jwt** | RFC 7523 | ✅ Implémenté |
| **DPoP** | RFC 9449 | ✅ Implémenté |
| **FAPI 2.0** | FAPI 2.0 Security Profile | ✅ Implémenté |
| **RP-Initiated Logout** | OIDC RP-Initiated Logout 1.0 | ✅ Implémenté |
| **Révocation Token** | RFC 7009 | ✅ Implémenté |
| **Introspection Token** | RFC 7662 | ✅ Implémenté |

### Mécanismes de Sécurité

**Authentification** :
- Hachage bcrypt des mots de passe (coût 12)
- Authentification client asymétrique (JWT RS256)
- Authentification basée session pour l'UI

**Sécurité des Tokens** :
- Signature RS256 (clés 2048-bit)
- Tokens de courte durée (access: 1h, auth code: 90s)
- Liaison de token avec DPoP (validation jkt)
- Suivi des révocations de tokens

**Sécurité des Requêtes** :
- PKCE (S256 uniquement, plain rejeté)
- Paramètre state (protection CSRF)
- Nonce (protection replay)
- PAR (protection altération paramètres)
- Objets de requête JWT signés

**Audit** :
- Toutes les tentatives d'authentification journalisées
- Opérations sur tokens suivies
- Adresse IP + User-Agent capturés

---

## Déploiement

### Support Docker

**Dockerfile** (serveur Rust) :
```dockerfile
FROM rust:1.83-alpine AS builder
# Construction binaire statique
FROM alpine:latest
# Structure conforme FHS: /opt/trusty/
```

**docker-compose.yml** :
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

### Déploiement Railway

**CI/CD avec GitHub Actions** :
- Branche `main` → Environnement Production
- Branche `dev` → Environnement Développement
- PR → Tests uniquement (pas de déploiement)

**Variables d'Environnement** :
- `BUILD_VERSION` - Tag git ou SNAPSHOT-{timestamp}
- `RUST_LOG` - Niveau de log
- `DATABASE_PATH` - Chemin fichier SQLite
- `ISSUER_URL` - URL de l'émetteur OIDC

### Structure Fichiers (Conforme FHS)

```
/opt/trusty/
├── bin/
│   ├── trusty              # Binaire
│   └── start.sh            # Script de démarrage
├── etc/
│   └── config.yaml         # Configuration
├── data/
│   └── trusty.db           # Base SQLite (volume)
├── logs/                   # Logs (volume)
└── migrations/             # Migrations SQL
```

---

## Configuration

### Config Serveur (`server/config.yaml`)

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
  access_token_ttl: 3600      # 1 heure
  id_token_ttl: 3600          # 1 heure
  refresh_token_ttl: 2592000  # 30 jours

logging:
  level: "info"
  directory: "server/logs"
  file_prefix: "trusty"

i18n:
  default_locale: "en"
  supported_locales: ["en", "fr"]
```

### Config Client (`client/.env`)

```bash
# Serveur OIDC
OIDC_ISSUER=http://localhost:8081
CLIENT_ID=demo_client
CLIENT_SECRET=demo_secret
REDIRECT_URI=http://localhost:5001/auth/callback

# Client FAPI (port 5002)
FAPI_CLIENT_ID=fapi_client
FAPI_REDIRECT_URI=http://localhost:5002/auth/callback
FAPI_PRIVATE_KEY_PATH=fapi_client_private_key.pem
FAPI_JWKS_PATH=fapi_client_public_jwks.json
```

---

## Développement

### Prérequis

**Serveur** :
- Rust 1.83+
- SQLite3
- OpenSSL

**Client** :
- Python 3.11+
- pip

### Installation

```bash
# Serveur
cd server
cargo build --release
cargo run

# Client (OIDC Standard)
cd client
python -m venv venv
source venv/bin/activate  # ou venv\Scripts\activate sur Windows
pip install -r requirements.txt
python app.py

# Client (FAPI 2.0)
python generate_fapi_keys.py
python fapi_client.py
```

### Tests

```bash
# Tests serveur
cd server
cargo test

# Tests d'intégration
cargo test --test '*'

# Couverture de code
cargo tarpaulin --out Html
```

---

## Endpoints API

Voir [OIDC_FLOW.fr.md](OIDC_FLOW.fr.md) pour la documentation détaillée des endpoints (à venir).

**Référence Rapide** :
- `GET /.well-known/openid-configuration` - Discovery OIDC
- `GET /.well-known/jwks.json` - Clés publiques (JWKS)
- `GET /auth` - Endpoint d'autorisation
- `POST /token` - Endpoint de token
- `GET /userinfo` - Endpoint UserInfo
- `POST /par` - Pushed Authorization Request
- `POST /revoke` - Révocation de token
- `POST /introspect` - Introspection de token
- `GET /logout` - Logout initié par RP
- `GET /health` - Vérification santé

---

## Monitoring & Observabilité

### Logging

- Logging structuré avec `tracing`
- Format JSON pour production
- Rotation de fichiers (logs quotidiens)
- Niveaux de log : TRACE, DEBUG, INFO, WARN, ERROR

### Métriques (Endpoints Observabilité)

- `GET /observability/sessions` - Nombre de sessions actives
- `GET /observability/tokens` - Statistiques de tokens
- `GET /observability/clients` - Informations clients

### OpenTelemetry (Futur)

- Support tracing distribué
- Configuration exportateur OTLP
- Intégration Jaeger/Zipkin prête

---

## Références

- [Documentation Flux OIDC](OIDC_FLOW.fr.md)
- [README Serveur](../server/README.md)
- [README Client](../client/README.md)

---

**Version du Document** : 2.0
**Statut** : Prêt pour production
**Dernière Révision** : 2025-11-30
