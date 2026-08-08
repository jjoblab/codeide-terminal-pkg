# CodeIDE Packages

Fork de [termux/termux-packages](https://github.com/termux/termux-packages)
pour l'application **CodeIDE** (`jo.codeide`).

Ce repo contient les scripts et patches pour compiler les paquets CodeIDE
depuis les sources, avec `TERMUX_APP_PACKAGE=jo.codeide` — tous les chemins
dans les binaires ELF pointent nativement vers `/data/data/jo.codeide/...`,
sans binary-patch runtime.

## Différences avec upstream Termux

- **Package name** : `com.termux` → `jo.codeide` (cf. `scripts/properties.sh`)
- **Dépôt APT** : publie vers GitHub Pages
  `https://jjoblab.github.io/codeide-terminal-pkg/apt/codeide-main` au lieu
  de `packages-cf.termux.dev`
- **Bootstrap** : utilise `build-bootstraps.sh` (build natif depuis les
  sources) au lieu de `generate-bootstraps.sh` (qui télécharge des `.deb`
  précompilés Termux)
- **Workflows** : simplifiés pour CodeIDE — `bootstrap_archives.yml` +
  `build-packages.yml` uniquement. Les workflows CodeQL, dependabot,
  golang/zig validation, etc. ont été supprimés.

## Workflows GitHub Actions

### `bootstrap_archives.yml`
Build le bootstrap (archive zip embarquée au premier lancement de CodeIDE).
- Compile depuis les sources via `build-bootstraps.sh` + `run-docker.sh`
- Architectures : `aarch64`, `arm`
- Publie une Release GitHub avec les zips + SHA256
- Déclencheur : `workflow_dispatch` ou push d'un tag `bootstrap-*`

### `build-packages.yml`
Build les paquets listés dans `packages.txt` et publie le dépôt APT sur
GitHub Pages.
- Compile depuis les sources via `build-package.sh` (sans `-I` — pas de
  téléchargement de `.deb` précompilés)
- Architectures : `aarch64`, `arm`
- Publie via `aptly` vers `gh-pages` branch → GitHub Pages
- Déclencheur : `workflow_dispatch`, push d'un tag `packages-*`, ou cron
  hebdomadaire (dimanche 00:00 UTC)

## Utilisation

### Build le bootstrap
1. Va dans l'onglet **Actions** → **Build Bootstrap Archives** → **Run workflow**
2. Attends 30–90 min par arch (compilation NDK de ~30 paquets)
3. Une Release GitHub est créée automatiquement avec les zips

### Build les paquets
1. Modifie `packages.txt` selon tes besoins
2. Va dans l'onglet **Actions** → **Build Packages & Publish APT Repo** → **Run workflow**
3. Attends plusieurs heures (dépend du nombre de paquets)
4. Le dépôt APT est publié sur GitHub Pages

### Configurer CodeIDE pour utiliser le dépôt
Une fois le dépôt APT publié, configure CodeIDE avec :
```
pkg install wget
echo "deb https://jjoblab.github.io/codeide-terminal-pkg/apt/codeide-main stable main" > $PREFIX/etc/apt/sources.list.d/codeide.list
pkg update
pkg install <paquet>
```

## Structure du repo

```
.
├── build-package.sh          # Script principal pour compiler un paquet
├── packages.txt              # Liste des paquets à compiler (pour build-packages.yml)
├── repo.json                 # Configuration des dépôts APT cibles
├── scripts/
│   ├── properties.sh         # Variables globales (TERMUX_APP_PACKAGE=jo.codeide)
│   ├── build-bootstraps.sh   # Build natif du bootstrap depuis les sources
│   ├── generate-bootstraps.sh # Build du bootstrap depuis .deb précompilés (PAS UTILISÉ)
│   ├── run-docker.sh         # Lance un build dans ghcr.io/termux/package-builder
│   └── build/                # 92 fichiers termux_*.sh sourcés par build-package.sh
├── packages/                 # ~1850 recettes de paquets
├── x11-packages/             # Paquets X11
├── root-packages/            # Paquets root
└── disabled-packages/        # Paquets désactivés
```

## Crédits

Basé sur [termux/termux-packages](https://github.com/termux/termux-packages)
sous licence GNU GPL v3 (cf. `LICENSE.md`).
