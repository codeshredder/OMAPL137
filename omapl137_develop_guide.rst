==========================================================
  TI OMAPL137 Develop Guide
==========================================================

.. contents::

Authors
==========

`codeshredder <https://github.com/codeshredder>`_ 

Reference
==========

most information from TI Support site::

   http://processors.wiki.ti.com/index.php?title=Getting_Started_Guide_for_OMAP-L137
   http://processors.wiki.ti.com/index.php/GSG:_Building_Software_Components_for_OMAP-L1#Rebuilding_the_Linux_kernel
   http://processors.wiki.ti.com/index.php/Omapl137/DA830_linux_bootup
   http://processors.wiki.ti.com/index.php/GSG_For_DA8xx_ARM%2BDSP_Devices_Using_Community_Linux#Flash_Image
   http://processors.wiki.ti.com/index.php/Building_PSP_Components_for_OMAP-L1x_on_v3.x_Kernel
   http://processors.wiki.ti.com/index.php/Getting_Started_Guide_for_C6747
   http://www.ti.com/lit/an/sprab04g/sprab04g.pdf
   http://processors.wiki.ti.com/index.php/MontaVista_Linux_PSP_for_OMAP-L137
   http://processors.wiki.ti.com/index.php/DaVinci_(ARM9)_PSP_Releases


What is it?
==============

It is a short way to build L137 develop environment. 


Overview
====================

about OMAPL137::

   http://www.ti.com/product/omap-l137


about L137_EVM::

   http://processors.wiki.ti.com/index.php/EVM_Overview_-_OMAP-L137


first of all, you must buy a TI L137 Demo Board first. you may need SN from it.


Host Prepare
============

check http://processors.wiki.ti.com/index.php/Linux_Host_Support
to install dependency packages in host linux(ubuntu 64-bit especially).

::

   #ubuntu 12.04 64-bit
   sudo apt-get install ia32-libs libjpeg62:i386
   sudo apt-get gawk
   
   #ubuntu 13.04 64-bit
   sudo apt-get install ia32-libs libjpeg62:i386 libgnomevfs2-0:i386 liborbit2:i386
   sudo apt-get gawk

   #ubuntu 14.04 64-bit
   sudo -i
   cd /etc/apt/sources.list.d
   echo "deb http://archive.ubuntu.com/ubuntu/ raring main restricted universe multiverse" >ia32-libs-raring.list
   apt-get update
   apt-get install ia32-libs
   rm ia32-libs-raring.list
   apt-get update
   apt-get gawk


Install SDK
============

* Download SDK:

1) Register TI account

2) Add SN from demo board

3) get the latest sdk from www.ti.com/myregisteredsoftware. such as

::

   OMAPL137_arm_setuplinux_1_00_00_11.bin  (Linux OMAPL137 ARM Installer)
   bios_setuplinux_5_33_05.bin   (BSP BIOS 5_33_05 Linux Installer)
   xdctools_setuplinux_3_10_05_61.bin   (Linux XDC Tools)
   LPTB-02.03.00.02-beta.bin   (Linux performance test bench)
   mvl_5_0_0801921_demo_sys_setuplinux.bin   (MVL 5.0 Tools)
   ti_cgt_c6000_6.1.9_setup_linux_x86.bin   (Codegen 6.1.9 For Linux)


4) install sdk


::

   # all below install to /home/<user>/<application>
   $ ./OMAPL137_arm_setuplinux_1_00_00_11.bin --mode console
   $ ./xdctools_setuplinux_3_10_05_61.bin --mode console
   $ ./bios_setuplinux_5_33_05.bin --mode console
   $ ./ti_cgt_c6000_6.1.9_setup_linux_x86.bin --mode console
   
   # all below install to the same dir(/home/<user>/mv_pro_5.0)
   $ ./mvl_5_0_0801921_demo_sys_setuplinux.bin --mode console
   $ cd /home/<user>/OMAPL137_arm_1_00_00_11/REL_LSP_02_20_00_07/
   $ ./mvl_5_0_0_demo_lsp_setuplinux_02_20_00_07.bin --mode console
   
   $ cd /home/<user>/mv_pro_5.0/
   $ sudo tar xvf LSP_02_20_00_07.tar.gz
   $ sudo tar xvf mvltools5_0_0801921_update.tar.gz
   

5) edit env

::

   vi env.sh

   C6000_C_DIR="/home/<user>/TI/TI_CGT_C6000_6.1.9/include;/home/<user>/TI/TI_CGT_C6000_6.1.9/lib"
   PATH="/home/<user>/mv_pro_5.0/montavista/pro/devkit/arm/v5t_le/bin:/home/<user>/mv_pro_5.0/montavista/pro/bin:/home/<user>/mv_pro_5.0/montavista/common/bin:$PATH"
   
   chmod +x env.sh
   source env.sh



Build Bootloader
====================


1) To compile SPI flash writer:

   open board_utils/flash_writers/spi_flash_writer/ccsv3.3/spiflash_writer.pjt in CCStudio v3.3
   Build the Project like any other CCStudio project
   
   spiflash_writer.out is placed in the Debug directory 
   Re-compiling DSP UBL should typically not be needed. If required, refer to "Additional Procedures" section of PSP User's Guide.


2) To compile DSP UBL:

   open board_utils/dspubl/ubl.pjt in CCStudio v3.3
   Build the Project like any other CCStudio project.
   we can get ubl-spi.out after build project.
   
   use AISgen.exe to convert ubl-spi.out to dsp-spi-ais.bin
   

3) To compile ARM UBL:

   open board_utils/armubl/ubl.pjt in CCStudio v3.3
   Build the Project like any other CCStudio project
   
   ubl-spi.bin file is placed in the board_utils/armubl directory 


4) To compile U-Boot:

untar board_utils/u-boot-1.3.3.tar.gz::

   cd /home/<user>/OMAPL137_arm_1_00_00_11/REL_LSP_02_20_00_07/PSP_02_20_00_07/board_utilities/
   tar xvf u-boot-1.3.3.tar.gz

Make sure MontaVista tools are in $PATH.

change to u-boot-1.3.3 directory and issue::

   cd /home/<user>/OMAPL137_arm_1_00_00_11/REL_LSP_02_20_00_07/PSP_02_20_00_07/board_utilities/u-boot-1.3.3
   
   make distclean
   make da830_omapl137_config
   make 

u-boot.bin in created in top level directory.


5) To flash Bootloader:

There are four modes for using the serial flasher::

    Erase the target flash type - This will erase the entire contents of the flash.
        C:\flasher>sfh_OMAP-L137.exe -erase 
    Flash the memory with a single application image - This will place an application image at address 0x0 of the flash.
        C:\flasher>sfh_OMAP-L137.exe -flash_noubl <binary application file> 
    Flash the memory with a UBL and application image - This will place the UBL at address 0x0 and an application image, such as u-boot, at address 0x10000. This is used for the AM1707 device.
        C:\flasher>sfh_OMAP-L137.exe -flash <UBL binary file> <binary application file> 
    Flash the memory with a DSP UBL, ARM UBL, and application image - This will place a DSP AIS file at address 0x0 of the flash, an ARM UBL at address 0x2000, and an application image, such as u-boot, at address 0x8000. This is used for the OMAPL137_v1 and OMAPL137_v2 devices.
        C:\flasher>sfh_OMAP-L137.exe -flash_dsp <DSP UBL AIS file> <ARM UBL binary file> <binary application file> 
    for example:
        sfh_OMAP-L137.exe -flash_dsp dsp-spi-ais.bin ubl-spi.bin u-boot.bin
    

reference::

   http://processors.wiki.ti.com/index.php/Serial_Boot_and_Flash_Loading_Utility_for_OMAP-L137



Build linux kernel
====================

Compile default kernel::

   cd /home/<user>/mv_pro_5.0/montavista/pro/devkit/lsp/ti-davinci/linux-2.6.18_pro500
   
   make distclean ARCH=arm CROSS_COMPILE=arm_v5t_le-
   make da830_omapl137_defconfig ARCH=arm CROSS_COMPILE=arm_v5t_le-
   
   make uImage -j8 ARCH=arm CROSS_COMPILE=arm_v5t_le-
   make modules -j8 ARCH=arm CROSS_COMPILE=arm_v5t_le-
   make modules_install INSTALL_MOD_PATH=/home/<user>/fs/smallfs ARCH=arm CROSS_COMPILE=arm_v5t_le-


notice::

   1) make modules to filesystem directory.
   2) uImage in created in arch/arm/boot directory.


if want to change kernel config, you can do this::

   sudo apt-get install libncurses5-dev
   
   make menuconfig ARCH=arm CROSS_COMPILE=arm_v5t_le-


kernel config::

   # kernel config
   networking --> networking options --> IP：Kernel level autoconfiguration --> off



Build linux fs
====================

sometimes, need root

1) small fs

there is a small ramfs image in /home/<user>/mv_pro_5.0/montavista/pro/devkit/arm/v5t_le/images/ramdisk.gz

::

   # Create a working directory 
   mkdir -p /home/<user>/fs
   
   # Copy the example ramdisk.gz file to the working directory 

   cd /home/<user>/fs
   cp /home/<user>/mv_pro_5.0/montavista/pro/devkit/arm/v5t_le/images/ramdisk.gz ./

   # Gunzip and mount the ramdisk image to a temporary directory 

   mkdir ram
   gunzip ramdisk.gz
   mount ramdisk ram -o loop
   
   mkdir smallfs
   cp -rf ram/* smallfs/


2) big fs

There is a big filesystem directory in /home/<user>/mv_pro_5.0/montavista/pro/devkit/arm/v5t_le/target/

::

   mkdir /home/<user>/fs/bigfs
   cp -rf /home/<user>/mv_pro_5.0/montavista/pro/devkit/arm/v5t_le/target/* /home/<user>/fs/bigfs/
   cd /home/<user>/fs/bigfs

if want to customize rootfs,refer to this::

   http://zjbintsystem.blog.51cto.com/964211/339865/


3) use ramdisk

make fs::

   genext2fs -b 4096 -d smallfs ramdisk
   gzip -9 -f ramdisk

kernel config::

   Device Drivers --> Block devices --> Initial RAM filesystem and RAM disk (initramfs/initrd) support
   Device Drivers --> Block devices --> RAM disk support
   File systems --> Second extended fs support


u-boot cmdline::

   setenv bootargs mem=32M console=ttyS2,115200n8 root=/dev/ram0 rw initrd=0xc1180000,4M

4) use initramfs

make fs::

   # make initramfs
   
   cd /home/<user>/fs/smallfs
   ln -s ./sbin/init init
   
   find . | cpio -o -H newc | gzip > ../initramfs.cpio.gz
   
   # to uncompress
   zcat initramfs.cpio.gz | cpio -idmv
   # or
   gunzip  initramfs.cpio.gz
   cpio -idmv  < initramfs.cpio
   

kernel config::

   
   General setup --> Initramfs source file(s)                    off
   Device Drivers --> Block devices --> Initial RAM filesystem and RAM disk (initramfs/initrd) support
   Device Drivers --> Block devices --> RAM disk support         off


u-boot cmdline::

   setenv bootargs mem=32M console=ttyS2,115200n8 root=/dev/ram0 rw initrd=0xc1180000,<actual initramfs size>


5) kernel with initramfs

kernel config::

   General setup --> Initramfs source file(s)                    set initramfs file path
   Device Drivers --> Block devices --> Initial RAM filesystem and RAM disk (initramfs/initrd) support
   Device Drivers --> Block devices --> RAM disk support         off

indicate the fs directory in kernel config.then make uImage.the uImage will include initramfs.


u-boot cmdline::

   setenv bootargs mem=32M console=ttyS2,115200n8 root=/dev/ram0 rw

no need to indicate initrd=xxxx.


6) use flash fs

make fs::

   # Create the JFFS2 image of the file system mounted at /home/<user>/workdir/ram
   
   mkfs.jffs2 -r smallfs -e 64 -o rootfs.jffs2



Boot linux
====================

setup network::

   U-Boot > printenv
   bootdelay=3
   baudrate=115200
   bootfile="uImage"
   ethaddr=00:0e:99:03:18:98
   filesize=1B8994
   fileaddr=C0700000
   ipaddr=172.16.3.100
   serverip=172.16.3.203
   bootcmd=sf probe 0;sf read 0xc0700000 0x1E0000 0x220000; bootm 0xc0700000
   bootargs=console=ttyS2,115200n8 root=/dev/mmcblk0p1 noinitrd rw ip=off mem=32M
   stdin=serial
   stdout=serial
   stderr=serial
   ver=U-Boot 1.3.3 (Jun 28 2012 - 13:59:37)
   
   Environment size: 384/16380 bytes
   U-Boot > 


boot initramfs::

   tftp 0xc0700000 uImage
   tftp 0xc1180000 initramfs.cpio.gz
   setenv bootargs console=ttyS2,115200n8 root=/dev/ram0 rw initrd=0xc1180000,<actual initramfs size>
   bootm 0xc0700000



Develop kernel module
====================




Develop user application
====================

arm_v5t_le-gcc hello.c -o hello 



Update boot loader
====================

boot from seriel
----------

reference::

   http://processors.wiki.ti.com/index.php/Serial_Boot_and_Flash_Loading_Utility_for_OMAP-L137

install mono::

   apt-get install mono-complete

make Serial_Boot_and_Flash_Loading_Utility::

   tar xvf OMAP-L137_FlashAndBootUtils_2_40.tar.gz
   cd OMAP-L137_FlashAndBootUtils_2_40/OMAP-L137/
   make

use::
    	
    Erase the target flash type - This will erase the entire contents of the flash.
        mono ./sfh_OMAP-L137.exe -erase 
    Flash the memory with a single application image - This will place an application image at address 0x0 of the flash.
        mono ./sfh_OMAP-L137.exe -flash_noubl <binary application file> 
    Flash the memory with a UBL and application image - This will place the UBL at address 0x0 and an application image, such as u-boot, at address 0x10000. This is used for the AM1707 device.
        mono ./sfh_OMAP-L137.exe -flash <UBL binary file> <binary application file> 
    Flash the memory with a DSP UBL, ARM UBL, and application image - This will place a DSP AIS file at address 0x0 of the flash, an ARM UBL at address 0x2000, and an application image, such as u-boot, at address 0x8000. This is used for the OMAPL137_v1 and OMAPL137_v2 devices.
        mono ./sfh_OMAP-L137.exe -flash_dsp <DSP UBL AIS file> <ARM UBL binary file> <binary application file> 



Licensing
============

This project is licensed under Creative Commons License.

To view a copy of this license, visit [ http://creativecommons.org/licenses/ ].

Contacts
===========

codeshredder  : evilforce@gmail.com

