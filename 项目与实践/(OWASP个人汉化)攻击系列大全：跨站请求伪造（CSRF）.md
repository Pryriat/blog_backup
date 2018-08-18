[TOC]

*最新版本 (mm/dd/yy): 03/6/2018*

翻译自[Cross-Site Request Forgery (CSRF)](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF) "Cross-Site Request Forgery (CSRF)")

# 概览
跨站请求伪造（CSRF）是一种强制用户在当前经过认证的web应用端执行非预期操作的攻击。CSRF攻击针对状态改变请求，而不是盗窃数据，因为攻击者无法查看对伪造请求的响应。在一些社会工程学的帮助下（如通过邮件或者聊天发送链接），攻击者可以诱使web应用的用户执行攻击者选定的操作。如果受害者是一名普通用户，一次成功的CSRF攻击可以强制令其执行状态更改请求，如转移资金，更改其电子邮件地址等等。如果受害者是一名管理员账户，SCRF攻击可能会危害整个web应用程序。

# 相关安全活动
## 如何审计CSRF漏洞的代码
- 请参阅[OWASP代码审查指南](https://www.owasp.org/index.php/Category:OWASP_Code_Review_Project "OWASP代码审查指南")文章，了解[如何审计CSRF漏洞的代码](https://www.owasp.org/index.php/Reviewing_code_for_Cross-Site_Request_Forgery_issues "如何审计CSRF漏洞的代码")。

## 如何测试CSRF漏洞
- 请参阅[OWASP测试指南](https://www.owasp.org/index.php/Category:OWASP_Testing_Project "OWASP测试指南")文章，了解如何[测试CSRF漏洞](https://www.owasp.org/index.php/Testing_for_CSRF_(OTG-SESS-005) "测试CSRF漏洞")。

## 如何防止CSRF漏洞

- 有关预防措施，请参阅[CSRF预防备忘清单](https://www.owasp.org/index.php/CSRF_Prevention_Cheat_Sheet "CSRF预防备忘清单")。
- 聆听[OWASP十大CSRF播客](http://www.owasp.org/download/jmanico/owasp_podcast_69.mp3 "OWASP十大CSRF播客")。
- 大多数框架都有内置的CSRF防御支持，如[Joomla](http://docs.joomla.org/How_to_add_CSRF_anti-spoofing_to_forms "Joomla")，[Spring](http://blog.eyallupu.com/2012/04/csrf-defense-in-spring-mvc-31.html "Spring")，[Struts](http://web.securityinnovation.com/appsec-weekly/blog/bid/84318/Cross-Site-Request-Forgery-CSRF-Prevention-Using-Struts-2 "Struts")，[Ruby on Rails](http://guides.rubyonrails.org/security.html#cross-site-request-forgery-csrf "Ruby on Rails")，[.NET](http://www.troyhunt.com/2010/11/owasp-top-10-for-net-developers-part-5.html ".NET")等。
- 使用[OWASP CSRF Guard](https://www.owasp.org/index.php/Category:OWASP_CSRFGuard_Project "OWASP CSRF Guard")为您的Java应用程序添加CSRF保护。 您可以使用[CSRFProtector Project](https://www.owasp.org/index.php/CSRFProtector_Project "CSRFProtector Project")来保护您的php应用程序或任何使用Apache Server部署的项目。 在OWASP中也有一个[.Net CSRF Guard](https://www.owasp.org/index.php?title=.Net_CSRF_Guard&action=edit&redlink=1 ".Net CSRF Guard")，但它陈旧，看起来不完整。
- John Melton也有一篇优秀的[博客文章](http://www.jtmelton.com/2010/05/16/the-owasp-top-ten-and-esapi-part-6-cross-site-request-forgery-csrf/ "博客文章")，描述如何使用[OWASP ESAPI](http://www.owasp.org/index.php/Category:OWASP_Enterprise_Security_API "OWASP ESAPI")的本地反CSRF功能。

# 漏洞描述
- CSRF是一种诱骗受害者提交恶意请求的攻击。它继承了受害者的身份和特权，代表受害者执行非预期的功能。对于大多数站点来说，浏览器的请求会自动包含与该网站关联的所有凭据，如用户会话的cookie，IP地址，Windows域凭据等等。因此，如果用户已经过网站的验证，那么该站点将无法分辨受害者发送的伪造请求和受害者发送的合法请求。

- CSRF攻击的目标是导致服务器状态发生变化的功能，例如更改受害者的电子邮件地址或密码或购买某些东西。强制受害者检索数据对攻击者没有帮助，因为攻击者没有收到响应，这是受害者的工作。因此，CSRF攻击针对状态改变请求。

- 有时可能将CSRF存储在易受攻击的站点上。这种类型的漏洞被称作“存储型CSRF漏洞”。这可以简单地通过将IMG或IFRAME标记存储在接受HTML的字段中，或通过更复杂的跨站点脚本攻击来实现。如果攻击者可以将CSRF存储在站点，将放大攻击的严重程度。特别是，由于受害者比Internet上的某个随机页面更有可能查看包含攻击的页面，且受害者肯定已经拥有认证了该网站凭据，所以攻击成功的可能性大大增加。

## 同义词
CSRF有很多称呼，包括XSRF,"Sea Surf"，会话控制、跨站点参考伪造和敌对链接。微软在威胁建模以及在线文档中的许多地方将这种类型的攻击称为一键式攻击。

# 无效的预防措施

## 使用加密的cookies
记住所有的cookies，即使是经过加密的，将在每次请求中被提交。无论最终用户是否被欺骗地提交请求，所有认证令牌都将被提交。此外，应用程序容器仅使用会话标识符将请求与特定的会话对象相关联。会话标识符不会验证最终用户是否打算提交请求。

## 仅接受POST请求
应用程序可以开发为只接受POST请求以执行业务逻辑。误解在于攻击者无法创建恶意链接，就无法执行CSRF攻击。不幸的是，这个逻辑是错误的。有许多的方法可以让一个攻击者欺骗受害者以提交伪造的POST请求，例如一托管在攻击者网站中，含有隐藏值的简单表单。这种表单可以由JavaScript自动触发，也可以由认为表单会做其他事情的受害者触发。

随着时间的推移，许多有瑕疵的防御CSRF攻击的想法已经形成。 以下是我们建议您避免的一些问题。

## 多步骤事务
多步交易不足以预防CSRF。 只要攻击者可以预测或推断已完成事务的每个步骤，就可能执行CSRF攻击。

## URL重写
这可能被视为一种有用的CSRF预防技术，因为攻击者无法猜测受害者的会话ID。 但是，用户的会话ID将显示在URL中。 我们不建议通过引入其他方式来修复一个安全漏洞。

## HTTPS
HTTPS本身对CSRF没有任何防范。但是，应将HTTPS视为任何预防措施值得信赖的先决条件。

# 攻击示例

## 攻击是怎么奏效的
有许多方法可以欺骗最终用户从Web应用程序加载信息或向Web应用程序提交信息。为了执行攻击，我们必须首先了解如何为我们的受害者执行生成的有效恶意请求。让我们进入下列示例：Alice希望通过含有CSRF漏洞的bank.com网站向Bob转账100$。Maria，攻击者，想要让Alice将钱转到她那。本次攻击包括以下步骤

- 建立一个可利用URL或脚本
- 诱骗ALice执行社会工程学行动

## GET场景
- 如果应用被设计为主要使用GET请求来传递参数和执行攻击，本次转账操作可能简化到如下操作
  `GET http://bank.com/transfer.do?acct=BOB&amount=100 HTTP/1.1`

- Maria决定现在利用Alice作为受害者执行该网页应用的漏洞。Maria首先构建如下的漏洞URL，它将从Alice的账户转账100,000$到她的账户。她采用原始命令URL并用自己的名字取代受益人名称，同时显着提高转账金额：
  `http://bank.com/transfer.do?acct=MARIA&amount=100000`

- 本次攻击的社会工程方面欺骗Alice在登陆银行应用后访问这个URL。这通常通过以下技术之一来完成：
  - 发送包含HTML内容的未经请求的电子邮件
  - 在受害者使用在线银行的同时可能访问的网页上植入漏洞URL或者脚本

- 被利用的URL可以伪装成普通链接，促使受害者点击它
  `<a href="http://bank.com/transfer.do?acct=MARIA&amount=100000">View my Pictures!</a>`

或者作为一个0x0假图片
`<img src="http://bank.com/transfer.do?acct=MARIA&amount=100000" width="0" height="0" border="0">`

如果这个图片标签被包括在邮件内，Alice不会看见任何东西。但是，浏览器仍然会将请求提交给bank.com，而且没有任何可视的已经发生转移的指示。

- 一个使用GET的应用程序进行CSRF攻击的实际例子是从2008年开始的，一个被用来大规模下载恶意软件的[uTorrent漏洞](https://www.ghacks.net/2008/01/17/dos-vulnerability-in-utorrent-and-bittorrent/ "uTorrent漏洞")。

## POST场景
GET和POST攻击的唯一区别在于受害者执行的方式。让我们假设银行显示使用POST并且包含漏洞的requests请求看起来是这样：
```http
POST http://bank.com/transfer.do HTTP/1.1
acct=BOB&amount=100
```

这样的请求无法使用标准A或IMG标签传送，但可以使用FORM标签传送

```http
<form action="<nowiki>http://bank.com/transfer.do</nowiki>" method="POST">
<input type="hidden" name="acct" value="MARIA"/>
<input type="hidden" name="amount" value="100000"/>
<input type="submit" value="View my pictures"/>
</form>
```

这个表单仍然需要用户点击提交按钮，但这依然可以由JavaScript自动完成
This form will require the user to click on the submit button, but this can be also executed automatically using JavaScript:

```javascript
<body onload="document.forms[0].submit()">
<form...
```

## 其他HTTP方法
现代Web应用程序API经常使用其他HTTP方法，例如PUT或DELETE。 假设有漏洞的银行应用使用以JSON块为参数的PUT：

```http
PUT http://bank.com/transfer.do HTTP/1.1
{ "acct":"BOB", "amount":100 }
```

这些请求可以利用嵌入到漏洞页面的JavaScript执行：

```javascript
<script>
function put() {
	var x = new XMLHttpRequest();
	x.open("PUT","http://bank.com/transfer.do",true);
	x.setRequestHeader("Content-Type", "application/json"); 
	x.send(JSON.stringify({"acct":"BOB", "amount":100})); 
}
</script>
<body onload="put()">
```
幸运的是，由于同源策略限制，此请求不会被现代Web浏览器执行。 此限制是默认启用的，除非目标网站通过使用带有以下标头的[CORS](https://www.owasp.org/index.php/HTML5_Security_Cheat_Sheet#Cross_Origin_Resource_Sharing "CORS")明确地打开攻击者（或所有人）来源的跨域请求：
`Access-Control-Allow-Origin: *`

# 相关攻击

- [跨站点脚本（XSS）](https://tinytracer.com/archives/owasp%e4%b8%aa%e4%ba%ba%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%ef%bc%9a%e8%b7%a8%e7%ab%99%e7%82%b9%e8%84%9a%e6%9c%ac%ef%bc%88xss%ef%bc%89/ "跨站点脚本（XSS）")
- [跨站点历史操作（XSHM）](https://tinytracer.com/archives/%ef%bc%88owasp%e4%b8%aa%e4%ba%ba%e6%b1%89%e5%8c%96%ef%bc%89%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%ef%bc%9a%e8%b7%a8%e7%ab%99%e7%82%b9%e5%8e%86%e5%8f%b2%e6%93%8d%e4%bd%9c/ "跨站点历史操作（XSHM）")

# 相关防御措施

- 除标准会话外，还为URL和所有表单添加每个请求的随机数。 这也被称为“表单键”。 许多框架（例如Drupal.org 4.7.4+）已经或者正在开始将这种类型的“内置”保护包括到每个表单中，因此程序员不需要手动编写这种保护。
- 向所有表单添加散列（会话ID，函数名称，服务器端秘密）。
- 对于.NET，使用MAC为ViewState添加一个会话标识符（详见[CSRF预防备忘录](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)_Prevention_Cheat_Sheet#Viewstate_.28ASP.NET.29 "CSRF预防备忘录")）
- 检查客户端的HTTP请求中的引用标头可以防止CSRF攻击。 确保HTTP请求来自原始站点意味着来自其他站点的攻击将不起作用。 由于内存限制，在嵌入式网络硬件上查看引用标头检查非常常见。
  - XSS可用于同时绕过基于引用和基于标记的检查。 例如，[Samy蠕虫](http://en.wikipedia.org/wiki/Samy_%28computer_worm%29 "Samy蠕虫")使用XHR获取CSRF令牌来伪造请求。
- “虽然CSRF从根本上说是Web应用程序的问题，而不是用户，但用户可以通过在访问另一个网站之前注销网站或在每个浏览器会话结束时清除浏览器的Cookie来帮助保护他们在设计不佳的网站上的帐户。——[http://en.wikipedia.org/wiki/Cross-site_request_forgery#_note-1](http://en.wikipedia.org/wiki/Cross-site_request_forgery#_note-1 "http://en.wikipedia.org/wiki/Cross-site_request_forgery#_note-1")
- [符号化](https://www.owasp.org/index.php/Tokenizing "符号化")

# 参考

- OWASP[跨站点请求伪造（CSRF）预防备忘录](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)_Prevention_Cheat_Sheet "跨站点请求伪造（CSRF）预防备忘录")
- [跨站请求伪造（CSRF / XSRF）常见问题](http://www.cgisecurity.com/articles/csrf-faq.shtml "跨站请求伪造（CSRF / XSRF）常见问题")
> 引用：“本文作为跨站请求伪造问题的有效文档，该文档将作为现有论文，会谈和邮件列表发布信息的存储库，并将在发现新信息时进行更新。”

- [CSRF测试](https://www.owasp.org/index.php/Testing_for_CSRF_(OWASP-SM-005) "CSRF测试") 来自OWASP测试指南项目的CSRF（又称会话控制）论文。

- [CSRF漏洞：一个'沉睡的巨人'](http://www.darkreading.com/document.asp?doc_id=107651&WT.svl=news1_2 "CSRF漏洞：一个'沉睡的巨人'") 概述文章

- [客户端防御会话控制](http://www.owasp.org/index.php/Image:RequestRodeo-MartinJohns.pdf "客户端防御会话控制") Martin Johns和Justus Winter为第4届OWASP AppSec大会撰写了有趣的论文和演讲，介绍了浏览器可以采用的自动提供CSRF保护的潜在技术 - [PDF文件](http://www.owasp.org/index.php/Image:RequestRodeo-MartinJohns.pdf "PDF文件")

- [OWASP CSRF防御](https://www.owasp.org/index.php/Category:OWASP_CSRFGuard_Project "OWASP CSRF防御")
  J2EE，.NET和PHP过滤器，它为每个表单附加一个唯一的请求标记并链接到HTML响应中，以便在整个应用程序中提供对CSRF的全面防御。

- [OWASP CSRF保护器](https://www.owasp.org/index.php/CSRFProtector_Project "OWASP CSRF保护器")
  反CSRF方法，减少Web应用程序中的CSRF。 目前以PHP库和Apache 2.x.x模块的形式实现

- [关于跨站点请求伪造（CSRF）的最被忽视的事实](http://yehg.net/lab/pr0js/view.php/A_Most-Neglected_Fact_About_CSRF.pdf "关于跨站点请求伪造（CSRF）的最被忽视的事实")
  Aung Khant，[http://yehg.net](http://yehg.net/ "http://yehg.net")，解释了CSRF在危险情况下的威胁和影响。

- [OWASP CSRF测试器](https://www.owasp.org/index.php/Category:OWASP_CSRFTester_Project "OWASP CSRF测试器")
  OWASP CSRFTester为开发人员提供了测试他们的应用程序中CSRF漏洞能力。

- [Pinata-CSRF-Tool：CSRF POC工具](http://code.google.com/p/pinata-csrf-tool/ "Pinata-CSRF-Tool：CSRF POC工具")
  Pinata可以很容易地创建概念CSRF页面的证明。 协助应用程序漏洞评估。




