# Maintainer: Chaiwat Suttipongsakul <cwt@bashell.com>

pkgname=mesa-pvr-ddk117
pkgdesc="Mesa wrapper for PVR DDK 1.17 blobs"
pkgver=21.2.1+2revyos2+glvnd
pkgrel=1
arch=('riscv64')
makedepends=('git' 'python-mako' 'xorgproto'
              'libxml2' 'libx11'  'libvdpau' 'libva' 'elfutils' 'libxrandr'
              'wayland-protocols' 'meson' 'ninja' 'glslang' )
depends=('mesa' 'th1520-img-gpu')
url="https://www.mesa3d.org"
license=('custom')
_commit=aa33b87be4483997c3f584dce5c4288d22c96be9
_srcname=mesa-${_commit}
source=("https://github.com/revyos/mesa/archive/${_commit}.tar.gz"
        '0001-redirect-glapi.patch'
        '0002-change-LIBGL_ALWAYS_SOFTWARE-to-PVRGL_ALWAYS_SOFTWAR.patch'
        '0003-Revert-gbm-add-gbm_bo_blit.patch'
        '0004-gbm-backend-ize.patch')

sha256sums=('fb254d1ce9e54387732478cb89ab911953432c0611a980044940a02a56b6c6d4'
            '25859f3c2befcdd780bde247f9213ae5619bc96c33d827f95a20135f70fbe507'
            'a6edde6043acce39e00460af9a8aa3bd6201c26e80cd1b8e4cee36d279245293'
            '66dbd56931919e993963b51078352015131b42f93aacd0c1d954b2bec0c5783c'
            '2cb173cda7a6df1a67080a13cc2bdf49a8a9174c9fb9fd1022edc2fb9fc69b68')

prepare() {
    # although removing _build folder in build() function feels more natural,
    # that interferes with the spirit of makepkg --noextract
    if [  -d _build ]; then
        rm -rf _build
    fi

    sed -i 's/\r$//' ${srcdir}/${_srcname}/src/mesa/main/formats.csv;

    local _patchfile
    for _patchfile in "${source[@]}"; do
        _patchfile="${_patchfile%%::*}"
        _patchfile="${_patchfile##*/}"
        [[ $_patchfile = *.patch ]] || continue
        echo "Applying patch $_patchfile..."
        patch --directory="${_srcname}" --forward --strip=1 --input="${srcdir}/${_patchfile}"
    done
}

build () {
    cd ${srcdir}
    mkdir _build
    cd _build
    meson setup ${srcdir}/${_srcname} --prefix=/usr -Ddri-drivers-path=/usr/lib/pvr-dri \
        -Dglvnd=true -Dglvnd-vendor-name=pvr \
        -Ddri-drivers=pvr -Dvulkan-drivers=pvr -Dgallium-drivers= \
        -Dglx=disabled -Dllvm=disabled -Dgbm=enabled \
	-Dvalgrind=disabled
    ninja $NINJAFLAGS -C .
}

package() {
    mkdir -p ${pkgdir}/usr/lib ${pkgdir}/usr/share/glvnd/egl_vendor.d
    DESTDIR=${srcdir}/tmp ninja $NINJAFLAGS -C ${srcdir}/_build install
    cp -a ${srcdir}/tmp/usr/lib/*pvr* ${pkgdir}/usr/lib/
    cp -r ${srcdir}/tmp/usr/lib/gbm ${pkgdir}/usr/lib/
    ln -sf pvr_gbm.so ${pkgdir}/usr/lib/gbm/vs-drm_gbm.so
    cp ${srcdir}/tmp/usr/share/glvnd/egl_vendor.d/50_pvr.json ${pkgdir}/usr/share/glvnd/egl_vendor.d/40_pvr.json
}
