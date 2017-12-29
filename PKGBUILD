# vim:set ft=sh:
# Maintainer: BlackEagle < ike DOT devolder AT gmail DOT com >
# Contributor: Tobias Powalowski <tpowa@archlinux.org>
# Contributor: Thomas Baechler <thomas@archlinux.org>

_kernelname=-bede
pkgbase="linux$_kernelname"
pkgname=("linux$_kernelname" "linux$_kernelname-headers")
_basekernel=4.14
_patchver=10
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
pkgrel=1
arch=('x86_64')
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
)
if [[ ${#_extrapatches[@]} -ne 0 ]]; then
    source=( "${source[@]}"
        "${_extrapatches[@]}"
    )
fi

sha512sums=('77e43a02d766c3d73b7e25c4aafb2e931d6b16e870510c22cef0cdb05c3acb7952b8908ebad12b10ef982c6efbe286364b1544586e715cf38390e483927904d8'
            'SKIP'
            'dbc23db58e7b31d83963029dd3894bbdee2bb635a933e982f7802571abc78001dade65755256bc7318a32ad8349d1521a9dc9092285785a8a5b2602c0248cc59'
            '501627d920b5482b99045b17436110b90f7167d0ed33fe3b4c78753cb7f97e7f976d44e2dae1383eae79963055ef74b704446e147df808cdcb9b634fd406e757'
            'f54d4186ed8e1de75185157007097b68f2e58982898f69e7d24fce253018819e2bbc80542f70434f6f81a7766c9c6df7431d859f44b2f53e7e801ac06a1bd3e5'
            'cf65a3f068422827dd3a70abbfe11ddbcc2b1f2d0fb66d7163446ce8e1a46546c89c9c0fbb32a889d767c7b774d6eb0a23840b1ac75049335ec4ec7544453ffd'
            '1a57af338f73100c5f93f4bb92a57295fd53fb6c989096a6b96f242d31cf6b837ccb9b339a30b9c1870c8c4adb5d98ed314683a9b92f4d8d1a28e2d66b77898e'
            'cc249aa48d362a570ec7e16fa9760552fd5fcc3615a29c154b2ee97e51c3c1c1c7449efd031bca59a7b65c473a2afaff075a043dbcb0fbf4a600c83cc9cb8f83'
            '93b642201235c78ef6c8253ef6338a82f6c38e5b6741c7ec06c3dde84433683809c56fe30aab0117607ab09d3367d1dafbbc81af3353f267676357bf72cd7280'
            'SKIP')

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
    cat "$srcdir/config-desktop.x86_64" >./.config
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

    # sync-check does not have executable flag
    chmod +x tools/objtool/sync-check.sh

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

    # add objtool for external module building and enabled VALIDATION_STACK option
    install -Dm755 tools/objtool/objtool \
        "$pkgdir/usr/src/linux-$_kernver/tools/objtool/objtool"

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
