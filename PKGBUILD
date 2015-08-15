pkgname=gcc34
pkgver=3.4.6
pkgrel=4
pkgdesc="The GNU Compiler Collection"
arch=(i686 x86_64)
url="http://gcc.gnu.org"
license=('GPL') # with exception, per fsf.org
depends=('libstdc++5' 'binutils>=2.24' 'lib32-glibc>=2.14' )
BUILDENV=('!fakeroot')
options=('!libtool' '!emptydirs' '!buildflags' 'staticlibs')
source=(http://mirrors.ustc.edu.cn/gnu/gcc/gcc-${pkgver}/gcc-${pkgver}.tar.bz2 \
        gcc-localeversion.patch
        gcc-multilib-fix.patch )
md5sums=('4a21ac777d4b5617283ce488b808da7b'
         'e93d6f49b254dc2879a4e181603599b0'
         'cd370d3bf1af1d779a710556a64df66f')

prepare() {
  if ! locale -a | grep ^de_DE; then
    echo "You need the de_DE locale to build gcc."
    return 1
  fi

  export CFLAGS="${CFLAGS/-mtune=generic/}"
  export CXXFLAGS="${CXXFLAGS/-mtune=generic/}"

  cd $srcdir/gcc-${pkgver}

  patch -Np0 -i ${srcdir}/gcc-localeversion.patch || return 1

  patch -Np0 -i ../gcc-multilib-fix.patch

  # Don't run fixincludes
  sed -i -e 's@\./fixinc\.sh@-c true@' gcc/Makefile.in
  # Don't install libiberty
  sed -i 's/install_to_$(INSTALL_DEST) //' libiberty/Makefile.in

  # fix siginfo
  sed -i 's/struct siginfo/siginfo_t/g'  gcc/config/i386/linux.h
  sed -i 's/struct siginfo/siginfo_t/g'  gcc/config/i386/linux64.h
  # Disables generating documentation (new texinfo does not like the old .texi files)
  echo "MAKEINFO = :" >> ${srcdir}/gcc-${pkgver}/Makefile.in

  if [ ! -d ../gcc-build ] ; then
        mkdir ../gcc-build
  fi
}

build(){
  cd $srcdir/gcc-build
  unset CFLAGS
  unset CXXFLAGS
  unset CPPFLAGS
  ../gcc-${pkgver}/configure --prefix=/usr --libdir=/usr/lib \
      --libexecdir=/usr/lib --mandir=/usr/share/man \
      --infodir=/usr/share/info --disable-multilib \
      --enable-shared --enable-languages=c,c++,f77,objc  \
      --enable-threads=posix --enable-__cxa_atexit  \
      --enable-clocale=gnu --program-suffix=-3.4 \
      --enable-version-specific-runtime-libs
  make bootstrap || return 1
}
package(){
  cd $srcdir/gcc-build
  make -j1 DESTDIR=$pkgdir install

  ## Move conflicting libraries
  _gccbasedir=${pkgdir}/usr/lib/gcc/x86_64-unknown-linux-gnu
  mv ${_gccbasedir}/lib64/* ${_gccbasedir}/${pkgver}/
  rmdir ${_gccbasedir}/lib64
  mv ${_gccbasedir}/lib* ${_gccbasedir}/${pkgver}/
  rm -r $pkgdir/usr/share
}
