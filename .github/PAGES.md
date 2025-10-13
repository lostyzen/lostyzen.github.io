# Configuration GitHub Pages

## 🚀 Déploiement automatique

Ce repository est configuré pour un déploiement automatique sur GitHub Pages via GitHub Actions.

### ⚙️ Configuration requise dans GitHub

1. **Aller dans les paramètres du repository** : `Settings` → `Pages`

2. **Configurer la source** :
   - Source : `GitHub Actions`
   - ✅ **Ne pas** sélectionner "Deploy from a branch"
   - ✅ Sélectionner **"GitHub Actions"**

3. **Le workflow se déclenche automatiquement** :
   - À chaque push sur la branche `main`
   - Manuellement via l'onglet "Actions"

### 🔗 URL du site

Une fois configuré, votre site sera disponible à :
**https://lostyzen.github.io**

### 📊 Suivi des déploiements

- **Actions** : Voir les builds en cours dans l'onglet "Actions"
- **Environments** : Suivre les déploiements dans "Settings" → "Environments"
- **Status** : Badge de statut disponible pour le README

### 🛠️ Build local

Pour tester en local avant déploiement :

```bash
# Development server
hugo server -D

# Production build (identique à GitHub Actions)
hugo --gc --minify --baseURL "https://lostyzen.github.io/"
```

### 🎯 Optimisations incluses

Le workflow GitHub Actions inclut :
- ✅ **Hugo Extended** (dernière version)
- ✅ **Minification** des assets
- ✅ **Garbage collection** pour optimiser la taille
- ✅ **Submodules** (thème PaperMod)
- ✅ **Cache** pour accélérer les builds
- ✅ **Environnement de production**

### 🚨 Troubleshooting

**Erreur de build ?**
1. Vérifier les logs dans l'onglet "Actions"
2. Tester le build en local avec `hugo --gc --minify`
3. Vérifier que les submodules sont à jour

**Site non accessible ?**
1. Attendre 5-10 minutes après le premier déploiement
2. Vérifier la configuration Pages dans Settings
3. Vérifier que le workflow s'est exécuté sans erreur

**Modification non visible ?**
1. Vider le cache du navigateur (Ctrl+F5)
2. Attendre la propagation CDN (quelques minutes)
3. Vérifier que le commit a bien déclenché un build