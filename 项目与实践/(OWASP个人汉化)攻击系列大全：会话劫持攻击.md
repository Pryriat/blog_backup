- [TOC]

  *最新版本 (mm/dd/yy): 08/14/2014*

  翻译自[Session hijacking attack](https://www.owasp.org/index.php/Session_hijacking_attack "Session hijacking attack")

  # 漏洞描述
  会话劫持攻击利用了Web会话控制机制，该机制通常是被会话令牌管理的。因为HTTP交流使用了许多不同的TCP连接进行通讯，因此web服务器需要一种辨识每个用户连接的方法。最有效的方法是基于Web服务器在完成对客户端的认证后向客户端的浏览器发送令牌。会话令牌通常由一个可变宽度的字符串组成，并且可以以不同的方式使用，例如在URL中，http请求的标头中作为cookie，http请求的标头的其他部分，或者在http请求的正文中。会话劫持攻击通过窃取或预测有效的会话令牌来破坏认证，以获得对Web服务器的未经授权的访问。会话令牌可能以不同的方式受到损害; 最常见的是：

  - 可预测的会话令牌
  - 会话嗅探
  - 客户端攻击（XSS，恶意JavaScript代码，特洛伊木马等）
  - [中间人攻击](https://www.andseclab.cn/2018/04/15/owasp%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%e5%9b%9b%e5%8d%81%e4%ba%8c%e4%b8%ad%e9%97%b4%e4%ba%ba%e6%94%bb%e5%87%bb%ef%bc%88mitm/ "中间人攻击")
  - [浏览器中间人攻击](https://www.andseclab.cn/2018/04/14/owasp%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%e5%9b%9b%e5%8d%81%e4%b8%80%e6%b5%8f%e8%a7%88%e5%99%a8%e4%b8%ad%e9%97%b4%e4%ba%ba%e6%94%bb%e5%87%bb/ "浏览器中间人攻击")

  # 攻击示例
  ## 示例1 - 会话嗅探
  在这个例子中，我们可以看到，首先攻击者使用嗅探器来捕获一个称为“会话ID”的有效会话令牌，然后他使用有效的会话令牌获得对Web服务器的未经授权的访问。
  ![](https://www.owasp.org/images/c/cb/Session_Hijacking_3.JPG "图2.操作令牌会话执行会话劫持攻击。")
  图2.操作令牌会话执行会话劫持攻击。

  ## 示例2 - 跨站脚本攻击
  攻击者可以通过使用运行在客户端的恶意代码或程序来破坏会话令牌。本示例展现了攻击者是如何使用一次XSS攻击来窃取对话令牌的。如果攻击者向受害者发送包含恶意JavaScript精心制作的链接，当受害者单击该链接时，JavaScript将运行并完成攻击者所做的指示。图3的示例使用了一次XSS攻击来显示当前会话cookie的值；使用相同的技术可以创建一个特殊的JavaScript脚本，将cookie发送至攻击者。
  `<SCRIPT>alert(document.cookie);</SCRIPT>`
  ![](https://www.owasp.org/images/b/b6/Code_Injection.JPG)
  图3.代码注入

  ## 其他示例
  以下攻击通过拦截客户端和服务器之间的信息交换完成：

  - [中间人攻击](https://www.andseclab.cn/2018/04/15/owasp%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%e5%9b%9b%e5%8d%81%e4%ba%8c%e4%b8%ad%e9%97%b4%e4%ba%ba%e6%94%bb%e5%87%bb%ef%bc%88mitm/ "中间人攻击")
  - [浏览器中间人攻击](https://www.andseclab.cn/2018/04/14/owasp%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%e5%9b%9b%e5%8d%81%e4%b8%80%e6%b5%8f%e8%a7%88%e5%99%a8%e4%b8%ad%e9%97%b4%e4%ba%ba%e6%94%bb%e5%87%bb/ "浏览器中间人攻击")

  # 相关的威胁代理
  - [类别：授权](https://www.owasp.org/index.php?title=Category:Authorization&action=edit&redlink=1 "类别：授权")

  # 相关攻击方式
  - [中间人攻击](https://www.andseclab.cn/2018/04/15/owasp%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%e5%9b%9b%e5%8d%81%e4%ba%8c%e4%b8%ad%e9%97%b4%e4%ba%ba%e6%94%bb%e5%87%bb%ef%bc%88mitm/ "中间人攻击")
  - [浏览器中间人攻击](https://www.andseclab.cn/2018/04/14/owasp%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%e5%9b%9b%e5%8d%81%e4%b8%80%e6%b5%8f%e8%a7%88%e5%99%a8%e4%b8%ad%e9%97%b4%e4%ba%ba%e6%94%bb%e5%87%bb/ "浏览器中间人攻击")
  - [会话预测](https://www.owasp.org/index.php/Session_Prediction "会话预测")

  # 相关漏洞
  - [类别：输入验证漏洞](https://www.owasp.org/index.php/Category:Input_Validation_Vulnerability "类别：输入验证漏洞")

  # 相关防御措施
  - [类别：会话管理](https://www.owasp.org/index.php/Category:Session_Management "类别：会话管理")

  # 参考
  - [http://www.iss.net/security_center/advice/Exploits/TCP/session_hijacking/default.htm](http://www.iss.net/security_center/advice/Exploits/TCP/session_hijacking/default.htm "http://www.iss.net/security_center/advice/Exploits/TCP/session_hijacking/default.htm")
  - [http://en.wikipedia.org/wiki/HTTP_cookie](http://en.wikipedia.org/wiki/HTTP_cookie "http://en.wikipedia.org/wiki/HTTP_cookie")