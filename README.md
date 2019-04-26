#                                     Linux源码阅读笔记

​	Linux源码阅读笔记用于记录阅读源码过程中遇到的问题。我阅读源码本身是一个自底向上的过程，并且对于每一层中，如何编程与操作也有简单的介绍。

直观的

1、base目录

简单的介绍现代计算机的组成，包括PCIE总线、CPU、GPU、内存、主板等。

操作：在硬件的层次中，除去了解硬件的架构以外，CPU只能简单的调节主频

率，内存除去调整主频以外还能调整时序等，GPU与主板可以刷BIOS。

搜索关键词：GPU BIOS 超频

2、efi目录

现代主板大部分都采用了efi的引导方式，这里主要介绍的是efi引导过

程中，主板固件部分的efi运行原理以及过程。

操作：存储在主板固件中的数据是可以导出的，导出的数据为aml格式的，具体

操作见：github/strli/aml

3、grub2目录

在主板固件中Bios初始化以后（这里指的是efi），将会由系统引导器来

负责为操作系统提供一些准备，然后引导操作系统，grub2是一个开源的

gnu项目，目前三大主流系统中，windos使用winNT引导，linux主要采

用grub2，MacOS主要采用Clover（这里是针对非主流Mac用户）

4、vmlinuzefi与initrdefi

grub2执行完成后，将会向系统内核传递参数，其中vmlinuz用来初始化

操作系统，initrd用来初始化内存盘（开辟内存空间，把操作系统载入进去）

5、systemd

linux内核初始化完成以后，所有的硬件都已经就绪，程序需要的运行环境也

已经准备好了，此时系统的第一个进程systemd就会运行（早起第一个运行的

程序都是init，但是由于init是线性执行，影响效率），systemd是一个相对复

的进程，由多个部分组成，与系统其他功能耦合度较高，由于linux本身是微内

核架构，所以也有人反对使用。



Ps：接下来的几个部分介绍类似与操作系统原理，cs的本科生都学过会，在操作系统

原理中，会把操作系统功能分为五大部分，分别为：进程管理，内存管理，网络管理，

存储管理，

MM(内存管理)

内存管理部分，主要阐述linux的内存管理的机制









在阅读源码的时候参考了很多资料

参考部分如下：

https://github.com/torvalds/linux        	      					源码树

https://github.com/MintCN/linux-insides-zh 					Linux内核揭秘

https://wiki.archlinux.org/index.php/GRUB					Archwiki写的逆天的好

https://blog.csdn.net/qq_28629687/article/details/82887477 	自己的博客对UEFI的一些总结

https://blog.csdn.net/qq_28629687/article/details/82894307     图解UEFI部分画了两个小时



















































人生最大的理想，莫过于参与Kernel开发了！

