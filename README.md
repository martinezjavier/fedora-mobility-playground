Fedora experimental images for mobile devices
=============================================

This repository contains OSBuild manifests to build experimental images to
install on mobile devices, such as the PinePhone Pro. These are of course
not official images, but just what I use to do some testing on my phone.

For now there is only a single manifest to build an image containing most
of default Fedora Workstation package set plus the Phosh shell for mobile.

But it should be easy to manifest for other types of Fedora images and it
could allow to do some experimentation. The existing manifest also adds a
COPR repository that contain custom packages to support the PinePhone Pro.

The images built though could be used in any other device with support for
EFI, and also the manifest allows to build QCOW2 images to import on a VM.

### Requirements

The osbuild and osbuild-tools packages must be installed to build images:

```sh
sudo dnf install osbuild osbuild-tools -y
```

### Building

As mentioned above, the images are generated using the OSBuild tool, which
still does not support cross-platform image generation so the images have
to be created on a machine that uses the same architecture than the target.

First an OSBuild manifest must be generated from the Manifest Pre-Processor
(MPP) file. This can be done with the following command:

```sh
osbuild-mpp fedora-phosh.mpp.json -D arch=\"$(uname -m)\" fedora-phosh.json
```

Then an image can be built, this is achieved using the following command:

```sh
sudo osbuild --store osbuild_store --output-directory image_output --export image fedora-phosh.json
```

A QCOW2 image to be used as a storage device for a virtual VM can also be
built. To do that instead the following command can be used:

```sh
sudo osbuild --store osbuild_store --output-directory image_output --export qcow2 fedora-phosh.json
```

### Install

To install the generated OS image, dd the `image_output/image/disk.img` to
the block device of the eMMC or SD card:

```sh
sudo dd if=image_output/image/disk.img of=/dev/sda bs=4M conv=fsync
```
Replace `/dev/sda` by the correct block device in your system.

If the PinePhone Pro does not have a firmware in its SPI flash or eMMC that
is able to EFI images, a bootloader must be installed in the block device.

This can be done by downloading the uboot-images-armv8 package and writing
the uboot binaries to the block device of the eMMC or SD card:

```sh
sudo dnf install uboot-images-armv8 -y
sudo dd if=/usr/share/uboot/pinephone-pro-rk3399/idbloader.img of=/dev/sda seek=64
sudo dd if=/usr/share/uboot/pinephone-pro-rk3399/u-boot.itb of=/dev/sda seek=16384
```

### Images default setup

The images default configuration is to log in automatically the `fedora` user
and start a Phosh Wayland session. These options can be modified by editing
the `/etc/gdm/custom.conf` and `/var/lib/AccountsService/users/fedora` files.

There are two users created, `root` that has as default password `password`
and `fedora` that has as default password `1234`. The password is also the
PIN that is used to unlock the screen.
