SimpleYunix
===========

    build a simple linux system based on Linux Kernel 4.0.2

## Prepare
    Host: CentOS6.5( x86_64 )
    mkdir SimpleYunix
    cd SimpleYunix
    mkdir -p kernel/rootfs  userspace opensrc/tgz


## [QEMU]( http://qemu-project.org/Main_Page )

    cd opensrc
    yum install libuuid-devel.x86_64 gnutls-devel  libaio-devel  spice-server-devel.x86_64
    wget http://wiki.qemu-project.org/download/qemu-2.2.0.tar.bz2 -P tgz
    tar -xjf tgz/qemu-2.2.0.tar.bz2 -C .
    cd qemu-2.2.0
    ./configure --prefix=/home/xyfeng/install/ --target-list=x86_64-softmmu --enable-vnc
    --disable-xen --enable-vnc-tls --enable-vnc-sasl --enable-kvm  --enable-linux-aio
    --disable-docs --enable-vhost-net --disable-libiscsi --disable-smartcard-nss --enable-debug
    --enable-spice --enable-uuid

    make
    make install


## Build Root FS

### Create Dir: rootfs

        cd kernel/rootfs
        mkdir -p bin sbin dev proc dev lib lib64 usr/bin usr/sbin
        cat >init
        #!/bin/sh

        export PATH=/bin:/sbin:/usr/bin:/usr/sbin

        echo "SimpleYunix Startup: Userspace"

        #mount none /proc -t proc
        #mount -t sysfs none /sys

        bin/sh

        C-D

### Build Busybox

        cd opensrc
        wget http://busybox.net/downloads/busybox-1.22.1.tar.bz2 -P tgz
        tar -xjf tgz/busybox-1.22.1.tar.bz2 -C .
        cd busybox-1.22.1
        make allnoconfig
        make menuconfig
        ( According to the need to customize )
        make -s
        make install CONFIG_PREFIX=../../kernel/rootfs
        rm -f ../../kernel/rootfs/linuxrc


 ## Build Kernel

### Download

        cd kernel
        wget https://www.kernel.org/pub/linux/kernel/v4.x/linux-4.0.2.tar.gz -P ../opensrc/tgz
        tar -xzf ../opensrc/tgz/linux-4.0.2.tar.gz -C .

### Config Kernel

        cd linux-4.0.2
        make allnoconfig
        make menuconfig

#### 64-bit Kernel
            [*] 64-bit kernel

#### RAM FS & Disk
            General setup  --->
                 [*] Initial RAM filesystem and RAM disk (initramfs/initrd) support
                        (../rootfs) Initramfs source file(s)
                        (0)     User ID to map to 0 (user root) (NEW)
                        (0)     Group ID to map to 0 (group root) (NEW)

#### Disable EMBEDDED

            [ ] Embedded system

#### Support for printk

            [*] Configure standard kernel features (expert users)  --->
                    [*]   Enable support for printk

#### Select Exec File Format( busybox & init script )

            Executable file formats / Emulations  --->
                [*] Kernel support for ELF binaries
                [*] Kernel support for scripts starting with #!

#### Support for Keyboard, Not Support for Mouse

            Device Drivers  --->
                    Input device support  --->
                        [*] Generic input layer (needed for keyboard, mouse, ...)
                            [ ]   Mouse interface
                            [*]   Keyboards (NEW)  --->
                            [ ]   Mice  ----
#### Enable TTY

            Device Drivers  --->
                   Character devices  --->
                        [*] Enable TTY
                            [*]   Virtual terminal (NEW)
                            [*]     Enable character translations in console (NEW)
                            [*]     Support for console on virtual terminal (NEW)
                            [ ]     Support for binding and unbinding console drivers (NEW)
                            [*]   Unix98 PTY support (NEW)
                            [ ]     Support multiple instances of devpts (NEW)
                            [*]   Legacy (BSD) PTY support (NEW)

#### Enable Console Output

            Device Drivers  --->
                    Graphics support  --->
                        Console display driver support  --->
                            [*] VGA text console (NEW)
                            [ ]   Enable Scrollback Buffer in System RAM (NEW)
                            (80) Initial number of console screen columns (NEW)
                            (25) Initial number of console screen rows (NEW)


### Running
        make bzImage
        qemu-system-x86_64 -kernel arch/x86/boot/bzImage  -append "noapic" -curses



