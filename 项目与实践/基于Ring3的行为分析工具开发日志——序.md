# 前言

我们在分析一个程序的时候，通常的步骤是拖进`PEid`等工具查壳，看区段和引用，然后拖进`OD`等调试器脱壳，或者`IDA`等工具查看源码，来对程序有个大体上的了解。然而在分析病毒样本的时候，脱壳要费一番功夫，反调试又是一道坎。接着在关键API或者字符串下断，查看运行情况，还得担心可能的暗桩或程序自带的钩子。往往一番功夫下来，连病毒是做什么的，怎么做的都不知道，绕进反调试或反混淆里面出不来。就算是定位到了关键API，参数的获取和处理也是一个难题，毕竟涉及到内存的分布、系统内部类或结构体的变量定义与解析的问题。

换一种思路，在对样本进行分析的时候，不关注于样本行为代码如何实现，而是监控样本的行为，推导出样本的行为特征和实施方式，在不知道伪源码的情况下对某一类样本的行为进行分析和汇总，似乎是一种很有意义和价值做法。这种方式不需要针对不同的样本投入大量的精力进行定制化的流程，因为恶意行为的实施归根到底还是通过系统调用，在此之前的反调试也好，加密混淆也好，都是为了调用系统`API`铺路，只要监控系统API的调用轨迹，就能大体知道样本的行为信息。比如下图是`Word 365`启动时的部分行为

![](https://github.com/Pryriat/blog_backup/blob/master/Image/ExeAnalyze/1/1.png?raw=true)

不难发现，在本行为片段中`Word`首先读取了本地某个文件，然后与服务器建立连接，读取数据，并将数据写入了本地文件，不断循环。结合文件夹的目录信息，不难分析出这是与服务器同步缓存，读取配置的行为片段。

提取程序的行为信息，进行汇总，对可能的恶意行为进行警告和拦截——似乎有点眼熟，这好像是xx卫士的实现方式。将恶意行为汇总为特征，即可以减少数据量，面对新出的恶意程序时无论如何变换，只要行为的特征不变，特征仍然有效。

# 行为分析向量

对程序进行行为的监控，就得通过`Hook`或`Debug`等方式对系统API进行的执行流程进行修改。监控哪些系统`API`呢？

举个简单的例子，为了监控程序的行为，对系统所有`API`下了钩子，简简单单一个`Hello World`得到了涵盖系统环境初始化、变量分配、线程同步等等所有过程——看起来十分详细，可对`Hello World`而言，我们想得到的无非就是字符串而已，外加一点进程和控制台的写入信息，过多的分析向量将导致有效信息被埋没，加大了分析的工作量。

再或者，我关心程序的读取了哪些文件敏感信息，于是对`ReadFile`下了钩子，比如

```cpp
BOOL WINAPI ReadFile(
	HANDLE       hFile,
	LPCVOID      lpBuffer,
	DWORD        nNumberOfBytesToRead,
	LPDWORD      lpNumberOfBytesRead,
	LPOVERLAPPED lpOverlapped
)//原始函数
{
    MyReadFile(hFile, lpBuffer, nNumberOfBytesToWrite,lpNumberOfBytesWritten, lpOverlapped);//劫持并跳转到自定函数
    ....
}
```

看起来似乎无懈可击，可获取文件信息一定是`ReadFile`吗？

```cpp
HANDLE FindFirstFileA(
  LPCSTR             lpFileName,
  LPWIN32_FIND_DATAA lpFindFileData
);//搜索文件，ANSI版本
HANDLE FindFirstFileW(
  LPCWSTR             lpFileName,
  LPWIN32_FIND_DATAA lpFindFileData
);//搜索文件，Unicode版本
BOOL ReadFileEx(
  HANDLE                          hFile,
  LPVOID                          lpBuffer,
  DWORD                           nNumberOfBytesToRead,
  LPOVERLAPPED                    lpOverlapped,
  LPOVERLAPPED_COMPLETION_ROUTINE lpCompletionRoutine
);//另一种读取文件信息的方式
BOOL SetFileAttributesA(
  LPCSTR lpFileName,
  DWORD  dwFileAttributes
);//修改文件属性
DWORD SetFilePointer(
  HANDLE hFile,
  LONG   lDistanceToMove,
  PLONG  lpDistanceToMoveHigh,
  DWORD  dwMoveMethod
);//修改读取指针
.....
```

获取信息，既可以通过`ReadFile`，也可以通过`ReadFileEx`；读取的时候，既可以顺序读取，也可以通过`SetFilePointer`进行花式读取——你监控到的是`Hello World`，进行变换后可能就是一段`ShellCode`；监控到了文件，想打开看看文件的内容，结果被`SetFileAttributesA`拒之门外，更别提`Win32`的传统艺能`xxApiA`和`xxApiW`，一份函数，双倍快乐。

获取过多或过少的信息都会得到灾难性的结果，在进行行为分析设计的时候，需要做出权衡，针对哪些行为对哪些`API`进行监控。下表为某木马植入、隐蔽和恶意操作所需资源：

> | 控制类别               | 控制项                                                       |
> | ---------------------- | ------------------------------------------------------------ |
> | 脚本运行控制           | `javascript`                                                 |
> |                        | `vbscript`                                                   |
> |                        | 网页中ActiveX控件                                            |
> |                        | `WSH`                                                        |
> | 协议安全配置           | `TCP`监听                                                    |
> |                        | `UDP`监听                                                    |
> |                        | `ICMP`协议                                                   |
> |                        | 允许的TCP端口                                                |
> |                        | 允许的UDP端口                                                |
> | 系统文件及目录安全设置 | <windows>写入可执行文件                                      |
> |                        | <system>写入可执行文件                                       |
> | 注册表安全设置         | `SessionManager\\KnownDLLs`                                  |
> |                        | `HKEYLOCALMACHINE\\System\\`                                 |
> |                        | `CurrentControlSet\\Services\\`                              |
> |                        | `Run\\RunOnce\\RunServices\\RunServicesOnce\\`               |
> |                        | `HKLM\\SoftWare\\Microsoft\\WindowsNT\\CurrentVersion\\Windows]“AppInitDLLs"` |
> |                        | `HKLM\\SYSTEM\\ControlSet001\\Control\\SessionManager\\KnownDLLs` |
>
> 来源:胡卫,张昌宏,马明田.基于动态行为监测的木马检测系统设计[J].火力与指挥控制,2010,35(02):128-132.

管中窥豹，我们可以根据该木马的行为顺序。得出以下的分析向量：

- 浏览器脚本控件
- `Socket`等端口控制`API`
- 文件写入（特定目录）
- 注册表修改（特定目录）

根据特定的过程缩小范围，最终得到的目标API少了很多。在此基础上看看`Wanacry`的行为分析

![](https://github.com/Pryriat/blog_backup/blob/master/Image/ExeAnalyze/1/2.png?raw=true)

相较于之前木马的四个方面，`Wanacry`多了释放进程和系统服务两个方面。观察两份分析报告，我们不难看出，二者的着重点在于程序做了什么，相应地，在进行行为分析的时候，我们的侧重点在于程序对现有系统进行了哪些修改，而非程序读取的信息或执行的流程。综上，在设计行为分析工具的时候的分析向量可以限定为以下6个方面：

- 浏览器脚本控件
- `Socket`等端口控制`API`
- 文件写入（特定目录）
- 注册表修改（特定目录）
- 进程创建与注入
- 系统服务修改

# 实现方式

监控系统API主要有两种方式，一种是通过调试器，另一种是通过注入等手段修改函数流程。

## 调试器

通过调试器监控系统API是一种简单易行的方法，前提是目标样本没有反调试的功能，如果有则需要用调试器来解决反调试的问题（套娃）。

![](3.jpg)

通过调试器以调试权限创建目标进程，目标进程是调试器的子进程，类似于上图中的总-分架构。通过在系统API入口处设置断点，然后通过自定义的断点事件捕获函数进行处理，如

```cpp
STARTUPINFO si = { 0 };
si.cb = sizeof(si);
PROCESS_INFORMATION pi = { 0 };
if (CreateProcess(TEXT("Target.exe"),NULL,NULL,NULL,FALSE,DEBUG_PROCESS | CREATE_NEW_CONSOLE,NULL,NULL,&si,&pi) == FALSE)
{
	std::wcout << TEXT("CreateProcess failed: ") << GetLastError() << std::endl;
	return -1;
}//创建目标进程

BOOL waitEvent = TRUE;
DEBUG_EVENT debugEvent;//注册调试事件

SetBp( (GetHandleByProcName("Target.exe"), "kernel32", "WriteFile");//设置断点

while (waitEvent == TRUE && WaitForDebugEvent(&debugEvent, INFINITE))//调试循环    {
    case EXCEPTION_DEBUG_EVENT:
    	if (debugEvent.dwDebugEventCode == EXCEPTION_DEBUG_EVENT)
            if(debugEvent.u.ExceptionCode == EXCEPTION_BREAKPOINT)//断点事件
            {
                ....
            }
}
```

### 优势

- 代码直观
- 进程间数据通信便捷
- 确保调试程序结束后无样本进程残留
- 样本程序崩溃后可快速定位原因`debug`

### 劣势

- 实现较复杂，不仅需要通过调试事件确定断点地址、对应的API名称，还需要手动解析参数
- 效率较低
- 可能会遇到程序的反调试检测与暗桩

## 修改函数流程

实现系统进程监控的另一种手段是通过`DLL`注入、自定义驱动或`IAT HOOK`等方式，修改系统API的执行流程。通过这种方式修改类似于下图中的离散点架构，同一种注入手段可能会产生多个实例，存在于不同的进程中，与分析器的主进程无关。

![](4.png)

以`Ring3 DLL`注入的`Inline Hook`为例，一般来说，`WIN32 API`遵循的是`__stdcall`函数约定，其汇编前5字节如下：

```assembly
mov edi,edi
mov ebp
mov ebp,esp
```

5字节的空间刚好是`jmp xxxx`指令所占用的空间，因此可以通过`jmp Func_Addr`指令来跳转到自定的函数空间，此时栈上刚好排列了参数，不需要额外的定位和解析。由于是非正常调用，在函数中需要内联汇编手动平衡堆栈。

```cpp
#pragma pack(1)
typedef struct _JMPCODE
{
    BYTE jmp = 0xe9;
    DWORD addr = (DWORD)MyApi - (DWORD)RealFunc - 5;//注意jmp指令本身的长度
}JMPCODE,*PJMPCODE;

_declspec(naked) void WINAPI MyApi(....)//内联汇编需要_declspec(naked)前缀
{
    __asm
    {
        PUSH ebp
        mov ebp,esp
        /*
        vs2010 debug 编译后的代码由于要cmp esi esp来比较堆栈。
        所以这里在调用非__asm函数前push一下esi
        */
        push esi
    }
    .....
    __asm
    {
        pop esi//平衡堆栈

        mov ebx,Re
        add ebx,5
        jmp ebx
    }
}

void Hook()
{
    JMPCODE a
    RealFunc* re = RealFunc;//保存原始地址
    WriteProcessMemory(GetCurrentProcess(),RealFunc,&a,sizeof(JMPCODE),NULL);
}
```

### 优势

- 不需要额外的参数定位和解析
- 执行效率高
- 可以跳过系统API原有流程直接返回预期值

### 劣势

- 实现复杂，在多线程情况下可能造成程序甚至系统崩溃
- 需要进程间通信与信息交换
- 注入模块出问题后会导致样本程序的崩溃，而对分析器本身无影响，难以定位`bug`
- ~~秃头~~



