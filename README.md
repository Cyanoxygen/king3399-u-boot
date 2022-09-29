# Mainline U-Boot for King3399 board

This repo contains various files and patches which brings mainline U-Boot for King3399 board.

Build instructions
======

1. Find a place to work

```sh
mkdir -p ~/king3399 
cd ~/king3399
```

2. Get U-Boot source tarball

```
wget https://github.com/u-boot/u-boot/archive/refs/tags/v2022.07.tar.gz
```

3. clone this repository

```
git clone https://github.com/Cyanoxygen/king3399-u-boot
```

4. Extract U-Boot tarball

```
tar xzf v2022.07.tar.gz
```

5. Install files

```
pushd king3399-u-boot/files
find . -type f -exec install -Dv {} ../../u-boot-2022.07/{} \;
popd
```

6. Apply patches

```
pushd u-boot-2022.07
find ../king3399-u-boot/patches -type f -name '*.patch' -print0 | xargs -0 -n1 patch -p1 -i
popd
```

7. Build U-Boot

```
cd u-boot-2022.07
make ARCH=arm king3399_defconfig
make ARCH=arm CROSS_COMPILE=whatever_your_distro_way_is -j$(nproc)
```

8. The output file we needed are `u-boot.itb` and `idbloader.img`.

Installation
======

- `idbloader.img` contains U-Boot TPL and SPL, and ARM Trusted Firmware which initializes TrustZone. With this build, it is no longer needed to flash `trust` into the board.
- `u-boot.itb` contains the actual U-Boot proper.

To install the new U-Boot to eMMC, just `dd` them into proper position of the media:

```
dd if=./idbloader.img of=/dev/mmcblk0 bs=512 conv=notrunc seek=64
dd if=./u-boot.itb of=/dev/mmcblk0 bs=512 conv=notrunc seek=16384
```

Connect your serial console (baudrate is 115200, 8N1), and reboot to see the new U-Boot in action.

Notes
======

This build of U-Boot runs `distro_bootcmd` automatically if the boot process is not interrupted.

### Boot Order

By default, U-Boot will look for any readable script called `boot.scr` to run. It will scan every partitions readable to it, with the order described as follows:

1. eMMC (`mmc 0`)
2. SD Card (`mmc 1`)
3. SDIO (`mmc 2`), there's no storage device down here
4. If all of above failed, U-Boot will start USB and scan for USB storage devices.
5. If U-Boot finds one, it will run the script found.
6. If there's no USB storage device available or no script was found, U-Boot then heads to Ethernet, trying to do TFTP boot.
7. If all of above failed, U-Boot drops to the shell.

Building ARM Trusted Firmware (ATF)
======

ATF firmware (for this instance, it is `bl31.elf`) is needed in order to boot the board.

This repository provides a prebuilt firmware at commit [d8d0ea9a7fcd5ace63a8c863176d9535adfc581d](https://github.com/ARM-software/arm-trusted-firmware/tree/d8d0ea9a7fcd5ace63a8c863176d9535adfc581d).

One can build this firmware manually BEFORE building U-Boot.

### Prerequesties

- ARM AArch64 cross build toolchain (`crossbuild-essential-arm64` as for Debian 11)
- ARM Crotex-R/M cross build toolchain (`gcc-arm-none-eabi`, `binutils-arm-none-eabi` as for Debian 11)
- Common build essential packages, e.g. GNU Make, GCC, flex, etc.

1. Clone ARM TF-A source

```
git clone https://github.com/ARM-software/arm-trusted-firmware
```

2. Build TF-A firmware

```
cd arm-trusted-firmware
make realclean
make CROSS_COMPILE=aarch64-linux-gnu- PLAT=rk3399
```

The output file is at `build/rk3399/release/bl31/bl31.elf`.

