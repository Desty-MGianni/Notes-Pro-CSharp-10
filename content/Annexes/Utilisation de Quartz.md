---
publish: false
---
# C'est quoi ?

*Quartz* est un outils qui permet de générer des pages HTML à partir d'un fichier markdown (comme pour l'exercice sur [boot.dev](https://www.boot.dev/courses/build-static-site-generator-python)). La particularité de *Quartz* est qu'il est 100% compatible avec *Obsidian* !

# Configuration de Quartz (vault externe + symlink)

>[!info] Contexte Setup choisi : 
>vault Obsidian **synchronisé via le cloud**, donc indépendant du repo Quartz. Le lien entre les deux se fait via un **symlink**, pas en plaçant le vault dans le repo.

## Prérequis

- Node.js (LTS, v20+)
- Git
- Un vault Obsidian existant, déjà synchronisé via votre solution cloud

## 1. Cloner Quartz

```bash
git clone https://github.com/jackyzha0/quartz.git nom-repo
cd nomm-repo
npm i
npx quartz create
# Choisir Obsidian puis "Empty Quartz" lors du prompt
```

## 2. Lier le vault via symlink

>[!warning] Le symlink n'est pas versionné.
> Git ne suit pas les symlinks de façon portable. Ce setup doit être recréé manuellement sur chaque machine où vous clonez le repo Quartz.

```bash
rm -rf content
ln -s /chemin/vers/MonVaultObsidian content
```

- macOS/Linux : commande ci-dessus
- Windows : `mklink /D content C:\chemin\vers\MonVaultObsidian` (en invite de commande admin)

✅ Résultat : tout ce qui est édité dans Obsidian apparaît automatiquement dans `mon-repo/content/`, sans copie manuelle.

## 3. Configuration de base — `quartz.config.yaml`

Avec le preset Obsidan, le ficher est généré avec une quantité monstre de plugins.

### Retirer le graph

```yaml
...
plugins:
  ...
  - source: github:quartz-community/graph
	 enabled: false
	 layout:
	   position: right
	   priority: 10
  ...
```

### Désactiver le listing dans le dossier (recommandé quand on fournis un *index.md* par dossier)

On masque la classe `page-listing` dans le fichier *cutom.scss*

```css
.page-listing{
	display: none;
}
```

## 4. Filtrer les notes privées (si le vault contient du contenu non destiné au public)

Dans chaque note à publier :

```yaml
---
title: "Titre de la note"
publish: true
---
```

**Le preset Obsidan active le plugin `remove-draft` et désactive `explicit-publish` dans le ficher config.** 

Ces deux filtres ont une logique inverse :

| Plugin             | Logique                                                                                  |
| ------------------ | ---------------------------------------------------------------------------------------- |
| `remove-draft`     | Publie **tout** sauf les notes avec `draft: true` — ignore complètement `publish: false` |
| `explicit-publish` | Publie **uniquement** les notes avec `publish: true` — tout le reste est exclu           |

### La correction dans `quartz.config.yaml`

```yaml
- source: github:quartz-community/remove-draft
  enabled: false    # ← désactiver

- source: github:quartz-community/explicit-publish
  enabled: true     # ← activer
```


> [!warning] Pièces jointes 
> Par défaut, tous les fichiers non-Markdown sont copiés dans le dossier publié, même sans lien direct. Utiliser `ignorePatterns` dans la config pour exclure les attachments privés :
> 
> ```ts
> ignorePatterns: [
>   ".obsidian/",
>   "Attachments/!(public-)*.*",
> ]
> ```

## 5. Tester en local

```bash
npx quartz build --serve
```

→ site visible sur `http://localhost:8080`, rebuild automatique à chaque modification.

## 6. Déployer sur GitHub Pages

### Étape 1 — Créer le workflow GitHub Actions

Créez un nouveau fichier `quartz/.github/workflows/deploy.yml` dans votre repo local avec ce contenu : [GitHub](https://github.com/refactoringhq/tolaria/commit/95bcf3b25a542c673a231b3b766337e9876805f1)

```yaml
name: Deploy Quartz site to GitHub Pages

on:
  push:
    branches:
      - v5

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v6
        with:
          fetch-depth: 0
      - uses: actions/setup-node@v6
        with:
          node-version: 24
      - name: Cache dependencies
        uses: actions/cache@v5
        with:
          path: ~/.npm
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node-
      - name: Cache Quartz plugins
        uses: actions/cache@v5
        with:
          path: .quartz/plugins
          key: ${{ runner.os }}-plugins-${{ hashFiles('quartz.lock.json') }}
          restore-keys: |
            ${{ runner.os }}-plugins-
      - name: Install Dependencies
        run: npm ci
      - name: Install Quartz plugins
        run: npx quartz plugin install
      - name: Build Quartz
        run: npx quartz build
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: public

  deploy:
    needs: build
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### Étape 2 — Changez le remote `origin` pour pointer vers votre repo

```bash
git remote set-url origin https://github.com/Desty-MGianni/nom-du-repo.git
```

pour vérifier que c'est correcte :

```bash
git remote -v
```

>[!tip] Upstream vs Origin  
Quartz configure automatiquement un remote `upstream` qui pointe vers `jackyzha0/quartz` — c'est normal et c'est utilisé par `npx quartz upgrade` pour récupérer les mises à jour de Quartz. Ne touchez pas à `upstream`, uniquement à `origin`.

### Étape 3 — Configurer GitHub Pages

Allez dans l'onglet "Settings" de votre repo GitHub, cliquez sur "Pages" dans la sidebar, et sous "Source" sélectionnez "GitHub Actions". [GitHub](https://github.com/refactoringhq/tolaria/commit/95bcf3b25a542c673a231b3b766337e9876805f1)

### Étape 4 — Pousser et déployer

Commitez ces changements en lançant `npx quartz sync`. Cela déploiera votre site sur `<votre-pseudo>.github.io/<nom-du-repo>`. [GitHub](https://github.com/refactoringhq/tolaria/commit/95bcf3b25a542c673a231b3b766337e9876805f1)

> [!warning] Erreur d'environnement protégé  
> Si vous obtenez une erreur "not allowed to deploy to github-pages due to environment protection rules", allez dans Settings → Environments et supprimez l'environnement existant via l'icône de corbeille. GitHub Actions le recréera correctement au prochain push. [GitHub](https://github.com/refactoringhq/tolaria/commit/95bcf3b25a542c673a231b3b766337e9876805f1)

> [!tip] `npx quartz sync` vs `git push`  
> `npx quartz sync` est la commande Quartz 5 qui remplace un simple `git push` — elle pull d'abord les éventuelles modifications distantes, puis commit et push vos changements locaux en une seule commande.```

## Pièges fréquents

|Problème|Cause|Solution|
|---|---|---|
|`command not found: npm`|Node.js non installé|Réinstaller Node.js, redémarrer le terminal|
|Erreur 404 après déploiement|Source ≠ GitHub Actions|Vérifier `Settings → Pages → Source`|
|Erreur d'environnement protégé|Environnement Pages existant|Supprimer dans `Settings → Environments`, relancer le workflow|
|Push refusé (authentification)|Mot de passe au lieu de token|Utiliser un Personal Access Token|
|Symlink "cassé" après clone sur une autre machine|Symlink non versionné par Git|Recréer le symlink manuellement (étape 2)|
|Liens internes cassés|`markdownLinkResolution` mal réglé|Aligner avec le format de lien Obsidian (`shortest`)|

## Notes complémentaires

- Repo **public** recommandé → GitHub Actions gratuit et illimité
- Vault et repo Quartz restent indépendants sur le disque grâce au symlink
- Aucune modification nécessaire sur les chapitres déjà rédigés (wikilinks, callouts, surlignage `==texte==` tous nativement supportés)