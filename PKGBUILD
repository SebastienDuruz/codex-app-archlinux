# Maintainer: Local package build

pkgname=codex-app
pkgver=1.0.89
pkgrel=1
pkgdesc="Linux package of Codex Desktop"
arch=('x86_64')
url="https://developers.openai.com/codex/app/"
license=('MIT')

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

source=(
  "Codex.dmg::https://persistent.oaistatic.com/codex-app-prod/Codex.dmg"
  "codex-app.sh"
  "codex-app.desktop"
)

noextract=(
  'Codex.dmg'
)

sha256sums=('86cdc52ab2c14f8d9535809121b2512e9b89138feaafa6ac697bae8710ea19db'
            'c6c7a3f61e963020d1cfeb1b6f56e42d89b98e4c0e8f9af73a84ade8b518ff59'
            '7d4460887df563d7fd5465db0ff950fb9a0b119556c9d659302359f8b12c6a7a')

prepare() {
  cd "${srcdir}"
  rm -rf dmg-extracted app-extracted app.asar app.asar.unpacked native-build icon
  mkdir -p dmg-extracted

  local dmg_listing appdir icon_icns resource_prefix app_asar_path app_unpacked_prefix
  dmg_listing="$(7z l -slt "Codex.dmg")"

  appdir="$(awk -F' = ' '
    $1 == "Path" && $2 ~ /\.app$/ {
      print $2
      exit
    }
  ' <<<"${dmg_listing}")"
  [[ -n "${appdir}" ]] || {
    echo "Could not find .app bundle in DMG"
    return 1
  }

  resource_prefix="${appdir}/Contents/Resources/"
  app_asar_path="${resource_prefix}app.asar"
  app_unpacked_prefix="${resource_prefix}app.asar.unpacked/"

  icon_icns="$(awk -F' = ' -v prefix="${resource_prefix}" '
    $1 == "Path" && index($2, prefix) == 1 && $2 ~ /\.icns$/ {
      suffix = substr($2, length(prefix) + 1)
      if (suffix !~ /\//) {
        print $2
        exit
      }
    }
  ' <<<"${dmg_listing}")"
  [[ -n "${icon_icns}" ]] || {
    echo "Could not find application icon in ${appdir}/Contents/Resources"
    return 1
  }

  awk -F' = ' -v app_asar_path="${app_asar_path}" '
    $1 == "Path" && $2 == app_asar_path {
      found = 1
      exit
    }
    END {
      exit(found ? 0 : 1)
    }
  ' <<<"${dmg_listing}" || {
    echo "Could not find ${app_asar_path} in DMG"
    return 1
  }

  7z x -y "Codex.dmg" \
    "${app_asar_path}" \
    "${icon_icns}" \
    -o"${srcdir}/dmg-extracted" >/dev/null

  if awk -F' = ' -v prefix="${app_unpacked_prefix}" '
    $1 == "Path" && index($2, prefix) == 1 {
      found = 1
      exit
    }
    END {
      exit(found ? 0 : 1)
    }
  ' <<<"${dmg_listing}"; then
    7z x -y "Codex.dmg" \
      "${app_unpacked_prefix}*" \
      -o"${srcdir}/dmg-extracted" >/dev/null
  fi

  local resource_dir
  resource_dir="${srcdir}/dmg-extracted/${appdir}/Contents/Resources"

  mkdir -p icon
  icns2png -x -o icon "${resource_dir}/$(basename "${icon_icns}")"

  find icon -maxdepth 1 -type f -name '*.png' | grep -q . || {
    echo "No PNG icon was extracted from ${icon_icns}"
    return 1
  }

  npx --yes asar extract \
    "${resource_dir}/app.asar" \
    app-extracted

  [[ -d "${resource_dir}/app.asar.unpacked" ]] &&
    cp -a "${resource_dir}/app.asar.unpacked" .

  rm -rf app-extracted/node_modules/sparkle-darwin
  find app-extracted -type f \( -name '*.dylib' -o -name 'sparkle.node' \) -delete

  local bs3_ver npty_ver
  bs3_ver="$(node -p "require('${srcdir}/app-extracted/node_modules/better-sqlite3/package.json').version")"
  npty_ver="$(node -p "require('${srcdir}/app-extracted/node_modules/node-pty/package.json').version")"

  mkdir native-build
  cd native-build

  cat >package.json <<'EOF_PKG'
{
  "name": "codex-native-rebuild",
  "private": true,
  "license": "UNLICENSED"
}
EOF_PKG

  npm pack --pack-destination "${srcdir}" "better-sqlite3@${bs3_ver}"
  npm pack --pack-destination "${srcdir}" "node-pty@${npty_ver}"

  npm install \
    --ignore-scripts \
    --no-audit \
    --no-fund \
    "${srcdir}/better-sqlite3-${bs3_ver}.tgz" \
    "${srcdir}/node-pty-${npty_ver}.tgz"

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
  local icon_png

  icon_png="$(find icon -maxdepth 1 -type f -name '*.png' | sort -V | tail -n1)"
  [[ -n "${icon_png}" ]] || {
    echo "No extracted PNG icon available for packaging"
    return 1
  }

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

  install -Dm644 "${icon_png}" \
    "${pkgdir}/usr/share/icons/hicolor/512x512/apps/codex-app.png"

  install -Dm644 codex-app.desktop \
    "${pkgdir}/usr/share/applications/codex-app.desktop"
}
