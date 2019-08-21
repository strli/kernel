#                                     Linux源码阅读笔记

​	Linux源码阅读笔记用于记录阅读源码过程中遇到的问题。阅读源码本身是一个自底向上的过程，并且对于每一层中，如何编程与操作也有简单的介绍。

1、base目录

简单的介绍现代计算机的组成，包括PCIE总线、CPU、GPU、内存、主板等。

操作：在硬件的层次中，除去了解硬件的架构以外，CPU只能简单的调节主频

率，内存除去调整主频以外还能调整时序等，GPU与主板可以刷BIOS。



搜索关键词：GPU BIOS 超频

2、efi目录

现代主板大部分都采用了efi的引导方式，这里主要介绍的是efi引导过

程中，主板固件部分的efi运行原理以及过程。



操作：存储在主板固件中的数据是可以导出的，导出的数据为aml格式的，具体

操作见：https://github.com/strli/efi



3、grub2目录

在主板固件中Bios初始化以后（这里指的是efi），将会由系统引导器来

负责为操作系统提供一些准备，然后引导操作系统，grub2是一个开源的

gnu项目，目前三大主流系统中，windos使用winNT引导，linux主要采

用grub2，MacOS主要采用Clover（这里是针对非主流Mac用户）。



4、linuzefi与initrdefi

grub2执行完成后，将会向系统内核传递参数，并将控制权交给内核。

vmlinux：（https://en.wikipedia.org/wiki/Vmlinux#bzImage）

​	linux系统的内核，编译内核源码之后生成的文件。

initrd：（https://en.wikipedia.org/wiki/Initrd）

​	是一个临时的根文件系统，它在初始化期间被 Linux 内核使用，而那时其他

的文件系统尚未被挂载。



5、systemd

linux内核初始化完成以后，所有的硬件都已经就绪，程序需要的运行环境也

已经准备好了，此时系统的第一个进程systemd就会运行（早起第一个运行的

程序都是init，但是由于init是线性执行，影响效率），systemd是一个相对复

的进程，由多个部分组成，与系统其他功能耦合度较高，由于linux本身是微内

核架构，所以也有人反对使用。











