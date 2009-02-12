# Contributor: Jakub Schmidtke <sjakub@gmail.com>

pkgname=firefox
pkgver=3.0.6
pkgrel=1
pkgdesc="Standalone web browser from mozilla.org"
arch=(i686 x86_64)
license=('MPL' 'GPL' 'LGPL')
depends=('xulrunner=1.9.0.6' 'desktop-file-utils' 'shared-mime-info')
makedepends=('zip' 'pkgconfig' 'diffutils' 'libgnomeui>=2.24.0' 'python' 'xorg-server')
replaces=('firefox3')
install=firefox.install
url="http://www.mozilla.org/projects/firefox"
source=(http://releases.mozilla.org/pub/mozilla.org/firefox/releases/${pkgver}/source/firefox-${pkgver}-source.tar.bz2
        mozconfig
        firefox.desktop
        firefox-safe.desktop
        mozilla-firefox-1.0-lang.patch
	mozbug421977.patch
	firefox-appversion.patch)
md5sums=('1b6063d29fe9785b929a36f249866360'
         '8b6e5f7d0a9e3f64747a024cf8f12069'
         '68cf02788491c6e846729b2f2913bf79'
         '5e68cabfcf3c021806b326f664ac505e'
         'bd5db57c23c72a02a489592644f18995'
         '7976e3ff52e01af3388dfc3a479c4955'
         'c6f27fca2e6bd2a570b271ec3ce35782')

build() {
  cd ${srcdir}/mozilla

  patch -Np1 -i ${srcdir}/mozilla-firefox-1.0-lang.patch || return 1

  # FS#10836: fixes backgroundcolor parsing with gnome
  patch -Np0 -i ${srcdir}/mozbug421977.patch || return 1

  patch -Np1 -i ${srcdir}/firefox-appversion.patch || return 1

  cp ${srcdir}/mozconfig .mozconfig

  unset CFLAGS
  unset CXXFLAGS

  export LDFLAGS="-Wl,-rpath,/usr/lib/firefox-3.0"

  LD_PRELOAD="" /usr/bin/Xvfb -nolisten tcp -extension GLX :99 &
  XPID=$!
  export DISPLAY=:99

  LD_PRELOAD="" make -j1 -f client.mk profiledbuild MOZ_MAKE_FLAGS="${MAKEFLAGS}" || return 1
  kill $XPID

  make -j1 DESTDIR=${pkgdir} -C ff-opt-obj install || return 1

  rm -f ${pkgdir}/usr/lib/firefox-3.0/libjemalloc.so

  install -m755 -d ${pkgdir}/usr/share/applications
  install -m755 -d ${pkgdir}/usr/share/pixmaps
  install -m644 ${srcdir}/mozilla/browser/branding/unofficial/default48.png ${pkgdir}/usr/share/pixmaps/firefox.png || return 1
  install -m644 ${srcdir}/firefox.desktop ${pkgdir}/usr/share/applications/ || return 1
  install -m644 ${srcdir}/firefox-safe.desktop ${pkgdir}/usr/share/applications/ || return 1
}
