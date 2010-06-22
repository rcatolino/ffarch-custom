# Contributor: Jakub Schmidtke <sjakub@gmail.com>

pkgname=firefox
pkgver=3.6.4
pkgrel=1
_xulver=1.9.2.4
pkgdesc="Standalone web browser from mozilla.org"
arch=('i686' 'x86_64')
license=('MPL' 'GPL' 'LGPL')
depends=("xulrunner=${_xulver}" 'desktop-file-utils')
makedepends=('zip' 'pkg-config' 'diffutils' 'libgnomeui>=2.24.1' 'python' 'wireless_tools')
install=firefox.install
url="http://www.mozilla.org/projects/firefox"
source=(http://releases.mozilla.org/pub/mozilla.org/firefox/releases/${pkgver}/source/firefox-${pkgver}.source.tar.bz2
        mozconfig
        firefox.desktop
        firefox-safe.desktop
        mozilla-firefox-1.0-lang.patch
        browser-defaulturls.patch
        firefox-version.patch
	firefox-agent.patch)
md5sums=('66b1610362ed6009d66fe762645679a0'
         '1e4283a24953f8d0c6a5c00a36ae0981'
         'ba96924ece1d77453e462429037a2ce5'
         '6f38a5899034b7786cb1f75ad42032b8'
         'bd5db57c23c72a02a489592644f18995'
         '1807651225b021e043154f8bba715a19'
         '92c11c66dd69b03f214002fededd1fc8'
         'f437e94acff8f810991271ef4677d859')

build() {
  cd "${srcdir}/mozilla-1.9.2"
  patch -Np1 -i "${srcdir}/mozilla-firefox-1.0-lang.patch"
  patch -Np1 -i "${srcdir}/browser-defaulturls.patch"
  patch -Np1 -i "${srcdir}/firefox-version.patch"
  patch -Np1 -i "${srcdir}/firefox-agent.patch"

  cp "${srcdir}/mozconfig" .mozconfig
  unset CFLAGS
  unset CXXFLAGS

  export LDFLAGS="-Wl,-rpath,/usr/lib/firefox-3.6"

  make -j1 -f client.mk build MOZ_MAKE_FLAGS="${MAKEFLAGS}"
  make -j1 DESTDIR="${pkgdir}" install

  rm -f ${pkgdir}/usr/lib/firefox-3.6/libjemalloc.so

  install -m755 -d ${pkgdir}/usr/share/{applications,pixmaps}
  install -m644 ${srcdir}/mozilla-1.9.2/browser/branding/unofficial/default48.png ${pkgdir}/usr/share/pixmaps/firefox.png
  install -m644 ${srcdir}/firefox.desktop ${pkgdir}/usr/share/applications/
  install -m644 ${srcdir}/firefox-safe.desktop ${pkgdir}/usr/share/applications/
}
