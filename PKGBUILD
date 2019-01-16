# vim:set ft=sh:
# Maintainer: BlackEagle < ike DOT devolder AT gmail DOT com >
# Contributor: Tobias Powalowski <tpowa@archlinux.org>
# Contributor: Thomas Baechler <thomas@archlinux.org>

_kernelname=-bede
pkgbase="linux$_kernelname"
pkgname=("linux$_kernelname" "linux$_kernelname-headers")
_basekernel=4.20
_patchver=3
if [[ "$_patchver" == rc* ]]; then
    _tag=v${_basekernel}-${_patchver}
    pkgver=${_basekernel}${_patchver}
elif [[ $_patchver -ne 0 ]]; then
    _tag=v${_basekernel}.${_patchver}
    pkgver=${_basekernel}.${_patchver}
else
    _tag=v${_basekernel}
    pkgver=${_basekernel}
fi

_folder="linux-stable"
_gitrepo="$_folder::git+https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git?signed#tag=${_tag}"
if [[ "$_patchver" == rc* ]]; then
    _folder="linux-mainline"
    _gitrepo="$_folder::git+https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git?signed#tag=${_tag}"
fi

pkgrel=1
arch=('x86_64')
license=('GPL2')
makedepends=('git' 'bc' 'kmod')
url="http://www.kernel.org"
options=(!strip)

validpgpkeys=(
    'ABAF11C65A2970B130ABE3C479BE3E4300411886'
    '647F28654894E3BD457199BE38DBBDC86092693E'
)

source=(
    "$_gitrepo"
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

## extra patches
_extrapatches=(
)
if [[ ${#_extrapatches[@]} -ne 0 ]]; then
    source=( "${source[@]}"
        "${_extrapatches[@]}"
    )
fi

sha512sums=('SKIP'
            '20aa392ef147e273da05fa4aedbdcca02884e66636ff52c04b0643374ce6e2921d759b7dccc14b78d0906a7b0b1253dfba355828080db96ea7bc7a1bf5f21462'
            '501627d920b5482b99045b17436110b90f7167d0ed33fe3b4c78753cb7f97e7f976d44e2dae1383eae79963055ef74b704446e147df808cdcb9b634fd406e757'
            '7689b3aea73e7f0f1833d20463a898d956e8d9e3a420397c2494d985d4996e6b62d07e91001e44ee193ba5eb79f1af6b6cf95e1cced8625c0e7255a111ed5fe0'
            'cf65a3f068422827dd3a70abbfe11ddbcc2b1f2d0fb66d7163446ce8e1a46546c89c9c0fbb32a889d767c7b774d6eb0a23840b1ac75049335ec4ec7544453ffd'
            '1a57af338f73100c5f93f4bb92a57295fd53fb6c989096a6b96f242d31cf6b837ccb9b339a30b9c1870c8c4adb5d98ed314683a9b92f4d8d1a28e2d66b77898e'
            'ae8c812f0021d38cd881e37a41960dc189537c52042a7d37c47072698b01de593412de1e30eb0d45504924c415bf086624493a22ae18ee5d24a196ec5b31a9f3')

prepare() {
    cd "$srcdir/$_folder"

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
    cp "$srcdir/config-desktop.x86_64" ./.config
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
    echo "" > "$srcdir/$_folder/.scmversion"
}

build() {
    cd "$srcdir/$_folder"

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

    cd "$srcdir/$_folder"

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
    cd "$srcdir/$_folder"
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
    find $(find arch/x86 -name kernel -type d -print) -iname '*.s' -type f \
        | bsdcpio -pdm "$pkgdir/usr/src/linux-$_kernver"
    install -Dm644 Module.symvers "$pkgdir/usr/src/linux-$_kernver"

    # cleanup other architectures
    for arch in "$pkgdir/usr/src/linux-$_kernver"/arch/*/; do
        [[ $arch = */$KARCH/ ]] && continue
        echo "Removing ./arch/$(basename "$arch")"
        rm -r "$arch"
    done
    for arch in "$pkgdir/usr/src/linux-$_kernver"/tools/perf/arch/*/; do
        [[ $arch = */$KARCH/ ]] && continue
        echo "Removing ./tools/perf/arch/$(basename "$arch")"
        rm -r "$arch"
    done

    # remove object files
    find "$pkgdir/usr/src/linux-$_kernver" -type f -name '*.o' -printf 'Removing %P\n' -delete

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
            *application/x-pie-executable*) # Relocatable binaries
                /usr/bin/strip $STRIP_SHARED "$binary"
                ;;
        esac
    done

    # "fix" owner
    chown -R root:root "$pkgdir/usr/src/linux-$_kernver"
    # "fix" permissions
    chmod -Rc u=rwX,go=rX "$pkgdir"
}
