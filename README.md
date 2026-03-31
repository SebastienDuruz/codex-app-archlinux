# Codex App Packaging for Arch Linux

This repository provides an Arch Linux package build for Codex Desktop (`x86_64`) from the macOS DMG source.

## Scope

- Target distro: Arch Linux
- Target architecture: `x86_64`
- Package format: pacman package (`.pkg.tar.zst`)

## Prerequisites

Install base packaging tools:

```bash
sudo pacman -S --needed base-devel git
```

`makepkg -s` installs package dependencies declared in `PKGBUILD` automatically.

Network access is required to download:

- `Codex.dmg`
- `better-sqlite3` npm tarball
- `node-pty` npm tarball

## Build Workflow

From repository root:

```bash
makepkg -si
```

## Run

```bash
codex-app
```

## Output Layout

- Build workspace: `src/`, `pkg/`
- Built package: `*.pkg.tar.zst` (repo root)

## Update Process

When updating package inputs:

1. Update `pkgver`, source URLs/versions, and checksums in `PKGBUILD`.
2. Regenerate checksums if needed:

```bash
updpkgsums
```

3. Regenerate `.SRCINFO`:

```bash
makepkg --printsrcinfo > .SRCINFO
```

4. Rebuild:

```bash
makepkg -f
```