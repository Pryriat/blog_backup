[TOC]

*最新版本: 04/16/2015*

翻译自[Format string attack](https://www.owasp.org/index.php/Format_string_attack "Format string attack")

# 漏洞描述
- 当输入的字符串被应用评估为命令时，就会发生格式化字符串漏洞。由此，攻击者可以执行代码、读取栈上的信息或者在运行的程序上引发段错误，和其它导致可能危及系统安全性或稳定性的新行为。
- 为了理解这种攻击，有必要了解构成它的组件。
  - 格式化函数是ANSI C 的转换函数，类似于printf,fprintf，它将编程语言的原始变量转换为可读的字符串表示形式。
  - 格式化字符串是格式化函数的参数，是一串包含文本和格式化参数的ASCII字符串，如`printf ("The magic number is: %d\n", 1911);`
  - 格式化字符串参数，如`%x %s`定义格式函数的转换类型。

- 当应用程序没有正确验证提交的输入时攻击者可以执行攻击。在这种情况下，如果将格式字符串参数（如％x）插入到发布的数据中，则该字符串将由格式化函数进行分析，并执行参数中指定的转换。但是，格式化函数需要更多参数作为输入，并且如果不提供这些参数，则该函数可以读取或写入堆栈。这样，就存在通过一个精心设计过的输入来改变格式化函数行为的可能，允许攻击者引发拒绝服务或执行任意命令。如果应用程序在能够解释格式化字符的源代码中使用格式化函数，攻击者可以通过在网站的表单中插入格式化字符来探索漏洞。例如，如果使用`printf`函数打印插入到页面某些字段中的用户名，则该网站容易受到这种攻击，如下所示：
  `printf (userName);`
  以下是格式化函数的一些示例，如果不加以处理，会将应用程序暴露在格式字符串攻击下。

表1：格式化函数

| 格式化函数 | 描述                                 |
| ---------- | ------------------------------------ |
| fprint     | 格式化输出到文件                     |
| printf     | 输出格式化的字符串                   |
| sprintf    | 把格式化的数据写入字符串             |
| snprintf   | 把格式化的数据写入字符串，有长度检查 |
| vfprintf   | 将va_arg结构输出到文件               |
| vprintf    | 将va_arg结构输出到标准输出           |
| vsprintf   | 将va_arg结构写入字符串               |
| vsnprintf  | 将va_arg结构写入字符串，有长度检查   |

以下是一些可以使用的格式化参数及其后果：

- `%x`从堆栈读取数据
- `%s`读取进程内存中的字符串
- `%n`将整数写入进程内存中的位置

为了发现应用程序是否容易受到这种类型的攻击，有必要验证格式化函数是否接受并分析表2中显示的格式字符串参数。

| 参数 | 输出                     | 传递形式 |
| ---- | ------------------------ | -------- |
| %%   | ％字符（文字）           | 引用     |
| %p   | 指向void的指针的外部表示 | 引用     |
| %d   | 十进制                   | 值       |
| %c   | 字符                     |          |
| %u   | 无符号十进制             | 值       |
| %x   | 十六进制                 | 值       |
| %s   | 字符                     | 引用     |
| %n   | 将字符数写入指针         | 引用     |

# 风险因素
待定

# 攻击示例
## 示例1
这个例子演示了当格式化函数在格式化字符串的输入中没有收到必要的验证处理时，应用程序会如何工作。首先是应用程序在正常行为和正常输入下操作，然后，应用程序在攻击者输入格式字符串和恶意行为下运行。
接下来是示例所用的源代码

```c
#include  <stdio.h>
#include  <string.h>
#include  <stdlib.h>

int main (int argc, char **argv)
{
	char buf [100];
	int x = 1 ;
	snprintf ( buf, sizeof buf, argv [1] ) ;
	buf [ sizeof(buf) -1 ] = 0;
	printf ( “Buffer size is: (%d) \nData input: %s \n” , strlen (buf) , buf ) ;
	printf ( “X equals: %d/ in hex: %x\nMemory address for x: (%p) \n” , x, x, &x) ;
	return 0 ;
}
```

接下来是程序在使用预期输入运行时提供的输出。在这种情况下程序接收字符串"Bob"作为输入然后返回它作为输入

```shell
./formattest “Bob”

Buffer size is (3)
Data input : Bob
X equals: 1/ in hex: 0x1
Memory address for x (0xbffff73c)
```

现在将探索格式字符串漏洞。如果格式化字符串参数"%x %x"被插入到输入字符串中，当格式化函数解析参数时，输出本该是名字"Bob"，但被显示`%x`的字符串代替，该应用将显示一个内存地址的内容。

```shell
./formattest “Bob %x %x”

Buffer size is (14)
Data input : Bob bffff 8740
X equals: 1/ in hex: 0x1
Memory address for x (0xbffff73c)
```

输入Bob和格式字符串参数将归于代码中的变量buf，它应取代数据输入中的％s。 所以现在printf参数看起来像：
`printf ( “Buffer size is: (%d) \n Data input: Bob %x %x \n” , strlen (buf) , buf ) ;`
当应用程序打印结果时，格式化函数将解释格式字符串输入，显示内存地址的内容。

## 示例2
在这种情况下，当请求无效的存储器地址时，通常会导致程序终止。
`printf (userName);`
攻击者可以插入一系列格式字符串，使程序显示存储许多其他数据的内存地址，然后，攻击者增加了程序读取非法地址的可能性，导致程序崩溃，失去可用性。
`printf (%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s%s);`

# 相关攻击代理
- [承包商](https://www.owasp.org/index.php/Contractors "承包商")
- [内部软件开发商](https://www.owasp.org/index.php?title=Internal_software_developer&action=edit&redlink=1 "内部软件开发商")

# 相关攻击类型
- [代码注入](https://www.andseclab.cn/2018/04/09/owasp%E6%B1%89%E5%8C%96%E6%94%BB%E5%87%BB%E7%B3%BB%E5%88%97%E5%A4%A7%E5%85%A8%E5%85%AB%E4%BB%A3%E7%A0%81%E6%B3%A8%E5%85%A5/ "代码注入")

# 相关漏洞
- [缓冲区溢出](https://www.owasp.org/index.php/Buffer_Overflow "缓冲区溢出")

# 相关防御措施
- [类别：输入验证](https://www.owasp.org/index.php/Category:Input_Validation "类别：输入验证")

# 参考

- [http://www.webappsec.org/projects/threat/classes/format_string_attack.shtml](http://www.webappsec.org/projects/threat/classes/format_string_attack.shtml "http://www.webappsec.org/projects/threat/classes/format_string_attack.shtml")
- [http://en.wikipedia.org/wiki/Format_string_attack](http://en.wikipedia.org/wiki/Format_string_attack "http://en.wikipedia.org/wiki/Format_string_attack")
- [http://seclists.org/bugtraq/2005/Dec/0030.html](http://seclists.org/bugtraq/2005/Dec/0030.html "http://seclists.org/bugtraq/2005/Dec/0030.html")