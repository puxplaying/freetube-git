# Maintainer: picokan <todaysoracle@protonmail.com>

pkgname=freetube-git
_pkgname=FreeTube
_electron=electron42
pkgver=0.24.0.beta.r10231.e25f76a
pkgrel=4
pkgdesc='An open source desktop YouTube player built with privacy in mind - built from git source tree.'
arch=('x86_64' 'i686' 'arm' 'armv6h' 'armv7h' 'aarch64')
url="https://freetubeapp.io"
license=('AGPL-3.0-or-later')
depends=($_electron)
makedepends=('git' 'pnpm')
provides=('freetube')
conflicts=('freetube')
source=(git+https://github.com/FreeTubeApp/FreeTube.git#branch=development
        freetube.desktop
        freetube.sh)
sha256sums=(SKIP SKIP SKIP)

pkgver() {
  cd "$srcdir/$_pkgname"
  printf "%s.r%s.%s" "$(git tag --sort=committerdate | tail -1 | sed 's/^v//;s/\([^-]*-g\)/r\1/;s/-/./g')" "$(git rev-list --count HEAD)" "$(git rev-parse --short=7 HEAD)"
}

prepare() {
  cd "$srcdir/$_pkgname"
  
  # Create pnpm-workspace.yaml to allow build scripts for ALL required dependencies
  # pnpm v11+ requires this YAML file; .npmrc is ignored for allowBuilds
  cat > pnpm-workspace.yaml <<EOF
allowBuilds:
  '@parcel/watcher': true
  'electron-winstaller': true
  'lefthook': true
  'unrs-resolver': true
EOF

  # Existing patches
  sed -i "5i electronDist: '/usr/lib/$_electron'," "$srcdir/$_pkgname/_scripts/ebuilder.config.mjs"
  sed -i "s/targets = Platform.LINUX.*/targets = Platform.LINUX.createTarget(['dir'], arch)/" "$srcdir/$_pkgname/_scripts/build.mjs"
  sed -i "s/_electron_/$_electron/" "$srcdir/freetube.sh"
}

build() {
  cd "$srcdir/$_pkgname"
  
  # Ensure the workspace config exists (redundant safety check)
  if [ ! -f pnpm-workspace.yaml ]; then
    cat > pnpm-workspace.yaml <<EOF
allowBuilds:
  '@parcel/watcher': true
  'electron-winstaller': true
  'lefthook': true
  'unrs-resolver': true
EOF
  fi

  # CRITICAL: Use --no-frozen-lockfile to allow pnpm to accept the new allowBuilds config
  # The Arch build env is detected as CI, which forces --frozen-lockfile by default.
  pnpm install --no-frozen-lockfile
  
  pnpm run build
}

package() {
  install -d "${pkgdir}"/{usr/bin,usr/lib/freetube-git}
  
  if [ ! -d "./$_pkgname/build/linux-unpacked/resources" ]; then
    echo "Error: Build output not found. Check build logs."
    return 1
  fi
  
  cp -R "./$_pkgname/build/linux-unpacked/resources/app.asar" "$pkgdir/usr/lib/$pkgname"
  install -Dm755 "./freetube.sh" "$pkgdir/usr/bin/freetube"
  
  cd $_pkgname
  install -Dm644 LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
  install -Dm644 "./_icons/icon.svg" "$pkgdir/usr/share/pixmaps/freetube.svg"
  cd ..
  install -Dm644 "freetube.desktop" "$pkgdir/usr/share/applications/$pkgname.desktop"
}   
