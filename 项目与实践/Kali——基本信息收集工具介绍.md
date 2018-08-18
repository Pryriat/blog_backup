[TOC]

# whois（域名信息查询）
- 使用方法：whois + 查询域名
  ![](http://ozhtfx691.bkt.clouddn.com/kali/whois.png)

# host（获取主机IP地址）
- 功能：获取主机的ip地址
- DNS类型介绍
  ![](http://ozhtfx691.bkt.clouddn.com/kali/host1.png)

- 使用方法
  - host + 域名
    ![](http://ozhtfx691.bkt.clouddn.com/kali/host2.png)
    不带参数的host获取的是ipv4、ipv6和邮件交换记录
  - hots -a +域名 +指定dns服务器（查询详细信息）
    ![](http://ozhtfx691.bkt.clouddn.com/kali/host3.png)
  - host -v + 域名（查询所有类型）
    ![](http://ozhtfx691.bkt.clouddn.com/kali/host4.png)
  - 其他用法：
    ![](http://ozhtfx691.bkt.clouddn.com/kali/host5.png)
  - host命令查找记录是通过Kali的DNS服务器系统文件，该文件位于/etc/resolv.conf，可以直接在命令行中指定DNS服务器

# dig（获取主机ip地址）
- dig可对主机地址进行挖掘，相对于host命令，dig命令更具有灵活和清晰的显示信息
- 使用方法
  - dig + 域名
    ![](http://ozhtfx691.bkt.clouddn.com/kali/dig1.png)
    不带参数的dig只返回ipv4
  - dig 域名 + any +dns（返回所有类型地址）
    ![](http://ozhtfx691.bkt.clouddn.com/kali/dig2.png)

# dnsenum、dnsdict（获取域名下所有ip地址、dns、MX）

- dnsenum使用方法
  - dnsenum + 域名（获取ipv4地址）
    ![](http://ozhtfx691.bkt.clouddn.com/kali/dnsenum1.png)
  - 其他用法
    ![](http://ozhtfx691.bkt.clouddn.com/kali/dnsenum2.png)

- dnsdict6使用方法
  - dnsdict6 + 域名
    ![](http://ozhtfx691.bkt.clouddn.com/kali/dnsdict1.png)
    不带参数的dnsdict6为扫描ipv6地址
  - dnsdict6 + -4(ipv4) + -d(收集DNS和NS) +域名
    ![](http://ozhtfx691.bkt.clouddn.com/kali/dnsdict2.png)
    ![](http://ozhtfx691.bkt.clouddn.com/kali/dnsdict3.png)

# fierce（定位独立IP空间对应域名和主机名）
- 使用方法
  - fierce + 查找项 域名 +设置项
    ![](http://ozhtfx691.bkt.clouddn.com/kali/fierce1.png)
  - 使用`fierce -h`查看详细用法

# DMitry（收集多种信息)
- DMirty可以收集以下信息：
  - 端口扫描
  - whois主机IP和域名信息
  - 从Netcraft.com获取主机信息
  - 子域名
  - 域名中包含的邮件地址
  - 将收集的信息保存在一个文件中
- 使用方法
  ![](http://ozhtfx691.bkt.clouddn.com/kali/dmitry1.png)
- 使用示例
  - 获取 whois ，ip，主机信息，子域名，电子邮件
    ![](http://ozhtfx691.bkt.clouddn.com/kali/dmitry2.png)
  - 扫描网站端口
    ![](http://ozhtfx691.bkt.clouddn.com/kali/dmitry3.png)

# maltegoce（多样化的信息查询）（需翻墙）
- 功能：搜集：
  - 域名
  - DNS
  - Whois
  - IP地址
  - 网络块
  - 公司、组织
  - 电子邮件
  - 社交网络关系
  - 电话号码
- 多样化的查询源：
  - URL
  - 网址
  - 手机号码
  - 邮箱
  - 图片
  - 人名
  - 社交网站昵称
    ![](http://ozhtfx691.bkt.clouddn.com/kali/maltegoce.png)

# theharvester(电子邮件，用户名和主机名/子域名)
- 使用方法
  - theharvester -d 域名 -l 结果数 -b搜索源  -e指定DNS
    ![](http://ozhtfx691.bkt.clouddn.com/kali/theharvester.png)

# zenmap(nmap图形界面版本)
- 相比于nmap，zenmap简单易用，扫描方式丰富
  ![](http://ozhtfx691.bkt.clouddn.com/kali/zenmap.png)
  本质上与nmap无区别

# aircrack -ng(无限网络监听、破解)
| 组件名称    | 描述                                                         |
| :---------- | :----------------------------------------------------------- |
| aircrack-ng | 主要用于WEP及WPA-PSK密码的恢复，只要airodump-ng收集到足够数量的数据包，aircrack-ng就可以自动检测数据包并判断是否可以破解 |
| airmon-ng   | 用于改变无线网卡工作模式，以便其他工具的顺利使用             |
| airodump-ng | 用于捕获802.11数据报文，以便于aircrack-ng破解                |
| aireplay-ng | 在进行WEP及WPA-PSK密码恢复时，可以根据需要创建特殊的无线网络数据报文及流量 |
| airserv-ng  | 可以将无线网卡连接至某一特定端口，为攻击时灵活调用做准备     |
| airolib-ng  | 进行WPA Rainbow Table攻击时使用，用于建立特定数据库文件      |
| airdecap-ng | 用于解开处于加密状态的数据包                                 |
| tools       | 其他用于辅助的工具，如airdriver-ng、packetforge-ng等         |

- 破解无线网络步骤（示例）
  - airmon-ng check kill（关闭冲突进程）
  - airmon-ng start wlan0（将waln0设置为监听模式）
  - airdump-ng -c (频道)  -w 输出文件名 wlan0mon（监听频道6的wlan信号，a、s键切换显示）
  - aireplay -ng -0 攻击次数 -c 客户端地址 -a AP地址 wlan0mon(采用deauth攻击模式获取握手包)
  - 查看airdump-ng的输出是否获取了握手包信息
  - aircrack-ng -w 字典名 获取的cap文件(跑字典破密码)

# ss(显示socket状态)
- SS命令可以提供如下信息：
  - 所有的TCP sockets
  - 所有的UDP sockets
  - 所有ssh/ftp/ttp/https持久连接
  - 所有连接到Xserver的本地进程
  - 使用state（例如：connected, synchronized, SYN-RECV, SYN-SENT,TIME-WAIT）、地址、端口过滤）
  - 所有的state FIN-WAIT-1 tcpsocket连接
- 常用ss命令
  - `ss -l` 显示本地打开的所有端口
  - `ss -pl` 显示每个进程具体打开的socket
  - `ss -t -a` 显示所有tcp socket
  - `ss -u -a` 显示所有的UDP Socekt
  - `ss -o state established '( dport = \:smtp or sport = \:smtp )' `显示所有已建立的SMTP连接
  - `ss -o state established '( dport = :http or sport = :http )' `显示所有已建立的HTTP连接
  - `ss -x src /tmp/.X11-unix/* `找出所有连接X服务器的进程
  - `ss -s` 列出当前socket详细信息:
  - 其他用法：
    ![](http://ozhtfx691.bkt.clouddn.com/kali/ss.png)
