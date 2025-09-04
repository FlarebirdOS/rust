pkgname=(rust rust-libs-32bit)
pkgver=1.89.0
pkgrel=1
pkgdesc="Systems programming language focused on safety, speed and concurrency"
arch=('x86_64')
url="https://blog.rust-lang.org/"
license=('Apache-2.0 OR MIT')
depends=(
    'bash'
    'curl'
    'gcc'
    'gcc-libs'
    'glibc'
    'libssh2'
    'llvm-libs'
    'openssl'
    'sqlite'
    'zlib'
)
makedepends=(
    'clang'
    'cmake'
    'gcc-libs-32bit'
    'glibc-32bit'
    'libffi'
    'lld'
    'llvm'
    'ninja'
    'perl'
    'python'
)
options=('!emptydirs' '!lto')
source=(https://static.rust-lang.org/dist/rustc-${pkgver}-src.tar.xz
    0002-bootstrap-Change-bash-completion-dir.patch
    0003-compiler-Change-LLVM-targets.patch
    0001-compiler-rt-Fix-compilation-with-glibc-2.42.patch
    0007-bootstrap-Workaround-for-system-stage0.patch)
sha256sums=(0b9d55610d8270e06c44f459d1e2b7918a5e673809c592abed9b9c600e33d95a
    fd16f7ec09c293adf56c674b8946eb074ed9b839f9313578afe008aab60df939
    616c2aa7ef452071abc16b3deef4030d0c3cb51ac2fab9cf53f078883f758d09
    cffd2619f549ac94d98e17831b186d48447a2d694d9cf4ae723026fb794a7f92
    8d7a96d99ad0e7e1ebc8c41fb5c8410bb6943acabf6e54a32fe6c5f6de545774)

prepare() {
    cd rustc-${pkgver}-src

    cat << EOF > config.toml
# See bootstrap.toml.example for more possible options,
# and see src/bootstrap/defaults/bootstrap.dist.toml for a few options
# automatically set when building from a release tarball
# (unfortunately, we have to override many of them).

# Tell x.py the editors have reviewed the content of this file
# and updated it to follow the major changes of the building system,
# so x.py will not warn us to do such a review.
change-id = 142379

[llvm]
download-ci-llvm = false

# When using system llvm prefer shared libraries
link-shared = true

# If building the shipped LLVM source, only enable the x86 target
# instead of all the targets supported by LLVM.
targets = "X86"

[build]
description = "Flarebird Linux ${pkgbase} ${pkgver}-${pkgrel}"

target = [
    "x86_64-unknown-linux-gnu",
    "i686-unknown-linux-gnu"
]

# Omit docs to save time and space (default is to build them).
docs = false

# Do not query new versions of dependencies online.
locked-deps = true

# Specify which extended tools (those from the default install).
tools = ["cargo", "clippy", "rustdoc", "rustfmt"]

[install]
prefix = "/usr"
libdir = "/usr/lib64"
docdir = "share/doc/rustc-${pkgver}"

[rust]
channel = "stable"

# Enable the same optimizations as the official upstream build.
lto = "thin"
codegen-units = 1

# Do not build lld which is not part of this package and does not appear
# to be very useful. Even if it turns out to be very useful, we will
# build it as part of the LLVM package.
lld = false

# Don't build llvm-bitcode-linker which is only useful for the NVPTX
# backend that we don't enable.
llvm-bitcode-linker = false

[target.x86_64-unknown-linux-gnu]
cc = "/usr/bin/${CHOST}-gcc"
cxx = "/usr/bin/${CHOST}-g++"
ar = "/usr/bin/${CHOST}-gcc-ar"
ranlib = "/usr/bin/${CHOST}-gcc-ranlib"
llvm-config = "/usr/bin/llvm-config"

[target.i686-unknown-linux-gnu]
cc = "/usr/bin/${CHOST}-gcc"
cxx = "/usr/bin/${CHOST}-g++"
ar = "/usr/bin/${CHOST}-gcc-ar"
ranlib = "/usr/bin/${CHOST}-gcc-ranlib"
llvm-config = "/usr/bin/llvm-config"
EOF

    # Put bash completions where they belong
    patch -Np1 < ${srcdir}/0002-bootstrap-Change-bash-completion-dir.patch

    # Use our *-pc-linux-gnu targets, making LTO with clang simpler
    patch -Np1 < ${srcdir}/0003-compiler-Change-LLVM-targets.patch

    # Fix build with glibc 2.42
    patch -Np1 < ${srcdir}/0001-compiler-rt-Fix-compilation-with-glibc-2.42.patch

    # Fix build with system rustc
    # https://github.com/rust-lang/rust/issues/143735
    patch -Np1 < ${srcdir}/0007-bootstrap-Workaround-for-system-stage0.patch
}

build() {
    cd rustc-${pkgver}-src

    export RUST_BACKTRACE=1

    { [ ! -e /usr/include/libssh2.h ] || export LIBSSH2_SYS_USE_PKG_CONFIG=1; }
    { [ ! -e /usr/include/sqlite3.h ] || export LIBSQLITE3_SYS_USE_PKG_CONFIG=1; }

    export RUSTUP_DIST_SERVER="https://rsproxy.cn"
    export RUSTUP_UPDATE_ROOT="https://rsproxy.cn/rustup"

    python3 x.py build -j "$(nproc)"
}

package_rust() {
    cd rustc-${pkgver}-src

    DESTDIR=${pkgdir} python3 x.py install -j "$(nproc)"

    # delete unnecessary files, e.g. files only used for the uninstall script
    rm -v ${pkgdir}/usr/lib64/rustlib/{components,install.log,rust-installer-version,uninstall.sh}
    rm -v ${pkgdir}/usr/lib64/rustlib/manifest-*


    rm -fv ${pkgdir}/usr/share/doc/rustc-${pkgver}/*.old
    install -vm644 README.md ${pkgdir}/usr/share/doc/rustc-${pkgver}

    install -vdm755 ${pkgdir}/usr/share/zsh/site-functions
    ln -sfv /usr/share/zsh/site-functions/_cargo \
            ${pkgdir}/usr/share/zsh/site-functions

    unset LIB{SSH2,SQLITE3}_SYS_USE_PKG_CONFIG

    _pick rust-libs-32bit ${pkgdir}/usr/lib64/rustlib/i686-unknown-linux-gnu
}

package_rust-libs-32bit() {
    pkgdesc="32-bit target and libraries for Rust"
    depends=(
        'gcc-libs-32bit'
        'glibc-32bit'
        'rust'
    )

    mv ${pkgname}/* ${pkgdir}
}
