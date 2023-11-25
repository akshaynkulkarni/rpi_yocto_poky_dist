# Yocto tutorials

The main building blocks for Embedded linux:
1. Bootloader(s)
2. Kernel (linux)
3. RootFS: libs and bins.
4. Toolchain
5. The Hardware ofcourse


## What is yocto project?

An OSS by linux foundation, that provides tools that help to create a custom embedded linux distribution, across various hardwares, vendors, manufacturers, etc.
It takes configs as input and produces image(s) for the above building blocks.

Yocto project (YP) has reference distribution called "poky", that has following components as repos:

1. oe-core (meta): OpenEmbedded core, offers core meta-data for various arches, features etc
2. Bitbake: pocky's (multi-threaded) build system, py and shell scripts. Process recpies: fetch pkgs, build and image gen.
3. meta-yocto-bsp: BSP

### Meta-data:
In the context of yocto, its basically build instructions, it has config files (.config), recipies (.bb, .bbapend), recipie classes (.bbclasses) and includes (.inc);
Build instr has versions of sw and fetch details of them and additional patches to be applied on.
Example: How to build a kernel: defines which config file  for make defconfig, etc; Additional custom patches for project or board, etc


## Building poky

The `conf` files can be found [here](./conf/).

0. This repo has oe, bsp, poky as submodules, however, it can be setup the following way:
1. Clone poky repo: git clone https://git.yoctoproject.org/poky/ -b <branch>
2. Setup poky shell env:
      ```
      . oe-init-build-env <build_dir>
      or
      source oe-init-build-env <build_dir>
      ```
   Now, under build dir, we have generated conf/local.conf and bblayers.cfg
3. For BSP layer of RPI boards,
   ```
   git clone https://git.yoctoproject.org/meta-raspberrypi
   ```
4. For OE,
   ```
   git clone git@github.com:openembedded/meta-openembedded.git
   ```
5. In order for the poky to recognize rpi, we have to append
   1. `MACHINE ??= "raspberrypi3"` in build/conf/local.conf
   2. build/conf/bblayers.conf: add RPI BSP layer (meta-raspberrypi) path to BBLAYERS.

   ```shell
         BBLAYERS ?= " \
               <path>/yocto/poky/meta \
               <path>/yocto/poky/meta-poky \
               <path>/yocto/poky/meta-yocto-bsp \
               <path>/yocto/poky/meta-raspberrypi \
               <path>/yocto/poky/meta-openembedded/meta-oe \
               <path>/yocto/poky/meta-openembedded/meta-python \
               <path>/yocto/poky/meta-openembedded/meta-networking \
      "
   ```

6. For enabling ssh, wifi, bt, appending the following to local.conf:

   ```shell
   DISTRO_FEATURES_append = " \
                              bluez5 \
                              bluetooth \
                              wifi \
                              "
   ```

   ```shell
   IMAGE_INSTALL_append = " \
                           openssh \
                           linux-firmware-bcm43430 \
                           bluez5 \
                           i2c-tools \
                           bridge-utils \
                           hostapd \
                           dhcp-server \
                           iptables \
                           wpa-supplicant \
                           "
   ```

   Enable Uart in RPI, append to local.conf:
   ```shell
   ENABLE_UART = "1"
   ```


7. Build minimal image and install on sdcard:
   ```
   bitbake core-image-minimal
   ```
   The image is generated at `<path>/yocto/poky/build/tmp/deploy/images/raspberrypi3-64`, image is `core-image-minimal-raspberrypi3-64-xxyyzzaa.rootfs.wic.bz2`

   extract the image:
   ```shell
   bzip2 -d -f core-image-minimal-raspberrypi3-64-xxyyzzaa.rootfs.wic.bz2
   ```
   flash the wic file to sd card in case of RPI
   ```shell
   sudo dd bs=4M if=core-image-minimal-raspberrypi3-64-xxyyzzaa.rootfs.wic of=/dev/<device> status=progress conv=fsync
   ```


8. Build the cross compiler toolchain:

   ```
   bitbake meta-toolchain
   ```
   The toolchain is generated at `<path>/yocto/poky/build/tmp/deploy/sdk/`.

   Example: `poky-glibc-x86_64-meta-toolchain-aarch64-raspberrypi3-64-toolchain-3.1.29.sh`

9. Additional: Append the following in local.conf
   ```shell
   BB_NUMBER_THREADS = "<number of threads>"
   PARALLEL_MAKE = "-j <number of threads>"
   ```