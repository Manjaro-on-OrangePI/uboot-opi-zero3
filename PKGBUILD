# U-Boot: Orange Pi Zero 3
# Maintainer: Furkan Kardame  <f.kardame@manjaro.org>
# Contributor: Mikhail Kalashnikov <iuncuim@gmail.com>

pkgname=uboot-opi-zero3
pkgver=2024.01
pkgrel=1
_tfaver=2.10.23
pkgdesc="U-Boot for orangepi zero3"
arch=('aarch64')
url='http://www.denx.de/wiki/U-Boot/WebHome'
license=('GPL')
makedepends=('bc' 'python' 'python-setuptools' 'swig' 'dtc' 'arm-none-eabi-gcc' 'bison' 'flex')
provides=('uboot')
conflicts=('uboot')
replaces=('uboot-opi-zero3')
install=${pkgname}.install
source=("https://source.denx.de/u-boot/u-boot/-/archive/v${pkgver}/u-boot-v${pkgver}.tar.gz"
        "https://github.com/ARM-software/arm-trusted-firmware/archive/refs/tags/lts-v${_tfaver}.tar.gz"
        "0001-arm64-dts-allwinner-opi-zeroX-usb.patch"
        "0010-sunxi-add-barrier-in-memory-check.patch"
        "0011-sunxi-H616-GPU-enable-hack.patch"
        )
sha256sums=('20bcde63e9c5c635d75c74fd98809ad3a1026c6e36f0523f2c8fee8c56bc5a53'
            'db9bfd2380ffa31f5939c0ed79c9f1afdf58859c841f0b4a0c9abe91ce403c63'
            '195659c5db4faabe08306c6d6e72f00bf978289e263f42de116600eb1afe703d'
            'b8e5d45a3b494257ff016e6facb0a93308d5c194ddf4ff87d4874c182f9d6820'
            '10aac0b0125ae73de7af36716aa55e28c35a1c1cc100bbd59e8fa695c5fc21a7')

prepare() {
  apply_patches() {
      local PATCH
      for PATCH in "${source[@]}"; do
          PATCH="${PATCH%%::*}"
          PATCH="${PATCH##*/}"
          [[ ${PATCH} = *.patch ]] || continue
          msg2 "Applying patch: ${PATCH}..."
          patch -N -p1 < "../${PATCH}"
      done
  }

  cd u-boot-v${pkgver}
  apply_patches
}

build() {
  # Avoid build warnings by editing a .config option in place instead of
  # appending an option to .config, if an option is already present
  update_config() {
    if ! grep -q "^$1=$2$" .config; then
      if grep -q "^# $1 is not set$" .config; then
        sed -i -e "s/^# $1 is not set$/$1=$2/g" .config
      elif grep -q "^$1=" .config; then
        sed -i -e "s/^$1=.*/$1=$2/g" .config
      else
        echo "$1=$2" >> .config
      fi
    fi
  }

  unset CFLAGS CXXFLAGS CPPFLAGS LDFLAGS

  cd arm-trusted-firmware-lts-v${_tfaver}

  echo -e "\nBuilding TF-A for H616 devices...\n"
  # Not performing regulator setup in TF-A allows Linux to boot on
  # this board correctly, which otherwise fails because the Ethernet
  # PHY requires a coordinated bringup of two regulators
  make PLAT=sun50i_h616 SUNXI_SETUP_REGULATORS=0 bl31
  cp build/sun50i_h616/release/bl31.bin ../u-boot-v${pkgver}

  cd ../u-boot-v${pkgver}

  echo -e "\nBuilding U-Boot for OrangePi Zero 3...\n"
  make orangepi_zero3_defconfig

  update_config 'CONFIG_IDENT_STRING' '" Manjaro Linux ARM"'
  update_config 'CONFIG_OF_LIBFDT_OVERLAY' 'y'

  make EXTRAVERSION=-${pkgrel}
  cp u-boot-sunxi-with-spl.bin u-boot-sunxi-with-spl-opi-zero3.bin
}

package() {
  cd u-boot-v${pkgver}

  mkdir -p "${pkgdir}/boot/extlinux"
  install -D -m 0644 u-boot-sunxi-with-spl-opi-zero3.bin -t "${pkgdir}/boot"
}
