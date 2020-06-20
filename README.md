# Microchip Yocto BSP for the MPFS Polarfire-SoC


Support for Microchip Polarfire-SoC (MPFS-DEV-KIT and LC-MPFS-DEV-KIT) are available in Yocto/OE provided by either the OpenEmbedded Core or for additional and more complete support the meta-polarfire-soc-yocto-bsp layer. 
This Yocto BSP layer builds a complete compressed uSD Image for booting the development board.

The 'Polarfire SoC Yocto BSP' is build on top of the RISCV Architectural layer (meta-riscv) to provide additional hardware specific features. 
Using Yocto 'Openembedded' you will build the following:
  - RISCV Toolchain
  - Predefined Disk Images 
  - Bootloader Binaries (FSBL, HSS, U-Boot)
  - Device Tree Binary (DTB)
  - Linux Kernel Images

Currently the following development platforms are supported:
- [MPFS-DEV-KIT](doc/MPFS-DEV-KIT_user_guide.md) (HiFive Unleashed Expansion Board)
- [LC-MPFS-DEV-KIT](doc/LC-MPFS-DEV-KIT_user_guide.md)
- [ICICLE-KIT-ES] (tbd)


The complete User Guides for each development platform, containing board and boot instructions, are available in the `doc/` subdirectory. 


## Building Linux Using Yocto
This section describes the procedure to build the Disk image and loading it into an uSD card using
bitbake and standard disk utilities.

Yocto Release Activity:
Dunfell (Revision 3.1)	(Released April 2020)

### Required packages for the Build Host
This document assumes you are running on a modern Linux system. The process documented here was tested using Ubuntu 18.04 LTS. 
It should also work with other Linux distributions if the equivalent prerequisite packages are installed.

Detailed instructions for various distributions can be found in "[Required Packages for the Build Host](https://www.yoctoproject.org/docs/3.1/ref-manual/ref-manual.html#required-packages-for-the-build-host)" section in Yocto Project Reference Manual.

### Dependencies

The BSP uses the Yocto RISCV Architecture Layer.
 For Ubuntu 18.04 (or newer) install python3-distutils package.

**Make sure to install the [repo command](https://source.android.com/setup/build/downloading#installing-repo) by Google first.**

**The Microchip 'Hart Software Services' (HSS) requires the Python library kconfiglib:**

Use the default install location '~/.local/bin/'
```
pip3 install kconfiglib
```
 
### Supported Machine Targets
The `MACHINE` (board) option can be used to set the target board for which linux is built, and if left blank it will default to `MACHINE=lc-mpfs`.           
The following table details the available targets:

| `MACHINE` | Board Name |
| --- | --- |
| `MACHINE=mpfs` | MPFS-DEV-KIT, HiFive Unleashed Expansion Board |
| `MACHINE=lc-mpfs` | LC-MPFS-DEV-KIT |
| `MACHINE=icicle-kit-es` | ICICLE-KIT-ES, Icicle Kit engineering samples (supports emmc boot) |
| `MACHINE=icicle-kit-es-sd` | ICICLE-KIT-ES, Icicle Kit engineering samples (supports SD card boot)|
| `MACHINE=qemuriscv64` | Simulation |


## Linux Images

 - 'mpfs-dev-cli' A console image with development tools.
```
     *You can login with `root` account. The password is `microchip`.
```

```     
    * With the OE core-image-*  you can login with `root` account, there is no password set.
```
 - 'core-image-minimal' A small console image to allow you to boot.
 - 'core-image-full-cmdline' A console only image with more full Featured Linux support.

 For more information on available images refer to [Yocto reference manual](https://www.yoctoproject.org/docs/3.1/ref-manual/ref-manual.html#ref-images)

## Quick Start

### Create the Workspace

This needs to be done every time you want a clean setup based on the latest layers.

```bash
mkdir yocto-dev && cd yocto
repo init -u https://github.com/polarfire-soc/meta-polarfire-soc-yocto-bsp -b master -m tools/manifests/riscv-yocto.xml
repo sync
```

### Updating Existing Workspace

If you want to pull in the latest changes in all layers.

```bash
cd yocto-dev
repo sync
repo rebase
```

### HSS Hardware Configuration from Libero Design

(Support for the Icicle-kit only)

The HSS recipe generates embedded software header files from information 
supplied by Libero from the Libero design. Libero supplies the information in 
the form of an xml file. This can be found in the Libero component subdirectory
e.g: /component/work/PFSOC_MSS_C0/PFSOC_MSS_C0_0

Update the following folder with the updated XML file (use the same name) :

```bash
meta-polarfire-soc-yocto-bsp/recipes-bsp/hss/files/${MACHINE}
```

### Setting up Build Environment

```bash
cd yocto-dev
. ./meta-polarfire-soc-yocto-bsp/polarfire-soc_yocto_setup.sh
```

### Building Disk Image

Using yocto bitbake command and setting the MACHINE and image requried.

```bash
MACHINE=<machine> bitbake <image>
```
Example: MACHINE=icicle-kit-es bitbake mpfs-dev-cli

### Yocto Image and Binaries directory
```
build/tmp-glibc/deploy/images/{MACHINE}
```

### Running on Icicle Kit

Boot plan: (hss)->(u-boot)->(linux).
All Binaries required for the boot process are located in the Yocto image directory:
```
build/tmp-glibc/deploy/images/icicle-kit-es
```





### Running wic.gz image on hardware

Compressed Disk images files use `<image>-<machine>.wic.gz` format, for example,

`mpfs-dev-cli-<machine>.wic.gz`. We are interested in `.wic.gz` disk images for writing to emmc/uSD/SD card.

> Be very careful while picking /dev/sdX device! Look at dmesg, lsblk, GNOME Disks, etc. before and after plugging in your usb flash device/uSD/SD to find a proper device. Double check it to avoid overwriting any of system disks/partitions!
> 
> Unmount any mounted partitions from uSD card before writing!
> 
> We advice to use 16GB or 32GB uSD cards. 8GB cards (shipped with HiFive Unleashed) can still be used with CLI images.

Example write the disk image to the SD card for the icicle kit:

```bash
zcat mpfs-dev-cli-icicle-kit-es.wic.gz | sudo dd of=/dev/sdb bs=512K iflag=fullblock oflag=direct conv=fsync status=progress
```

## Run in Simulation (QEMU)

```bash
./openembedded-core/scripts/runqemu nographic
```

## Additional Reading

[Yocto User Manual](https://www.yoctoproject.org/docs/3.1/mega-manual/mega-manual.html) 

[Yocto Development Task Manual](https://www.yoctoproject.org/docs/3.1/dev-manual/dev-manual.html) 
 
[Yocto Bitbake User Manual](https://www.yoctoproject.org/docs/3.1/bitbake-user-manual/bitbake-user-manual.html)
 
[PolarFire SoC Buildroot BSP](https://github.com/polarfire-soc/polarfire-soc-buildroot-sdk) 
 
[MPFS-DEV-KIT User Guide](doc/MPFS-DEV-KIT_user_guide.md) 

[LC-MPFS-DEV-KIT User Guide](doc/LC-MPFS-DEV-KIT_user_guide.md) 

[U-Boot Documentation](https://www.denx.de/wiki/U-Boot/Documentation) 

[Kernel Documentation for v5.4](https://www.kernel.org/doc/html/v5.4/) 

## Known issues

### Issue 001: Required binaries not available before creating the disk image
  We sometimes get dependencies not building correctly.
  During the process do_wic_install payload may not be present for hss or opensbi.

  For example after requesting a complete build:

  ```bash
  MACHINE=icicle-kit-es bitbake mpfs-dev-cli
  ```
  If payload is missing execute the following:
  ```bash
  MACHINE=icicle-kit-es bitbake hss -c install
  ```
  If fsbl is missing execute the following:
  ```bash
  MACHINE=icickle-kit-es bitbake u540-c000-bootloader -c install
  ```
  And finally a complete build:
  ```bash
  MACHINE=mpfs bitbake mpfs-dev-cli
  ```

### Issue 002 fs.inotify.max_user_watches

Listen uses inotify by default on Linux to monitor directories for changes. It's not uncommon to encounter a system limit on the number of files you can monitor. For example, Ubuntu Lucid's (64bit) inotify limit is set to 8192.

You can get your current inotify file watch limit by executing:

```bash
$ cat /proc/sys/fs/inotify/max_user_watches
```

When this limit is not enough to monitor all files inside a directory, the limit must be increased for Listen to work properly.

Run the following command:

  ```bash
 echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p
  ```

[Details on max_user_watches](https://github.com/guard/listen/wiki/Increasing-the-amount-of-inotify-watchers)


