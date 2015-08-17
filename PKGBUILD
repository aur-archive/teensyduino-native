# Maintainer: Niels Martignène <niels.martignene@gmail.com>
# Contributor: PyroPeter <googlemail.com@abi1789>
# Contributor: darkapex <me@jailuthra.in>
# Contributor: tty0 <vt.tty0[d0t]gmail.com>

pkgname=teensyduino-native
pkgver=1.21
_arduino=1.6.1
_loader=2.1
_gdkpixbuf=2.31.3
pkgrel=2
pkgdesc="Arduino SDK with Teensyduino (Arch compiler toolchain)"
arch=('i686' 'x86_64')
url="http://www.pjrc.com/teensy/teensyduino.html"
options=(!strip staticlibs)
license=('GPL' 'LGPL' 'custom')
depends=('gtk2' 'arm-none-eabi-gcc' 'arm-none-eabi-binutils' 'arm-none-eabi-newlib'
         'avr-gcc' 'avr-libc' 'avrdude' 'libusb-compat' 'libusb' 'java-runtime'
         'libpng12' 'libsm' 'desktop-file-utils')
makedepends=('xorg-server-xvfb' 'libxft' 'xdotool' 'icoutils')
provides=('arduino' 'teensyduino')
conflicts=('arduino' 'teensyduino' 'arduino-toolchain' 'teensyduino-toolchain'
           'teensy-loader-cli' 'teensyduino-libraries')
install="teensyduino.install"
source=('use-arch-toolchains.patch'
        'newlib-2.2.0-compatibility.patch'
        'arduino.desktop'
        'arduino.xml'
        'teensyduino.sh'
        'teensy-loader.desktop'
        "http://www.pjrc.com/teensy/teensy_loader_cli.${_loader}.zip"
        "http://www.pjrc.com/teensy/49-teensy.rules"
        'LICENSE')
source_i686+=("http://downloads.arduino.cc/arduino-${_arduino}-linux32.tar.xz"
              "http://www.pjrc.com/teensy/td_${pkgver//./}/teensyduino.32bit"
              "http://www.pjrc.com/teensy/beta/teensy_loader_32bit_exclude_syms.tar.gz")
source_x86_64+=("http://downloads.arduino.cc/arduino-${_arduino}-linux64.tar.xz"
                "http://www.pjrc.com/teensy/td_${pkgver//./}/teensyduino.64bit")
sha256sums=('1f17166300e282074bebdb27a5b6f450ebffd43074578574ed341da0b3315ad5'
            'd1d9beec25aa5790edd94071457cb7ec38ebb0588f99969d50389846b871c713'
            'd817829bb2830cb690ed63f14d8a990bb513bef4a4ebc6227a82abdfc8bcd35d'
            '473b82156505e9bd903e4d8484e8d183f2e3bf3c1f7e29940b815929ae597b68'
            'bdd3da81cad5429e1d59c7950f40e75a96d2dd6cab07c2ffb77153e6e860f4b3'
            '270b55353eb438d3790c7245e5ae16ff8bac9f98cfe927d6c9f2146a34499323'
            'dafd040d6748b52e0d4a01846d4136f3354ca27ddc36a55ed00d0a0af0902d46'
            'fa7eff0e0f1e8230941c3b016c40617887f52f1991db655a498309824291ca54'
            '25980feb5927b8bea8b8e999f5002e110825b1bc3d546fa902c2db5c824d33f3')
sha256sums_i686+=('8e64d32c56c116a8bad4741bfcbe715b2040447fbcc1634c99f486790a0021a4'
                  '9921067a71b99ad488442f38f51a86fb7bdca925098a5c0889bd291f54ab478c'
                  '4a39870bf9e8472d8e0151cf18493847aa36fc1164a2542fe613b2567bfdcd7d')
sha256sums_x86_64+=('c344b166efe3a6839cc54af16510b6869cbb4f1fcfaf06a45ca4eba3fca8b2b9'
                    'c0c2262793d6309d9200c5200f52add988e50ffc346222a199b192f135e32a50')

if [ "$CARCH" == 'x86_64' ]; then
  _bits=64
elif [ "$CARCH" == 'i686' ]; then
  _bits=32
fi

prepare() {
  cd "arduino-${_arduino}"

  icotool -x -o .. lib/arduino_icon.ico
}

build() {
  msg2 "Running Teensyduino installer (takes around 60 seconds)"

  chmod +x "teensyduino.${_bits}bit"
  xvfb-run ./teensyduino.sh "./teensyduino.${_bits}bit" "${srcdir}/arduino-${_arduino}"

  cd "arduino-${_arduino}"
  patch -Np1 <"${srcdir}/use-arch-toolchains.patch"
  patch -p1 <"${srcdir}/newlib-2.2.0-compatibility.patch"

  msg2 "Building Teensy Loader command line"

  cd ../teensy_loader_cli
  make
}

package() {
  cd "arduino-${_arduino}"

  mkdir -p "${pkgdir}/usr/bin"
  mkdir -p "${pkgdir}/usr/share/"{doc,applications,mime/packages,licenses/teensyduino}
  mkdir -p "${pkgdir}/usr/lib/udev/rules.d"

  # use Arch's avr toolchain
  mkdir -p "${pkgdir}/usr/arm-none-eabi/lib"
  install -m644 hardware/tools/arm/arm-none-eabi/lib/libarm_cortexM{0,4}l_math.a "${pkgdir}/usr/arm-none-eabi/lib/"
  rm -rf hardware/tools/{arm,gcc-arm-none-eabi-4.8.3-2014q1,avr}
  rm -f hardware/tools/{avrdude,avrdude64,avrdude.conf,readme.txt}

  # copy the whole SDK to /usr/share/arduino/
  cp -a . "${pkgdir}/usr/share/arduino"

  # use system's RXTX library
  ln -sf /usr/lib/librxtxSerial.so "${pkgdir}/usr/share/arduino/lib/librxtxSerial.so"
  ln -sf /usr/lib/librxtxSerial.so "${pkgdir}/usr/share/arduino/lib/librxtxSerial64.so"
  ln -sf /usr/share/java/rxtx/RXTXcomm.jar "${pkgdir}/usr/share/arduino/lib/RXTXcomm.jar"

  # we don't need these sources
  rm -rf "${pkgdir}/usr/share/arduino/src"

  # at least support the FHS a little bit
  ln -s /usr/share/arduino/arduino "${pkgdir}/usr/bin/arduino"
  ln -s /usr/share/arduino/reference "${pkgdir}/usr/share/doc/arduino"

  # desktop icon
  for size in 16 32 48 256; do
    install -Dm644 ../arduino_icon_*_${size}x${size}x32.png \
      "${pkgdir}/usr/share/icons/hicolor/${size}x${size}/apps/arduino.png"
  done

  # desktop and mimetype files
  install -m644 "${srcdir}/arduino.desktop" "${pkgdir}/usr/share/applications/"
  install -m644 "${srcdir}/arduino.xml" "${pkgdir}/usr/share/mime/packages/"

  # install custom PJRC license
  install -m644 "${srcdir}/LICENSE" "${pkgdir}/usr/share/licenses/teensyduino/"

  # install teensy loader files
  install -m644 "${srcdir}/49-teensy.rules" "${pkgdir}/usr/lib/udev/rules.d"
  ln -s /usr/share/arduino/hardware/tools/teensy "${pkgdir}/usr/bin/teensy-loader"
  install -m644 "${srcdir}/teensy-loader.desktop" "${pkgdir}/usr/share/applications/"

  # install command-line teensy loader
  install -m755 "${srcdir}/teensy_loader_cli/teensy_loader_cli" "${pkgdir}/usr/bin/teensy-loader-cli"

  # fixed version to feature no segfault :)
  if [ "$CARCH" == 'i686' ]; then
    install -m755 "${srcdir}/teensy" "${pkgdir}/usr/share/arduino/hardware/tools/"
  fi
}
