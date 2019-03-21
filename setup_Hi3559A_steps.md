# Setup Hi355A

***By ZHF***

***V1.1***

***2019-03-21***

## 交叉工具链安装

`aarch64-himix100-linux.tgz` 为编译在arm内核上64位程序的工具链，`gcc-arm-none-eabi-4_9-2015q3.tgz` 为编译在arm内核上32位程序的工具链。

解压并安装工具链

```
tar -xvf aarch64-himix100-linux.tgz
cd aarch64-himix100-linux/
sudo ./aarch64-himix100-linux.install
tar -xvf gcc-arm-none-eabi-4_9-2015q3.tgz
cd gcc-arm-none-eabi-4_9-2015q3/
sudo ./aarch64-himix100-linux.install
```

## 解压并安装SDK

```
tar -xvf Hi3559AV100.tgz
cd Hi3559AV100/
source skd.unpack # 解压缩 （若要压缩，运行 source skd.cleanup）
```

## 编译镜像源码

进入`osdrv/`目录，采用`aarch64-himix100-linux` 64bit工具链进行编译。默认采用**SPI Nand** flash

### 0. 添加linux系统源码

将文件 `linux-4.9.y.tgz` 解压并拷贝到`osdrv/opensource/kernel/` 目录下

### 1. 编译 mkimage 工具

运行下面命令先编译一遍uboot，生成 uimage 工具

```
make BOOT_MEDIA=spi AMP_TYPE=linux hiboot
```

进入文件夹 `osdrv/opensource/uboot/u-boot-2016.11/tools` 并运行

```
sudo cp mkimage /usr/local/bin
```

复制可执行文件 *mkimage* 到系统中为了以后生成 *uImage*。

> *为确保万无一失，执行 `make clean` 以清除编译过的文件*

### 2. 编译整个osdrv目录 (kernel+uboot+文件系统)：

编译(A53MP+A73MP)多核linux的命令：

```
    make BOOT_MEDIA=spi AMP_TYPE=linux all
```

编译(A53MP+A73MP)多核linux+A53UP单核liteos的命令：

```
    make BOOT_MEDIA=spi AMP_TYPE=linux_liteos all
```

### *清除整个osdrv目录的编译文件：

```
make clean
```

### *彻底清除整个osdrv目录的编译中间文件：

```
make distclean
```

### 3.单独编译kernel image：

* 方法1(推荐)：

待进入内核源代码目录后，执行以下操作

```
cp arch/arm64/configs/hi3559av100_arm64_big_little_defconfig .config 或
cp arch/arm64/configs/hi3559av100_arm64_big_little_nand_defconfig .config 或
cp arch/arm64/configs/hi3559av100_arm64_big_little_emmc_defconfig .config 或
cp arch/arm64/configs/hi3559av100_arm64_big_little_ufs_defconfig .config

make ARCH=arm64 CROSS_COMPILE=aarch64-himix100-linux- menuconfig

cp .config arch/arm64/configs/hi3559av100_arm64_big_little_defconfig 或
cp .config arch/arm64/configs/hi3559av100_arm64_big_little_nand_defconfig 或
cp .config arch/arm64/configs/hi3559av100_arm64_big_little_emmc_defconfig 或
cp .config arch/arm64/configs/hi3559av100_arm64_big_little_ufs_defconfig
```

osdrv顶层目录下执行：

```
make BOOT_MEDIA=spi AMP_TYPE=linux atf
(BOOT_MEDIA,AMP_TYPE根据需要进行传参)
```

* 方法2：

1. 待进入内核源代码目录后，执行以下操作

```
cp arch/arm64/configs/hi3559av100_arm64_big_little_defconfig .config 或
cp arch/arm64/configs/hi3559av100_arm64_big_little_nand_defconfig .config 或
cp arch/arm64/configs/hi3559av100_arm64_big_little_emmc_defconfig .config 或
cp arch/arm64/configs/hi3559av100_arm64_big_little_ufs_defconfig .config

make ARCH=arm64 CROSS_COMPILE=aarch64-himix100-linux- menuconfig
make ARCH=arm64 CROSS_COMPILE=aarch64-himix100-linux- uImage
```

2. 进入`opensource/arm-trusted-firmware/arm-trusted-firmware`目录，执行mk.sh脚本(参考主Makefile中atf命令进行适配)，在`opensource/arm-trusted-firmware/arm-trusted-firmware/build/hi3559av100/debug`目录下，
生成的fip.bin文件就是ATF+kernle的镜像

### 4. 单独编译uboot：

* 方法1(推荐)：

osdrv顶层目录下执行：

```
make BOOT_MEDIA=spi AMP_TYPE=linux hiboot
(BOOT_MEDIA,AMP_TYPE根据需要进行传参)
```

* 方法2：

待进入boot源代码目录后，执行以下操作

```
make CROSS_COMPILE=aarch64-himix100-linux- hi3559av100_defconfig 或
make CROSS_COMPILE=aarch64-himix100-linux- hi3559av100_nand_defconfig 或
make CROSS_COMPILE=aarch64-himix100-linux- hi3559av100_emmc_defconfig 或
make CROSS_COMPILE=aarch64-himix100-linux- hi3559av100_ufs_defconfig
make CROSS_COMPILE=aarch64-himix100-linux- -j 20
```

Windowns下进入到`osdrv/tools/pc/uboot_tools/`目录下打开对应平台的Excel文件,在`main`标签中点击"Generate reg bin file"按钮,生成reg_info.bin即为对应平台的表格文件。

从`osdrv/tools/pc/uboot_tools`目录拷贝reg_info.bin到boot目录,重命名为.reg

```
cp ../../../tools/pc/uboot_tools/reg_info.bin .reg
make CROSS_COMPILE=aarch64-himix100-linux- u-boot-z.bin
```

`opensource/uboot/u-boot-2016.11`下生成的u-boot-hi3559av100.bin即为可用的u-boot镜像

### 5. 制作文件系统镜像：

在`osdrv/pub/`中有已经编译好的文件系统，因此无需再重复编译文件系统，只需要根据单板上启动介质的规格型号制作文件系统镜像即可。

* spi flash使用jffs2格式的镜像，制作jffs2镜像时，需要用到spi flash的块大小。这些信息会在uboot启动时会打印出来。建议使用时先直接运行mkfs.jffs2工具，根据打印信息填写相关参数。下面以块大小为64KB为例：

```
osdrv/pub/bin/pc/mkfs.jffs2 -d osdrv/pub/rootfs_glibc_xxx -l -e 0x40000 -o osdrv/pub/rootfs_hi3559av100_256k.jffs2
```

* nand flash使用yaffs2格式的镜像，制作yaffs2镜像时，需要用到nand flash的pagesize和ecc。这些信息会在uboot启动时会打印出来。建议使用时先直接运行mkyaffs2image工具，根据打印信息填写相关参数。下面以2KB pagesize、4bit ecc为例：

```
osdrv/pub/bin/pc/mkyaffs2image100 osdrv/pub/rootfs_glibc_xxx osdrv/pub/rootfs_hi3559av100_2k_4bit.yaffs2 1 2
```

* emmc/ufs使用ext4格式的镜像：以96MB镜像为例：

```
make_ext4fs -l 96M -s osdrv/pub/rootfs_hi3559av100_96M.ext4 osdrv/pub/rootfs_glibc_xxx
```

### 编译完成

编译后的二进制文件在`image_glibc_big-little_arm64\` (双系统) 或 `image_glibc_multi-core_arm64\`(单系统) 中。

### 注意事项

1. 在windows下复制源码包时，linux下的可执行文件可能变为非可执行文件，导致无法编译使用；u-boot或内核下编译后，会有很多符号链接文件，在windows下复制这些源码包, 会使源码包变的巨大，因为linux下的符号链接文件变为windows下实实在在的文件，因此源码包膨胀。因此使用时请注意不要在windows下复制源代码包。
2. 编译板端软件
Hi3559AV100板端代码编译时, 需要在Makefile里面添加选项`-mcpu=cortex-a53`或`mcpu=cortex-a73.cortex-a53`。
如：

```makefile
CFLAGS += -mcpu=cortex-a53
CXXFlAGS += -mcpu=cortex-a53
```

或:

```makefile
CFLAGS += -mcpu=cortex-a73.cortex-a53
CXXFlAGS += -mcpu=cortex-a73.cortex-a53
```

其中`CXXFlAGS中`的XX根据用户Makefile中所使用宏的具体名称来确定，e.g: `CPPFLAGS`。

## 烧写镜像

### 单系统 Linux 方案烧写步骤 (SPI NAND FLASH)

1. 配置 tftp 服务器

可以使用任意的 tftp 服务器，将发布包 image_glibc_multi-core_arm64 目录下的相关文件拷贝到 tftp 服务器目录下。具体配置步骤参见文档 `setup-tftp.txt`。

2. 参数配置

单板上电后，敲任意键进入 u-boot。设置 ipaddr（单板 ip）、ethaddr（单板的 MAC 地址）和 serverip（即 tftp 服务器的 ip）。

```
setenv netmask 255.255.255.0        # 子网掩码
setenv ipaddr 10.214.211.29         # ip地址
setenv serverip 10.214.211.30       # 服务器ip地址
setenv gatewayip 10.214.211.1       # 网关地址
```

`ping serverip`，确保网络畅通。

3. 烧写 multi-core 版本映像文件到 SPI NAND

 > *注意：单 Linux 方案要烧写 image_glibc_multi-core_arm64 目录中的镜像文件！*

* 地址空间说明

 |uboot|kernel|rootfs|
 |-----|------|------|
 |  1M |  9M  |  16M |

以下操作基于图示的地址空间分配，也可以根据实际情况进行调整。

* 烧写 u-boot

```
mw.b 0x44000000 0xff 0x100000;tftp 0x44000000 u-boot-hi3559av100.bin;nand erase 0x0 0x100000;nand write 0x44000000 0x0 0x100000
```

> *指令含义*
> 
> ```
> mw.b <start> 0xff <size>                # 初始化内存
> tftp <start> u-boot-hi3559av100.bin     # 通过tftp向以<start>的内存写入文件
> nand erase <start> <size>               # 擦掉内存内容
> nand write <scr> <dst> <size>           # 内存写入
> nand write.yaffs <scr> <dst> <size>     # 文件系统写入
> ```

* 烧写内核

```
mw.b 0x44000000 0xff 0x900000;tftp 0x44000000 uImage_hi3559av100_multi-core;nand erase 0x100000 0x900000;nand write 0x44000000 0x100000 0x900000
```

* 烧写文件系统

```
mw.b 0x44000000 0xff 0x1000000;tftp 0x44000000 rootfs_hi3559av100_2k_4bit.yaffs2;nand erase 0xA00000 0x1000000;
nand write.yaffs 0x44000000 0xA00000 0xcd1700 （0xcd1700 为 rootfs 文件实际大小）
```

* 设置启动参数

```
setenv bootargs 'mem=512M console=ttyAMA0,115200 root=/dev/mtdblock2 rw;rootfstype=yaffs2 mtdparts=hinand:1M(boot),9M(kernel),16M(rootfs)';setenv bootcmd 'nand read 0x44000000 0x100000 0x900000;bootm 0x44000000';saveenv;
```

* 重启系统
```
reset
```

### 单系统 Linux 方案烧写步骤 (EMMC)

初始化步骤同 SPI NAND FLASH

* 烧写 multi-core 版本映像文件到 EMMC

 > *注意：单 Linux 方案要烧写 image_glibc_multi-core_arm64 目录中的镜像文件！*

* 地址空间说明

 |uboot|kernel|rootfs|
 |-----|------|------|
 |  1M |  9M  |  96M |

以下操作基于图示的地址空间分配，也可以根据实际情况进行调整。

* 烧写 u-boot

```
mw.b 0x42000000 0xff 0x100000; tftp 0x42000000 u-boot-hi3559av100EMMC.bin; mmc write 0 0x42000000 0x0 0x800
```

> 注意： emmc 写入时按照块写入, 一块为512字节，因此写入的大小为实际大小除512

* 烧写内核

```
mw.b 0x42000000 0xff 0x900000; tftp 0x42000000 uImageEMMC.bin; mmc write 0 0x42000000 0x800 0x4800
```

* 烧写文件系统

```
mw.b 0x42000000 0xff 0x6000000; tftp 0x42000000 rootfs_hi3559av100_96M.ext4; mmc write.ext4sp 0 0x42000000 0x5000 0x30000;
```

* 设置启动参数

```
setenv bootargs 'mem=512M console=ttyAMA0,115200 clk_ignore_unused rw rootwait root=/dev/mmcblk0p3 rootfstype=ext4 blkdevparts=mmcblk0:1M(boot),9M(kernel),96M(rootfs)';
setenv bootcmd 'mmc read 0 0x44000000 0x800 0x4800; bootm 0x44000000'; saveenv; re;
```

### 进入系统设置网络参数

```
ifconfig eth0 hw ether 00:10:67:20:81:70
ifconfig eth0 10.214.211.29 netmask 255.255.255.0
route add default gw 10.214.211.1
mount -t nfs -o nolock -o tcp -o rsize=32768,wsize=32768 10.214.211.30:/media/Data/project/ /mnt
```

## SVP 工具安装(nnie 编译器)

> 目前经过测试，nnie工具所依赖的protobuf只能在ubuntu 14.04 GCC 4.9 平台编译，否则会出现nnie工具链接不到符号的错误。

### Protobuf 安装

1. 到 https://github.com/google/protobuf/releases/tag/v2.5.0 ，进入下载页面点击`protobuf-2.5.0.tar.gz`：

2. 解压后执行

```
./configure --prefix=/path/you/want/to/set
make -j8        # 8线程编译
make install    # 若安装在系统目录，需要sudo权限
```

3. 添加系统环境变量

运行命令 `gedit ~/.bashrc` 打开`.bashrc` 文件，添加如下内容： 
```
export PATH=(protobuf path)/protobuf2.5.0/bin:$PATH
export LD_LIBRARY_PATH=(protobuf path)/protobuf2.5.0/lib:$LD_LIBRARY_PATH
export PKG_CONFIG_PATH=(protobuf path)/protobuf2.5.0/lib/pkgconfig:PKG_CONFIG_PATH
```
关闭文件，运行 `source ~/.bashrc`

### 安装OpenCV 3.4.5

具体安装教程见 `opencv.txt` 。

### 安装mapper

解压 `nnie_mapper` 到合适的目录，并将该目录添加到系统的 `PATH` 变量中。Done！
