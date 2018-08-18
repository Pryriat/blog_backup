[TOC]

*最新版本 (mm/dd/yy): 04/23/2009*

翻译自[Traffic flood](https://www.owasp.org/index.php/Traffic_flood "Traffic flood")

# 漏洞描述
流量洪水是针对服务器DoS攻击的一种。该攻击利用了TCP连接的管理方式。该攻击包括生成大量精心设计的TCP请求，其目的是停止Web服务器或导致性能下降。该攻击还利用了HTTP协议的一个特点，在一个请求中同时打开多个连接。http协议的这个特殊功能，包括为每个html对象打开一个TCP连接并关闭它，有两种利用方式。连接攻击在连接建立期间完成，关闭攻击则在连接关闭期间完成。

# 攻击示例
## 连接攻击
- 这种类型的攻击包括使用不完整的HTTP请求建立大量假TCP连接，直到Web服务器被连接淹没并停止响应。
- 不完整的HTTP请求的目的是让Web服务器保持TCP连接处于建立状态，等待请求完成，如图1所示。根据Web服务器的实现，连接将保持此状态，直到TCP连接或Web服务器超时。因此可以在第一个连接超时前建立大量的新连接。不仅如此，新连接的增长速度比超时的要快得多。
  ![](https://www.owasp.org/images/b/b4/Trafficatual.jpg)
- 该攻击还可能影响类似Checkpoint FW1防火墙那样的访问控制代理。

## 关闭攻击

- 关闭攻击在TCP的结束阶段完成，该攻击利用了一些Web服务器处理TCP连接的终结的方式，特别是使用FIN_WAIT_1状态的方式。正如Stanislav Shalunov解释的那样，攻击有两种：缓存耗尽和过程饱和。
- 执行缓存耗尽攻击的前置条件是另一端的用户级进程写入数据时不会被阻塞并关闭描述符。内核将不得不处理所有数据，并且用户级进程将被释放。所以可以通过这种方式发送更多请求，并最终消耗所有缓存或所有物理内存（如果缓存是动态分配的）。
- 执行过程饱和的前置条件是用户级进程在尝试写入数据时被阻塞。许多HTTP服务器被设计为同一时刻只对一个用户提供服务，当大量的连接到来时，服务器会停止响应合法用户。如果服务器没有对连接数量进行限制，则资源将被捆绑，最终机器会陷入无法运行的状态。

# 相关的威胁代理

- [类别：逻辑攻击](https://www.owasp.org/index.php?title=Category:Logical_Attacks&action=edit&redlink=1 "类别：逻辑攻击")

# 相关攻击类型

- [拒绝服务](https://www.andseclab.cn/2018/04/14/owasp%E6%B1%89%E5%8C%96%E6%94%BB%E5%87%BB%E7%B3%BB%E5%88%97%E5%A4%A7%E5%85%A8%E4%BA%8C%E5%8D%81%E5%85%AD%E6%8B%92%E7%BB%9D%E6%9C%8D%E5%8A%A1/ "拒绝服务")

# 相关漏洞

-[ 类别：常规逻辑错误漏洞](https://www.owasp.org/index.php/Category:General_Logic_Error_Vulnerability " 类别：常规逻辑错误漏洞")

# 相关防御措施
待定

# 参考
[http://shlang.com/netkill/netkill.html](http://shlang.com/netkill/netkill.html "http://shlang.com/netkill/netkill.html")