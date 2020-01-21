# Maintainer: Sven-Hendrik Haase <svenstaro@gmail.com>
# Maintainer: Felix Yan <felixonmars@archlinux.org>
# Contributor: Thomas Baechler <thomas@archlinux.org>

pkgname=nvidia-dkms-fix-drm-drmph-not-found
pkgver=440.44
pkgrel=13
pkgdesc="NVIDIA drivers for linux, fix 5.5 or higher <drm/drmP.h> not found"
arch=('x86_64')
url="https://www.nvidia.com/"
depends=('dkms' "nvidia-utils=$pkgver" 'libglvnd')
makedepends=("nvidia-utils=${pkgver}" 'libglvnd' 'linux-headers')
optdepends=('linux-headers: Build the module for Arch kernel'
            'linux-lts-headers: Build the module for LTS Arch kernel')
provides=("nvidia=$pkgver")
conflicts+=('nvidia')
license=('custom')
options=('!strip')
_pkg="NVIDIA-Linux-x86_64-${pkgver}"
source=("https://us.download.nvidia.com/XFree86/Linux-x86_64/${pkgver}/${_pkg}.run"
  "kernel-5.5.patch")
sha512sums=('c0c0e19cdb82d47575adbcf46e23580977cf7a5097edfb9d76464c2e678a44f556d8c2d0d49515a86b6765f57176460193c6951927e24c278e6a7f411f89f26b'
            '55452108bb63ef19b4c64c4e1f58da60a10ac9bd95d077a4472e3557be7e2d87182274e430260c88e36f915be467a97eb63d145fbdea293525e96db503953d33')

prepare() {
    sh "${_pkg}.run" --extract-only
    cd "${_pkg}"

    # Fix linux 5.5 (or higher) <drm/drmP.h> not found
    # Patch from https://gitlab.com/snippets/1923197
    patch -p1 <"${srcdir}/kernel-5.5.patch"

    cp -a kernel kernel-dkms
    cd kernel-dkms
    sed -i "s/__VERSION_STRING/${pkgver}/" dkms.conf
    sed -i 's/__JOBS/`nproc`/' dkms.conf
    sed -i 's/__DKMS_MODULES//' dkms.conf
    sed -i '$iBUILT_MODULE_NAME[0]="nvidia"\
DEST_MODULE_LOCATION[0]="/kernel/drivers/video"\
BUILT_MODULE_NAME[1]="nvidia-uvm"\
DEST_MODULE_LOCATION[1]="/kernel/drivers/video"\
BUILT_MODULE_NAME[2]="nvidia-modeset"\
DEST_MODULE_LOCATION[2]="/kernel/drivers/video"\
BUILT_MODULE_NAME[3]="nvidia-drm"\
DEST_MODULE_LOCATION[3]="/kernel/drivers/video"' dkms.conf

    # Gift for linux-rt guys
    sed -i 's/NV_EXCLUDE_BUILD_MODULES/IGNORE_PREEMPT_RT_PRESENCE=1 NV_EXCLUDE_BUILD_MODULES/' dkms.conf
}

build() {
    cd "${_pkg}"/kernel
    make SYSSRC=/usr/src/linux module
}

package() {
    cd ${_pkg}

    install -dm 755 "${pkgdir}"/usr/src
    cp -dr --no-preserve='ownership' kernel-dkms "${pkgdir}/usr/src/nvidia-${pkgver}"

    echo "blacklist nouveau" |
        install -Dm644 /dev/stdin "${pkgdir}/usr/lib/modprobe.d/${pkgname}.conf"

    install -Dt "${pkgdir}/usr/share/licenses/${pkgname}" -m644 "${srcdir}/${_pkg}/LICENSE"
}
