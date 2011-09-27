# Contributor: Jakub Schmidtke <sjakub@gmail.com>

pkgname=firefox
pkgver=7.0
pkgrel=1
pkgdesc="Standalone web browser from mozilla.org"
arch=('i686' 'x86_64')
license=('MPL' 'GPL' 'LGPL')
depends=('gtk2' 'mozilla-common' 'nss' 'libxt' 'hunspell' 'startup-notification' 'mime-types' 'dbus-glib' 'alsa-lib' 'sqlite3' 'libnotify' 'desktop-file-utils' 'libvpx' 'libevent' 'hicolor-icon-theme')
makedepends=('unzip' 'zip' 'pkg-config' 'diffutils' 'python2' 'wireless_tools' 'yasm' 'mesa' 'autoconf2.13' 'libidl2' 'xorg-server-xvfb')
url="http://www.mozilla.org/projects/firefox"
install=firefox.install
source=(ftp://ftp.mozilla.org/pub/mozilla.org/firefox/releases//${pkgver}/source/firefox-${pkgver}.source.tar.bz2
        mozconfig firefox.desktop mozilla-firefox-1.0-lang.patch)
md5sums=('673a015a23e3fefa18d80db68c8b9fc1'
         'd88607e98806db2a7eb222b0c8635af0'
         'bdeb0380c7fae30dd0ead6d2d3bc5873'
         'bd5db57c23c72a02a489592644f18995')

build() {
  cd "$srcdir/mozilla-release"

  cp "$srcdir/mozconfig" .mozconfig
  patch -Np1 -i "$srcdir/mozilla-firefox-1.0-lang.patch"

  # Fix PRE_RELEASE_SUFFIX
  sed -i '/^PRE_RELEASE_SUFFIX := ""/s/ ""//' \
      browser/base/Makefile.in

  export LDFLAGS="-Wl,-rpath,/usr/lib/firefox-$pkgver -Wl,-O1,--sort-common,--hash-style=gnu,--as-needed"
  export PYTHON="/usr/bin/python2"

  # PGO
  sed -i '/^NO_PROFILE_GUIDED_OPTIMIZE = 1$/d' \
    memory/jemalloc/Makefile.in
  echo 'LDFLAGS += -lX11 -lXrender' \
    >> layout/build/Makefile.in

  LD_PRELOAD="" /usr/bin/Xvfb -nolisten tcp -extension GLX -screen 0 1280x1024x24 :99 &
  LD_PRELOAD="" DISPLAY=:99 make -j1 -f client.mk profiledbuild MOZ_MAKE_FLAGS="$MAKEFLAGS"
  kill $! || true
}

package() {
  cd "$srcdir/mozilla-release"
  make -j1 -f client.mk DESTDIR="$pkgdir" install

  for i in 16x16 22x22 24x24 32x32 48x48 256x256; do
      install -Dm644 browser/branding/official/default${i/x*/}.png \
        "$pkgdir/usr/share/icons/hicolor/$i/apps/firefox.png"
  done

  install -Dm644 "$srcdir/firefox.desktop" \
    "$pkgdir/usr/share/applications/firefox.desktop"

  rm -rf "$pkgdir"/usr/lib/firefox-$pkgver/{dictionaries,hyphenation}
  ln -sf /usr/share/hunspell "$pkgdir/usr/lib/firefox-$pkgver/dictionaries"
  ln -sf /usr/share/hyphen "$pkgdir/usr/lib/firefox-$pkgver/hyphenation"

  # We don't want the development stuff
  rm -r "$pkgdir"/usr/{include,lib/firefox-devel-$pkgver,share/idl}
}
