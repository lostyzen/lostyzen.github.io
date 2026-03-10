---
title: "Benchmark & Audit de sécurité — Viva Monica"
date: 2026-03-10
lastmod: 2026-03-10
draft: false
showToc: true
TocOpen: true
searchHidden: true
---

# Benchmark & Audit de sécurité — Viva Monica

> Sites analysés : `vivamonica.fr` (Wix) vs `viva-monica.netlify.app` (v1 statique)

---

## 1. Comparatif technique

| Critère | vivamonica.fr (Wix) | viva-monica.netlify.app (v1) |
|---|---|---|
| **Plateforme** | Wix (Thunderbolt v1.16956.0) | HTML/CSS/JS vanilla |
| **Hébergement** | Wix (intégré) | Netlify |
| **Poids estimé** | ~500 KB+ (non compressé) | ~35 KB |
| **Scripts chargés** | 15+ (Thunderbolt, polyfills, analytics, security) | 1 script inline |
| **CSS** | 2+ fichiers minifiés + CSS inline (~200 KB+) | 1 stylesheet inline |
| **Fonts** | Wix Madefor Text, Helvetica, Meiryo | Inter, Playfair Display (Google Fonts) |
| **Framework JS** | React (via Thunderbolt) | Vanilla JS (Intersection Observer) |
| **Bundler** | Rspack v1.6.6 | Aucun |
| **Analytics** | Google Analytics 4 (G-B0L8D1C7MF) | Aucun |
| **Services tiers** | GA4, Wix consent manager | Typeform (2 formulaires), LinkedIn |

---

## 2. SEO

| Critère | vivamonica.fr | viva-monica.netlify.app |
|---|---|---|
| Meta description | Non trouvée | Non trouvée |
| Open Graph tags | Non détectés | Absents |
| Twitter Card | Non détecté | Absent |
| Canonical URL | Non visible | Absent |
| Structured data (JSON-LD) | Absent | Absent |
| Sitemap | Non détecté | Absent |
| Lang attribute | `fr` (locale `fr-be`) | Absent |
| Viewport meta | Oui | **Non trouvé** |

**Verdict SEO** : Les deux sites sont faibles en SEO. Aucun n'a de meta description, d'Open Graph, ni de données structurées. La v1 Netlify n'a même pas de `viewport` meta tag ni de `lang`, ce qui est critique.

---

## 3. Accessibilité

| Critère | vivamonica.fr | viva-monica.netlify.app |
|---|---|---|
| Alt text images | Non confirmé | Absent |
| ARIA attributes | Minimal | Absent |
| HTML sémantique | Faible (divs) | Faible |
| Skip-to-content | Possible (non confirmé) | Absent |
| Lang sur `<html>` | Oui | **Non** |

**Verdict** : Les deux sites ont des lacunes significatives en accessibilité.

---

## 4. Performance

| Critère | vivamonica.fr | viva-monica.netlify.app |
|---|---|---|
| Poids total | ~500 KB+ | ~35 KB |
| Minification | Oui (CSS/JS) | Non |
| Lazy loading images | Oui (blur progressif) | Oui (Intersection Observer) |
| WebP/AVIF | Non détecté | Non |
| Font preloading | Non visible | Non |
| Cache busting | Oui (hashes dans URLs) | Non visible |
| Ratio poids/contenu | Très mauvais (beaucoup de framework) | Bon (code léger) |

**Verdict performance** : La v1 Netlify est **~14x plus légère**. Wix embarque une quantité massive de code framework pour un site vitrine simple.

---

## 5. Mobile / Responsive

| Critère | vivamonica.fr | viva-monica.netlify.app |
|---|---|---|
| Viewport meta | Oui | **Non trouvé** |
| Responsive design | Oui (device classes) | Oui (clamp, auto-fit, media queries) |
| Mobile menu | Oui (TINY_MENU) | Caché via media query |
| Min width | 320px | 300px (minmax grid) |

---

## 6. Audit de sécurité

### 6.1 vivamonica.fr (Wix)

#### Informations exposées

| Trouvaille | Sévérité | Détail |
|---|---|---|
| API endpoints visibles | Moyenne | `/_api/v1/access-tokens`, `/_api/v2/dynamicmodel` |
| Versions exposées | Basse | Thunderbolt v1.16956.0, Rspack v1.6.6, revision 298 |
| Google Analytics ID | Info | `G-B0L8D1C7MF` |
| Config site publique | Basse | `"isSEO":false`, `"debug":false`, `"enableTestApi":false` |
| Locale/langue | Info | `fr-be` exposé dans la config |

#### Headers de sécurité

| Header | Statut |
|---|---|
| Content-Security-Policy | Non détecté |
| X-Frame-Options | Non visible |
| X-Content-Type-Options | Non visible |
| Strict-Transport-Security | Non visible |
| Referrer-Policy | Non visible |

#### Points positifs (Wix)

- Hardening fetch/XHR activé
- Blocage ServiceWorker registration
- Isolation des cookies de session (`client-session-bind`, `svSession`, etc.)
- Restriction `setTimeout`/`setInterval` avec arguments string
- Consent manager RGPD intégré

#### Points négatifs (Wix)

- JavaScript inline massif et obfusqué
- Manipulation étendue de l'objet `window`
- Pas de SRI (Subresource Integrity) sur les scripts externes
- API endpoints exposés côté client
- Mode debug/test visible dans la config (même si désactivé)

---

### 6.2 viva-monica.netlify.app (v1)

#### Informations exposées

| Trouvaille | Sévérité | Détail |
|---|---|---|
| URLs Typeform | Info | `form.typeform.com/to/Iyb6kmyA`, `form.typeform.com/to/PakkfpOY` |
| Pas d'API keys exposées | OK | Rien de sensible détecté |

#### Headers de sécurité

| Header | Statut |
|---|---|
| Content-Security-Policy | Non implémenté |
| X-Frame-Options | Non visible |
| X-Content-Type-Options | Non visible |
| Strict-Transport-Security | Non visible |
| Referrer-Policy | Non visible |

#### Points négatifs (v1)

- JavaScript inline (surface d'attaque XSS si compromis)
- Aucun header de sécurité visible
- Soumission de CV via Typeform (données sensibles vers un tiers)
- Pas de banner de consentement cookies visible

---

## 7. Synthèse globale

| Domaine | Gagnant | Commentaire |
|---|---|---|
| **Performance** | v1 Netlify | 14x plus léger, pas de framework lourd |
| **SEO** | Aucun | Les deux sont insuffisants |
| **Accessibilité** | Wix (légèrement) | Au moins le `lang` et viewport sont présents |
| **Sécurité** | Wix (légèrement) | Hardening intégré, mais surface d'attaque plus large |
| **Mobile** | Wix | Viewport meta manquant sur la v1 |
| **Maintenabilité** | v1 Netlify | Code simple et lisible |
| **Portabilité** | v1 Netlify | Wix = vendor lock-in total |
| **Coût** | v1 Netlify | Gratuit vs abonnement Wix |

---

## 8. Recommandations

### Pour vivamonica.fr (Wix)

1. Ajouter les meta tags SEO (description, OG, canonical)
2. Implémenter les données structurées (JSON-LD)
3. Vérifier l'accessibilité (alt text, ARIA)
4. Les headers de sécurité sont gérés par Wix (limité en contrôle)

### Pour viva-monica.netlify.app (v1)

1. **Critique** : Ajouter `<meta name="viewport">` et `lang="fr"` sur `<html>`
2. Ajouter les meta tags SEO
3. Déplacer le JS inline vers un fichier externe
4. Configurer les headers de sécurité dans `netlify.toml` (CSP, HSTS, X-Frame-Options)
5. Ajouter un bandeau de consentement si des cookies sont utilisés
6. Précharger les fonts Google (preload/preconnect)
7. Vérifier la politique de confidentialité Typeform pour les CV

### Recommandation générale

Si le site devait être refait, un générateur statique (Hugo, Astro) hébergé sur Netlify/Vercel offrirait le meilleur compromis performance/SEO/sécurité/contrôle, tout en éliminant le vendor lock-in Wix.
