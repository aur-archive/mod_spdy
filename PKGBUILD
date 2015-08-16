# Maintainer: André R. de Miranda <ardemiranda@gmail.com>
pkgname=mod_spdy
pkgver=0.9.4.3
pkgrel=1
pkgdesc="SPDY module for Apache 2.2 that allows your web server to take advantage of SPDY features like stream multiplexing and header compression."
arch=('i686' 'x86_64')
url="http://code.google.com/p/mod-spdy/"
license=('APACHE')
makedepends=('svn' 'curl' 'depot_tools-svn' 'python2')
depends=('apache' 'depot_tools-svn')
#backup=('etc/httpd/conf/extra/spdy.conf')


source=()
sha512sums=(
)

build() {
   cd "${srcdir}"

   # python2
   PATH_PYTHON_HACK="${srcdir}/python_hack"
   [ -d $PATH_PYTHON_HACK ] || mkdir $PATH_PYTHON_HACK
   ln -sf /usr/bin/python2 "$PATH_PYTHON_HACK/python"
   OLD_PATH=$PATH
   PATH="${PATH_PYTHON_HACK}:${PATH}"

   GCLIENT="/opt/depot_tools-svn/gclient"

  # checkout
  if [ -d ${pkgname}-${pkgver} ] 
  then
    cd "${pkgname}-${pkgver}"
    $GCLIENT revert
    [ -f src/mod_ssl.so ] && rm -f src/mod_ssl.so;
    
  else
    mkdir "${pkgname}-${pkgver}"
    cd "${pkgname}-${pkgver}"
    $GCLIENT config "http://mod-spdy.googlecode.com/svn/tags/${pkgver}/src"
    $GCLIENT sync --force
  fi

  cd ./src
 
  # mod_ssl_npn
  ./build_modssl_with_npn.sh

  # mod_spdy
  make BUILDTYPE=Release CXXFLAGS="-Wno-unused-local-typedefs -Wno-error" CFLAGS="-Wno-unused-local-typedefs -Wno-error"

  # python2
  PATH=$OLD_PATH
}

package() {
  cd "${pkgname}-${pkgver}/src"

  install -m 644 -D mod_ssl.so $pkgdir/usr/lib/httpd/modules/mod_ssl_npn.so
  install -m 644 -D ./out/Release/libmod_spdy.so $pkgdir/usr/lib/httpd/modules/mod_spdy.so

# /etc/conf
  mkdir -p ${pkgdir}/etc/httpd/conf/extra/
  touch ${pkgdir}/etc/httpd/conf/extra/spdy.conf
  echo "" > ${pkgdir}/etc/httpd/conf/extra/spdy.conf
  echo "LoadModule ssl_module modules/mod_ssl_npn.so" >> ${pkgdir}/etc/httpd/conf/extra/spdy.conf
  echo "LoadModule spdy_module modules/mod_spdy.so" >> ${pkgdir}/etc/httpd/conf/extra/spdy.conf
  echo "Include conf/extra/httpd-ssl.conf" >> ${pkgdir}/etc/httpd/conf/extra/spdy.conf
  echo "SpdyEnabled on" >> ${pkgdir}/etc/httpd/conf/extra/spdy.conf
  echo "" >> ${pkgdir}/etc/httpd/conf/extra/spdy.conf
  echo "#Use SPDY even over non-SSL connections; DO NOT USE IN PRODUCTION" >> ${pkgdir}/etc/httpd/conf/extra/spdy.conf
  echo "SpdyDebugUseSpdyForNonSslConnections off" >> ${pkgdir}/etc/httpd/conf/extra/spdy.conf


}
