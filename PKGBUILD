# Maintainer: Dave Reisner <dreisner@archlinux.org>
# Contributor: Angel Velasquez <angvp@archlinux.org>
# Contributor: Eric Belanger <eric@archlinux.org>
# Contributor: Lucien Immink <l.immink@student.fnt.hvu.nl>
# Contributor: Daniel J Griffiths <ghost1227@archlinux.us>

pkgbase=curl
pkgname=(curl libcurl-compat libcurl-gnutls)
pkgver=7.84.0
pkgrel=2
pkgdesc='An URL retrieval utility and library'
arch=('x86_64')
url='https://curl.haxx.se'
license=('MIT')
options=('debug')
depends=('ca-certificates' 'brotli' 'libbrotlidec.so' 'krb5' 'libgssapi_krb5.so'
         'libidn2' 'libidn2.so' 'libnghttp2' 'libpsl' 'libpsl.so' 'libssh2' 'libssh2.so'
         'openssl' 'zlib' 'zstd' 'libzstd.so')
provides=('libcurl.so')
source=("https://curl.haxx.se/download/${pkgname}-${pkgver}.tar.gz"{,.asc}
        '0001-select-do-not-return-fatal-error-on-EINTR-from-poll.patch')
sha512sums=('8133baf48dfd93531ce0a226b54cb153fd58bb0c1ffe8159cee0c0aa23ce210192c572e8ee01f3d75a87b609a580e76929df1e66635be59c177b0cb8076043b2'
            'SKIP'
            '3d771caae23f4b602e57788c61d3f4eaccbd3a73e7256ef7dc2699e574554246280afde1f9eac10c54a498aa0d25de6e696ea36f65389e45ec8ec9abcb3bfb88')
validpgpkeys=('27EDEAF22F3ABCEB50DB9A125CC908FDB71E12C2') # Daniel Stenberg

_configure_options=(
  --prefix='/usr'
  --mandir='/usr/share/man'
  --disable-ldap
  --disable-ldaps
  --disable-manual
  --enable-ipv6
  --enable-threaded-resolver
  --with-gssapi
  --with-libssh2
  --with-openssl
  --with-random='/dev/urandom'
  --with-ca-bundle='/etc/ssl/certs/ca-certificates.crt'
)

prepare() {
  cd "${srcdir}/${pkgbase}-${pkgver}"

  patch -Np1 < ../0001-select-do-not-return-fatal-error-on-EINTR-from-poll.patch
}

build() {
  mkdir build-curl{,-compat,-gnutls}

  # build curl
  cd "${srcdir}"/build-curl

  "${srcdir}/${pkgbase}-${pkgver}"/configure \
    "${_configure_options[@]}" \
    --enable-versioned-symbols
  sed -i -e 's/ -shared / -Wl,-O1,--as-needed\0/g' libtool
  make

  # build libcurl-compat
  cd "${srcdir}"/build-curl-compat

  "${srcdir}/${pkgbase}-${pkgver}"/configure \
    "${_configure_options[@]}" \
    --disable-versioned-symbols
  sed -i -e 's/ -shared / -Wl,-O1,--as-needed\0/g' libtool
  make -C lib

  # build libcurl-gnutls
  cd "${srcdir}"/build-curl-gnutls

  "${srcdir}/${pkgbase}-${pkgver}"/configure \
    "${_configure_options[@]}" \
    --disable-versioned-symbols \
    --without-ssl \
    --with-gnutls='/usr'
  sed -i -e 's/ -shared / -Wl,-O1,--as-needed\0/g' libtool
  make -C lib
}

package_curl() {
  cd build-curl

  make DESTDIR="${pkgdir}" install
  make DESTDIR="${pkgdir}" install -C scripts

  cd "${srcdir}/${pkgname}-${pkgver}"

  # license
  install -Dt "${pkgdir}/usr/share/licenses/$pkgname" -m0644 COPYING
}

package_libcurl-compat() {
  pkgdesc='An URL retrieval library (without versioned symbols)'
  depends=('curl' 'openssl')

  cd "${srcdir}"/build-curl-compat

  make -C lib DESTDIR="${pkgdir}" install

  mv "${pkgdir}"/usr/lib/libcurl{,-compat}.so.4.8.0
  rm "${pkgdir}"/usr/lib/libcurl.{a,so}*
  for version in 3 4.0.0 4.1.0 4.2.0 4.3.0 4.4.0 4.5.0 4.6.0 4.7.0; do
    ln -s libcurl-compat.so.4.8.0 "${pkgdir}"/usr/lib/libcurl.so.${version}
  done

  install -dm 0755 "${pkgdir}"/usr/share/licenses
  ln -s curl "${pkgdir}"/usr/share/licenses/libcurl-compat
}

package_libcurl-gnutls() {
  pkgdesc='An URL retrieval library (without versioned symbols and linked against gnutls)'
  depends=('curl' 'gnutls')

  cd "${srcdir}"/build-curl-gnutls

  make -C lib DESTDIR="${pkgdir}" install

  mv "${pkgdir}"/usr/lib/libcurl{,-gnutls}.so.4.8.0
  rm "${pkgdir}"/usr/lib/libcurl.{a,so}*
  for version in 3 4 4.0.0 4.1.0 4.2.0 4.3.0 4.4.0 4.5.0 4.6.0 4.7.0; do
    ln -s libcurl-gnutls.so.4.8.0 "${pkgdir}"/usr/lib/libcurl-gnutls.so.${version}
  done

  install -dm 0755 "${pkgdir}"/usr/share/licenses
  ln -s curl "${pkgdir}"/usr/share/licenses/libcurl-gnutls
}
