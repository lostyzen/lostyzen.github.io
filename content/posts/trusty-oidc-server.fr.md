---
title: "TRusTY : Développer un serveur OIDC en Rust avec FAPI 2.0"
date: 2025-12-01T10:00:00+02:00
lastmod: 2025-12-01T10:00:00+02:00
draft: false
weight: 1
tags: ["Rust", "OIDC", "OAuth2", "FAPI-2.0", "DPoP", "PAR", "Security", "Access Management", "Axum", "SQLite", "JWT"]
categories: ["Security", "Rust", "Access Management"]
author: "lostyzen"
showToc: true
TocOpen: false
hidemeta: false
comments: true
description: "Développer un serveur OIDC conforme FAPI 2.0 en Rust : architecture DDD, flux sécurisés et documentation technique."
canonicalURL: "https://lostyzen.github.io/posts/trusty-oidc-server/"
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
    alt: "TRusTY OIDC Server en Rust"
    caption: "Développer un serveur OIDC moderne avec Rust"
    relative: false
    hidden: false
editPost:
    URL: "https://github.com/lostyzen/lostyzen.github.io/tree/main/content"
    Text: "Suggérer des changements"
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

🔐 **TRusTY : Développer un serveur OIDC en Rust avec FAPI 2.0**

Après avoir exploré Rust pour le développement web (voir le [guide d'installation Rust sur Windows](/posts/installation-environnement-rust-windows/)), j'ai voulu aller plus loin en m'attaquant à un défi technique ambitieux : **construire un fournisseur OpenID Connect (OIDC) complet**, conforme aux spécifications de sécurité FAPI 2.0. Pas juste un prototype académique - un vrai serveur d'authentification avec les standards de sécurité les plus exigeants du secteur financier.

## 🎯 **Pourquoi développer un serveur OIDC en Rust ?**

### **L'origine : un challenge d'équipe**

Tout a commencé par un **défi lancé au sein de mon ancienne équipe DevOps**, responsable du SSO d'une grande banque française. Entre deux sprints, on s'est posé la question : "Et si on construisait notre propre serveur OIDC from scratch en Rust ?" Un challenge technique stimulant alliant **exploration technologique**, **optimisation des coûts d'infrastructure** et **maîtrise totale de la stack**.

### **Le constat terrain**

Dans le monde de l'Access Management, on utilise généralement des solutions établies : **Keycloak**, **Auth0**, **Okta**, ou encore **ForgeRock**. Ces outils sont matures, éprouvés en production, et couvrent un large spectre de cas d'usage. Cependant, après plusieurs années à les opérer en production, quelques **points de friction** sont apparus :

• **Complexité de configuration** : Des centaines de paramètres, des interfaces riches mais parfois lourdes - la courbe d'apprentissage peut être significative  
• **Consommation de ressources** : Certaines solutions Java peuvent nécessiter 500 MB à 1 GB de RAM par instance - un coût non négligeable en environnement cloud multiplié par les instances  
• **Opacité des flux** : Comprendre ce qui se passe sous le capot lors d'un échec d'authentification peut demander du temps de debug  
• **Extensibilité** : La customisation via plugins/SPIs est puissante mais nécessite de bien maîtriser les APIs internes  
• **Vendor lock-in** : La portabilité entre solutions peut s'avérer complexe selon les fonctionnalités utilisées  

L'idée de TRusTY ? **Reprendre le contrôle** en construisant un serveur OIDC moderne, performant et compréhensible. Un outil où chaque ligne de code est maîtrisée, où les flux sont transparents. Rust offre exactement ce qu'il faut : sécurité mémoire, performance native, et un écosystème web mature avec **Axum**.

## 🚀 **L'architecture : DDD + Clean Architecture**

### **Séparation en couches**

Plutôt que d'empiler du code dans des controllers monolithiques, j'ai structuré le projet selon les principes **Domain-Driven Design** et **Clean Architecture** :

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
│      Infrastructure Layer (Techniques)           │
│  Repositories (SQLite), JWT, DPoP, PAR...        │
└──────────────────────────────────────────────────┘
```

**Pourquoi cette organisation ?**
✅ **Tests isolés** : Logique métier testable sans démarrer le serveur  
✅ **Maintenabilité** : Responsabilités claires, couplage faible  
✅ **Évolutivité** : Facile d'ajouter PostgreSQL, Redis, ou d'autres backends  
✅ **Compréhension** : L'architecture reflète les concepts métier OIDC  

### **Les entités du domaine**

Le cœur du système repose sur des entités métier bien définies :

- **`User`** : Représente un utilisateur avec ses credentials (mot de passe hashé via bcrypt) et claims
- **`Client`** : Application cliente enregistrée (redirect_uris, JWKS, méthodes d'auth...)
- **`Session`** : Session utilisateur avec durée de vie configurable
- **`AuthorizationCode`** : Code éphémère (90s TTL) pour l'échange Authorization Code Flow
- **`AuthorizationSession`** : Contexte d'une requête d'autorisation (state, nonce, PKCE...)

Chaque entité encapsule **sa propre logique métier** et **ses règles de validation**. Par exemple, `AuthorizationCode` vérifie automatiquement :
- Expiration (90 secondes)
- Usage unique (pas de réutilisation)
- Liaison avec le bon client
- Validation PKCE code_verifier

## 🔒 **FAPI 2.0 : Sécurité de grade financier**

### **Pourquoi FAPI 2.0 ?**

**FAPI** (Financial-grade API) est un profil de sécurité développé par l'**OpenID Foundation** pour répondre aux exigences du secteur bancaire et financier. FAPI 2.0 (sorti en 2023) va encore plus loin avec des mécanismes anti-vol et anti-rejeu robustes.

📖 **Spécification complète** : [FAPI 2.0 Security Profile](https://openid.net/specs/fapi-security-profile-2_0.html)

**Les fonctionnalités clés implémentées** :

| Fonctionnalité | RFC/Spec | Objectif |
|----------------|----------|----------|
| **PKCE (S256)** | [RFC 7636](https://datatracker.ietf.org/doc/html/rfc7636) | Empêcher l'interception du code d'autorisation |
| **PAR** | [RFC 9126](https://datatracker.ietf.org/doc/html/rfc9126) | Protéger les paramètres d'autorisation (pré-enregistrement côté serveur) |
| **private_key_jwt** | [RFC 7523](https://datatracker.ietf.org/doc/html/rfc7523) | Authentification client asymétrique (pas de secrets partagés) |
| **DPoP** | [RFC 9449](https://datatracker.ietf.org/doc/html/rfc9449) | Liaison de token à une clé cryptographique (anti-vol de token) |
| **RP-Initiated Logout** | OIDC Logout | Déconnexion initiée par le client |
| **Token Revocation** | [RFC 7009](https://datatracker.ietf.org/doc/html/rfc7009) | Révocation explicite de tokens |
| **Token Introspection** | [RFC 7662](https://datatracker.ietf.org/doc/html/rfc7662) | Validation de tokens par des Resource Servers |

### **DPoP en détail : la vraie innovation**

**Le problème classique** : Les access tokens Bearer sont comme des billets de banque - **qui les possède peut les utiliser**. Si un attaquant intercepte le token (man-in-the-middle, malware, XSS...), il peut l'utiliser librement jusqu'à expiration.

**La solution DPoP (Demonstrating Proof-of-Possession)** :
1. Le client génère une **paire de clés RSA éphémère** au début de la session
2. À chaque requête token, il signe une **preuve cryptographique** (JWT) avec sa clé privée
3. Le serveur lie l'access token à l'empreinte de la clé publique (`cnf.jkt` dans le token)
4. Pour utiliser le token, le client doit **prouver possession de la clé privée** à chaque appel API

**Résultat** : Même si un attaquant vole le token, il ne peut pas l'utiliser sans la clé privée correspondante. Le token devient **inutilisable hors contexte**.

```rust
// Validation DPoP simplifié (côté serveur)
pub async fn validate_dpop_proof(
    proof_jwt: &str,
    access_token: &str,
    http_method: &str,
    http_uri: &str,
) -> Result<Jwk, DpopError> {
    // 1. Parser et valider la signature du proof JWT
    let proof = decode_dpop_proof(proof_jwt)?;
    
    // 2. Vérifier htm (méthode HTTP) et htu (URI)
    if proof.htm != http_method || proof.htu != http_uri {
        return Err(DpopError::InvalidProof);
    }
    
    // 3. Vérifier que le jkt du token correspond à la clé publique du proof
    let token_jkt = extract_jkt_from_token(access_token)?;
    let proof_jkt = calculate_jkt(&proof.jwk)?;
    
    if token_jkt != proof_jkt {
        return Err(DpopError::KeyMismatch);
    }
    
    Ok(proof.jwk)
}
```

### **PAR : Pushed Authorization Requests**

Autre innovation importante : **PAR** (RFC 9126) permet de **pré-enregistrer** les paramètres d'autorisation côté serveur avant de rediriger l'utilisateur.

**Flux classique (risqué)** :
```
GET /auth?client_id=app&redirect_uri=https://evil.com&scope=admin&...
```
👉 Paramètres visibles dans l'URL, modifiables par un attaquant

**Flux PAR (sécurisé)** :
```
1. POST /par (avec tous les paramètres + signature client)
   ← 201 Created { request_uri: "urn:ietf:params:oauth:request_uri:abc123" }

2. GET /auth?client_id=app&request_uri=urn:ietf:params:oauth:request_uri:abc123
```
👉 Paramètres protégés côté serveur, URL courte et sécurisée

**Avantages** :
- Intégrité garantie des paramètres
- Pas de limite de longueur d'URL
- Expiration automatique (90 secondes)

## 🛠️ **Stack technique et choix d'implémentation**

### **Pourquoi Axum pour le serveur ?**

Après avoir testé plusieurs frameworks web Rust (**Actix-web**, **Rocket**, **Warp**), j'ai choisi **Axum** pour plusieurs raisons :

✅ **Ergonomie** : API élégante avec extracteurs typed, routing expressif  
✅ **Performance** : Construit sur **Tokio** et **Tower**, optimisé pour la production  
✅ **Écosystème** : Compatible avec tout l'écosystème Tower (middleware, rate limiting...)  
✅ **Type-safety** : Compilation qui garantit la cohérence des routes et handlers  
✅ **Maintenance** : Développé par Tokio team, support long terme assuré  

### **SQLite : choix pragmatique**

Pour le MVP, j'ai opté pour **SQLite** plutôt que PostgreSQL/MySQL :

✅ **Zero-configuration** : Pas de serveur à gérer, base embarquée  
✅ **Portable** : Fichier unique, facile à backuper et migrer  
✅ **Performance suffisante** : Jusqu'à 100k req/s en lecture, largement suffisant pour démarrer  
✅ **Facilité de dev** : Tests rapides, pas de Docker nécessaire  

**Migration PostgreSQL prévue** : L'architecture Repository Pattern permet de switcher facilement vers PostgreSQL pour la production (haute disponibilité, réplication...).

### **Dépendances clés**

```toml
[dependencies]
# Framework web
axum = "0.8"
tokio = { version = "1.0", features = ["full"] }
tower = "0.4"

# Base de données
sqlx = { version = "0.8", features = ["sqlite", "runtime-tokio"] }

# JWT et Crypto
jsonwebtoken = "9.0"
rsa = "0.9"
bcrypt = "0.15"

# Sérialisation
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"

# Templates et i18n
handlebars = "4.0"
fluent-templates = "0.11"

# Observabilité
tracing = "0.1"
opentelemetry = "0.24"
```

## 📊 **Flux implémentés : démonstration complète**

### **1. Flux OIDC Standard (Authorization Code + PKCE)**

Le flux classique pour une application web :

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

### **2. Flux FAPI 2.0 avec PAR + DPoP**

Le flux sécurisé pour les applications critiques :

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

**Remarque** : Chaque appel API avec le token nécessite une **nouvelle preuve DPoP** signée. Impossible de réutiliser une ancienne preuve (protection anti-rejeu via `jti` + vérification temporelle).

## 💡 **Client de démonstration : tester en conditions réelles**

Pour valider l'implémentation, j'ai développé **un client Python Flask unique** qui offre **deux modes d'authentification** au choix sur la page d'accueil :

### **Mode OIDC Standard**
- Authorization Code Flow classique
- PKCE S256
- Authentification `client_secret_post`
- Session Flask pour gérer l'état
- Compte de test : utilisateur standard

### **Mode FAPI 2.0 Sécurisé**
- Flux PAR complet
- Authentification `private_key_jwt` avec clés RSA
- Génération de preuves DPoP à chaque requête
- Clés DPoP éphémères par session
- Compte de test : utilisateur FAPI

**🎯 Démo interactive disponible** : J'ai déployé le serveur et le client sur **[Railway](https://railway.com)** - un super service qui offre une intégration GitHub native et qui m'a permis d'héberger gratuitement le MVP pendant 1 mois avec leur offre de base. Parfait pour démarrer rapidement sans infrastructure complexe !

**Environnements disponibles** :
- 🧪 **[Démo DEV](https://trusty-client-dev.up.railway.app)** : Version de développement avec logs détaillés
- 🚀 **[Démo PROD](https://trusty-client-prd.up.railway.app)** : Version de production stable

**Comment tester** : Rendez-vous sur la page d'accueil, choisissez votre mode (Standard ou FAPI 2.0), et suivez le flux complet. Vous verrez les tokens générés, les preuves DPoP (en mode FAPI), et pourrez inspecter les claims JWT.

## 📚 **Documentation technique complète**

Le projet inclut une documentation détaillée. En attendant l'ouverture du repository GitHub pour la v1.0 (courant S1 2026), les documentations techniques sont **hébergées directement sur ce blog** :

### **Architecture & Stack**
📄 **[Documentation Technique](/docs/trusty/technical-fr/)**
- Structure DDD complète (Domain, Application, Infrastructure, Presentation)
- Stack technologique avec justifications
- Schémas de base de données SQLite
- Configuration Docker et déploiement

### **Flux OIDC & FAPI 2.0**
📄 **[Documentation des Flux](/docs/trusty/oidc-flow-fr/)**
- Diagrammes Mermaid de tous les flux
- Exemples de requêtes/réponses
- Détails de validation DPoP et PAR
- Gestion des erreurs et cas limites

### **Conformité aux standards**

| Spécification | Statut | Niveau de conformité |
|---------------|--------|---------------------|
| **OpenID Connect Core 1.0** | ✅ Implémenté | Authorization Code Flow complet |
| **OAuth 2.0 PKCE (RFC 7636)** | ✅ Implémenté | S256 uniquement (plain rejeté) |
| **PAR (RFC 9126)** | ✅ Implémenté | TTL 90s, stockage en mémoire |
| **private_key_jwt (RFC 7523)** | ✅ Implémenté | RS256, validation JWKS |
| **DPoP (RFC 9449)** | ✅ Implémenté | Liaison jkt, validation htm/htu/ath |
| **FAPI 2.0 Security Profile** | ✅ Implémenté | Subset MVP (pas Hybrid Flow) |
| **RP-Initiated Logout** | ✅ Implémenté | Avec confirmation utilisateur |
| **Token Revocation (RFC 7009)** | ✅ Implémenté | Access + Refresh tokens |
| **Token Introspection (RFC 7662)** | ✅ Implémenté | Validation pour Resource Servers |

## 🔮 **Roadmap : Prochaines étapes**

### **Version 0.9.0 (Q1 2026)**
- 🔄 **Refresh Token Flow** : Renouvellement de tokens sans ré-authentification
- 🔑 **Support SAML (full)** : Implémentation complète du protocole SAML 2.0 (IdP et SP)
- 🔐 **Backend PostgreSQL** : Migration pour haute disponibilité
- 🧪 **Tests de conformité** : Suite de tests OpenID Certification
- 📊 **Télémétrie avancée** : Métriques Prometheus, tracing distribué

### **Version 1.0.0 (Q2-Q3 2026) - Idées en cours de réflexion**

**Priorité 1 - Certification et tooling** :
- ✅ **Certification OpenID** : Conformité officielle
- 🎛️ **Console d'administration** : Interface de gestion des paramètres du provider OIDC et configuration des IdP/SP SAML

**Priorité 2 - Parcours d'authentification** :
- 🚶 **Parcours d'authentification** : Première implémentation de base avec modules standards (à déterminer - MFA, conditional access, step-up auth...)

**À explorer selon priorités** :
- 🔑 **WebAuthn/FIDO2** : Authentification sans mot de passe
- 📱 **CIBA (Client Initiated Backchannel Authentication)** : Pour mobile

⚠️ **Note** : Ces fonctionnalités sont des pistes de réflexion et ne sont pas encore totalement fixées. Le développement se fait avec mes moyens et mon temps disponible (j'ai un boulot à côté ! 😅), donc la roadmap peut évoluer en fonction des retours et des priorités.

### **Fonctionnalités exploratoires**
- **Distinction clients internes/externes** : Gestion différenciée selon le type de client (à ne pas confondre avec Device Flow)
- **JAR (JWT Secured Authorization Request - RFC 9101)** : Encapsulation des paramètres d'autorisation dans un JWT signé (alternative/complément à PAR)
- **mTLS (Mutual TLS)** : Authentification mutuelle au niveau TLS pour les clients hautement sécurisés
- **Rich Authorization Requests (RAR - RFC 9396)** : Permet de demander des autorisations fines et structurées au-delà des simples scopes. Exemple : au lieu de `scope=read`, demander `{"type":"account","actions":["read","transfer"],"limits":{"max_amount":1000}}`. Très utile pour les APIs bancaires où les autorisations sont complexes.

## 🤝 **Besoin de feedback d'experts Access Management**

Si vous travaillez dans le domaine de l'**Identity & Access Management**, vos retours seraient précieux :

🔍 **Points d'attention particuliers** :
- **Conformité des flux FAPI 2.0** : Les implémentations de DPoP (génération/validation des preuves, liaison jkt) et PAR (stockage, TTL, validation client_assertion) sont-elles conformes aux specs ? Y a-t-il des edge cases non couverts ?
- **Gestion des erreurs et cas limites** : Les codes d'erreur OAuth/OIDC sont-ils pertinents ? Comment gérez-vous les attaques par timing, les rejeux, les codes expirés dans vos implémentations ?
- **Sécurité génération/stockage tokens** : Les JWT sont signés RS256 avec rotation de clés. Le stockage des codes d'autorisation et refresh tokens en SQLite est-il suffisamment sécurisé pour un MVP ? Quelles sont vos pratiques en production ?
- **Performance et scalabilité** : Avec SQLite et stockage PAR en mémoire, le système est limité à un nœud unique. Quels sont les patterns recommandés pour passer à PostgreSQL + Redis distribués sans tout refactoriser ?
- **Interopérabilité** : Avez-vous testé l'intégration avec des Resource Servers tiers (Spring Boot, .NET, Node.js) ? Quels problèmes rencontrez-vous typiquement avec les ID tokens et access tokens DPoP ?

💬 **Comment remonter vos feedbacks** :

Pour l'instant, le projet n'est pas encore ouvert publiquement (prévu pour la v1.0 en S1 2026). En attendant, vous pouvez m'envoyer vos retours, questions ou recommandations par email :

📧 **Email** : [lostyzen@gmail.com](mailto:lostyzen@gmail.com)  
🔐 **PGP Key** : [Download from keys.openpgp.org](https://keys.openpgp.org/search?q=lostyzen%40gmail.com)  
📌 **Fingerprint** : `0205 9854 864D EE62 C2BB 455F D825 3521 EDD1 630B`

N'hésitez pas à :
- Tester les démos (DEV/PROD) et remonter bugs/incohérences
- Challenger la documentation technique
- Proposer des use cases métier spécifiques de votre secteur
- Signaler des écarts avec les spécifications OpenID/FAPI

## 🔗 **Ressources et liens utiles**

### **Spécifications officielles**
- [OpenID Connect Core 1.0](https://openid.net/specs/openid-connect-core-1_0.html)
- [OAuth 2.0 PKCE (RFC 7636)](https://datatracker.ietf.org/doc/html/rfc7636)
- [Pushed Authorization Requests (RFC 9126)](https://datatracker.ietf.org/doc/html/rfc9126)
- [JWT Client Authentication (RFC 7523)](https://datatracker.ietf.org/doc/html/rfc7523)
- [DPoP (RFC 9449)](https://datatracker.ietf.org/doc/html/rfc9449)
- [FAPI 2.0 Security Profile](https://openid.net/specs/fapi-security-profile-2_0.html)

### **Projet TRusTY**
- 🧪 **[Démo DEV](https://trusty-client-dev.up.railway.app)** : Version développement avec logs
- 🚀 **[Démo PROD](https://trusty-client-prd.up.railway.app)** : Version production stable
- 📖 **[Documentation Technique](/docs/trusty/technical-fr/)**
- 📖 **[Documentation Flux OIDC](/docs/trusty/oidc-flow-fr/)**
- 🏠 **Repository GitHub** : Ouverture prévue avec la v1.0 (S1 2026)

## 🎉 **Conclusion**

Construire un serveur OIDC from scratch en Rust, c'est un sacré défi technique. Mais TRusTY est avant tout le fruit de **près de 10 ans d'expérience dans l'Access Management**, acquise notamment dans le **secteur bancaire** où les exigences de sécurité et de conformité sont particulièrement élevées. Cette expertise, je la mets aujourd'hui à disposition de mes clients pour les accompagner sur leurs enjeux d'authentification et d'autorisation.

**TRusTY n'est pas encore prêt pour la production** (c'est un MVP !), mais il démontre déjà qu'il est possible de construire un serveur OIDC performant, compréhensible et conforme aux standards les plus exigeants, en s'appuyant sur une **connaissance approfondie des problématiques terrain**.

**Ce que vous pouvez faire dès maintenant** :
- Tester les démos (DEV/PROD) pour voir les flux en action
- Parcourir la documentation technique détaillée hébergée sur ce blog
- Partager vos retours et cas d'usage par email

La suite ? Continuer à implémenter les fonctionnalités manquantes, durcir la sécurité, améliorer les performances... et pourquoi pas viser la **certification OpenID officielle** pour la v1.0 !

**Des questions ? Des suggestions ?** N'hésitez pas à me contacter ou à commenter ci-dessous. Les échanges avec la communauté sont essentiels pour faire évoluer ce genre de projet.
