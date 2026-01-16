# Projet lostyzen.github.io - Blog DevOps

## Description
Blog technique Hugo hébergé sur GitHub Pages, axé DevOps, architecture logicielle et bonnes pratiques.

## Stack technique
- **Générateur** : Hugo (static site generator)
- **Thème** : PaperMod (submodule Git)
- **Hébergement** : GitHub Pages
- **URL** : https://lostyzen.github.io

## Structure du projet

### Répertoires principaux
```
lostyzen.github.io/
├── archetypes/           # Templates pour nouveaux contenus
│   ├── default.md
│   └── posts.md          # Template complet pour articles
├── content/              # Contenu du blog
│   ├── posts/            # Articles publics (menu)
│   ├── docs/             # Documentation technique (caché du menu)
│   │   └── trusty/       # Docs TRusTY OIDC
│   └── pro/              # Documentation professionnelle (caché)
│       └── aws/          # Docs AWS EKS/Karpenter
├── data/                 # Fichiers de données
├── layouts/              # Layouts personnalisés
│   └── partials/
│       └── extend_head.html  # Support Mermaid.js
├── static/               # Fichiers statiques (images, HTML standalone)
├── themes/               # Thème PaperMod (submodule)
└── hugo.toml             # Configuration principale
```

### Organisation du contenu

#### Articles publics (`content/posts/`)
- Visibles dans le menu principal
- Format bilingue : `article.fr.md` et `article.en.md`
- Frontmatter complet avec SEO, tags, catégories

#### Documentation cachée (`content/docs/`, `content/pro/`)
- NON visible dans le menu
- Accessible uniquement par URL directe
- Frontmatter minimal
- Utilisé pour documentation technique à partager avec collègues

### Conventions

#### Nommage des fichiers
- Articles : `nom-article.fr.md` / `nom-article.en.md`
- Documentation : `nom-doc.md` (français par défaut)

#### Frontmatter minimal (docs cachés)
```yaml
---
title: "Titre"
date: 2026-01-16
draft: false
showToc: true
TocOpen: true
searchHidden: true
---
```

#### Frontmatter complet (articles publics)
Voir `archetypes/posts.md` pour le template complet.

### Fonctionnalités
- **Multilingue** : FR (défaut) + EN
- **Mermaid.js** : Diagrammes dans le markdown
- **Syntax highlighting** : Style GitHub
- **SEO** : Schema.org, Open Graph, sitemap
- **RSS** : Feed automatique

### Commandes utiles
```bash
# Serveur de développement
hugo server -D

# Build production
hugo --minify

# Nouveau post
hugo new posts/mon-article.fr.md
```

### Sections cachées existantes
- `/docs/trusty/` - Documentation OIDC TRusTY
- `/pro/aws/` - Documentation AWS EKS/Karpenter
