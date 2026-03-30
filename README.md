# Codex App macOS -> Arch Linux (x86_64)

Ce dossier contient un portage local de l'application Codex Desktop macOS (`Codex.dmg`) vers Arch Linux AMD64 (`x86_64`) via `makepkg`.

## Prerequis

- Arch Linux x86_64
- `base-devel`
- Acces reseau pour telecharger:
  - `Codex.dmg`
  - archives npm `better-sqlite3` et `node-pty`

## Installation

```bash
cd /home/sebastien/Applications/codex-app
makepkg -si
```

## Lancement

```bash
codex-app
```

## Notes techniques

- L'app macOS est extraite depuis le DMG (`7z`), puis `app.asar` est decompresse.
- Les modules natifs (`better-sqlite3`, `node-pty`) sont recompiles pour Linux et Electron 39.
- Le resultat est empaquete dans `app.asar` et installe avec un launcher Linux.

## Workflow Git minimal (push propre)

Objectif: pousser uniquement les fichiers sources utiles (`PKGBUILD`, launcher, desktop file, docs, `.SRCINFO`).

### 1) Nettoyer l'index Git des artefacts lourds (sans supprimer les fichiers locaux)

```bash
cd /home/sebastien/Applications/codex-app
git rm -r --cached --ignore-unmatch src pkg Codex.dmg better-sqlite3-*.tgz node-pty-*.tgz *.pkg.tar.zst *.pkg.tar.zst.sig
```

### 2) Verifier que `.gitignore` couvre bien les artefacts

Le `.gitignore` de ce repo ignore deja:
- `src/`
- `pkg/`
- `Codex.dmg`
- `better-sqlite3-*.tgz`
- `node-pty-*.tgz`
- `*.pkg.tar.zst`
- `*.pkg.tar.zst.sig`

### 3) Commit minimal

```bash
git add .gitignore README.md PKGBUILD .SRCINFO codex-app.sh codex-app.desktop
git commit -m "chore: keep only packaging sources and docs"
```

### 4) Push

```bash
git push
```
