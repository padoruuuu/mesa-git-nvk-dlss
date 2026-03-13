# Maintainer: Reza Jahanbakhshi <reza.jahanbakhshi at gmail dot com
# Contributor: Lone_Wolf <lone_wolf@klaas-de-kat.nl>
# Contributor: Armin K. <krejzi at email dot com>
# Contributor: Kristian Klausen <klausenbusk@hotmail.com>
# Contributor: Egon Ashrafinia <e.ashrafinia@gmail.com>
# Contributor: Tavian Barnes <tavianator@gmail.com>
# Contributor: Jan de Groot <jgc@archlinux.org>
# Contributor: Andreas Radke <andyrtr@archlinux.org>
# Contributor: Thomas Dziedzic < gostrc at gmail >
# Contributor: Antti "Tera" Oja <antti.bofh@gmail.com>
# Contributor: Diego Jose <diegoxter1006@gmail.com>
#
# mesa-git + MR !37898 (VK_NVX_binary_import) merged in.
# GPU: Turing (RTX 20xx / GTX 16xx) or newer, drm/nouveau (not nvidia.ko).
# Enable DLSS: DXVK_ENABLE_NVAPI=1 PROTON_ENABLE_NVAPI=1 %command%

pkgname=mesa-nvk-dlss
pkgdesc="Mesa git with VK_NVX_binary_import (NVK DLSS support, MR !37898)"
pkgver=26.1.0_devel.219811.d0abbe9a4fb.d41d8cd
pkgrel=1
arch=('x86_64')
makedepends=(
    'git'
    'xorgproto'
    'libxml2'
    'libva'
    'elfutils'
    'libxrandr'
    'meson'
    'ninja'
    'glslang'
    'directx-headers'
    'python-mako'
    'python-ply'
    'cbindgen'
    'wayland-protocols'
    'python-packaging'
    'python-pyaml'
    'llvm'
    'clang'
    'libclc'
    'spirv-llvm-translator'
    'spirv-tools'
    'rust'
    'rust-bindgen'
)
depends=(
    'libdrm'
    'libxxf86vm'
    'libxdamage'
    'libxshmfence'
    'libelf'
    'libunwind'
    'libglvnd'
    'wayland'
    'lm_sensors'
    'vulkan-icd-loader'
    'zstd'
    'expat'
    'gcc-libs'
    'libxfixes'
    'libx11'
    'systemd-libs'
    'libxext'
    'libxcb'
    'glibc'
    'zlib'
    'python'
    'xcb-util-keysyms'
)
optdepends=('opengl-man-pages: for the OpenGL API man pages')
provides=(
    'vulkan-mesa-layers'
    'opencl-driver'
    'opengl-driver'
    'vulkan-driver'
    'vulkan-intel'
    'vulkan-nouveau'
    'vulkan-radeon'
    'vulkan-swrast'
    'vulkan-virtio'
    'libva-mesa-driver'
    'mesa-libgl'
    'mesa'
    'mesa-git'
    'vulkan-mesa-device-select'
    'vulkan-mesa-implicit-layers'
    'opencl-rusticl-mesa'
)
conflicts=(
    'vulkan-mesa-layers'
    'opencl-clover-mesa'
    'vulkan-intel'
    'vulkan-nouveau'
    'vulkan-radeon'
    'vulkan-swrast'
    'vulkan-virtio'
    'libva-mesa-driver'
    'mesa-libgl'
    'mesa'
    'mesa-git'
    'vulkan-mesa-device-select'
    'vulkan-mesa-implicit-layers'
    'opencl-rusticl-mesa'
)
url="https://www.mesa3d.org"
license=('custom')
source=('mesa::git+https://gitlab.freedesktop.org/mesa/mesa.git#branch=main')
sha256sums=('SKIP')
b2sums=('SKIP')

options=(!lto !debug)

pkgver() {
    cd mesa
    local _ver
    _ver=$(<VERSION)
    echo ${_ver/-/_}.$(git rev-list --count HEAD).$(git rev-parse --short HEAD).d41d8cd
}

prepare() {
    if [ -d _build ]; then
        rm -rf _build
    fi

    cd mesa

    git config user.email "makepkg@localhost"
    git config user.name "makepkg"

    echo "==> Merging MR !37898 (VK_NVX_binary_import) into main..."
    git fetch origin refs/merge-requests/37898/head
    git merge -X theirs --no-gpg-sign --no-edit FETCH_HEAD

    echo "==> Patching nvk_cubin.c: define missing ALIGN macro..."
    sed -i '1s|^|#ifndef ALIGN\n#define ALIGN(value, alignment) (((value) + (alignment) - 1) \& ~((alignment) - 1))\n#endif\n|' \
        src/nouveau/vulkan/nvk_cubin.c

    echo "==> Patching nvk_cmd_dispatch.c: remove stale printf_buffer_addr field..."
    sed -i '/printf_buffer_addr/d' \
        src/nouveau/vulkan/nvk_cmd_dispatch.c

    echo "==> Patching nak/api.rs: fix max_warps_per_sm signature mismatch..."
    python3 - << 'PYEOF'
import re, pathlib
p = pathlib.Path('src/nouveau/compiler/nak/api.rs')
s = p.read_text()
# Replace the MR-added function body that calls ir::max_warps_per_sm(num_gprs)
# with an inlined version that doesn't need ShaderModelInfo
old = (
    'pub extern "C" fn nak_max_warps_per_sm(num_gprs: u32) -> u32 {\n'
    '    crate::ir::max_warps_per_sm(num_gprs)\n'
    '}'
)
new = (
    'pub extern "C" fn nak_max_warps_per_sm(num_gprs: u32) -> u32 {\n'
    '    let total_regs: u32 = 65536;\n'
    '    let gprs = num_gprs.max(1).next_multiple_of(8);\n'
    '    crate::ir::prev_multiple_of((total_regs / 32) / gprs, 4)\n'
    '}'
)
assert old in s, "Pattern not found in api.rs"
p.write_text(s.replace(old, new))
print("Patched api.rs OK")
PYEOF
}

build() {
    local meson_options=(
        -D android-libbacktrace=disabled
        -D b_ndebug=true
        -D b_lto=false
        -D egl=enabled
        -D gallium-drivers=r300,r600,radeonsi,nouveau,virgl,svga,softpipe,llvmpipe,i915,iris,crocus,zink
        -D gallium-extra-hud=true
        -D gallium-rusticl=true
        -D gallium-va=enabled
        -D gbm=enabled
        -D gles1=disabled
        -D gles2=enabled
        -D glvnd=enabled
        -D glx=dri
        -D libunwind=enabled
        -D llvm=enabled
        -D lmsensors=enabled
        -D microsoft-clc=disabled
        -D platforms=x11,wayland
        -D valgrind=disabled
        -D video-codecs=all
        -D vulkan-drivers=amd,intel,intel_hasvk,swrast,virtio,nouveau
        -D vulkan-layers=device-select,intel-nullhw,overlay,anti-lag
        -D tools=[]
        -D zstd=enabled
        -D buildtype=plain
        --wrap-mode=nofallback
        --force-fallback-for=syn,paste,rustc-hash
        -D prefix=/usr
        -D sysconfdir=/etc
        -D legacy-x11=dri2
    )

    CFLAGS+=' -g1'
    CXXFLAGS+=' -g1'

    meson setup mesa _build "${meson_options[@]}"
    meson configure --no-pager _build
    ninja $NINJAFLAGS -C _build
}

package() {
    DESTDIR="${pkgdir}" ninja $NINJAFLAGS -C _build install

    rm "${pkgdir}/usr/bin/mesa-overlay-control.py"
    rmdir "${pkgdir}/usr/bin"

    ln -s /usr/lib/libGLX_mesa.so.0 "${pkgdir}/usr/lib/libGLX_indirect.so.0"

    install -Dm644 "${srcdir}/mesa/docs/license.rst" \
        "${pkgdir}/usr/share/licenses/${pkgname}/LICENSE"
}
