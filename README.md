intel_smart_environment
=======================

Intel Galileo
=========
Arduino sketch can be directly loaded onto Galileo however to use c/c++ code we need a cross compiler so that the code can be compiled on a high performance workstation and then loaded on Galileo. Following are the available options to setup an environment for development.
  - Virtual Machine
  - Live CD
  - Setup the environment on Ubuntu (Advanced users)

Virtual Machine   
---------------
Install Virtualbox from https://www.virtualbox.org/   
Download the virtual hard drive file from [VM path]   
Create a new virtual machine with type: Linux, Version: Other Linux (32 bit). Select 'use existing virtual hard drive file' and use the file downloaded in previous step.

Live CD   
-------
Download disk image from [Live CD Path]   
run the following command  
```sh
dd if=devkit-live-img-final.binblob of=/dev/sdX bs=8M conv=fsync"  
```
where /dev/sdX is the usb drive location   
boot laptop from this usb drive   
Setup the environment on Ubuntu (Advanced Users)
------------------------------------------------
Follow these steps to configure a cross compiler on your workstation. 

####Prerequisites

* Ubuntu 12.04
* 40GB free space
* Board Support Packages sources 1.0.0 for Intel Quark. Download from [BSP link]
* Install packages and libraries
```sh
sudo apt-get install build-essential sed wget cvs subversion git-core \
coreutils unzip texi2html texinfo libsdl1.2-dev docbook-utils gawk \
python-pysqlite2 diffstat help2man make gcc g++ desktop-file-utils \
chrpath libgl1-mesa-dev libglu1-mesa-dev mercurial autoconf automake \
groff libtool xterm p7zip-full bitbake
```
###Process
1. Uncompress the BSP source
```sh
7z x Board_Support_Package_Sources_for_Intel_Quark_v1.0.0.7z
```
2.  Unpack the meta-clanton tarball
```sh
tar -xvzf meta-clanton_v1.0.0.tar.gz
```
3. Edit the meta-clanton_v1.0.0\setup.sh file
>Set BB_NUMBER_THREADS and PARALLEL_THREADS to “number of cores your processor has multiply by 3”   
>Set DISTRO ?= “clanton-full” 
4. Download Compile and setup Poky
```sh
cd meta-clanton_v1.0.0
./setup.sh
source poky/oe-init-build-env yocto_build
```
5. Disable uClibc, to use EGLIBC   
Edit “../meta-clanton-distro/recipes-multimedia/v4l2apps/v4l-utils_0.8.8.bbappend”
Comment these 3 lines:
```sh
#FILESEXTRAPATHS_prepend := "${THISDIR}/files:“
#SRC_URI += file://uclibc-enable.patch
#DEPENDS += "libiconv"
```
6. Default configuration   
Edit ../meta-clanton-distro/recipes-core/images/image-full.bb :
> IMAGE_INSTALL = "packagegroup-core-boot ${ROOTFS_PKGMANAGE_BOOTSTRAP} ${CORE_IMAGE_EXTRA_INSTALL} packagegroup-core-basic packagegroup-core-lsb kernel-dev“   
>IMAGE_FEATURES += "package-management tools-sdk dev-pkgs tools-debug eclipse-debug tools-profile tools-testapps debug-tweaks’’   
>IMAGE_ROOTFS_SIZE = “2048000" (this is a minimum)   

7. Edit ../meta-oe/meta-oe/recipes-multimedia/x264/x264_git.bb file 
>SRCREV = "bfed708c5358a2b4ef65923fb0683cefa9184e6f"

8. Compile the toolchain
```sh
bitbake image-full -c populate_sdk
```

9. Install the toolshain
```sh
./tmp/deploy/sdk/clanton-full-eglibc-x86_64-i586-toolchain-1.4.2.sh
```

10. Link new headers and libraries
```sh
source /opt/clanton-full/1.4.2/environment-setup-i586-poky-linux
```

###Compiling
To compile a C program, use CC, a environment variable referring to gcc for Intel Quark architecture (C compiler).
```sh
${CC} myfile.c –o myfile
```
And use ${CXX} instead of ${CC} to use g++ for Intel Quark architecture (C++ compiler)   
If you want to use a library, just add `pkg-config LIBNAME --libs` at the end of the C++ compile command.

SD Card
========

```sh
sudo dd if=iot-devkit-mmcblkp0.direct of=/dev/sd? bs=4096
```



[BSP Link]: http://somelinkeforBSPsources.com
[VM path]:http://vmpath
[Live CD Path]:http://LiveCDPath
