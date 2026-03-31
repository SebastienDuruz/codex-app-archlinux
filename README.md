# Codex App macOS to Arch Linux Package

This repository packages the macOS Codex Desktop app for Arch Linux (`x86_64`) using `makepkg`.

## Requirements

- Arch Linux (`x86_64`)
- `base-devel`
- Network access to download:
  - `Codex.dmg`
  - npm tarballs for `better-sqlite3` and `node-pty`

## Build and Install

From the repository root:

```bash
makepkg -si
```

## Run

```bash
codex-app
```

## Build Notes

- The package extracts the app from `Codex.dmg`.
- Native modules (`better-sqlite3`, `node-pty`) are rebuilt for Linux and Electron 39.
- The final package installs a Linux launcher and desktop entry.
