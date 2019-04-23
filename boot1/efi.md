# 				EFI初始化

本部分主要参考 戴正华 著《UEFI原理与编程》

CPU在加电后会进入16位实模式状态运行，同时CPU的逻辑电路设计为加电瞬间将CS的值设置为

0xF000、IP的值置为0xFFF0，这样CS：IP就指向0xFFFF0这个地址位置。然后开始执行固件

固件的执行分为七个阶段：

![img](https://img-blog.csdn.net/20180928172939543?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI4NjI5Njg3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

1.SEC(Security Phase)阶段是平台初始化的第一个阶段，计算机系统加电后进入这个阶段。

​        （1）SEC阶段功能

​           UEFI系统开机或重启进入SEC阶段，从功能上说，它执行以下4种任务

​            1）接受并处理系统启动和重启信号：系统加电信号、系统重启信号、系统运行过程中的严重

异常信号

​             2）初始化临时存储区域：系统运行在SEC阶段时，仅CPU和CPU内部资源被初始化，各种外

部设备和内存都没有被初始化，因而系统需要一些临时RAM区域，用于代码和数据的存储，我们将之称

为临时RAM，以示与内存的区别。这些临时RAM只能位于CPU内部。最常用的临时RAM是Cache，当

Cache被配置为no--eviction模式时，可以作为内存使用，读命中时返回Cache中的数据，读缺失时不会向

主存发出缺失事件；写命中时讲数据写入Cache，写缺失时不会向主存发出缺失事件，这种技术称为CAR

（Cache As Ram）

​              3）作为可信系统的根：作为取得对系统控制权的第一部分，SEC阶段时整个可信系统的 根。SEC

能被系统信任，以后的各个阶段才有被信任的基础。通常，SEC在将控制权转移给PEI之前，课可以验证PEI

​              4）传递系统参数给下一阶段（即PEI）：SEC阶段的一切工作都是为PEI阶段做准备，最终SEC要把

控制权转交给PEI，同时要将现阶段的成果汇报给PEI。汇报的手段就是讲如下信息作为参数传递给PEI的入口函数。

​                 $系统当前状态，PEI可以根据这些状态判断系统的健康状况。

​                 $可启动固件（Boot Fireware Volume）的地址和大小。

​                 $临时RAM区域的地址和大小

​                 $栈的地址和大小

​              （2）SEC阶段执行流程

​                 上面介绍了SEC的功能，下面再来看看SEC的执行流程，如图所示：

​                 

![img](https://img-blog.csdn.net/20180928175159742?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI4NjI5Njg3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

​                 

​                    以临时RAM初始化为界，SEC的执行又分为两大部分：临时 RAM生效之前称为Reset Vector

阶段，临时RAM生效后调用SEC入口函数从而进入SEC功能区

​                    其中Reset Vector的执行流程如下：

​                      1）进入固件入口

​                      2）从实模式转换到32位平坦模式（包含模式）

​                      3）定位固件中的BFV（Boot Firmware Volume）

​                      4）定位BFV中的SEC映像

​                      5）若是64位系统，从32位模式转换到64位模式

​                      6）调用SEC入口函数

​                      下面代码描述了从固件入口Reset Vector到SEC入口函数的执行过程

​                      （EDK2   UEFICpuPkg/ResetVector/Vtf0/Ia16/ResetVectorVtf0.asm)

​                      resetVector:

​                      jmp           short     EarlyBspInitReal16

​                        (EDK2   UEFICpuPkg/ResetVector/Vtf0/Ia16/Init16.asm )

​                       EarlyBspInitReal16:

​                       mov        di, 'BP'

​                      jmp          short Main16

​                       (EDK2  UefiCpuPkg/ResetVector/Vtf0/Main.asm)

​                       Main16:

​                       OneTimeCall    EarlyInit16

​                       OneTimeCall    TransitionFromReal16To32BitFlat  ;  //从实模式转换到32位平坦模式

​                       OneTimeCall    Flat32SearchForBfvBase；               //定位固件中的BFV

​                       OneTimeCall    Flat32SearchForSecEntryPoint；      //定位BFV中的SEC镜像

​                       //esi寄存器存放了SEC的入口地址，ebp寄存器存放了BFV起始地址

​                      %ifdef    ARCH_IA32

​                      mov eax ,esp

​                      jmp esi; 跳转到SEC入口

​                      %else

​                     OneTimeCall  Transition32To64Flat；         //从32位模式转换到64位模式

​                      ................

​                      jmp rsi;                                                         //跳到SEC入口

​                      %endif

​                      在Reset Vector部分，因为系统还没有RAM，因而不能使用基于栈的程序设计

所有的函数调用都使用jmp指令模拟。OneTimeCall是宏，用于模拟call指令。例如宏调用

OneTimeCall  EarlyInit16，如图：

![img](https://img-blog.csdn.net/20180928181654711?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI4NjI5Njg3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

​                       进入SEC功能区后，首先利用CAR技术初始化栈，初始化IDT，初始化 EFI_SEC_PEI_HAND_OFF

将控制权转交给PEI，并讲EFI_SEC_PEI_HAND_OFF传递给PEI。

​                        不同的硬件平台，SEC代码会有不同的实现方式，但大致执行过程相似。下面以OVMF为例，介绍SEC

功能区的执行过程。

​     

![img](https://img-blog.csdn.net/20180928182308596?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI4NjI5Njg3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

​                                                                                           

![img](https://img-blog.csdn.net/20180928182338600?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI4NjI5Njg3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

![img](https://img-blog.csdn.net/20180928182359775?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI4NjI5Njg3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

![img](https://img-blog.csdn.net/20180928182421984?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI4NjI5Njg3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 2.PEI阶段

​            PEI（Pre-EFI Initialization）阶段资源仍然十分有限内存到了PEI后期才被初始化，其主要功能

是为DXE准备执行环境，将需要传递到DXE的信息组成HOB（Handoff Block)列表，最终将控制权转交

到DXE手中。PEI执行流程如图：

![img](https://img-blog.csdn.net/20180928183848553?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI4NjI5Njg3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

​                               从功能上讲，PEI可分为以下两部分。

​                              $PEI内核（PEI Foundation）：负责PEI基础服务和流程

​                              $PEIM(PEI Moudule)派遣器：主要功能是找出系统中所有 PEIM，并根据PEIM之间的依赖

关系按顺序执行PEIM。PEI阶段对系统的初始化主要由PEIM完成的。

​                              每个PEIM是一个独立的模块，模块的入口函数类型定义如下：

​                                    typedef EFI_STATUS(EFIAPI *EFI_PEIM_ENTRY_POINT32(

​                                           IN EFI_PEI_FILE_HANDLE FileHandle,IN CONST EFI_PEI_SERVICE **PeiServices

​                                            )

​                           通过PEIServices，PEIM可以使用PEI阶段提供的系统服务，通过这些系统服务，PEIM可以访问

PEI内核。PEIM之间的通信通过PPI（PEIM-to-PEIM Interfaces）完成。

​                           PPI与DXE阶段的Protocol类似，每个PPI是一个结构体，包含了函数指针和变量，

例如：

​                            struct    _EFI_PEI_DECOMPRESS_PPI{

​                                   EFI_PEI_DECOMPRESS_DECOMPRESS Decompress;

​                           }

​                           extern EFI_GUID   gEfiPeiDecompressPpiGuid;

​                         每个PPI都有一个GUID。根据GUID，通过PeiServices的LocatePpi服务可以得到GUID对应的PPI实例

​                         UEFI的一个重要特点是其模块化的设计。模块载入内存后生成Image。Image的的入口函数为

_ModuleEntryPoint。PEI也是一个模块，PEI Image的入口函数_ModuleEntryPoint,位于MdePkg/Library/PeimEntryPoint

/PeimEntryPoint.c。_ModuleEntryPoint最终调用PEI模块的入口函数PeiCore，位于MdePkg/Library/PeimEntryPoint/PerMain.c

进入PeiCore后，首先根据从SEC阶段传入的信息设置Pei Core Services，然后调用PeiDispatcher执行系统中的PEIM，当

内存初始化后，系统会发生栈切换并重新进入PeiCore。重新进入PeiCore后使用的内存为我们所熟悉的内存。所有PEIM都执行

完毕后，调用PeiServices的LocatePpi服务得到DXE IPL PPI，并调用DXE IPL PPI的Entry服务，这个Entry服务实际上是

DxeLoadCore，它找出DXE Image的入口函数，执行DXE image的入口函数并讲HOB列表传递给DXE。

 3.DXE阶段

​        DXE（Driver Execution Environment）阶段执行大部分系统初始化工作，进入此阶段时，内存已经可以被完全使用，因而此阶段

可以进行大量的的复杂工作。从程序设计的角度讲，DXE阶段与PEI阶段相似，执行流程如下：

![img](https://img-blog.csdn.net/20180929100000399?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI4NjI5Njg3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

​                 与PEI类似，从功能上讲，DXE可分为以下两部分。

​                 $DXE内核：负责DXE基础服务和执行流程

​                 $ DXE派遣器：负责调度执行DXE驱动，初始化系统设备

​                DXE提供的基础服务包括系统表、启动服务、Run Time Services。

​               每个DXE驱动是一个独立的模块，模块入口函数类型定义为：

​               typedef EFI_STATUS(EFIAPI *EFI_IMAGE_ENTRY_PINT)( 

​                   IN  EFI_HANDLE   ImageHandle,

​                   IN   EFI_SYSTEM_TABLE  *SystemTable

​               );

​              DXE驱动之间通过Protocol通信。Protocol是一种特殊的结构体，  每个Protocol对应一个

GUID，利用系统BootServices的OpenProtocol，并根据GUID来打开对应的Protocol，进而使用这个

Protocol 提供的服务。

​              当所有的Driver都执行完毕后，系统完成初始化，DXE通过EFI_BDS_ARCH_PROTOCOL找到

BDS并调用BDS的入口函数，从而进入BDS阶段。从本质上讲，BDS是一种特殊的DXE阶段的应用程序。

​             4.BDS阶段

​             BDS（Boot Services Selection）的主要功能是执行启动策略，其主要功能包括：

​            $初始化控制台设备。

​            $ 加载必要的设备驱动。

​            $根据系统设置加载和执行启动项

​            如果加载启动项失败，系统讲重新执行DXE dispatcher以加载更多的驱动，然后重新尝试加载启动项。

​            BDS策略通过全局NVRAM变量配置。这些变量可以通过运行时服务GetVariable()读取，通过SetVariable()

设置。例如，变量BootOrder定义了启动顺序，变量Boot####定义了各个启动项（####为4个十六进制大写符号）

​            用户选中某个启动项（或系统进入默认的启动）后，OS Loader启动，系统进入TSL阶段。

 5、TSL阶段

​            TSL（Transient System Load）是操作系统加载器（OS Loader）执行的第一阶段，在这一阶段OS Loader

作为一个UEFI应用程序运行，系统资源仍然由UEFI内核控制。当启动服务ExitBootServices()服务被调用后系统进入

RunTime阶段。

​            TSL阶段之所以称为临时系统，在于它存在的目的就是为操作系统加载器准备执行环境。虽然是临时系统，但

其功能已经很强大，已经具备了操作系统的雏形，UEFI Shell是这个临时系统的人机交互界面。正常情况下 ，系统不会

进入UEFI Shell，而是直接执行操作系统加载器，只有在用户干预下或操作系统加载器遇到严重错误时才会进入UEFI Shell。

6、RT阶段

​           系统进入RT（Run Time）阶段后，系统的控制权从UEFI内核转交到OS Loader手中，UEFI占用的各种资源被回收到

OS Loader，仅有UEFI运行时服务保留给OS Loader和OS使用。随着OS Loader的执行，OS最终取得对系统的控制权。

 7、AL阶段

​           在RT阶段，如果系统（硬件或软件）遇到灾难性错误，系统固件需要提供错误处理和灾难恢复机制，这种机制运行在

AL（After Life）阶段。UEFI和UEFI PI标准都没有定义此阶段的行为和规范。

 