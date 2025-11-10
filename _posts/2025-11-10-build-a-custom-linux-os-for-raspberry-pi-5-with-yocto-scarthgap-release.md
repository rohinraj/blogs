---
layout: post
title: "Build a Custom Linux OS for Raspberry Pi 5 with Yocto (Scarthgap Release)"
date: 2025-11-10 11:00:00
categories: jekyll update
tags: [linux, os, raspberrypi, yocto, embedded-systems, scarthgap, custom-linux, custom-os, operating system]
---


# Build a Custom Linux OS for Raspberry Pi 5 with Yocto (Scarthgap Release)


If you've ever wanted to tailor a Linux system exactly for your hardware, the **Yocto Project** is your go-to toolkit. 


Wait, What is Yocto? 
**Yocto** is an open-source project that offers tools, templates, and methods to help developers create custom Linux distributions for embedded systems.
It’s not an embedded Linux distribution, it creates a custom one for you. 


In this quick guide, we’ll walk through building a **custom OS image** for the **Raspberry Pi 5**, using the **Scarthgap** release of **Yocto** — from environment setup to booting your own image.


---


## Why Yocto and Raspberry Pi 5?


The Yocto Project lets you build a custom Linux system from the ground up — only what your project needs, nothing extra.
The Raspberry Pi 5 brings improved performance, modern interfaces, and strong community support, making it a great choice for experimentation and real-world deployment.


Together, they form a powerful platform for IoT and embedded development — whether you’re building a lightweight sensor gateway, a connected controller, or a fully featured edge device.


As a well-known and easily available development board, the Raspberry Pi is ideal for demonstrating how to design, build, and customize your own embedded operating system.


In this guide, you’ll learn how to:
- Set up a Yocto environment. 
- Configure layers and build your first image. 
- Flash and boot it on your Pi. 
- Customize the OS with your own packages or layers. 


---


## What You’ll Need?


Before we begin, make sure you've the following.


### Hardware
- Raspberry Pi 5. 
- SD Card (32GB or larger recommended). 
- Power supply and HDMI cable. 
- Keyboard, Monitor.


### Host System
- Linux host (Ubuntu 22.04+ or Fedora 39+ preferred),   
- At least 100 GB free disk space. 
- Minimum 8 GB RAM 


I will be using Ubuntu 22.04 with Intel core i7 and 16GB RAM for the demonstration.


---


## Setting Up the Build Environment


### Packages & Tools
Install the basic dependencies:
```bash
sudo apt-get install build-essential chrpath cpio debianutils diffstat file gawk gcc git iputils-ping libacl1 liblz4-tool locales python3 python3-git python3-jinja2 python3-pexpect python3-pip python3-subunit socat texinfo unzip wget xz-utils zstd
```


* This is for Ubuntu or Debian Linux distribution, For host package requirements on all supported Linux distributions, see the [Required Packages for the Build Host](https://docs.yoctoproject.org/ref-manual/system-requirements.html#required-packages-for-the-build-host) section in the Yocto Project Reference Manual.


### Configuring the Build Environment


We’ll use **Scarthgap**, the Yocto LTS release.


Download the source code:


```bash
# Get Poky (the core Yocto Project)
git clone -b scarthgap https://git.yoctoproject.org/poky poky-rpi


cd poky-rpi


# Add the Raspberry Pi layer
git clone -b scarthgap https://github.com/agherzan/meta-raspberrypi.git


```
Initialize the build environment
```bash
source oe-init-build-env
```


Add BSP layer meta-raspberrypi to conf/bblayers.conf:
```bash
bitbake-layers add-layer ../meta-raspberrypi
```


Append the following 3 lines to the end of conf/local.conf:
```
MACHINE = "raspberrypi5"
INIT_MANAGER = "systemd"
LICENSE_FLAGS_ACCEPTED = "synaptics-killswitch"
```
---


## Building Your First Image Using Bitbake
Build a basic image for Raspberry Pi 5:
```
bitbake core-image-base
```
**Note:** The build may take several hours as per your system performance. Provide good cooling for your host system.


After successful build, we'll have an image that we can flash on a micro SD card.


In case of any errors, [Yocto Dev Manual](https://docs.yoctoproject.org/dev-manual/) is an awesome place to learn about Debugging Tools and Techniques.


---


## Flashing and Booting on Raspberry Pi 5


You can find the image files that we've just build in the folder `tmp/deploy/images/<name of the machine>/`


In our case:
```
ls -l tmp/deploy/images/raspberrypi5/core-image-base-raspberrypi5.rootfs.wic.bz2
```


Attach a micro SD card to your host system and we shall use `lsblk` command to get the detail of SD card (In my case, it's `/dev/sda`)


We can use the `bzcat` and the `dd` commands together for writing the image to SD Card.
```
bzcat tmp/deploy/images/raspberrypi5/core-image-base-raspberrypi5.rootfs.wic.bz2 | sudo dd of=/dev/sda status=progress
```


Once it is done, insert the SD card into raspberry pi5 and power ON raspberry pi5 after connecting the HDMI monitor and keyboard.


Yayy!!! You can now see the raspberry pi5 turning on with the OS that we have just built :-)


Use `username: "root"`


Let's verify the OS with `uname -a` command.


---
## Add Additional Layers


### Add SSH
I am using [DropBear SSH](https://matt.ucc.asn.au/dropbear/dropbear.html) for the demonstration.


Navigate to the build folder and initialize the build environment.


```bash
source oe-init-build-env
```


Append the following line to the end of conf/local.conf using nano, vim or any text editor of your choice.
```
EXTRA_IMAGE_FEATURES:append = " ssh-server-dropbear"
```
Verify that the value of variable EXTRA_IMAGE_FEATURES contains ssh-server-dropbear using `bitbake-getvar` command.
```bash
bitbake-getvar EXTRA_IMAGE_FEATURES
```


Build the image :
```bash
bitbake core-image-base
```


Verify the image in raspberry pi5 as mentioned in the [Flashing and Booting on Raspberry Pi 5](#flashing-and-booting-on-raspberrypi5) section.


Also verify the network connectivity after connecting ethernet, use `ip -a` to verify the IP assignment.


Once the ip is noted, you can ssh in to raspberry pi 5 using another system in the same network.
In my case,
```bash
ssh root@10.74.17.24
```


### Add `apt` for Package Management
Navigate to the build folder and initialize the build environment.
```bash
source oe-init-build-env
```
Add the following line to conf/local.conf to enabled runtime package management and switch to deb packages:
```
PACKAGE_CLASSES = "package_deb"
EXTRA_IMAGE_FEATURES += "package-management"
```
Build the image:
```
bitbake core-image-base
```
Flash the image into raspberry pi 5.
Once the raspberry pi is turned on, update `sources.list.d` for `apt`, and you can use `apt` for package management.


## Create a new Layer and Recipe
Navigate to the build folder and initialize the build environment.
```bash
source oe-init-build-env
```
Create a new layer:
```bash
bitbake-layers create-layer ../meta-rohin
```
Add the new layer to conf/bblayers.conf:
```bash
bitbake-layers add-layer ../meta-rohin
```
Open another terminal and enter in to the layer we created on the previous step
```bash
cd meta-rohin
```
Create a directory for the "Hello, World" recipe:
```bash
mkdir -p recipes-apps/hello/
```
Create recipe `recipes-apps/hello/hello_git.bb` with the following content, using nano or vim.
```bash
DESCRIPTION = "Rohin's Hello World Test"
HOMEPAGE = "https://github.com/rohinraj/hello-world"
SECTION = "console/utils"
LICENSE = "MIT"
LIC_FILES_CHKSUM = "file://LICENSE;md5=f388cad0df1af35e3626518d587c0cb6"


SRC_URI = "git://github.com/rohinraj/hello-world.git;branch=master;protocol=https"
SRCREV = "e1a41536b307695064c2b1e4bb4d552089b8f3f0"


S = "${WORKDIR}/git"


inherit autotools
```
Let's now create a directory in the layer to extend core-image-base
```bash
mkdir -p recipes-core/images/
```
Create file `recipes-core/images/core-image-base.bbappend` and add in it the following line to install the hello recipe from layer meta-rohin to core-image-base:
```
IMAGE_INSTALL:append = " hello"
```
**Note:** Ensure that there is a blank space before `hello`


Go back to the terminal where the build environment has been initialized and build the new image:
```bash
bitbake core-image-base
```


Now flash the image in to sd card as in the [previous step](#flashing-and-booting-on-raspberrypi5), and verify the OS on the Raspberry Pi 5.


execute `hello` from the raspberry pi 5 and verify that our application has been successfully included in the image.
```bash
hello
```
It will be giving an output as:
```bash
Rohin Just tested a 'Hello, World!' :-)


```


Yess!! Our `Hello World` app is included in the image:-)
