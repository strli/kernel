#                      GRUB2  启动过程

参考文章：<http://www.php.cn/linux-371649.html>

GRUB2的启动过程，大部分都是针对硬盘位mbr分区进行讨论的。但mbr已经逐渐退出历史舞台。

因此本部分只讨论EFI+GPT的启动模式。

![img](https://img-blog.csdn.net/20180929154037975?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI4NjI5Njg3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

在Linux启动详解1和Linux启动详解2中，已经较为详尽的阐述GRUB2之前的加载过程。所以这里只讨论

在GRUB2（操作系统引导器）获得计算机控制权之后的过程（即TSL阶段执行完成，进入RT阶段）。

GRUB2由几个映像组成:以各种方式启动GRUB的各种启动映像、一个内核映像和一组模块，这些模块与内核映像组合在一起形成一个核心映像。下面是它们的简要概述。

1.boot.img

在PC BIOS系统上，这个映像是GRUB的第一部分。它被写入主引导记录(MBR)或分区的引导扇区。因为PC引导扇区是512字节，所以这个镜像的大小正好是512字节。由于大小限制，boot.img无法理解任何文件系统结构。（虽然只讨论EFI+GPT分区的情况，但是GRUB2是支持传统的mbr启动模式的。因此在我们日常安装Linux的时候，使用分区工具总会把ext4（或者boot分区）分区之前留出1mb的空间出来。）

2.diskboot.img

从硬盘启动时，此映像用作核心映像的第一个扇区。它将核心映像的其余部分读入内存并启动内核。由于文件系统处理尚未可用，因此它使用块列表格式对核心映像的位置进行编码

3.cdboot.img

从CD-ROM驱动器引导时，此映像被用作核心映像的第一个扇区。它执行类似于diskboot.img的函数。

4.pxeboot.img

当使用PXE从网络引导时，将使用此映像作为核心映像的开始。

5.1nxboot.img

这个映像可以放在核心映像的开头，以便使GRUB看起来足够像Linux内核，以便LILO使用' image= '部分引导它。

6.kernel.img

此映像包含GRUB的基本运行时工具:设备和文件处理框架、环境变量、救援模式命令行解析器等等。它很少被直接使用，但是被内建到所有的核心图像中。

7.core.img

这是GRUB的核心映象。它是由grub-mkimage程序从内核映像和任意模块列表中动态构建的。通常，它包含足够的模块来访问/引导/grub，并在运行时从文件系统加载所有其他内容(包括菜单处理、加载目标操作系统的能力等等)。模块化设计允许核心映像保持较小，因为必须安装它的磁盘区域通常只有32KB大小。

8.*.mod

GRUB中的其他内容都驻留在可动态加载的模块中。它们通常是自动加载的，或者内置到核心映像中(如果它们是必需的)，但是也可以使用insmod命令手动加载。

 