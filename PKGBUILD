#_                   _ _ _  _ _____ _  _
#| | _______   ____ _| | | || |___  | || |
#| |/ / _ \ \ / / _` | | | || |_ / /| || |_
#|   <  __/\ V / (_| | | |__   _/ / |__   _|
#|_|\_\___| \_/ \__,_|_|_|  |_|/_/     |_|

#Maintainer: kevall474 <kevall474@tuta.io> <https://github.com/kevall474>
#Credits: Bartłomiej Piotrowski <bpiotrowski@archlinux.org>
#Credits: Allan McRae <allan@archlinux.org>

# toolchain build order: linux-api-headers->glibc->binutils->gcc->binutils->glibc

pkgname=binutils-git
# grap pkver
#(cd binutils-gdb && cat binutils/configure | grep "PACKAGE_VERSION=")
pkgver=2.36.50
pkgrel=1
pkgdesc='A set of programs to assemble and manipulate binary and object files'
arch=(x86_64)
url='https://www.gnu.org/software/binutils/'
license=(GPL)
groups=('base-devel')
depends=('glibc' 'zlib' 'elfutils')
makedepends=('elfutils' 'git')
conflicts=('binutils-multilib' 'binutils')
replaces=('binutils-multilib')
provides=("binutils" "binutils=$pkgver")
options=(staticlibs !distcc !ccache !buildflags)
source=('binutils-gdb::git+https://sourceware.org/git/binutils-gdb.git')
md5sums=('SKIP')

pkgver(){
  cd binutils-gdb
  echo $(cat binutils/configure | grep 'PACKAGE_VERSION=' | sed 's/PACKAGE_VERSION=//' | sed "s/'//" | sed "s/'//").$(date -I | sed 's/-/_/' | sed 's/-/_/').$(git rev-list --count HEAD).$(git rev-parse --short HEAD)
}

prepare() {
  [[ ! -d binutils-gdb ]] && ln -s binutils-$pkgver binutils-gdb
  mkdir -p binutils-build

  cd binutils-gdb

  # Turn off development mode (-Werror, gas run-time checks, date in sonames)
  sed -i '/^development=/s/true/false/' bfd/development.sh

  # hack! - libiberty configure tests for header files using "$CPP $CPPFLAGS"
  sed -i "/ac_cpp=/s/\$CPPFLAGS/\$CPPFLAGS -O2/" libiberty/configure
}

build() {
  cd binutils-build

  "$srcdir/binutils-gdb/configure" \
    --prefix=/usr \
    --with-lib-path=/usr/lib:/usr/local/lib \
    --with-bugurl=https://bugs.archlinux.org/ \
    --enable-cet \
    --enable-deterministic-archives \
    --enable-gold \
    --enable-ld=default \
    --enable-lto \
    --enable-plugins \
    --enable-relro \
    --enable-shared \
    --enable-targets=x86_64-pep \
    --enable-threads \
    --disable-gdb \
    --disable-werror \
    --with-debuginfod \
    --with-pic \
    --with-system-zlib

  make -j$(nproc) configure-host
  make -j$(nproc) tooldir=/usr
}

#check() {
#  cd binutils-build
#
#  # unset LDFLAGS as testsuite makes assumptions about which ones are active
#  # ignore failures in gold testsuite...
#  make -j$(nproc) -k LDFLAGS="" check || true
#}

package() {
  cd binutils-build
  make -j$(nproc) prefix="$pkgdir/usr" tooldir="$pkgdir/usr" install

  # Remove unwanted files
  rm -f "$pkgdir"/usr/share/man/man1/{dlltool,nlmconv,windres,windmc}*

  # No shared linking to these files outside binutils
  rm -f "$pkgdir"/usr/lib/lib{bfd,opcodes}.so
  echo 'INPUT( /usr/lib/libbfd.a -liberty -lz -ldl )' > "$pkgdir/usr/lib/libbfd.so"
  echo 'INPUT( /usr/lib/libopcodes.a -lbfd )' > "$pkgdir/usr/lib/libopcodes.so"
}
