# Introduction

This is a Yocto project repository for Watermelon Wine boards designed by [DanielMartensson](https://github.com/DanielMartensson),
specifically the Watermelon Wine boards based on stm32 microcontrollers.

Initially the examples here will simply uses `STM32MP157D-DK1` board but the goal is to support Watermelon Wine boards.

Such as:

 * [STM32-Computer](https://github.com/DanielMartensson/STM32-Computer)
 * [Watermelon-Wine-1A](https://github.com/DanielMartensson/Watermelon-Wine-1A)

# Yocto ST-STM32MP Quick Start

## Preflight

`openstlinux` for `stm32mp157d-dk1-fastboot` used in these examples need an sdcard of 5GiB or more.

A source build requires a lot of diskspace, ram and cpu time;

 * No less then 30G free disk
 * 16GiB ram + 16GiB swap is acceptable

## Clone

This git repository uses a lightweight approach with git submodules to reference the various external meta-layer repositories;
a consideration is switching to [repo](https://gerrit.googlesource.com/git-repo) to make external change processes easier to manage,
but for now it will be git submodules.

`git clone --recurse-submodules https://github.com/evildeeds/yocto-watermelon-wine-stm32mp`

or if you already cloned but forgot to recurse submodules

`git submodule update --init --recursive`

## Machine setting

First verify that your device is supported; for example
the `STM32MP157D-DK1` devboard, look for a Yocto machine
configuration in `layers/meta-st/meta-st-stm32mp/conf/machine/`
then identify which configurations support it.

```
$ grep -i stm32mp157d-dk1 layers/meta-st/meta-st-stm32mp/conf/machine/*.conf
layers/meta-st/meta-st-stm32mp/conf/machine/stm32mp1.conf:STM32MP_DT_FILES_SDCARD += "stm32mp157a-dk1 stm32mp157d-dk1"
layers/meta-st/meta-st-stm32mp/conf/machine/stm32mp1.conf:LINUX_A7_EXAMPLES_DT += "stm32mp157d-dk1-a7-examples"
layers/meta-st/meta-st-stm32mp/conf/machine/stm32mp1.conf:CUBE_M4_EXAMPLES_DT += "stm32mp157d-dk1-m4-examples"
```

Thus `STM32MP157D-DK1` is supported by using `MACHINE=stm32mp1` since the relevant configuration file is named `stm32mp1.conf`.

Then select a distro configuration; for example `DISTRO=openstlinux-weston` comes with a weston desktop.

## Yocto Bitbake Envirnoment

Initialize the Yocto build environment; It may complain about your build host distro not being supported,
you may proceed anyway but make sure you have the
[appropriate packages installed](https://docs.yoctoproject.org/ref-manual/system-requirements.html#required-packages-for-the-build-host).

```bash
DISTRO=${DISTRO} MACHINE=${MACHINE} source layers/meta-st/meta-st-scripts/envsetup.sh --no-ui
```

## Yocto Image Build

Select an image recipe from `meta-st-openstlinux` to build; for example `st-image-weston`.

```bash
bitbake ${IMAGE}
```

## Finalize and Write Image to sdcard

After creating the sdcard image check that the final image size fits on the sdcard before writing it.

After writing the sdcard the GPT partion should be repaired to ensure that the backup GPT
table is placed at the end of the device as it should.

```bash
cd tmp-glibc/deploy/images/${MACHINE}
./scripts/create_sdcard_from_flashlayout.sh ./flashlayout_${IMAGE}/fastboot/FlashLayout_sdcard_stm32mp157d-dk1-fastboot.tsv
sudo dd bs=8M conv=fdatasync status=progress if=./FlashLayout_sdcard_stm32mp157d-dk1-fastboot.raw of=/dev/${SDCARD_DEVICE}
sudo parted -sf /dev/${SDCARD_DEVICE} print
```
