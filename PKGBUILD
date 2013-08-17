# Maintainer : Ionut Biru <ibiru@archlinux.org>
# Contributor: Jakub Schmidtke <sjakub@gmail.com>

pkgname=firefox
pkgver=23.0.1
pkgrel=1
pkgdesc="Standalone web browser from mozilla.org"
arch=('i686' 'x86_64')
license=('MPL' 'GPL' 'LGPL')
url="https://www.mozilla.org/firefox/"
depends=('gtk2' 'mozilla-common' 'libxt' 'startup-notification' 'mime-types'
         'dbus-glib' 'alsa-lib' 'libvpx' 'libevent' 'nss' 'hunspell' 'sqlite'
          'libnotify' 'desktop-file-utils' 'hicolor-icon-theme')
makedepends=('unzip' 'zip' 'diffutils' 'python2' 'yasm' 'mesa' 'imake'
             'xorg-server-xvfb' 'libpulse')
optdepends=('networkmanager: Location detection via available WiFi networks'
            'libpulse: PulseAudio audio driver')
install=firefox.install
options=('!emptydirs' '!makeflags')
source=(https://ftp.mozilla.org/pub/mozilla.org/firefox/releases/$pkgver/source/firefox-$pkgver.source.tar.bz2
        mozconfig firefox.desktop firefox-install-dir.patch vendor.js shared-libs.patch
        firefox-20.0.1-fixed-loading-icon.png)
sha256sums=('bb2c2e99a03859ebd8c02b8bc4c57b39ccc97c55872c2737c433212c0ebe01cf'
            '1d750088958ae2a2faf43f9bb3909c3e396f01c6504ba39f09b8ed7fc1b29492'
            'd2a7610393ba259c35e3227b92e02ec91095a95189f56ac93ccdf6732772719c'
            'ded67e8204bd5e1c0c5771c0d2c84ff80c998e1543711e7cd804cfe29e8dd1b0'
            '4b50e9aec03432e21b44d18c4c97b2630bace606b033f7d556c9d3e3eb0f4fa4'
            'e2b4a00d14f4ba69c62b3f9ef9908263fbab179ba8004197cbc67edbd916fdf1'
            '68e3a5b47c6d175cc95b98b069a15205f027cab83af9e075818d38610feb6213')

prepare() {
  cd mozilla-release

  cp ../mozconfig .mozconfig
  patch -Np1 -i ../firefox-install-dir.patch
  patch -Np1 -i ../shared-libs.patch

  # Fix PRE_RELEASE_SUFFIX
  sed -i '/^PRE_RELEASE_SUFFIX := ""/s/ ""//' \
    browser/base/Makefile.in

  mkdir "$srcdir/path"

  # WebRTC build tries to execute "python" and expects Python 2
  ln -s /usr/bin/python2 "$srcdir/path/python"

  # configure script misdetects the preprocessor without an optimization level
  # https://bugs.archlinux.org/task/34644
  sed -i '/ac_cpp=/s/$CPPFLAGS/& -O2/' configure

  # Fix tab loading icon (flickers with libpng 1.6)
  # https://bugzilla.mozilla.org/show_bug.cgi?id=841734
  cp "$srcdir/firefox-20.0.1-fixed-loading-icon.png" \
    browser/themes/linux/tabbrowser/loading.png
}

build() {
  cd mozilla-release

  export PATH="$srcdir/path:$PATH"
  export LDFLAGS="$LDFLAGS -Wl,-rpath,/usr/lib/firefox"
  export PYTHON="/usr/bin/python2"

  # Work around memory address space exhaustion during linking on i686
  if [[ $CARCH == i686 ]]; then
    LDFLAGS+=' -Wl,--no-keep-memory'
  fi

  export DISPLAY=:99
  Xvfb -nolisten tcp -extension GLX -screen 0 1280x1024x24 $DISPLAY &

  if ! make -f client.mk build MOZ_PGO=1; then
    kill $!
    return 1
  fi

  kill $! || true
}

package() {
  cd mozilla-release
  make -f client.mk DESTDIR="$pkgdir" install

  install -Dm644 ../vendor.js "$pkgdir/usr/lib/firefox/browser/defaults/preferences/vendor.js"

  for i in 16 22 24 32 48 256; do
      install -Dm644 browser/branding/official/default$i.png \
        "$pkgdir/usr/share/icons/hicolor/${i}x${i}/apps/firefox.png"
  done
  install -Dm644 browser/branding/official/content/icon64.png \
    "$pkgdir/usr/share/icons/hicolor/64x64/apps/firefox.png"
  install -Dm644 browser/branding/official/mozicon128.png \
    "$pkgdir/usr/share/icons/hicolor/128x128/apps/firefox.png"
  install -Dm644 browser/branding/official/content/about-logo.png \
    "$pkgdir/usr/share/icons/hicolor/210x210/apps/firefox.png"

  install -Dm644 ../firefox.desktop \
    "$pkgdir/usr/share/applications/firefox.desktop"

  # Use system-provided dictionaries
  rm -rf "$pkgdir"/usr/lib/firefox/{dictionaries,hyphenation}
  ln -s /usr/share/hunspell "$pkgdir/usr/lib/firefox/dictionaries"
  ln -s /usr/share/hyphen "$pkgdir/usr/lib/firefox/hyphenation"

  # We don't want the development stuff
  rm -r "$pkgdir"/usr/{include,lib/firefox-devel,share/idl}

  #workaround for now
  #https://bugzilla.mozilla.org/show_bug.cgi?id=658850
  ln -sf firefox "$pkgdir/usr/lib/firefox/firefox-bin"
}
