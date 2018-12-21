# 前言

今天上午去上课前在虚拟机配置生产环境，没锁屏离开了宿舍。由于插着电源，电脑并不会自动进入休眠超时锁定账户。但是下课回来发现需要登陆——电脑重启了。重启的原因是什么？~~不会是被人日了吧~~，以后该如何避免这种蛋疼的情况呢？

# 信息收集

## 日志信息获取

系统的重启、程序的执行、警告等信息都会被系统日志所记录，先去系统日志看看情况。由于重启发生于上课时间，所以日志的记录时间区间很好确定，大约一个半小时。首先设置下筛选器，将该时间段内来自系统事件的信息呈现出来。

![](https://tinytracer-1256246079.cos.ap-guangzhou.myqcloud.com/windbg/1.png)



在事件列表中，确定了计算机关闭的时间，且计算机非正常关闭，引起该意外的原因是系统错误~~或者停电~~。

![](https://tinytracer-1256246079.cos.ap-guangzhou.myqcloud.com/windbg/2.png)

将筛选器的时间范围缩小为关机前后的五分钟，类型限定为错误，找到了罪魁祸首

![](https://tinytracer-1256246079.cos.ap-guangzhou.myqcloud.com/windbg/3.png)

看来是系统错误导致的蓝屏重启，错误码为`0x00000139`，该错误码包含了四个参数，分别为`0x1d`、`0xffff8706afe3f8b0`、`0xffff8706afe3f808`和`0`。内存转储到了Windows目录。

MSDN官方文档对`0x139`错误码的介绍为

>此错误检查表明内核已检测到关键数据结构的损坏。
>
>## 错误检查0x139 KERNEL_SECURITY_CHECK_FAILURE参数
>
>| 参数 | 描述                                   |
>| ---- | -------------------------------------- |
>| 1    | 错误的类型。有关更多信息，请参阅下表。 |
>| 2    | 导致错误检查的异常的陷阱帧的地址       |
>| 3    | 导致错误检查的异常的异常记录的地址     |
>| 4    | 保留                                   |

日志中第一个参数`0x1d`的十进制为29，意味着`RTL_BALANCED_NODE RBTree条目已损坏`。MSDN上只有针对错误号为3的原因可能性分析和详细说明，对本错误并无更多信息。**RTL_BALANCED_NODE到底是什么鬼？**

## 何方神圣

参考

- [讲讲怎么枚举MmMapViewInSystemSpace分配的内存](https://bbs.pediy.com/thread-246843.htm)
- [Return Flow Guard](https://xlab.tencent.com/cn/2016/11/02/return-flow-guard/)

`_RTL_BALANCED_NODE`是一个win10中存储二叉树结构的结构体。在Win10中大量使用到了该结构体，比如用于内存分配的`MmMapViewInSystemSpace`、获取描述进程Thread Control Stack的_MMVAD结构等。

```cpp
typedef struct _RTL_BALANCED_NODE 
{
    union 
    {
    	struct _RTL_BALANCED_NODE* Children[2];
    	struct 
        {
      		struct _RTL_BALANCED_NODE* Left;
      		struct _RTL_BALANCED_NODE* Right;
    	}; 
  	}; /* size: 0x0010 */
  	union 
    {
    	unsigned char Red : 1; 
        unsigned char Balance : 2; 
        unsigned __int64 ParentValue;
  	}; /* size: 0x0008 */
} RTL_BALANCED_NODE, *PRTL_BALANCED_NODE;/* size: 0x0018 */
```

RTL_BALANCED_NODE损坏意味着系统无法遍历或修改关键结构体，导致蓝屏重启。但是什么原因导致数据的损坏还有待排查。初步的信息收集到此结束，接下来使用DUMP文件获取错误的更加详细信息。

# DUMP

## 更详细的事故现场

使用`windbg`打开DUMP文件，程序会自动加载蓝屏时的栈帧、内存状态、寄存器值等信息

![](https://tinytracer-1256246079.cos.ap-guangzhou.myqcloud.com/windbg/4.png)

蓝屏时栈帧信息如下

```cpp
nt!KeBugCheckEx（蓝屏）
nt!KiBugCheckDispatch+0x69（异常）
nt!KiFastFailDispatch+0xd0（异常）
nt!KiRaiseSecurityCheckFailure+0x308（引发异常）
nt!RtlRbInsertNodeEx+0x4c4（插入节点）
nt!KiSetClockInterval+0xa6[设置时钟间隔]
nt!KiSetVirtualHeteroClockIntervalRequest+0x7b[设置虚拟异步时钟间隔请求]
nt!KeUpdatePendingQosRequest+0x2f[更新待处理的Qos请求]
nt!KeCheckAndApplyBamQos+0x8b[检查并申请Bam Qos]
nt!SwapContext+0xb8（交换线程上下文）
nt!KiIdleLoop+0x126(空闲循环)
```

由于Windows闭源的特性，中间的API，诸如`KeCheckAndApplyBamQos`、`KeUpdatePendingQosRequest`网络上并无参考资料。通过分析事故现场可以判定：某个线程在插入内核数据的时候造成了数据损坏。

使用`!analyze -v`来让`windbg`自动分析dump文件，还原案发现场

```cpp
2: kd> !analyze -v
*******************************************************************************
*                                                                             *
*                        Bugcheck Analysis                                    *
*                                                                             *
*******************************************************************************
//错误说明
KERNEL_SECURITY_CHECK_FAILURE (139)
A kernel component has corrupted a critical data structure.  The corruption
could potentially allow a malicious user to gain control of this machine.
Arguments:
Arg1: 000000000000001d, Type of memory safety violation
Arg2: ffff8706afe3f8b0, Address of the trap frame for the exception that caused the bugcheck
Arg3: ffff8706afe3f808, Address of the exception record for the exception that caused the bugcheck
Arg4: 0000000000000000, Reserved

.....
    
//事故现场，包括出错位置、寄存器状态
TRAP_FRAME:  ffff8706afe3f8b0 -- (.trap 0xffff8706afe3f8b0)
NOTE: The trap frame does not contain all registers.
Some register values may be zeroed or incorrect.
rax=0000000000000000 rbx=0000000000000000 rcx=000000000000001d
rdx=fffff8013e0b9de8 rsi=0000000000000000 rdi=0000000000000000
rip=fffff8013dd798b4 rsp=ffff8706afe3fa40 rbp=0000000000000001
 r8=fffff8013e0d6a00  r9=0000000000000000 r10=fffff8013e18e950
r11=fffff8013e0b9de8 r12=0000000000000000 r13=0000000000000000
r14=0000000000000000 r15=0000000000000000
iopl=0         nv up di pl nz ac po cy
nt!RtlRbInsertNodeEx+0x4c4:
fffff801`3dd798b4 cd29            int     29h
Resetting default scope

EXCEPTION_RECORD:  ffff8706afe3f808 -- (.exr 0xffff8706afe3f808)
ExceptionAddress: fffff8013dd798b4 (nt!RtlRbInsertNodeEx+0x00000000000004c4)
   ExceptionCode: c0000409 (Security check failure or stack buffer overrun)
  ExceptionFlags: 00000001
NumberParameters: 1
   Parameter[0]: 000000000000001d
Subcode: 0x1d FAST_FAIL_INVALID_BALANCED_TREE

.....
    
//引发蓝屏的进程信息
BUGCHECK_STR:  0x139

PROCESS_NAME:  QQ.exe

CURRENT_IRQL:  f

DEFAULT_BUCKET_ID:  FAIL_FAST_INVALID_BALANCED_TREE


......
    
//蓝屏时的栈帧信息
BAD_STACK_POINTER:  ffff8706afe3f588

LAST_CONTROL_TRANSFER:  from fffff8013de78369 to fffff8013de66b40

STACK_TEXT:  
ffff8706`afe3f588 fffff801`3de78369 : 00000000`00000139 00000000`0000001d ffff8706`afe3f8b0 ffff8706`afe3f808 : nt!KeBugCheckEx
ffff8706`afe3f590 fffff801`3de78710 : ffffb001`30139180 fffff801`3dde1b08 00004364`c57bb5b0 00004364`c57bb5b0 : nt!KiBugCheckDispatch+0x69
ffff8706`afe3f6d0 fffff801`3de76b08 : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : nt!KiFastFailDispatch+0xd0
ffff8706`afe3f8b0 fffff801`3dd798b4 : ffffcd0a`82881300 fffff801`3dd2ddb2 fffff801`3e0b9de8 00000000`00000000 : nt!KiRaiseSecurityCheckFailure+0x308
ffff8706`afe3fa40 fffff801`3dd2ddb2 : fffff801`3e0b9de8 00000000`00000000 00000000`00002710 fffff801`3e18e950 : nt!RtlRbInsertNodeEx+0x4c4
ffff8706`afe3fa50 fffff801`3df4bf03 : 00000000`00000002 00000000`00000000 00000000`00000002 00000000`00000000 : nt!KiSetClockInterval+0xa6
ffff8706`afe3fa80 fffff801`3df4a813 : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : nt!KiSetVirtualHeteroClockIntervalRequest+0x7b
ffff8706`afe3fab0 fffff801`3df4a50b : ffffb001`30139180 00000000`00000000 00000000`00000002 00000000`00000001 : nt!KeUpdatePendingQosRequest+0x2f
ffff8706`afe3faf0 fffff801`3de6da78 : ffffb001`30139180 000024ed`bd9bbfff ffffcd0a`903e3080 ffffb001`30149100 : nt!KeCheckAndApplyBamQos+0x8b
ffff8706`afe3fb20 fffff801`3de6a626 : 00000000`00000000 ffffb001`30139180 ffffb001`30149100 ffffcd0a`8b2d5080 : nt!SwapContext+0xb8
ffff8706`afe3fb60 00000000`00000000 : ffff8706`afe40000 ffff8706`afe39000 00000000`00000000 00000000`00000000 : nt!KiIdleLoop+0x126

.....
//异常捕获API返回的额外信息
FAILURE_ID_HASH_STRING:  km:0x139_1d_invalid_balanced_tree_stackptr_error_nt!kifastfaildispatch
```

结合事故现场和API的返回信息看，引发蓝屏的进程是`QQ.exe`，其子线程在调用`RtlRbInsertNodeEx`时检查失败，引发了`invalid_balanced_tree_stackptr_error`，与内存写入有关。

## 溯源

通过`!process proc_addr`命令枚举QQ进程下所有的线程，定位问题的源点

```cpp
: kd> !process ffffcd0a`90ed0500
PROCESS ffffcd0a90ed0500
    SessionId: 1  Cid: 2df4    Peb: 00798000  ParentCid: 31fc
    DirBase: 2dfc00002  ObjectTable: ffff8b043ef70a00  HandleCount: 1876.
    Image: QQ.exe
	
......

THREAD ffffcd0a903e3080  Cid 2df4.2128  Teb: 000000000079a000 Win32Thread: ffffcd0a8d09f750 RUNNING on processor 2
    
IRP List:
ffffcd0a90dcb4e0: (0006,04c0) Flags: 00060000  Mdl: 00000000

    ...
    
Win32 Start Address 0x000000000133a59c
Stack Init ffff8706b64dab90 Current ffff8706b64d9a70
Base ffff8706b64db000 Limit ffff8706b64d4000 Call 0000000000000000
Priority 12 BasePriority 8 PriorityDecrement 32 IoPriority 2 PagePriority 5
    
Child-SP          RetAddr           Call Site
ffff8706`afe3f588 fffff801`3de78369 nt!KeBugCheckEx
ffff8706`afe3f590 fffff801`3de78710 nt!KiBugCheckDispatch+0x69
......
ffff8706`afe3fa40 fffff801`3dd2ddb2 nt!RtlRbInsertNodeEx+0x4c4
```

### IRP提供的信息

引发蓝屏的线程地址为`ffffcd0a903e3080`，在蓝屏时有一个未完成的IRP请求，会不会与这个请求有关？使用`!irp ffffcd0a90dcb4e0`来查看IRP详细信息

```assembly
2: kd> !irp ffffcd0a90dcb4e0
Irp is active with 13 stacks 12 is current (= 0xffffcd0a90dcb8c8)
cmd  flg cl Device   File     Completion-Context
....
>[IRP_MJ_DIRECTORY_CONTROL(c), N/A(2)]
            0 e1 ffffcd0a845cd030 ffffcd0a90e40850 fffff80d98ad3a80-ffffcd0a8dd419a0 Success Error Cancel pending
            \FileSystem\Ntfs	FLTMGR!FltpPassThroughCompletion
Args: 00000020 00000011 00000000 00000000
```

该IRP隶属于`minifilter`模块，一般见于应用于内核进行数据交互。MSDN上对`IRP_MJ_DIRECTORY_CONTROL`的解释为

>在处理某些目录控制操作时，安全性是一个考虑因素，特别是处理更改通知的操作。安全问题为目录更改通知可能会返回有关已更改的特定文件的信息。如果用户没有遍历目录路径的权限，则不能将有关更改的信息返回给用户。
>
>文件系统运行时库支持目录更改通知，允许文件系统在返回目录更改通知之前指定用于执行遍历检查的回调函数。
>
>发生更改时，文件系统会将此指示给文件系统运行时库。然后，文件系统运行时库将调用文件系统提供的回调函数，以验证是否可以向调用者提供有关更改的信息。

即：**该线程所更改数据的内存地址触发了安全性检查**

### 栈帧现场提供的信息

目前网上能查询到的API为`RtlRbInsertNodeEx`，其函数原型为

```cpp
pub unsafe extern "system" fn RtlRbInsertNodeEx(
    Tree: PRTL_RB_TREE, 
    Parent: PRTL_BALANCED_NODE, 
    Right: BOOLEAN, 
    Node: PRTL_BALANCED_NODE
)
```

在Windbg中定位到函数调用时的语句

```assembly
fffff801`3dd2dda4 4532c0         xor     r8b, r8b
fffff801`3dd2dda7 4c8bcb         mov     r9, rbx
fffff801`3dd2ddaa 488bce         mov     rcx, rsi
fffff801`3dd2ddad e83eb60400     call    nt!RtlRbInsertNodeEx (fffff801`3dd793f0)
```

函数读取顺序为从右至左，即在调用等价的C++语句为

```cpp
RtlRbInsertNodeEx(rcx,r9,false);
```

由于DUMP文件无法保存该函数调用时的寄存器现场，无法从寄存器值还原调用情况。不过该函数并不是最底层的调用，我们可以通过函数栈帧来还原参数。`Child-SP `为函数的栈基址，`RtlRbInsertNodeEx`对应值为`ffff8706 afe3fa40`。x64架构下栈帧的分配为

![](https://tinytracer-1256246079.cos.ap-guangzhou.myqcloud.com/windbg/5.png)

因此对`Child-SP+8 `为函数的返回地址，``Child-SP+16 `为第一个参数，依次类推。使用`dw`命令打印调用时的内存数据

```assembly
2: kd> dw ffff8706`afe3fa40+8
ffff8706`afe3fa48  ddb2 3dd2 f801 ffff 9de8 3e0b f801 ffff
ffff8706`afe3fa58  0000 0000 0000 0000 2710 0000 0000 0000
ffff8706`afe3fa68  e950 3e18 f801 ffff 0001 0000 0000 0000
```

函数返回地址为`fffff8013 dd2ddb2`，第一个参数为`fffff801 3e0b9de8`，第二个参数为`0`，第三个参数为`2710`——最低位为0，即`false`。该分析结果与windbg的自动分析结果一致。

**回到函数原型，第二个参数为待插入的节点——为0。**

## 问题分析

### 猜想

- 参考资料[us-16-Yason-Windows-10-Segment-Heap-Internals-wp.pdf](https://paper.seebug.org/papers/Security%20Conf/Blackhat/2016/us-16-Yason-Windows-10-Segment-Heap-Internals-wp.pdf)

> 系统堆区段使用RB树来跟踪已释放的内存和虚拟堆栈块。它还用于跟踪大块元数据。 系统除了确保RB树是平衡的之外，NTDLL导出的函数RtlRbInsertNodeEx（）和RtlRbRemoveNode（）分别执行节点插入和移除。 为了防止由于树节点损坏而导致的任意写入，上述功能在操作RB树节点时会执行验证。 与链表节点验证类似，RB树节点验证失败将导致调用FastFail机制。 在下面的示例验证中，将操纵左子项的父项，如果左子项的ParentValue指针由攻击者控制，则可能导致任意写入。 为了防止任意写入，检查父节点的子节点是否确实是左子节点。
>
> ![](https://tinytracer-1256246079.cos.ap-guangzhou.myqcloud.com/windbg/6.png)



按照推理，事故发生的原因为：QQ的有一个循环执行的子线程，用于虚拟内存的分配与回收。由于某种原因在插入数据的时候将待插入的节点置为了NULL，系统校验失败，触发了FastFail机制导致了蓝屏。

### 证据

1、调用堆栈中有针对节点安全检查和FastFail的调用

```cpp
nt!KeBugCheckEx
nt!KiBugCheckDispatch+0x69
nt!KiFastFailDispatch+0xd0
nt!KiRaiseSecurityCheckFailure+0x308
```

2、系统崩溃前，线程在等待安全性检查的IRP

3、在`RtlRbInsertNodeEx`的执行中过程出现了0x1D中断

```assembly
//nt!KiRaiseSecurityCheckFailure  rtn_addr = fffff801`3dd798b4

fffff801`3dd798af b91d000000           mov     ecx, 1Dh
fffff801`3dd798b4 cd29                 int     29h
fffff801`3dd798b6 cc                   int     3
fffff801`3dd798b7 cc                   int     3
fffff801`3dd798b8 cc                   int     3
```

# 结论

QQ的bug？