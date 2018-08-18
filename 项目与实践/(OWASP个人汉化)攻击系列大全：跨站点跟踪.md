*最新版本 (mm/dd/yy): 11/10/2014*

翻译自[Cross Site Tracing](https://www.owasp.org/index.php/Cross_Site_Tracing "Cross Site Tracing")

# 漏洞描述

- 跨站点跟踪（XST）攻击涉及使用跨站点脚本（XSS）以及TRACE或TRACK HTTP方法。根据RFC 2616的规定，“TRACE允许客户端查看请求链另一端正在接收的内容，并将该数据用于测试或诊断信息”，TRACK方法的工作原理相同,不过只适用于Microsoft的IIS网络服务器。即使cookie设置了“HttpOnly”标志并且/或者公开了用户的授权标头，XST也可以用作通过跨站点脚本（XSS）窃取用户Cookie的方法。

- 尽管TRACE方法显然无害，但在某些情况下可成功利用TRACE方法窃取合法用户的凭证。这种攻击技术在2003年被Jeremiah Grossman发现，试图绕过微软在Internet Explorer 6 SP1中引入的HttpOnly标签以防止Cookie被JavaScript访问。 事实上，跨站点脚本中最常见的攻击模式之一是访问document.cookie对象，并将其发送到攻击者控制的Web服务器，以便他/她可以劫持受害者的会话。 将Cookie标记为HttpOnly以禁止JavaScript访问它，防止它被发送给第三方。 但是，即使在这种情况下，TRACE方法也可以绕过此保护并访问cookie。


- 现代浏览器现在可以防止通过JavaScript创建TRACE请求，但是，还发现了其他使用浏览器发送TRACE请求的方式，例如使用Java。

# 风险因素
待定

# 攻击示例
使用命令行中的cURL将TRACE请求发送到启用TRACE的本地主机上的Web服务器的示例。 注意Web服务器如何响应发送给它的请求。

```shell
$ curl -X TRACE 127.0.0.1
TRACE / HTTP/1.1
User-Agent: curl/7.24.0 (x86_64-apple-darwin12.0) libcurl/7.24.0 OpenSSL/0.9.8r zlib/1.2.5
Host: 127.0.0.1
Accept: */*
```

在这个例子中，请注意我们如何发送带请求的Cookie头，并且它也在Web服务器的响应中。

```shell
$ curl -X TRACE -H "Cookie: name=value" 127.0.0.1
TRACE / HTTP/1.1
User-Agent: curl/7.24.0 (x86_64-apple-darwin12.0) libcurl/7.24.0 OpenSSL/0.9.8r zlib/1.2.5
Host: 127.0.0.1
Accept: */*
Cookie: name=value
```

在这个例子中，TRACE方法被禁用，注意我们如何得到一个错误，而不是我们发送的请求。

```shell
$ curl -X TRACE 127.0.0.1
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>405 Method Not Allowed</title>
</head><body>
<h1>Method Not Allowed</h1>
<p>The requested method TRACE is not allowed for the URL /.</p>
</body></html>
```

JavaScript XMLHttpRequest TRACE请求示例。在Firefox 19.0.2中，它将不起作用并返回“Illegal Value”错误。在谷歌浏览器25.0.1364.172环境下它不会工作，并返回一个“Uncaught Error: SecurityError: DOM Exception 18”错误。 这是因为现代浏览器现在阻止XMLHttpRequest中的TRACE方法来帮助缓解XST。

```javascript
<script>
  var xmlhttp = new XMLHttpRequest();
  var url = 'http://127.0.0.1/';

  xmlhttp.withCredentials = true; // send cookie header
  xmlhttp.open('TRACE', url, false);
  xmlhttp.send();
</script>
```

# 防御措施
## Apache
在Apache 1.3.34,2.0.55和更高版本中，在主配置文件中将TraceEnable指令设置为“off”，然后重新启动Apache。 有关更多信息，请参阅[TraceEnable](http://httpd.apache.org/docs/2.2/mod/core.html#traceenable "TraceEnable")。

# 相关的威胁代理
待定

# 相关攻击方式
[跨站点脚本（XSS）](https://tinytracer.com/archives/owasp%e4%b8%aa%e4%ba%ba%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%ef%bc%9a%e8%b7%a8%e7%ab%99%e7%82%b9%e8%84%9a%e6%9c%ac%ef%bc%88xss%ef%bc%89/ "跨站点脚本（XSS）")

# 相关漏洞
待定

# 相关防御措施

- [输入过滤](https://www.owasp.org/index.php/Input_Validation "输入过滤")

- [输出过滤](https://www.owasp.org/index.php?title=Output_Validation&action=edit&redlink=1 "输出过滤")

- [规范化](https://www.owasp.org/index.php/Canonicalization "规范化")

# 参考

- 跨网站跟踪（XST）：[http://www.cgisecurity.com/whitehat-mirror/WH-WhitePaper_XST_ebook.pdf](http://www.cgisecurity.com/whitehat-mirror/WH-WhitePaper_XST_ebook.pdf "http://www.cgisecurity.com/whitehat-mirror/WH-WhitePaper_XST_ebook.pdf")

- [HTTP方法测试和XST（OWASP-CM-008）](https://www.owasp.org/index.php/Testing_for_HTTP_Methods_and_XST_(OWASP-CM-008) "HTTP方法测试和XST（OWASP-CM-008）")

- [OSVDB 877](http://osvdb.org/show/osvdb/877 "OSVDB 877")

- [CVE-2005-3398](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2005-3398 "CVE-2005-3398")

- [XSS：获得对HttpOnly Cookie的访问（2012年）](http://seckb.yehg.net/2012/06/xss-gaining-access-to-httponly-cookie.html "XSS：获得对HttpOnly Cookie的访问（2012年）")

- [Mozilla Bug 302489](https://bugzilla.mozilla.org/show_bug.cgi?id=302489 "Mozilla Bug 302489")

- [Mozilla Bug 381264](https://bugzilla.mozilla.org/show_bug.cgi?id=381264 "Mozilla Bug 381264")