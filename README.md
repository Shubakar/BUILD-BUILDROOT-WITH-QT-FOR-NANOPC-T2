# BUILD-BUILDROOT-WITH-QT-FOR-NANOPC-T2


Introcuction.

Buildroot is a suite of tools for compiling a complete Linux based operating system, including bootloader, kernel and root file system. With a little configuration, it will stream line the process of creating an OS to your exacting specifications. This page will describe how to configure Buildroot and compile everything, including bootloader, Linux kernel and root FS.

This extension package applies to NanoPi and NanoPi2, the main difference is the preconfigured defconfig file, the manual configuration section and SD Flashing procedure.

##Useful Links##

    Buildroot User Manual
    Buildroot Mailling List
    On IRC #buildroot chat.freenode.net

##Prerequisites##

    Linux based computer system.
    Buildroot sources.
        As of this writing 2015.05 is the latest stable release.
        Using Git: git clone git://git.buildroot.net/buildroot
            git checkout tags/2015.05 (Unnecessary if you want the nightly build)
        Direct download: Buildroot 2015.05
    Preconfigured Buildroot files (Optional)
        Buildroot NanoPi Configuration Files
    Host computer dependencies
        Buildroot Requirements
            Mandatory Packages: Make sure all are installed on your system.
            Optional Packages
                Config Interface: ncurses5 and ncurses5-dev.
                Source Fetching: More than likely, you'll need all of them at some point.

#Preconfigured Files# This is the easiest way to get up and running. Also the Bluetooth takes some extra software to upload the firmware, which has been packaged in. There is also the necessary settings to get WiFi up and running. It is still worth while to look over the Manual Configuration, to get familiar with Buildroot. To get started run:

mkdir br_nanopi
cd br_nanopi
git clone https://github.com/jrspruitt/FriendlyARM_NanoPi_Buildroot_Ext.git
git clone git://git.buildroot.net/buildroot
cd buildroot/
git checkout 2015.11.x

This will have all your source code ready to go, so next we will set up Buildroot to use the NanoPi files.

NanoPi

make BR2_EXTERNAL=../FriendlyARM_NanoPi_Buildroot_Ext nanopi_defconfig

NanoPi2

make BR2_EXTERNAL=../FriendlyARM_NanoPi_Buildroot_Ext nanopi2_defconfig

Here we told Buildroot to include the NanoPi files, and use the provided configuration file. For further configuration, such as adding packages run:

make menuconfig

Once done, you can then compile it with:

make

Once finished, you will find u-boot.bin, zImage, and rootfs.tar.gz in buildroot/output/images you can now flash these to your SD card.

Once you have booted your board, through the serial terminal, you'll need to login, username:root password is blank. To set up wifi, run:

wpa_passphrase "Your SSID" your_password >> /etc/wpa_supplicant.conf
ifdown wlan0 #If its been configured.
ifup wlan0 

If all goes well, and your network has DHCP set up, you should get an IP address and be able to ping a location.

Be sure to check the Tips section at the bottom of this page for some more info.

#Manual Configuration# This method is not as easy as the Preconfigured way, but not very hard either, and will quickly get you familiar with the menu and options of Buildroot. One precaution, unlike the Preconfigured method, this will only get the basic functionality working. The WiFi/Bluetooth especially require special files and start up scripts to get working, along with the Touchscreen driver having a patch applied so that it is functional. I would recommend at least looking over the NanoPi Buildroot Files to see the additional steps taken and supporting files needed.

To get started run:

git clone git://git.buildroot.net/buildroot
cd buildroot/
git checkout 2015.11.x
make menuconfig

This brings up the configuration menu.

    Up/down buttons move the cursor.
    Space bar toggles selection.
    Tab or right/left buttons moves between commands.
    Enter executes commands or enters sub menu ---> options.

#Configuration# Configuration options are generally entering a string value, selecting from a multiple choice block, or just selecting the option.

Settings are shown in their make menuconfig hierarchy.

    Values in parenthesis are the values that should be selected or typed in that option setting.
    [*] denotes that option should be selected.
    A trailing "--->" indicates that option links to a sub menu.

##Target Options## These options define the target architecture and as such are mostly dictated by the SoC, in this case s3c2451. For other processors you will need to verify these settings with the specifications of the chip.

NanoPi

    Target options --->
        Target Architecture (ARM (little endian)) --->
        Target Binary Format (ELF) --->
        Target Architecture Variant (arm926t) --->
        Target ABI (EABI) --->
        Floating point strategy (Soft float) --->
        ARM instruction set (ARM) --->

NanoPi2

    Target options --->
        Target Architecture (ARM (little endian)) --->
        Target Binary Format (ELF) --->
        Target Architecture Variant (cortex-A9) --->
        Target ABI (EABIhf) --->
        [*] Enable NEO SIMD extension support
        Floating point strategy (NEON) --->
        ARM instruction set (ARM) --->

##Build options## Here are some Buildroot configurations, the default values are good to use here for a basic build setup.

##Toolchain## Options here define how the toolchain will be built. For the NanoPi 4.1.x kernel gcc 4.8 is the newest version that can be used. Some software packages will require specific settings here, such as C++, which is optional, if nothing you plan to install needs it. In Target Packages, it will often tell you, if you need a different C library or specific option to use that software.

NanoPi

    Toolchain --->
        Kernel Headers (Linux 4.1.x kernel headers) --->
        C library (glibc) --->
        GCC compiler Version (gcc 4.8.x) --->
        [*] Enable C++ support

NanoPi2

    Toolchain --->
        Kernel Headers (Linux 3.4.x kernel headers) --->
        C library (glibc) --->
        GCC compiler Version (gcc 4.9.x) --->
        [*] Enable C++ support

##System configuration## Here is some settings for the operating system you will build. You can change hostname and System banner to your desires. This set up will get you a single user (root) system, but here is where you can set up a users list, specific device handling, and other basic OS configuration if desired.

    System configuration --->
        (buildroot) System hostname
        (Welcome to Buildroot) System banner
        Passwords encoding (sha-256) --->
        Init system (BusyBox) --->
        /dev management (Dynamic using mdev) --->

##Kernel## These are the necessary settings for Buildroot to compile the Linux kernel for the NanoPi, the rest of the options should be blank, not selected or left unchanged.

NanoPi

    [*] Linux Kernel
        Kernel version (Custom Git repository) --->
        (https://github.com/friendlyarm/linux-4.x.y.git) URL of custom repository
        (nanopi-v4.1.y) Custom repository version
        (nanopi) Defconfig name
        Kernel binary format (zImage) --->

NanoPi2

    [*] Linux Kernel
        Kernel version (Custom Git repository) --->
        (https://github.com/friendlyarm/linux-3.4.y) URL of custom repository
        (s5p4418-nanopi2) Custom repository version
        (nanopi2_linux) Defconfig name
        Kernel binary format (uImage) --->

##Target Packages## This is the fun part, picking which applications will be compiled and installed for your OS. A handy trick here is type "/" and you can search. Some packages require other packages to be enabled, so may not be viewable until these requirements are met. Look in <buildroot>/package for a complete list of available software.

##Filesystem images## Since the NanoPi has it's root file system on an SD card, there is no need to create a file system image. So we'll just create a tar.gz archive that can be extracted to an SD Card.

    Filesystem images
        [*] tar the root filesystem
            Compression method (gzip) --->

##U-Boot## These are the necessary settings for Buildroot to compile U-Boot for the NanoPi, the rest of the options should be left blank, not selected or left unchanged.

NanoPi

    Bootloaders --->
        [*] U-Boot
            Build system(Legacy) --->
            (nanopi) U-Boot board name
            U-Boot Version (Custom Git repository) --->
            (https://github.com/friendlyarm/uboot_nanopi.git) URL of custom repository
            (nanopi) Custom repository version
            U-Boot binary format (u-boot.bin) --->

NanoPi2

    Bootloaders --->
        [*] U-Boot
            Build system(Legacy) --->
            (s5p4418_nanopi2) U-Boot board name
            U-Boot Version (Custom Git repository) --->
            (https://github.com/friendlyarm/uboot_nanopi2.git) URL of custom repository
            (s5p4418-nanopi2) Custom repository version
            U-Boot binary format (u-boot.bin) --->
            
QT5 Configuration.

make menuconfig ---> Target Packages --> Graphic libraries and Application(graphic/text)--> QT5

#Compiling# With the configuration done, it is now time to try and compile Buildroot. Exit the configuration utility saving when it asks. Then run:

make

Depending on your computer, this may take a while, from 20 minutes to a couple hours on very old machines. At this point errors are generally due to missing requirements. Such as missing a source fetching utility, or necessary feature for compiling software. If this happens, install the missing software and run make again.

Upon successful completion, look in the <buildroot>/output/images folder, you should see zImage (NanoPi) or uImage (NanoPi2), rootfs.tar.gz, and u-boot.bin. You can now put these to an SD Card and test your setup. It is recommended to use the fusing.sh script to configure the SD card, as the script will properly partition the SD card and install the boot loader automatically. Obtain the fusing.sh script by running:

NanoPi

     git clone https://github.com/friendlyarm/sd-fuse_nanopi.git
     cd sd-fuse_nanopi

In the sd-fuse_nanopi directory, you will have to replace the zImage, rootfs.tar.gz, and u-boot.bin with the ones found in <buildroot>/output/images. Once these have been replaced, you can run the script with the following command:

      ./fusing.sh /dev/sdx (replace the x with the letter assigned to your sd card)

Refer to http://wiki.friendlyarm.com/wiki/index.php/NanoPi#Get_Started for more detailed instructions regarding the fusing script.

NanoPi2

     git clone https://github.com/friendlyarm/sd-fuse_nanopi2
     cd sd-fuse_nanopi2

For flashing to the SD Card, I find it easier to just let it download Debian and install it, then copy the rootfs and uImage to the SD Card as needed.

     sudo tar -xf /path/to/buildroot/output/image/rootfs.tar -C /path/to/SDCard/rootfs/
     sudo cp /path/to/buildroot/output/images/uImage /path/to/SDCard/boot/uImage

Optionally, /path/to/SDCard/boot/uImage is normally a symbolic link, you may wish to use a unique name for your uImage and recreate the link to point to it.

#Tips#

    To start over and clean out the build environment run:

        make clean

    Buildroot can't "uninstall" packages you've selected, compiled and then deselected, with out running make clean and starting over. If build times are an issue, it is a good idea to plan ahead here.

    If you end up with multiple instances of Buildroot, you can save downloads, by configuring Build options -> Download dir to a common directory.

    Configuration changes can be saved to the defconfig you initially loaded or you can also change the saved path in Build options > Location to save buildroot config, then run:

        make savedefconfig

    When loading a config file, depending on your changes and the currently compiled state, you may need to run make clean for all the new settings to take place. To load run:

        make MyDefconfig

    To show make options run:

        make help

