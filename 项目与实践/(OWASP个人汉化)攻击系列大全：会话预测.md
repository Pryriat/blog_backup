[TOC]

*最新版本(mm/dd/yy): 12/6/2011*

翻译自[Session Prediction](https://www.owasp.org/index.php/Session_Prediction "Session Prediction")

# 漏洞描述
会话预测攻击重点在于预测允许攻击者绕过应用程序身份验证模式的会话ID值。通过分析和理解会话ID的生成过程，攻击者可以预测一个有效的会话ID值来获得应用的访问权限。第一步，攻击者需要收集一些有效的会话ID值以用于辨识认证的用户。然后，他必须了解会话ID的结构，用于创建它的信息以及应用程序用来保护它的加密或散列算法。一些错误的实现使用由用户名或其他可预测信息组成的会话ID，如时间戳或客户端IP地址。在最坏的情况下，这些信息以明文形式使用，或者使用像base64编码这样的弱算法进行编码。另外，攻击者可以实施暴力攻击技术来生成和测试不同的会话ID值，直到他成功访问应用程序。

# 攻击示例
某个应用程序的会话ID信息通常由固定宽度的字符串组成。随机性对避免其预测非常重要。查看图1中的示例，会话ID变量由JSESSIONID表示，其值为“user01”，它对应于用户名。通过尝试新的值，比如“user02”，可以在没有事先验证的情况下进入应用程序。
![](https://www.owasp.org/images/b/b8/Predictable_cookie.JPG)

# 外部参考

- [http://www.iss.net/security_center/advice/Exploits/TCP/session_hijacking/default.htm](http://www.iss.net/security_center/advice/Exploits/TCP/session_hijacking/default.htm "http://www.iss.net/security_center/advice/Exploits/TCP/session_hijacking/default.htm")

- [http://en.wikipedia.org/wiki/HTTP_cookie](http://en.wikipedia.org/wiki/HTTP_cookie "http://en.wikipedia.org/wiki/HTTP_cookie")


# 相关威胁
- [类别：授权](https://www.owasp.org/index.php?title=Category:Authorization&action=edit&redlink=1 "类别：授权")

# 相关攻击方式
- [中间人攻击](https://www.andseclab.cn/2018/04/15/owasp%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%e5%9b%9b%e5%8d%81%e4%ba%8c%e4%b8%ad%e9%97%b4%e4%ba%ba%e6%94%bb%e5%87%bb%ef%bc%88mitm/ "中间人攻击")
- [会话劫持攻击](https://tinytracer.com/archives/%ef%bc%88owasp%e4%b8%aa%e4%ba%ba%e6%b1%89%e5%8c%96%ef%bc%89%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%ef%bc%9a%e4%bc%9a%e8%af%9d%e5%8a%ab%e6%8c%81%e6%94%bb%e5%87%bb/ "会话劫持攻击")
- [操作用户权限标识符](https://tinytracer.com/archives/%ef%bc%88owasp%e4%b8%aa%e4%ba%ba%e6%b1%89%e5%8c%96%ef%bc%89%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%ef%bc%9a%e6%93%8d%e4%bd%9c%e7%94%a8%e6%88%b7%e6%9d%83%e9%99%90%e6%a0%87%e8%af%86/ "操作用户权限标识符")

# 相关漏洞
- [类别：输入验证漏洞](https://www.owasp.org/index.php/Category:Input_Validation_Vulnerability "类别：输入验证漏洞")


# 相关防御措施
- [ 类别：会话管理控制](https://www.owasp.org/index.php/Category:Session_Management_Control " 类别：会话管理控制")