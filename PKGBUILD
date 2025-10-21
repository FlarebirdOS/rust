pkgname=(
    rust
    rust-libs-32bit
    rust-src
)
pkgver=1.90.0
pkgrel=3
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
    'lld'
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
    'llvm'
    'ninja'
    'perl'
    'python'
)
options=('!emptydirs' '!lto')
source=(https://static.rust-lang.org/dist/rustc-${pkgver}-src.tar.xz
    0001-compiler-rt-Fix-compilation-with-glibc-2.42.patch
    0002-bootstrap-Change-bash-completion-dir.patch
    0003-compiler-Change-LLVM-targets.patch
    0004-compiler-Use-ld.lld-by-default.patch
    0007-bootstrap-Workaround-for-system-stage0.patch
    0008-bootstrap-Workaround-for-1.90.0-stage0.patch)
sha256sums=(6bfeaddd90ffda2f063492b092bfed925c4b8c701579baf4b1316e021470daac
    cffd2619f549ac94d98e17831b186d48447a2d694d9cf4ae723026fb794a7f92
    739772412af04ff6f92c9668e0130e17d4798cf843e4bbedd84e352cab795873
    55916bc444bac1ac2ff1f78ded47077316a1dab1e7bdde362bcf08be2a28b2f8
    913ea033f7d76e097ddcbd2015dd6c61911b8fbb25b071469f349dd8a0cd3799
    e7167380b91fa9c0b6590b0d97d9b506570a7c6930b635861ea48a493dac4096
    2ebaccc439f998da2658d346677d580f9accfd757874f44f1e6411c40619d1ff)

prepare() {
    cd rustc-${pkgver}-src

    # Fix build with glibc 2.42
    patch -Np1 < ${srcdir}/0001-compiler-rt-Fix-compilation-with-glibc-2.42.patch

    # Put bash completions where they belong
    patch -Np1 < ${srcdir}/0002-bootstrap-Change-bash-completion-dir.patch

    # Use our *-pc-linux-gnu targets, making LTO with clang simpler
    patch -Np1 < ${srcdir}/0003-compiler-Change-LLVM-targets.patch

    # Use our ld.lld
    patch -Np1 < ${srcdir}/0004-compiler-Use-ld.lld-by-default.patch

    # Fix build with system rustc
    # https://github.com/rust-lang/rust/issues/143735
    patch -Np1 < ${srcdir}/0007-bootstrap-Workaround-for-system-stage0.patch

    # Fix build with system rustc 1.90.0
    # https://github.com/rust-lang/rust/issues/143765
    patch -Np1 < ${srcdir}/0008-bootstrap-Workaround-for-1.90.0-stage0.patch

    cat << EOF > config.toml
# See bootstrap.toml.example for more possible options,
# and see src/bootstrap/defaults/bootstrap.dist.toml for a few options
# automatically set when building from a release tarball
# (unfortunately, we have to override many of them).

# Tell x.py the editors have reviewed the content of this file
# and updated it to follow the major changes of the building system,
# so x.py will not warn us to do such a review.
change-id = 144675

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
tools = ["cargo", "clippy", "rustdoc", "rustfmt", "src"]

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

    mv -v ${pkgdir}/etc/bash_completion.d/cargo ${pkgdir}/usr/share/bash-completion/completions

    unset LIB{SSH2,SQLITE3}_SYS_USE_PKG_CONFIG

    _pick rust-libs-32bit ${pkgdir}/usr/lib64/rustlib/i686-unknown-linux-gnu

    _pick rust-src ${pkgdir}/usr/lib64/rustlib/src
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

package_rust-src() {
    pkgdesc="Source code for the Rust standard library"
    depends=('rust')

    mv ${pkgname}/* ${pkgdir}
}
