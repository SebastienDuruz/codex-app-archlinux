# Maintainer: Local package build

pkgname=codex-app
pkgver=1.0.3
pkgrel=1
pkgdesc="Linux package of Codex Desktop"
arch=('x86_64')
url="https://developers.openai.com/codex/app/"
license=('custom')

depends=(
  'electron39'
  'python'
  'hicolor-icon-theme'
  'xdg-utils'
)

makedepends=(
  'libicns'
  'nodejs'
  'npm'
  'p7zip'
)

_electron_major=39
_better_sqlite3_ver=12.5.0
_node_pty_ver=1.1.0

source=(
  "Codex.dmg::https://persistent.oaistatic.com/codex-app-prod/Codex.dmg"
  "better-sqlite3-${_better_sqlite3_ver}.tgz::https://registry.npmjs.org/better-sqlite3/-/better-sqlite3-${_better_sqlite3_ver}.tgz"
  "node-pty-${_node_pty_ver}.tgz::https://registry.npmjs.org/node-pty/-/node-pty-${_node_pty_ver}.tgz"
  "codex-app.sh"
  "codex-app.desktop"
)

noextract=(
  'Codex.dmg'
  "better-sqlite3-${_better_sqlite3_ver}.tgz"
  "node-pty-${_node_pty_ver}.tgz"
)

sha256sums=('681a12ac481a3100fbf402c51e156830c65a0bf6f6cc51885e68df74686bdb7b'
            '0a3cd0554b063c3185b9912ef7059b84455a2e411d637faa0166fef9fefa04c2'
            'c7517f19083ddcb05f276904680eb2b11a6b5ecab778b8e4e5685a6d645b3f60'
            'c6c7a3f61e963020d1cfeb1b6f56e42d89b98e4c0e8f9af73a84ade8b518ff59'
            '7d4460887df563d7fd5465db0ff950fb9a0b119556c9d659302359f8b12c6a7a')

prepare() {
  cd "${srcdir}"
  rm -rf dmg app-extracted app.asar app.asar.unpacked native-build
  mkdir dmg

  7z x -y "Codex.dmg" -o"${srcdir}/dmg" >/dev/null

  icon_icns="$(find dmg -path '*/Contents/Resources/*.icns' | head -n1)"
  mkdir -p icon
  icns2png -x -o icon "${icon_icns}"

  local appdir
  appdir="$(find dmg -maxdepth 4 -type d -name '*.app' | head -n1)"
  [[ -n "${appdir}" ]] || {
    echo "Could not find .app bundle in DMG"
    return 1
  }

  npx --yes asar extract \
    "${appdir}/Contents/Resources/app.asar" \
    app-extracted

  [[ -d "${appdir}/Contents/Resources/app.asar.unpacked" ]] &&
    cp -a "${appdir}/Contents/Resources/app.asar.unpacked" .

  rm -rf app-extracted/node_modules/sparkle-darwin
  find app-extracted -type f \( -name '*.dylib' -o -name 'sparkle.node' \) -delete

  local bs3_ver npty_ver
  bs3_ver="$(node -p "require('${srcdir}/app-extracted/node_modules/better-sqlite3/package.json').version")"
  npty_ver="$(node -p "require('${srcdir}/app-extracted/node_modules/node-pty/package.json').version")"

  [[ "${bs3_ver}" == "${_better_sqlite3_ver}" ]] || {
    echo "better-sqlite3 version mismatch: app=${bs3_ver}, pkgbuild=${_better_sqlite3_ver}"
    return 1
  }

  [[ "${npty_ver}" == "${_node_pty_ver}" ]] || {
    echo "node-pty version mismatch: app=${npty_ver}, pkgbuild=${_node_pty_ver}"
    return 1
  }

  mkdir native-build
  cd native-build

  cat >package.json <<'EOF'
{
  "name": "codex-native-rebuild",
  "private": true,
  "license": "UNLICENSED"
}
EOF

  npm install \
    --ignore-scripts \
    --no-audit \
    --no-fund \
    "${srcdir}/better-sqlite3-${_better_sqlite3_ver}.tgz" \
    "${srcdir}/node-pty-${_node_pty_ver}.tgz"

  export npm_config_runtime=electron
  export npm_config_target="${_electron_major}.0.0"
  export npm_config_disturl="https://electronjs.org/headers"
  export npm_config_build_from_source=true

  npx --yes @electron/rebuild -v "${_electron_major}.0.0" --force

  rm -rf "${srcdir}/app-extracted/node_modules/better-sqlite3"
  rm -rf "${srcdir}/app-extracted/node_modules/node-pty"
  cp -a node_modules/better-sqlite3 "${srcdir}/app-extracted/node_modules/"
  cp -a node_modules/node-pty "${srcdir}/app-extracted/node_modules/"

  cd "${srcdir}"
  npx --yes asar pack app-extracted app.asar --unpack "{*.node,*.so}"
}

package() {
  cd "${srcdir}"

  install -Dm644 app.asar \
    "${pkgdir}/usr/lib/${pkgname}/resources/app.asar"

  if [[ -d app.asar.unpacked ]]; then
    cp -a app.asar.unpacked \
      "${pkgdir}/usr/lib/${pkgname}/resources/"
  fi

  if [[ -d app-extracted/webview ]]; then
    mkdir -p "${pkgdir}/usr/lib/${pkgname}/content"
    cp -a app-extracted/webview \
      "${pkgdir}/usr/lib/${pkgname}/content/"
  fi

  install -Dm755 codex-app.sh \
    "${pkgdir}/usr/bin/codex-app"

  install -Dm644 icon/*512x512*.png \
    "${pkgdir}/usr/share/icons/hicolor/512x512/apps/codex-app.png"

  install -Dm644 codex-app.desktop \
    "${pkgdir}/usr/share/applications/codex-app.desktop"
}
