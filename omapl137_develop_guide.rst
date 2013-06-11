==========================================================
  TI OMAPL137 Develop Guide
==========================================================


Authors
==========

`codeshredder <https://github.com/codeshredder>`_ 

Reference
==========

most information from TI Support site::

http://processors.wiki.ti.com/index.php?title=Getting_Started_Guide_for_OMAP-L137


Table of Contents
=================

::

  0. What is it?
  1. Overview
  2. Host Prepare
  3. Install SDK
  4. Build Bootloader
  5. Build linux kernel
  6. Build linux fs
  7. Boot linux
  8. Develop kernel module
  9. Develop user application
  10. Licensing
  11. Contacts
  
0. What is it?
==============

It is a short way to build L137 develop environment. 


1. Overview
====================

about OMAPL137::

   http://www.ti.com/product/omap-l137


about L137_EVM::

   http://processors.wiki.ti.com/index.php/EVM_Overview_-_OMAP-L137


first of all, you must buy a TI L137 Demo Board first. you may need SN from it.


2. Host Prepare
============

check http://processors.wiki.ti.com/index.php/Linux_Host_Support
to install dependency packages in host linux(ubuntu 64-bit especially).

::

   #ubuntu 13.04 64-bit
   sudo apt-get install ia32-libs libjpeg62:i386 libgnomevfs2-0:i386 liborbit2:i386


3. Install SDK
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

   $ ./OMAPL137_arm_setuplinux_1_00_00_11.bin --mode console
   $ ./xdctools_setuplinux_3_10_05_61.bin --mode console
   $ ./bios_setuplinux_5_33_05.bin --mode console
   $ ./ti_cgt_c6000_6.1.9_setup_linux_x86.bin --mode console
   
   # all install to /home/<user>
   
   $ ./mvl_5_0_0801921_demo_sys_setuplinux.bin --mode console
   $ cd /home/<user>/OMAPL137_arm_1_00_00_11/REL_LSP_02_20_00_07/
   $ ./mvl_5_0_0_demo_lsp_setuplinux_02_20_00_07.bin --mode console
   
   # install to the same dir(/home/<user>/mv_pro_5.0)
   
   $ cd /home/<user>/mv_pro_5.0/
   $ sudo tar xvf mvltools5_0_0801921_update.tar.gz
   $ sudo tar xvf LSP_02_20_00_07.tar.gz
   
   $ sudo chown -R xxx:xxx ./


5) edit env

::

   vi env.sh

   C6000_C_DIR="/home/<user>/TI/TI_CGT_C6000_6.1.9/include;/home/<user>/TI/TI_CGT_C6000_6.1.9/lib"
   PATH="/home/<user>/mv_pro_5.0/montavista/pro/devkit/arm/v5t_le/bin:/home/<user>/mv_pro_5.0/montavista/pro/bin:/home/<user>/mv_pro_5.0/montavista/common/bin:$PATH"

   source env.sh



4. Build Bootloader
====================



1) To compile SPI flash writer::

   open board_utils/flash_writers/spi_flash_writer/ccsv3.3/spiflash_writer.pjt in CCStudio v3.3
   Build the Project like any other CCStudio project
   
   spiflash_writer.out is placed in the Debug directory 
   Re-compiling DSP UBL should typically not be needed. If required, refer to "Additional Procedures" section of PSP User's Guide.

2) To compile ARM UBL::

   open board_utils/armubl/ubl.pjt in CCStudio v3.3
   Build the Project like any other CCStudio project
   
   ubl-spi.bin file is placed in the board_utils/armubl directory 

3) To compile U-Boot:

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


5. Build linux kernel
====================

Compile default kernel::

   cd /home/<user>/mv_pro_5.0/montavista/pro/devkit/lsp/ti-davinci/linux-2.6.18_pro500
   
   make distclean ARCH=arm CROSS_COMPILE=arm_v5t_le-
   make da830_omapl137_defconfig ARCH=arm CROSS_COMPILE=arm_v5t_le-
   
   make uImage -j8 ARCH=arm CROSS_COMPILE=arm_v5t_le-
   make modules ARCH=arm CROSS_COMPILE=arm_v5t_le-
   make modules modules_install INSTALL_MOD_PATH=/home/<user>/fs/target ARCH=arm CROSS_COMPILE=arm_v5t_le-


notice: 
1) make modules to filesystem directory.

2) uImage in created in arch/arm/boot directory.


if want to change kernel config, you can do this::

   sudo apt-get install libncurses5-dev
   
   make menuconfig ARCH=arm CROSS_COMPILE=arm_v5t_le-



6. Build linux fs
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

   # Create initramfs
   
   make initramfs
   cp -rf ram/* initramfs/
   cd initrfamfs
   ln -s ./sbin/init init
   
   find . | cpio -o -H newc | gzip > ../initramfs.cpio.gz
   
   # to uncompress
   
   zcat initramfs.cpio.gz | cpio -idmv
   # or
   gunzip  initramfs.cpio.gz
   cpio -idmv  < initramfs.cpio
   
   # Create the JFFS2 image of the file system mounted at /home/<user>/workdir/ram
   
   mkfs.jffs2 -r ram -e 64 -o rootfs.jffs2


2) big fs

There is a big filesystem directory in /home/<user>/mv_pro_5.0/montavista/pro/devkit/arm/v5t_le/target/

::

   mkdir /home/<user>/fs/target
   cp -rf /home/<user>/mv_pro_5.0/montavista/pro/devkit/arm/v5t_le/target/* /home/<user>/fs/target/
   cd /home/<user>/fs/target


7. Boot linux
====================




8. Develop kernel module
====================




9. Develop user application
====================

arm_v5t_le-gcc hello.c -o hello 





10. Licensing
============

This project is licensed under Creative Commons License.

To view a copy of this license, visit [ http://creativecommons.org/licenses/ ].

11. Contacts
===========

codeshredder  : evilforce@gmail.com

