# Maintainer: Arglebargle < arglebargle at arglebargle dot dev>
# Contributor: Joan Figueras <ffigue at gmail dot com>
# Contributor: Torge Matthies <openglfreak at googlemail dot com>
# Contributor: Jan Alexander Steffens (heftig) <jan.steffens@gmail.com>
# --
# shellcheck disable=SC2034,SC2164

## The following variables can be customized at build time
##
##  Usage:  env _microarchitecture=98 use_numa=n use_tracers=n makepkg -sc
##     or:  makepkg -sc -- _microarchitecture=98 use_numa=n use_tracers=n
##     or:  export use_numa=n use_tracers=n; makepkg -sc

##
## Xanmod-ROG options:
##
## 'amd_pstate' - Setting this variable to 'n' will disable the amd-pstate driver
##
: "${amd_pstate:=y}"

## '_O3' -  Enable compiler -O3 optimizations, set _O3 to anything to enable
##
_O3="${_O3:+y}"

## 'makeflags_check' - If MAKEFLAGS in /etc/makepkg.conf is null or unset we'll throw a big
##                     warning and pause long enough for the user to cancel their build
##                     Set "makeflags_check" to n to skip this
##
: "${makeflags_check:=y}"

##
## Xanmod options:
##
## Look inside 'choose-gcc-optimization.sh' to choose your microarchitecture
## Valid numbers between: 0 to 99
## Default is: 93 => x86-64-v3
## Good option if your package is for one machine: 98 (Intel native) or 99 (AMD native)
: "${_microarchitecture:=93}"

## Disable NUMA since most users do not have multiple processors. Breaks CUDA/NvEnc.
## Archlinux and Xanmod enable it by default.
## Set variable "use_numa" to: n to disable (possibly increase performance)
##                             y to enable  (stock default)
: "${use_numa:=y}"

## For performance you can disable FUNCTION_TRACER/GRAPH_TRACER. Limits debugging and analyzing of the kernel.
## Stock Archlinux and Xanmod have this enabled.
## Set variable "use_tracers" to: n to disable (possibly increase performance)
##                                y to enable  (stock default)
: "${use_tracers:=y}"

## Choose between GCC or CLANG config (default is GCC)
case "${_compiler,,}" in
  "clang" | "gcc") _compiler=${_compiler,,} ;;
                *) _compiler=gcc            ;; # default to GCC
esac

## Compress modules by default (following Arch's kernel)
## Set variable "_compress_modules" to: n to disable
##                                      y to enable (default)
: "${_compress_modules:=y}"

# Compile ONLY used modules to VASTLY reduce the number of modules built
# and the build time.
#
# To keep track of which modules are needed for your specific system/hardware,
# give module_db script a try: https://aur.archlinux.org/packages/modprobed-db
# This PKGBUILD read the database kept if it exists
#
# More at this wiki page ---> https://wiki.archlinux.org/index.php/Modprobed-db
: "${_localmodcfg:=n}"

# Tweak kernel options prior to a build via nconfig
_makenconfig=

### IMPORTANT: Do not edit below this line unless you know what you're doing

pkgbase=linux-xanmod-rog
xanmod=5.16.7-xanmod1
pkgver=${xanmod//-/.}
#pkgver=5.16.6rc1.prexan0
pkgrel=2
pkgdesc='Linux Xanmod'
url="http://www.xanmod.org/"
arch=(x86_64)
license=(GPL2)
makedepends=(
  bc kmod libelf pahole cpio perl tar xz
  "gcc>=11.0"
)
if [ "$_compiler" = "clang" ]; then
  pkgver="${pkgver}+clang"
  makedepends+=(clang llvm lld python)
  _LLVM=1
fi
options=('!strip' '!ccache')
_major=${xanmod%\.*\-*}           # eg: 5.15
_branch=${xanmod%%\.*\-*}.x       # eg: 5.x

source=("https://cdn.kernel.org/pub/linux/kernel/v${_branch}/linux-${_major}.tar."{xz,sign}
        "https://github.com/xanmod/linux/releases/download/${xanmod}/patch-${xanmod}.xz"
        "choose-gcc-optimization.sh"

        # apply incremental kernel updates ahead of official Xanmod release
        #"https://cdn.kernel.org/pub/linux/kernel/v5.x/patch-5.16.1.xz"
        #"https://cdn.kernel.org/pub/linux/kernel/v5.x/incr/patch-5.16.4-5.xz"
        #"Linux-5.16.6-rc1.patch"

        # Arch: misc hotfixes
        "Bluetooth-btintel-Fix-bdaddress-comparison-with-garb.patch"
        "Bluetooth-Read-codec-capabilities-only-if-supported.patch"

        # amd-pstate included in Xanmod
        # multigenerational lru included in Xanmod
        # 5.17 TCP Optimizations included in Xanmod
        # 5.17 TCP csum optimization included in Xanmod

        # 5.17: UDP/IPv6 optimizations
        "UDP-IPv6-Optimizations-from-5.17-partial.patch"

        # 5.17: remove CONFIG_RCU_FAST_NO_HZ for improved latency
        "CONFIG_RCU_FAST_NO_HZ-removal-for-v5.17.patch"

        # ????: Parallelize x86_64 CPU bringup (v4 patchset)
        # see: https://lore.kernel.org/lkml/20220201205328.123066-1-dwmw2@infradead.org/
        "Parallel-boot-v4-on-5.16.5.patch"

        # -- patch from Chromium developers; more accurately report battery state changes
        "acpi-battery-Always-read-fresh-battery-state-on-update.patch"

        # -- squelch overzealous 802.11 regdomain not set warnings
        "cfg80211-dont-WARN-if-a-self-managed-device.patch"

        # -- Reduce hid-asus object size by consolidating calls
        "HID-asus-Reduce-object-size-by-consolidating-calls.patch"

        # 5.17: ASUS ROG laptop custom fan curve support
        "v16-asus-wmi-Add-support-for-custom-fan-curves.patch"

        # mediatek mt7921 bt/wifi patches
        "mt76-mt7921-enable-VO-tx-aggregation.patch"
        "1-2-Bluetooth-btusb-Add-Mediatek-MT7921-support-for-Foxconn.patch"
        "2-2-Bluetooth-btusb-Add-Mediatek-MT7921-support-for-IMC-Network.patch"
        "Bluetooth-btusb-Add-support-for-IMC-Networks-Mediatek-Chip.patch"
        "Bluetooth-btusb-Add-support-for-Foxconn-Mediatek-Chip.patch"
        "Bluetooth-btusb-Add-support-for-IMC-Networks-Mediatek-Chip-MT7921.patch"
        # XXX: this patch could be buggy, recommended to only build mt76 as a module; see the mailing list
        "mt76-mt7921e-fix-possible-probe-failure-after-reboot.patch"

        # squashed s0ix enablement
        "9001-v5.16.8-s0ix-patch-2022-02-08.patch"
        )
validpgpkeys=(
    'ABAF11C65A2970B130ABE3C479BE3E4300411886' # Linux Torvalds
    '647F28654894E3BD457199BE38DBBDC86092693E' # Greg Kroah-Hartman
)

sha256sums=('027d7e8988bb69ac12ee92406c3be1fe13f990b1ca2249e226225cd1573308bb'
            'SKIP'
            '834dd6603ec3fb0a72e3a1a97669a7198cd6921c4f355b155b3d587a8f0fcea4'
            '278118011d7a2eeca9971ac97b31bf0c55ab55e99c662ab9ae4717b55819c9a2'
            '241f01f06849fcec462d72355ca3ab6bd34931731dec89876d785912ac532398'
            'dd01bd3f774c3a9af42b6d89f534f39c4a5f200db32cd6d4b72a29325645100e'
            '41b0cef536675f10fdfb69eeca2f1927653c69eb351dd92f4085ab6590485851'
            'e74649fb883f2c8c3703c730e45119fca7126dc0599d6bc814de6dcf5a07e7cb'
            'ad73b179ea1a309205744fbf415a7b19108e1c1b452b913894752bb717165d81'
            'f7a4bf6293912bfc4a20743e58a5a266be8c4dbe3c1862d196d3a3b45f2f7c90'
            '3d8961438b5c8110588ff0b881d472fc71a4304d306808d78a4055a4150f351e'
            '544464bf0807b324120767d55867f03014a9fda4e1804768ca341be902d7ade4'
            '0c422d8f420c1518aab1b980c6cdb6e029a4fa9cde1fd99a63670bb105a44f36'
            '1ce9fd988201c4d2e48794c58acda5b768ec0fea1d29555e99d35cd2712281e4'
            '236cdadf0b1472945c0d7570caeed7b95929aabed6872319c9d0969a819689e9'
            'cc2aa580d69801aa1afb0d72ecf094fe13c797363d3d5928c868d3a389910b7b'
            '292a7e32b248c7eee6e2f5407d609d03d985f367d329adb02b9d6dba1f85b44c'
            '7dbfdd120bc155cad1879579cb9dd1185eb5e37078c8c93fef604a275a163812'
            '1444af2e125080934c67b6adb4561fd354a72ce47d3de393b24f53832ee492ac'
            '63ebf908ba2a66865a94e3a4af579d41ec15573522d3ebb07bf8ded3bc57e833'
            '70c3253a70ce50b1241591df14e4749a1040398a1b2a5a5d0f899fe079208319')

export KBUILD_BUILD_HOST=${KBUILD_BUILD_HOST:-archlinux}
export KBUILD_BUILD_USER=${KBUILD_BUILD_USER:-"$pkgbase"}
export KBUILD_BUILD_TIMESTAMP=${KBUILD_BUILD_TIMESTAMP:-$(date -Ru${SOURCE_DATE_EPOCH:+d @$SOURCE_DATE_EPOCH})}

# shellcheck disable=SC2154,SC2155
prepare() {
  cd "linux-${_major}"

  # Apply patches
  local src
  for src in "${source[@]}"; do
    src="${src%%::*}"
    src="${src##*/}"
    case "$src" in
      patch-${_major}*xanmod*.xz)
        # Apply Xanmod patch
        msg2 "Applying Xanmod patch..."
        patch -Np1 -i "../${src%\.xz}"
        ;;
      patch-${_major}*xz|Linux-${_major}*.xz)
        # Apply kernel.org point releases if we're building ahead of Xanmod official
        msg2 "Applying kernel.org point release ${src%\.xz} ..."
        patch -Np1 -i "../${src%\.xz}"
        ;;
      *.patch|*.patch.xz|*.diff|*.diff.xz)
        # Apply any other patches
        msg2 "Applying patch ${src%\.xz} ..."
        patch -Np1 -i "../${src%\.xz}"
        ;;
    esac
  done

  # Set kernel version
  msg2 "Setting version..."
  scripts/setlocalversion --save-scmversion
  echo "-$pkgrel" > localversion.99-pkgrel
  echo "${pkgbase#linux-xanmod}" > localversion.20-pkgname

  # Extract localversions and update source if needed
  local _xanlver=${xanmod##*\-}   # strip longest match '^*\-'
  local _pkglver=${pkgver##*\.}   # strip longest match '^*\.'
  _pkglver=${_pkglver%\+*}        # strip shortest match '\+*$' (removes +clang tag)
  [[ "$_xanlver" != "$_pkglver" ]] &&
    sed -Ei "s/xanmod[0-9]+/${_pkglver}/" localversion

  # Apply configuration
  msg2 "Applying kernel config..."
  cp -vf CONFIGS/xanmod/${_compiler}/config .config

  # enable LTO_CLANG_THIN
  if [ "$_compiler" = "clang" ]; then
    msg2 "Enabling Clang ThinLTO ..."
    scripts/config --disable LTO_CLANG_FULL \
                   --enable LTO_CLANG_THIN
  fi

  # enable O3 optimizations
  if [ "$_O3" = "y" ]; then
    msg2 "Enabling -O3 optimizations ..."
    scripts/config --disable CONFIG_CC_OPTIMIZE_FOR_PERFORMANCE \
                   --enable CONFIG_CC_OPTIMIZE_FOR_PERFORMANCE_O3
  fi

  # CONFIG_STACK_VALIDATION gives better stack traces. Also is enabled in all official kernel packages by Archlinux team
  scripts/config --enable CONFIG_STACK_VALIDATION

  # Enable IKCONFIG following Arch's philosophy
  scripts/config --enable CONFIG_IKCONFIG \
                 --enable CONFIG_IKCONFIG_PROC

  # User set. See at the top of this file
  if [ "$use_tracers" = "n" ]; then
    msg2 "Disabling FUNCTION_TRACER/GRAPH_TRACER only if we are not compiling with clang..."
    if [ "$_compiler" = "gcc" ]; then
      scripts/config --disable CONFIG_FUNCTION_TRACER \
                     --disable CONFIG_STACK_TRACER
    fi
  fi

  # User set. See at the top of this file
  if [ "$use_numa" = "n" ]; then
    msg2 "Disabling NUMA..."
    scripts/config --disable CONFIG_NUMA
  fi

  # Compress modules
  if [ "$_compress_modules" = "y" ]; then
    scripts/config --disable CONFIG_MODULE_COMPRESS_NONE \
                   --enable CONFIG_MODULE_COMPRESS_ZSTD
  fi

  # Disable AMD pstate if requested
  if [ "$amd_pstate" = "n" ]; then
    msg2 "Disabling amd-pstate driver..."
    scripts/config --disable CONFIG_X86_AMD_PSTATE
  else
    scripts/config --enable CONFIG_X86_AMD_PSTATE
  fi

  # Select microarchitecture optimization target
  sh "${srcdir}/choose-gcc-optimization.sh" $_microarchitecture

  # Apply package config customizations
  if [[ -s ${startdir}/xanmod-rog-config ]]; then
    msg2 "Applying package config customization..."
    bash -x "${startdir}/xanmod-rog-config"
  fi

  # This is intended for the people that want to build this package with their own config
  # Put the file "myconfig" at the package folder (this will take precedence) or "${XDG_CONFIG_HOME}/linux-xanmod-rog/myconfig"
  # If we detect partial file with scripts/config commands, we execute as a script
  # If not, it's a full config, will be replaced
  for _myconfig in "${startdir}/myconfig" "${HOME}/.config/linux-xanmod-rog/myconfig" "${XDG_CONFIG_HOME}/linux-xanmod-rog/myconfig" ; do
    # if file exists and size > 0 bytes
    if [ -s "${_myconfig}" ]; then
      if grep -q 'scripts/config' "${_myconfig}"; then
        # myconfig is a partial file. Executing as a script
        msg2 "Applying myconfig..."
        bash -x "${_myconfig}"
      else
        # myconfig is a full config file. Replacing default .config
        msg2 "Using user CUSTOM config..."
        cp -f "${_myconfig}" .config
      fi
      echo
      break
    fi
  done

  ### Optionally load needed modules for the make localmodconfig
  # See https://aur.archlinux.org/packages/modprobed-db
  if [ "$_localmodcfg" = "y" ]; then
    if [ -f "$HOME/.config/modprobed.db" ]; then
      msg2 "Running Steven Rostedt's make localmodconfig now"
      make ${_LLVM:+LLVM=$_LLVM LLVM_IAS=$_LLVM} LSMOD="$HOME/.config/modprobed.db" localmodconfig
    else
      msg2 "No modprobed.db data found"
      exit 1
    fi
  fi

  msg2 "Finalizing kernel config..."
  make ${_LLVM:+LLVM=$_LLVM LLVM_IAS=$_LLVM} olddefconfig

  make -s kernelrelease > version
  msg2 "Prepared %s version %s" "$pkgbase" "$(<version)"

  [[ -z "$_makenconfig" ]] || make ${_LLVM:+LLVM=$_LLVM LLVM_IAS=$_LLVM} nconfig

  # save build configuration for reuse or inspection
  cat .config > "${startdir}/config.last"
}

build() {

  if [[ -z "$MAKEFLAGS" ]] && [[ "$makeflags_check" == "y" ]]; then
    show_makeflags_warning
  fi

  cd "linux-${_major}"
  make ${_LLVM:+LLVM=$_LLVM LLVM_IAS=$_LLVM} all
}

# shellcheck disable=SC2154,SC2155
_package() {
  pkgdesc="The $pkgdesc kernel and modules with ASUS ROG laptop patches (Zephyrus G14, G15, etc)"
  depends=(coreutils kmod initramfs)
  optdepends=('crda: to set the correct wireless channels of your country'
              'linux-firmware: firmware images needed for some devices')
  provides+=(ASUS-WMI-FAN-CONTROL linux-rog)

  cd "linux-${_major}"
  local kernver="$(<version)"
  local modulesdir="$pkgdir/usr/lib/modules/$kernver"

  msg2 "Installing boot image..."
  # systemd expects to find the kernel here to allow hibernation
  # https://github.com/systemd/systemd/commit/edda44605f06a41fb86b7ab8128dcf99161d2344
  install -Dm644 "$(make -s image_name)" "$modulesdir/vmlinuz"

  # Used by mkinitcpio to name the kernel
  echo "$pkgbase" | install -Dm644 /dev/stdin "$modulesdir/pkgbase"

  msg2 "Installing modules..."
  make INSTALL_MOD_PATH="$pkgdir/usr" INSTALL_MOD_STRIP=1 modules_install

  # remove build and source links
  rm "$modulesdir"/{source,build}
}

# shellcheck disable=SC2154,SC2155
_package-headers() {
  pkgdesc="Headers and scripts for building modules for the $pkgdesc kernel"
  depends=(pahole)

  cd "linux-${_major}"
  local builddir="$pkgdir/usr/lib/modules/$(<version)/build"

  msg2 "Installing build files..."
  install -Dt "$builddir" -m644 .config Makefile Module.symvers System.map \
    localversion.* version vmlinux
  install -Dt "$builddir/kernel" -m644 kernel/Makefile
  install -Dt "$builddir/arch/x86" -m644 arch/x86/Makefile
  cp -t "$builddir" -a scripts

  # required when STACK_VALIDATION is enabled
  install -Dt "$builddir/tools/objtool" tools/objtool/objtool

  # required when DEBUG_INFO_BTF_MODULES is enabled
  [[ -x tools/bpf/resolve_btfids/resolve_btfids ]] &&
    install -Dt "$builddir/tools/bpf/resolve_btfids" tools/bpf/resolve_btfids/resolve_btfids

  msg2 "Installing headers..."
  cp -t "$builddir" -a include
  cp -t "$builddir/arch/x86" -a arch/x86/include
  install -Dt "$builddir/arch/x86/kernel" -m644 arch/x86/kernel/asm-offsets.s

  install -Dt "$builddir/drivers/md" -m644 drivers/md/*.h
  install -Dt "$builddir/net/mac80211" -m644 net/mac80211/*.h

  # https://bugs.archlinux.org/task/13146
  install -Dt "$builddir/drivers/media/i2c" -m644 drivers/media/i2c/msp3400-driver.h

  # https://bugs.archlinux.org/task/20402
  install -Dt "$builddir/drivers/media/usb/dvb-usb" -m644 drivers/media/usb/dvb-usb/*.h
  install -Dt "$builddir/drivers/media/dvb-frontends" -m644 drivers/media/dvb-frontends/*.h
  install -Dt "$builddir/drivers/media/tuners" -m644 drivers/media/tuners/*.h

  # https://bugs.archlinux.org/task/71392
  install -Dt "$builddir/drivers/iio/common/hid-sensors" -m644 drivers/iio/common/hid-sensors/*.h

  msg2 "Installing KConfig files..."
  find . -name 'Kconfig*' -exec install -Dm644 {} "$builddir/{}" \;

  msg2 "Removing unneeded architectures..."
  local arch
  for arch in "$builddir"/arch/*/; do
    [[ $arch = */x86/ ]] && continue
    echo "Removing $(basename "$arch")"
    rm -r "$arch"
  done

  msg2 "Removing documentation..."
  rm -r "$builddir/Documentation"

  msg2 "Removing broken symlinks..."
  find -L "$builddir" -type l -printf 'Removing %P\n' -delete

  msg2 "Removing loose objects..."
  find "$builddir" -type f -name '*.o' -printf 'Removing %P\n' -delete

  msg2 "Stripping build tools..."
  local file
  while read -rd '' file; do
    case "$(file -bi "$file")" in
      application/x-sharedlib\;*)      # Libraries (.so)
        strip -v "$STRIP_SHARED" "$file" ;;
      application/x-archive\;*)        # Libraries (.a)
        strip -v "$STRIP_STATIC" "$file" ;;
      application/x-executable\;*)     # Binaries
        strip -v "$STRIP_BINARIES" "$file" ;;
      application/x-pie-executable\;*) # Relocatable binaries
        strip -v "$STRIP_SHARED" "$file" ;;
    esac
  done < <(find "$builddir" -type f -perm -u+x ! -name vmlinux -print0)

  msg2 "Stripping vmlinux..."
  strip -v "$STRIP_STATIC" "$builddir/vmlinux"
  msg2 "Adding symlink..."
  mkdir -p "$pkgdir/usr/src"
  ln -sr "$builddir" "$pkgdir/usr/src/$pkgbase"
}

pkgname=("${pkgbase}" "${pkgbase}-headers")
for _p in "${pkgname[@]}"; do
  eval "package_$_p() {
    $(declare -f "_package${_p#"$pkgbase"}")
    _package${_p#"$pkgbase"}
  }"
done

# shellcheck disable=SC2059
show_makeflags_warning() {
  echo
  warning "makepkg.conf MAKEFLAGS is not set, your kernel build may take many hours to complete!"
  plainerr ""
  plainerr "See https://wiki.archlinux.org/title/Makepkg#Improving_compile_times for help"
  plainerr "or scan this handy-dandy QR code to go straight to the wiki article:"
  plainerr ""
  if [[ "$(/bin/date +%m%d)" == "0401" ]]; then
    # Happy April 1!
    cat <<-YEET >&2
     ▄▄▄▄▄▄▄ ▄▄    ▄ ▄▄▄▄▄ ▄▄▄▄▄▄▄  
     █ ▄▄▄ █ ▄  ▄▄█▄  ▀█ ▄ █ ▄▄▄ █  
     █ ███ █ ██▄█ █ █▀▀▀█  █ ███ █  
     █▄▄▄▄▄█ ▄▀▄ █▀▄ ▄▀█▀█ █▄▄▄▄▄█  
     ▄▄▄▄  ▄ ▄▀ ▀ ▄▄▀▀███▀▄  ▄▄▄ ▄  
     ▄▄█▄█▀▄▀▄▀   ▄▀ █ ▄▀█ ███ ▄▄▀  
      █▄█▀▄▄▀ ▄ █▀██▄█▄▀▄▀▀▀▀▀▄▄ ▀  
     █▀▄▀██▄ ▀▄█▀▄ █ █▀ ██▄▀█▄ ███  
     █▀▄██ ▄ ▀ ▄▄▀ ▀▀▀ ▄ █▄▀▀█▄ █   
     ▄▀▀▄▀ ▄▀██▄▄█ ▀█▄ ▀ ▀▀ █ ▀█▀   
      ▄▀█▀▀▄▄▄▄▄▄█ █▄▀█▄███▄▄▄▄█    
     ▄▄▄▄▄▄▄ ▀██▄█▄▄   ▀▄█ ▄ ██▀█▀  
     █ ▄▄▄ █  ▀▄ ▄▀██▄▄▀ █▄▄▄█▀▄█▄  
     █ ███ █ █ ▄█▀▄ ▀▀  ▀▀█ ▄▀▀▄ █  
     █▄▄▄▄▄█ █  ▀  █▄█ ▀██  ▀ █ █
YEET
  else
    cat <<-YEET >&2
     ▄▄▄▄▄▄▄  ▄    ▄ ▄  ▄▄▄▄▄  ▄▄▄▄▄▄▄
     █ ▄▄▄ █ ▀█▄█▀ ▀▀█ █ ██▀▄  █ ▄▄▄ █
     █ ███ █ ▀▀▄▀▄█▀▄█▀ █▄ ▀██ █ ███ █
     █▄▄▄▄▄█ █ █ ▄ ▄▀█▀█ █▀▄ ▄ █▄▄▄▄▄█
     ▄▄▄▄▄ ▄▄▄▄▄ █▄██▀▄▀▄ ▀█▀▄▄ ▄ ▄ ▄ 
     █ ██▀ ▄▄▀█ ▄▄▀▄▄▀█▀ ███▀▄ ▀▄▄▄▀█▀
     ▄▀▄█▀▀▄ ▀█▀█ ▄█▀▀▄ ▀ ▀▄▀▀█▀▀▄▄▀  
     █ ▀▀▀█▄▄█▄█ ▀ ▄▄█▄ ▀▄▄▄▀  █    █▀
     ▀█▀▀██▄ ▀█  █▄█▀ ▄▀▀▀▀▄█▄▄ █▄ ▀▄ 
     ▄▀▀ ▀█▄▀▄▀ ▄▄▄▀▄▀▀█▀██▄▀▄▄█▄▄█ █▀
     ▀ ▄▄▀ ▄█▄█ █  ▀█▄▄ █ ▀▀█▄▄ ▄█ ▀▄ 
     █▀▀▀█▄▄▄▄▄▀ ▀█▄▄██▀▀▀ ▄▀█ █▀ █ █▀
     █ ▀ ▀▀▄█▀▄  █▀▄▀▀▄▀ ▄██▄██▄███▀ ▄
     ▄▄▄▄▄▄▄ █▄▄▄▄  ▄█▄▀▀█▄▄██ ▄ ██▀▄▀
     █ ▄▄▄ █ ▄███  ▀ ▀   ▀▀█▀█▄▄▄█▀▀  
     █ ███ █ █▀▄ ▀██▄██ ▀██ ██▀▄▀▀▄▄▄ 
     █▄▄▄▄▄█ █▄▀ █▀▄▄ ▄ █ ▀██▄▀█▄█▀▀▄ 
YEET
  fi
  plainerr ""
  plainerr "Edit your makepkg.conf and set appropriate MAKEFLAGS for your system to remove this warning"
  plainerr "or run makepkg with 'env makeflags_check=n makepkg ...'"
  plainerr ""
  printf -- "${ALL_OFF}${BOLD} > ${YELLOW}Pressing CTRL-C now and fixing this before continuing is a very good idea ${ALL_OFF}${BOLD}<\n" >&2
  plainerr ""
  printf -- "-> Continuing in 60s " >&2
  for i in {1..60}; do printf "." >&2 && sleep 1; done
  echo
}

# vim:set ts=8 sts=2 sw=2 et:
