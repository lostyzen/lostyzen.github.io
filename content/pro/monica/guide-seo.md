---
title: "Guide SEO — Viva Monica"
date: 2026-03-10
lastmod: 2026-03-10
draft: false
showToc: true
TocOpen: true
searchHidden: true
---

# Guide SEO — Viva Monica

> Objectif : lister tous les points SEO à améliorer sur vivamonica.fr, expliqués simplement.

---

## 1. Les points à améliorer

### 1.1 Balise `<title>` — Le titre de ta page dans Google

**C'est quoi ?**
C'est le texte bleu cliquable qui apparaît dans les résultats Google. C'est la première chose que les gens voient.

**Exemple dans le code :**
```html
<title>Viva Monica — Recrutement tech & IT à Bruxelles</title>
```

**Exemple dans Google :**
> **Viva Monica — Recrutement tech & IT à Bruxelles**
> www.vivamonica.fr

**Pourquoi c'est important ?**
Sans titre clair, Google invente un titre à partir du contenu de la page. Le résultat est souvent peu attractif et mal ciblé. Un bon titre doit contenir les mots-clés principaux et donner envie de cliquer.

**Règles :**
- Entre 50 et 60 caractères (au-delà, Google le coupe)
- Placer les mots-clés importants en début de titre
- Unique pour chaque page du site

---

### 1.2 Balise `<meta description>` — Le résumé sous le titre dans Google

**C'est quoi ?**
C'est le petit texte gris qui apparaît sous le titre dans les résultats de recherche. C'est ta "vitrine" dans Google.

**Exemple dans le code :**
```html
<meta name="description" content="Viva Monica connecte les talents tech avec les entreprises à Bruxelles. Recrutement IT, missions freelance et accompagnement carrière.">
```

**Exemple dans Google :**
> **Viva Monica — Recrutement tech & IT**
> www.vivamonica.fr
> *Viva Monica connecte les talents tech avec les entreprises à Bruxelles. Recrutement IT, missions freelance et accompagnement carrière.*

**Pourquoi c'est important ?**
Sans meta description, Google pioche un bout de texte au hasard dans la page. Le résultat est souvent décousu et peu engageant. Une bonne description augmente le taux de clic (CTR).

**Règles :**
- Entre 150 et 160 caractères
- Résumer clairement ce que propose la page
- Inclure un appel à l'action ("Découvrez", "Rejoignez", etc.)
- Unique pour chaque page

---

### 1.3 Balises Open Graph — L'aperçu quand on partage sur les réseaux sociaux

**C'est quoi ?**
Quand quelqu'un partage le lien du site sur LinkedIn, Facebook ou WhatsApp, ces balises définissent ce qui s'affiche : le titre, la description, et surtout l'image.

**Exemple dans le code :**
```html
<meta property="og:title" content="Viva Monica — Recrutement tech & IT">
<meta property="og:description" content="Connecte les talents tech avec les entreprises à Bruxelles.">
<meta property="og:image" content="https://www.vivamonica.fr/images/preview.jpg">
<meta property="og:url" content="https://www.vivamonica.fr/">
<meta property="og:type" content="website">
```

**Pourquoi c'est important ?**
Sans ces balises, quand quelqu'un partage le lien sur LinkedIn, ça affiche juste une URL nue ou un aperçu moche. Avec, ça affiche une belle carte avec image + titre + description. Sur LinkedIn surtout (contexte recrutement), c'est crucial pour la crédibilité.

**Règles :**
- Image : 1200x630 pixels minimum (format paysage)
- Titre : court et accrocheur
- Description : complémentaire au titre

---

### 1.4 Balise `canonical` — Éviter le contenu dupliqué

**C'est quoi ?**
Elle dit à Google : "la version officielle de cette page, c'est celle-ci". Ça évite que Google pénalise le site s'il trouve le même contenu accessible via plusieurs URLs.

**Exemple dans le code :**
```html
<link rel="canonical" href="https://www.vivamonica.fr/">
```

**Pourquoi c'est important ?**
Un même contenu peut être accessible via `vivamonica.fr`, `www.vivamonica.fr`, `vivamonica.fr/index.html`, etc. Sans canonical, Google peut considérer ça comme du contenu dupliqué et baisser le classement.

---

### 1.5 Attribut `lang` — La langue de la page

**C'est quoi ?**
On indique à Google (et aux lecteurs d'écran) dans quelle langue est écrit le contenu.

**Exemple dans le code :**
```html
<html lang="fr">
```

**Pourquoi c'est important ?**
Sans ça, Google doit deviner la langue. Il peut mal classifier la page et ne pas la proposer aux francophones. Pour un site belge francophone, c'est indispensable. Le site Wix l'a (`fr`) mais avec une locale `fr-be`, ce qui est correct.

---

### 1.6 Sitemap XML — La carte du site pour Google

**C'est quoi ?**
Un fichier `sitemap.xml` à la racine du site qui liste toutes les pages. C'est comme donner un plan à Google pour qu'il sache exactement quoi indexer.

**Exemple :**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://www.vivamonica.fr/</loc>
    <lastmod>2026-03-10</lastmod>
  </url>
</urlset>
```

**Pourquoi c'est important ?**
Sans sitemap, Google doit découvrir les pages tout seul en suivant les liens. Il peut en rater. Avec un sitemap, on est sûr que toutes les pages importantes sont connues.

**Note :** Pour un site d'une seule page, c'est moins critique, mais c'est une bonne pratique.

---

### 1.7 Fichier `robots.txt` — Les règles pour les robots

**C'est quoi ?**
Un fichier texte à la racine du site qui dit aux moteurs de recherche ce qu'ils ont le droit de crawler ou pas.

**Exemple :**
```
User-agent: *
Allow: /
Sitemap: https://www.vivamonica.fr/sitemap.xml
```

**Pourquoi c'est important ?**
Il permet de pointer vers le sitemap et d'empêcher l'indexation de pages inutiles (pages d'admin, pages de test, etc.). Sans lui, les robots font ce qu'ils veulent.

---

### 1.8 Données structurées (JSON-LD) — Aider Google à comprendre le contenu

**C'est quoi ?**
Un bloc de code invisible pour l'utilisateur mais lisible par Google. Il décrit précisément ce qu'est le site : une entreprise, un service, un article, etc.

**Exemple pour Viva Monica :**
```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "Viva Monica",
  "url": "https://www.vivamonica.fr",
  "description": "Recrutement tech et IT à Bruxelles",
  "address": {
    "@type": "PostalAddress",
    "addressLocality": "Bruxelles",
    "addressCountry": "BE"
  }
}
</script>
```

**Pourquoi c'est important ?**
Ça permet à Google d'afficher des résultats enrichis (rich snippets) : adresse, horaires, avis, logo, etc. Ça rend le résultat plus visible et plus cliquable dans la page de recherche.

---

### 1.9 Balises d'en-tête (`<h1>`, `<h2>`, etc.) — La structure du contenu

**C'est quoi ?**
Les titres et sous-titres de la page, organisés hiérarchiquement. Google les utilise pour comprendre la structure du contenu.

**Règles :**
- Un seul `<h1>` par page (le titre principal)
- `<h2>` pour les sections, `<h3>` pour les sous-sections
- Ne pas sauter de niveaux (`<h1>` puis `<h3>` sans `<h2>`)

**Pourquoi c'est important ?**
Google accorde plus de poids aux mots dans les titres. Une hiérarchie claire aide Google à comprendre de quoi parle chaque section.

---

### 1.10 Texte `alt` sur les images — Décrire les images pour Google

**C'est quoi ?**
Un attribut sur chaque image qui décrit ce qu'elle montre. Google ne "voit" pas les images, il lit ce texte.

**Exemple :**
```html
<img src="equipe.jpg" alt="Équipe Viva Monica en réunion dans leurs bureaux de Bruxelles">
```

**Pourquoi c'est important ?**
- Google Images indexe les images grâce au texte alt → source de trafic supplémentaire
- Les lecteurs d'écran lisent ce texte aux personnes malvoyantes (accessibilité)
- Si l'image ne charge pas, ce texte s'affiche à la place

---

### 1.11 URLs propres et lisibles

**C'est quoi ?**
Les URLs des pages doivent être courtes, descriptives et contenir des mots-clés.

**Bon exemple :** `vivamonica.fr/recrutement-tech-bruxelles`
**Mauvais exemple :** `vivamonica.fr/page?id=12345&ref=abc`

**Pourquoi c'est important ?**
Google utilise l'URL comme signal de pertinence. Une URL avec des mots-clés a plus de poids qu'une URL générée aléatoirement.

---

### 1.12 Vitesse de chargement — Core Web Vitals

**C'est quoi ?**
Google mesure 3 métriques de performance :
- **LCP** (Largest Contentful Paint) : en combien de temps le contenu principal s'affiche
- **FID/INP** (Interaction to Next Paint) : en combien de temps le site réagit au premier clic
- **CLS** (Cumulative Layout Shift) : est-ce que la page "saute" pendant le chargement

**Pourquoi c'est important ?**
Depuis 2021, Google utilise la vitesse comme facteur de classement. Un site lent est pénalisé. Avec ses ~500 KB de framework, le site Wix est désavantagé par rapport à un site statique léger.

---

### 1.13 HTTPS — Le cadenas dans le navigateur

**C'est quoi ?**
Le site doit être accessible en HTTPS (connexion sécurisée). Les deux versions du site le sont déjà.

**Pourquoi c'est important ?**
Google favorise les sites HTTPS. Un site en HTTP simple affiche un avertissement "Non sécurisé" dans le navigateur, ce qui fait fuir les visiteurs.

**Statut :** OK sur les deux versions.

---

### 1.14 Mobile-friendly — Le site s'adapte au téléphone

**C'est quoi ?**
Le site doit être lisible et utilisable sur mobile sans zoomer.

**Pourquoi c'est important ?**
Google utilise le "mobile-first indexing" : il évalue la version mobile en priorité. Si le site n'est pas mobile-friendly, il est pénalisé. La balise `<meta name="viewport">` est indispensable (elle manque sur la v1 Netlify).

---

### 1.15 Liens internes et navigation

**C'est quoi ?**
Les liens entre les différentes pages/sections du site. Ils aident Google à découvrir tout le contenu et à comprendre la structure.

**Pourquoi c'est important ?**
Un bon maillage interne distribue le "jus SEO" (l'autorité) entre les pages. Les pages orphelines (sans lien qui pointe vers elles) sont rarement indexées.

---

## 2. Ce que ça améliore concrètement

| Amélioration | Effet concret |
|---|---|
| **Visibilité Google** | Le site apparaît dans plus de recherches pertinentes |
| **Position dans les résultats** | Le site monte dans le classement (page 1 vs page 5) |
| **Taux de clic (CTR)** | Un beau résultat avec titre + description + image = plus de clics |
| **Partage sur les réseaux** | LinkedIn/Facebook affichent une belle carte au lieu d'un lien nu |
| **Trafic organique** | Plus de visiteurs sans payer de publicité |
| **Crédibilité** | Un site bien référencé inspire confiance aux candidats et aux entreprises |
| **Accessibilité** | Le site est utilisable par tous, y compris les personnes en situation de handicap |
| **Pérennité** | Un bon SEO de base continue de générer du trafic dans le temps, contrairement à la pub |

---

## 3. Les 5 points prioritaires à retenir

Si tu ne devais retenir que 5 choses à faire en priorité, ce serait celles-ci :

### 1. Ajouter `<title>` et `<meta description>` sur chaque page
> C'est le **minimum vital**. Sans ça, Google affiche n'importe quoi dans les résultats. C'est la première chose que voient les gens. 5 minutes de travail, impact immédiat.

### 2. Ajouter les balises Open Graph (og:title, og:description, og:image)
> Pour le **recrutement**, LinkedIn est le canal principal. Quand quelqu'un partage le site, il faut que ça affiche une belle carte professionnelle, pas un lien brut. C'est une question de crédibilité.

### 3. Ajouter les données structurées JSON-LD (Organization)
> Ça permet à Google d'afficher des **résultats enrichis** (logo, localisation, description). Ça prend 10 lignes de code et ça change la manière dont le site apparaît dans Google.

### 4. Optimiser la vitesse de chargement
> Le site Wix est **14x plus lourd** que la v1. Google pénalise les sites lents. Pour un site vitrine, un site statique léger sera toujours mieux classé qu'une usine à gaz Wix. Si rester sur Wix, au minimum optimiser les images et limiter les widgets.

### 5. Vérifier le mobile et ajouter `<meta name="viewport">`
> **60%+ du trafic web vient du mobile**. Google indexe la version mobile en priorité. Si le site ne s'affiche pas correctement sur téléphone, il sera pénalisé. Sur la v1, cette balise manque — c'est critique.

---

> **En résumé** : le SEO ce n'est pas de la magie, c'est une checklist de bonnes pratiques. Les 5 points ci-dessus couvrent 80% du travail pour 20% de l'effort. Le reste (backlinks, contenu régulier, maillage) vient dans un second temps.
