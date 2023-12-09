# Yocto tutorials

The main building blocks for Embedded Linux:
1. Bootloader(s)
2. Kernel (Linux)
3. RootFS: libs and bins.
4. Toolchain
5. The Hardware of course


## What is Yocto project?

An OSS by Linux Foundation provides tools that help create a custom embedded Linux distribution across various hardwares, vendors, manufacturers, etc.
It takes configs as input and produces image(s) for the above building blocks.

Yocto Project (YP) has a reference distribution called "poky", that has the following components as repos:

1. oe-core (meta): OpenEmbedded core, offers core meta-data for various arches, features etc
2. Bitbake: pocky's (multi-threaded) build system, py and shell scripts. Process recipes: fetch pkgs, build and image gen.
3. meta-yocto-bsp: BSP

### Meta-data:
In the context of Yocto, it is build instructions, it has config files (.config), recipes (.bb, .bbapend), recipe classes (.bbclasses) and includes (.inc);
Build instr has versions of sw and fetches details of them and additional patches to be applied on.
Example: How to build a kernel: defines which config file  for make defconfig, etc; Additional custom patches for project or board, etc


## Building poky

The `conf` files can be found [here](./conf/).

0. This repo has oe, BSP, poky as submodules, however, it can be setup the following way:
1. Clone poky repo: git clone https://git.yoctoproject.org/poky/ -b <branch>
2. Setup poky shell env:
      ```
      . oe-init-build-env <build_dir>
      or
      source oe-init-build-env <build_dir>
      ```
   Under the build dir, we have generated conf/local.conf and bblayers.cfg
3. For the BSP layer of RPI boards,
   ```
   git clone https://git.yoctoproject.org/meta-raspberrypi
   ```
4. For OE,
   ```
   git clone git@github.com:openembedded/meta-openembedded.git
   ```
5. For the poky to recognize RPI, we have to append
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

   OR
   with use bitbake script:
   ```shell
      bitbake-layers add-layer <layer-location>
   ```
   A layer must contain conf/layer.conf to be added to bitbake layers. One can remove layer using  option `remove-layer`
   and to display `show-layers`, create with `create-layer`. Use `bitbake-layers -h` for help.

   ** Notes on BSP layer **
   - All details of the layer are always under README.md :P, like it's dependent layers, etc, how to build, flash, tools etc.
   - A BSP layer defines different supported machines under meta-<any-bsp-layer>/conf/machine/<Machines>.conf
   - Possible build image types are at  meta-<any-bsp-layer>/recipes-core/images/*.bb. For example, rpi-hwup-image.bb is for basic hardware up and  (deprecated now, use core-image-base), recipes-core/images/rpi-basic-image.bb with additional features like ssh server, etc. and so on. Test image: recipes-core/images/rpi-test-image.bb.
   - 
   
7. For enabling ssh, wifi, bt, appending the following to local.conf:

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


8. Build a minimal image and install on sdcard:
   ```
   bitbake core-image-minimal
   ```
   The image is generated at `<path>/yocto/poky/build/tmp/deploy/images/<machine>`, image is `core-image-minimal-raspberrypi3-64.tar.bz2`, a symlink to
   `core-image-minimal-raspberrypi3-64-xxyyzzaa.rootfs.wic.bz2`. There are base files including device tree blob, kernel image, RFS symlinked to current build.

   extract the image:
   ```shell
   bzip2 -d -f core-image-minimal-raspberrypi3-64-xxyyzzaa.rootfs.wic.bz2
   ```
   flash the wic file to sd card in case of RPI
   ```shell
   sudo dd bs=4M if=core-image-minimal-raspberrypi3-64-xxyyzzaa.rootfs.wic of=/dev/<device> status=progress conv=fsync
   ```
   Tip: after sourcing Yocto env, one can use `wic ls <wic-image>` to list all the partitions under that wic image.

10. Build the cross compiler toolchain:

   ```
   bitbake meta-toolchain
   ```
   The toolchain is generated at `<path>/yocto/poky/build/tmp/deploy/sdk/`.

   Example: `poky-glibc-x86_64-meta-toolchain-aarch64-raspberrypi3-64-toolchain-3.1.29.sh`

11. Additional: Append the following in local.conf
   ```shell
   BB_NUMBER_THREADS = "<number of threads>"
   PARALLEL_MAKE = "-j <number of threads>"
   ```
