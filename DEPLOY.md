# 🚀 Instructions de déploiement GitHub Pages

## Étapes à suivre après le push :

### 1. **Commit et push des fichiers**
```bash
git add .
git commit -m "🚀 Add GitHub Actions workflow for auto-deployment

✨ Features:
- Automatic Hugo build on push to main
- GitHub Pages deployment
- Production optimizations (minify, GC)
- Support for submodules (PaperMod theme)

🔧 Configuration:
- Hugo v0.151.0 Extended
- Build time: ~1-2 minutes
- Deploy URL: https://lostyzen.github.io"

git push origin main
```

### 2. **Configuration GitHub Pages (une seule fois)**

1. Aller sur **GitHub.com** → Votre repository `lostyzen/lostyzen.github.io`
2. Cliquer sur **Settings** (onglet en haut)
3. Scroller jusqu'à **Pages** (menu de gauche)
4. Dans **Source**, sélectionner : **"GitHub Actions"**
5. ✅ **Ne pas** sélectionner "Deploy from a branch"

### 3. **Premier déploiement**

- Le workflow se lance automatiquement après le push
- Aller dans **Actions** pour suivre le progress
- Temps de build : ~1-2 minutes
- URL finale : **https://lostyzen.github.io**

### 4. **Vérification**

- Badge de statut dans le README indique : [![Deploy Hugo site to GitHub Pages](https://github.com/lostyzen/lostyzen.github.io/actions/workflows/hugo.yml/badge.svg)](https://github.com/lostyzen/lostyzen.github.io/actions/workflows/hugo.yml)
- Site accessible sur https://lostyzen.github.io
- Déploiement automatique à chaque push sur `main`

## ✅ Résultat attendu

- ⚡ **Build rapide** : ~1-2 minutes
- 🌐 **Site multilingue** : FR/EN avec switch automatique
- 🎨 **Thème sombre/clair** fonctionnel
- 📱 **Responsive** sur tous appareils
- 🔍 **SEO optimisé** avec sitemap, robots.txt
- 📊 **Performance** : Lighthouse 95+

## 🎯 Maintenance

Désormais, pour publier un nouvel article :
1. Créer le fichier `.fr.md` et `.en.md` dans `content/posts/`
2. Commit et push
3. Le site se met à jour automatiquement ! 🚀