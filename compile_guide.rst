http://processors.wiki.ti.com/index.php?title=Getting_Started_Guide_for_OMAP-L137


1) prepare host

check http://processors.wiki.ti.com/index.php/Linux_Host_Support
to install dependency packages in host linux(ubuntu 64-bit especially).

::

   #ubuntu 13.04 64-bit
   sudo apt-get install ia32-libs libjpeg62:i386 libgnomevfs2-0:i386 liborbit2:i386


2) install sdk






edit env::

   vi env.sh

   C6000_C_DIR="/home/<user>/TI/TI_CGT_C6000_6.1.9/include;/home/<user>/TI/TI_CGT_C6000_6.1.9/lib"
   PATH="/home/<user>/mv_pro_5.0/montavista/pro/devkit/arm/v5t_le/bin:/home/<user>/mv_pro_5.0/montavista/pro/bin:/home/<user>/mv_pro_5.0/montavista/common/bin:$PATH"

   source env.sh



3) compile bootloader



Optional if using pre-compiled binaries from bin directory of PSP package
    To compile SPI flash writer:
        open board_utils/flash_writers/spi_flash_writer/ccsv3.3/spiflash_writer.pjt in CCStudio v3.3
        Build the Project like any other CCStudio project

        spiflash_writer.out is placed in the Debug directory 
    Re-compiling DSP UBL should typically not be needed. If required, refer to "Additional Procedures" section of PSP User's Guide.

To compile ARM UBL
        open board_utils/armubl/ubl.pjt in CCStudio v3.3
        Build the Project like any other CCStudio project

        ubl-spi.bin file is placed in the board_utils/armubl directory 

To compile U-Boot:

untar board_utils/u-boot-1.3.3.tar.gz
Make sure MontaVista tools are in $PATH
change to u-boot-1.3.3 directory and issue::

   cd ~/OMAPL137_arm_1_00_00_11/REL_LSP_02_20_00_07/PSP_02_20_00_07/board_utilities/u-boot-1.3.3
   
   make distclean
   make da830_omapl137_config
   make 

u-boot.bin in created in top level directory 



4) compile kernel

::

   cd ~/mv_pro_5.0/montavista/pro/devkit/lsp/ti-davinci/linux-2.6.18_pro500
   
   make distclean ARCH=arm CROSS_COMPILE=arm_v5t_le-
   make da830_omapl137_defconfig ARCH=arm CROSS_COMPILE=arm_v5t_le-
   make uImage -j8 ARCH=arm CROSS_COMPILE=arm_v5t_le-

uImage in created in arch/arm/boot directory 

::

   apt-get install libncurses5-dev
   make menuconfig ARCH=arm CROSS_COMPILE=arm_v5t_le-



5) compile filesystem


 第一种方法，源码包中给出了一个ramdisk.gz，可以直接做成镜像。其位置为/opt/mv_pro_5.0/montavista/pro/devkit/arm/v5t_le/images/ramdisk.gz，具体操作：

   Create a working directory 

   host$ mkdir -p /home/user/workdir

    Copy the example ramdisk.gz file to the working directory 

   host$ cd /home/user/workdir

   host$ cp <ramdisk location>/ramdisk.gz .

    Gunzip and mount the ramdisk image to a temporary directory 

   host$ mkdir ram

   host$ gunzip ramdisk.gz

   host$ mount ramdisk ram -o loop

    Create the JFFS2 image of the file system mounted at /home/user/workdir/ram

host$ mkfs.jffs2 -r ram -e 64 -o rootfs.jffs2

生成的rootfs.jffs2就是所要的镜像文件

第二种方法，通过裁剪target文件自己制作一个镜像，此方法比较复杂，具体见TI Davinci DM6446 开发攻略.doc

 

 
/mv_pro_5.0/montavista/pro/devkit/arm/v5t_le/target/

::

   ln -s ./sbin/init init
   
   find . | cpio -o -H newc | gzip > ../initramfs.cpio.gz
   find . | cpio -o -H newc | bzip2 > ../initramfs.cpio.bz2
   
   解压
   zcat initramfs.cpio.gz | cpio -idmv
   
   gunzip  initramfs.cpio.gz
   cpio -idmv  < initramfs.cpio



6) comile kernel module




7) compile user application


arm_v5t_le-gcc hello.c -o hello 

