# vim:set ft=sh:
# Maintainer: BlackEagle < ike DOT devolder AT gmail DOT com >
# Contributor: Tobias Powalowski <tpowa@archlinux.org>
# Contributor: Thomas Baechler <thomas@archlinux.org>

_kernelname=-bede
pkgbase="linux$_kernelname"
pkgname=("linux$_kernelname" "linux$_kernelname-headers")
_basekernel=4.11
_patchver=6
if [[ "$_patchver" == rc* ]]; then
    # rc kernel
    _baseurl='https://www.kernel.org/pub/linux/kernel/v4.x/testing'
    pkgver=${_basekernel}$_patchver
    _linuxname="linux-${_basekernel}-$_patchver"
else
    # $_patchver is no RC build normal
    _baseurl='https://www.kernel.org/pub/linux/kernel/v4.x'
    pkgver=$_basekernel
    _linuxname="linux-$_basekernel"
fi
pkgrel=2
arch=('i686' 'x86_64')
license=('GPL2')
makedepends=('bc' 'kmod')
url="http://www.kernel.org"
options=(!strip)

validpgpkeys=(
    'ABAF11C65A2970B130ABE3C479BE3E4300411886'
    '647F28654894E3BD457199BE38DBBDC86092693E'
)

source=(
    "$_baseurl/$_linuxname.tar.xz"
    "$_baseurl/$_linuxname.tar.sign"
    # the main kernel config files
    "config-desktop.i686"
    "config-desktop.x86_64"
    # standard config files for mkinitcpio ramdisk
    "linux$_kernelname.preset"
    # pacman hooks
    'linux-bede-01-depmod.hook'
    'linux-bede-02-initcpio.hook'
    'linux-bede-remove.hook'
    # sysctl tweaks
    'sysctl-linux-bede.conf'
)

# revision patches
if [[ "$_patchver" =~ ^[0-9]*$ ]]; then
    if [[ $_patchver -ne 0 ]]; then
        pkgver=$_basekernel.$_patchver
        _patchname="patch-$pkgver"
        source=( "${source[@]}"
            "$_baseurl/$_patchname.xz"
            "$_baseurl/$_patchname.sign"
        )
    fi
fi

## extra patches
_extrapatches=(
    'apple-gmux.patch'
    'poweroff-quirk-workaround.patch'
    'macbook-suspend.patch'
    'https://git.kernel.org/pub/scm/linux/kernel/git/stable/stable-queue.git/plain/queue-4.11/mm-larger-stack-guard-gap-between-vmas.patch'
)
if [[ ${#_extrapatches[@]} -ne 0 ]]; then
    source=( "${source[@]}"
        "${_extrapatches[@]}"
    )
fi

sha512sums=('6610eed97ffb7207c71771198c36179b8244ace7222bebb109507720e26c5f17d918079a56d5febdd8605844d67fb2df0ebe910fa2f2f53690daf6e2a8ad09c3'
            'SKIP'
            '1a5cc5577b1ab3147d11074882b9776e45a91901bb746501c6b33a00b5322ec1df0d4162fd3309f2fbc0a255e11432ff6cefe14e5e5b745d04839c60a1215b9f'
            '4b86038152c979d2cb6c23381d43e0da9fde57e027e2a240debd1c3da24de82f633f5fd85bdcb18b5285b06b25a85a0bb4df4b4f0ad19c31b470a1826e9837cb'
            '501627d920b5482b99045b17436110b90f7167d0ed33fe3b4c78753cb7f97e7f976d44e2dae1383eae79963055ef74b704446e147df808cdcb9b634fd406e757'
            'f54d4186ed8e1de75185157007097b68f2e58982898f69e7d24fce253018819e2bbc80542f70434f6f81a7766c9c6df7431d859f44b2f53e7e801ac06a1bd3e5'
            'cf65a3f068422827dd3a70abbfe11ddbcc2b1f2d0fb66d7163446ce8e1a46546c89c9c0fbb32a889d767c7b774d6eb0a23840b1ac75049335ec4ec7544453ffd'
            '1a57af338f73100c5f93f4bb92a57295fd53fb6c989096a6b96f242d31cf6b837ccb9b339a30b9c1870c8c4adb5d98ed314683a9b92f4d8d1a28e2d66b77898e'
            'cc249aa48d362a570ec7e16fa9760552fd5fcc3615a29c154b2ee97e51c3c1c1c7449efd031bca59a7b65c473a2afaff075a043dbcb0fbf4a600c83cc9cb8f83'
            'e0e2de7d721575cd2770fa4fa61a1ecdfd54bb4239725363a90ab3b670aab44531a7c0f198ff769080643e86ce7e4806d26bb436a43437747e123715061b278b'
            'SKIP'
            '7ec816bfb2e56016eb79614d1619a4921f46a55940b1a4e44d9490375bb63c15c6b61d6275354378d4edc1c88f93afbc08d193c269bcc57a350f9da095e91e10'
            '5bf7e9487d3b31c0207a797b7abfd89794249f1dd16689423203722b201a7d1e40735ed957596ffb10b1dacb87d16b99d4560ff87aed7b24322c257c979d5acc'
            'f27b52dbf081cb6402b651b02744c99d340eac886cc3deff95ad426246976f37c8c0acae5b5dc80c8b0a642d66882d3dddc841810ce08ae1519a3d0e8c8ce423'
            '3c40f5d22ca1061802fb17607f752a99caa8702e03f9adc1ef8e3d6a8e932170c4e14d65a7c8ec83b4de0f725bf9bf2becc08f259effb00e47e30a4648aa1213')

prepare() {
    cd "$srcdir/$_linuxname"

    # Add revision patches
    if [[ "$_patchver" =~ ^[0-9]+$ ]]; then
        if [[ $_patchver -ne 0 ]]; then
            msg2 "apply $_patchname"
            patch -Np1 -i "$srcdir/$_patchname"
        fi
    fi

    # extra patches
    for patch in ${_extrapatches[@]}; do
        patch="$(basename "$patch" | sed -e 's/\.\(gz\|bz2\|xz\)//')"
        pext=${patch##*.}
        if [[ "$pext" == 'patch' ]] || [[ "$pext" == 'diff' ]]; then
            msg2 "apply $patch"
            patch -Np1 -i "$srcdir/$patch"
        fi
    done

    # set configuration
    msg2 "copy configuration"
    if [[ "$CARCH" = "x86_64" ]]; then
        cat "$srcdir/config-desktop.x86_64" >./.config
    else
        cat "$srcdir/config-desktop.i686" >./.config
    fi
    if [[ "$_kernelname" != "" ]]; then
        sed -i "s|CONFIG_LOCALVERSION=.*|CONFIG_LOCALVERSION=\"\U$_kernelname\"|g" ./.config
    fi

    # set extraversion to pkgrel
    msg2 "set extraversion to $pkgrel"
    if [[ "$_patchver" == rc* ]]; then
        sed -ri "s|^(EXTRAVERSION =).*|\1 ${_patchver}-$pkgrel|" Makefile
    else
        sed -ri "s|^(EXTRAVERSION =).*|\1 -$pkgrel|" Makefile
    fi

    # don't run depmod on 'make install'. We'll do this ourselves in packaging
    sed -i '2iexit 0' scripts/depmod.sh

    # hack to prevent output kernel from being marked as dirty or git
    msg2 "apply hack to prevent kernel tree being marked dirty"
    echo "" > "$srcdir/$_linuxname/.scmversion"

    # tmp bc fail workaround
    sed -e 's/$(CONFIG_HZ)/"&;"/g' -i Kbuild
}

build() {
    cd "$srcdir/$_linuxname"

    msg2 "prepare"
    make prepare
    # load configuration
    # Configure the kernel. Replace the line below with one of your choice.
    #make menuconfig # CLI menu for configuration
    #make xconfig # X-based configuration
    #make oldconfig # using old config from previous kernel version
    # ... or manually edit .config
    ####################
    # stop here
    # this is useful to configure the kernel
    #msg "Stopping build"
    #return 1
    ####################
    # yes "" | make config
    # build!
    msg2 "build"
    make $MAKEFLAGS bzImage modules
}

package_linux-bede() {
    pkgdesc="The Linux Kernel and modules, BlackEagle Desktop Edition"
    provides=('linux')
    backup=(
        "etc/mkinitcpio.d/$pkgname.preset"
    )
    depends=('coreutils' 'kmod>=10' 'mkinitcpio>=0.9')
    optdepends=(
        'crda: to set the correct wireless channels of your country'
        'linux-firmware: when having some hardware needing special firmware'
    )

    install=$pkgname.install

    KARCH=x86
    cd "$srcdir/$_linuxname"

    mkdir -p "$pkgdir"/{lib/modules,lib/firmware,boot,usr}

    # get kernel version
    _kernver=$(make kernelrelease)

    # install modules
    make INSTALL_MOD_STRIP=1 INSTALL_MOD_PATH="$pkgdir" modules_install

    # install bzImage
    install -m644 arch/$KARCH/boot/bzImage "$pkgdir/boot/vmlinuz$_kernelname"

    # install fallback mkinitcpio.conf file and preset file for kernel
    install -m644 -D "$srcdir/$pkgname.preset" "$pkgdir/etc/mkinitcpio.d/$pkgname.preset"

    # set correct depmod command for install
    sed \
        -e  "s/KERNEL_NAME=.*/KERNEL_NAME=$_kernelname/g" \
        -e  "s/KERNEL_VERSION=.*/KERNEL_VERSION=$_kernver/g" \
        -i "$startdir/$pkgname.install"
    sed \
        -e "s|source .*|source /etc/mkinitcpio.d/$pkgname.kver|g" \
        -e "s|default_image=.*|default_image=\"/boot/initramfs$_kernelname.img\"|g" \
        -e "s|fallback_image=.*|fallback_image=\"/boot/initramfs$_kernelname-fallback.img\"|g" \
        -i "$pkgdir/etc/mkinitcpio.d/$pkgname.preset"

    echo -e "# DO NOT EDIT THIS FILE\nALL_kver='$_kernver'" > "$pkgdir/etc/mkinitcpio.d/$pkgname.kver"

    # remove build and source links
    rm -f "$pkgdir/lib/modules/$_kernver"/{source,build}

    # remove the firmware
    rm -rf "$pkgdir/lib/firmware"

    _fldkernelname=$(echo $_kernelname | tr "[:lower:]" "[:upper:]")
    # make room for external modules
    ln -s "../${_basekernel}$_fldkernelname-external" "$pkgdir/lib/modules/$_kernver/external"
    # add real version for building modules and running depmod from post_install/upgrade
    mkdir -p "$pkgdir/lib/modules/$_basekernel$_fldkernelname-external"
    echo "$_kernver" > "$pkgdir/lib/modules/${_basekernel}$_fldkernelname-external/version"

    # gzip all modules
    find "$pkgdir" -name '*.ko' -exec gzip -9 {}  \;

    # Now we call depmod...
    depmod -b "$pkgdir" -F System.map "$_kernver"

    # move module tree /lib -> /usr/lib
    mv "$pkgdir/lib" "$pkgdir/usr/"

    # install pacman hooks
    install -Dm644 "$srcdir/linux-bede-01-depmod.hook" "$pkgdir/usr/share/libalpm/hooks/linux-bede-01-depmod.hook"
    install -Dm644 "$srcdir/linux-bede-02-initcpio.hook" "$pkgdir/usr/share/libalpm/hooks/linux-bede-02-initcpio.hook"
    install -Dm644 "$srcdir/linux-bede-remove.hook" "$pkgdir/usr/share/libalpm/hooks/linux-bede-remove.hook"

    # install sysctl tweaks
    install -Dm644 "$srcdir/sysctl-linux-bede.conf" "$pkgdir/usr/lib/sysctl.d/60-linux-bede.conf"
}

package_linux-bede-headers() {
    pkgdesc="Header files and scripts for building modules for linux$_kernelname"
    provides=('linux-headers')

    KARCH=x86

    install -dm755 "$pkgdir/usr/lib/modules/$_kernver"
    cd "$pkgdir/usr/lib/modules/$_kernver"
    ln -sf ../../../src/linux-$_kernver build
    cd "$srcdir/$_linuxname"
    install -D -m644 Makefile \
        "$pkgdir/usr/src/linux-$_kernver/Makefile"
    install -D -m644 kernel/Makefile \
        "$pkgdir/usr/src/linux-$_kernver/kernel/Makefile"
    install -D -m644 .config \
        "$pkgdir/usr/src/linux-$_kernver/.config"

    find . -path './include/*' -prune -o -path './scripts/*' -prune \
        -o -type f \( -name 'Makefile*' -o -name 'Kconfig*' \
        -o -name 'Kbuild*' -o -name '*.sh' -o -name '*.pl' \
        -o -name '*.lds' \) | bsdcpio -pdm "$pkgdir/usr/src/linux-$_kernver"
    cp -a scripts include "$pkgdir/usr/src/linux-$_kernver"
    find $(find arch/$KARCH -name include -type d -print) -type f \
        | bsdcpio -pdm "$pkgdir/usr/src/linux-$_kernver"
    install -Dm644 Module.symvers "$pkgdir/usr/src/linux-$_kernver"

    # add docbook makefile
    install -D -m644 Documentation/DocBook/Makefile \
        "$pkgdir/usr/src/linux-$_kernver/Documentation/DocBook/Makefile"

    # strip scripts directory
    find "$pkgdir/usr/src/linux-$_kernver/scripts" -type f -perm -u+w 2>/dev/null | while read binary ; do
        case "$(file -bi "$binary")" in
            *application/x-sharedlib*) # Libraries (.so)
                /usr/bin/strip $STRIP_SHARED "$binary"
                ;;
            *application/x-archive*) # Libraries (.a)
                /usr/bin/strip $STRIP_STATIC "$binary"
                ;;
            *application/x-executable*) # Binaries
                /usr/bin/strip $STRIP_BINARIES "$binary"
                ;;
        esac
    done

    chown -R root:root "$pkgdir/usr/src/linux-$_kernver"
    find "$pkgdir/usr/src/linux-$_kernver" -type d -exec chmod 755 {} \;
}
