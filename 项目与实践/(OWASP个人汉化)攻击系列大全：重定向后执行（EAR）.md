[TOC]

*最新版本 (mm/dd/yy): 05/2/2017*

翻译自[Execution After Redirect (EAR)](https://www.owasp.org/index.php/Execution_After_Redirect_(EAR) "Execution After Redirect (EAR)")

# 概览
重定向后执行（EAR）是攻击者忽略重定向并检索供认证用户使用的敏感内容的攻击。 一个成功的EAR漏洞可能会导致应用程序的完全破坏。

# 如何测试EAR漏洞
使用大多数代理可以忽略重定向并显示返回的内容。 在这个测试中我们使用Burp Proxy。
- 拦截请求`https://vulnerablehost.com/managment_console`
- 发送到repeater模块
- 查看响应

# 如何防御EAR漏洞
重定向后应该正确终止。 在一个函数中应该执行返回。 在其他情况下，应执行die（）等函数。 这将告诉不管页面是否被重定向应用程序都应被终止。

# 攻击示例
以下代码将检查参数“loggedin”是否为true。 如果不是这样，它会使用JavaScript将用户重定向到登录页面。使用“如何测试EAR漏洞”部分或通过在浏览器中禁用JavaScript，可以重复相同的请求，而不必遵循JavaScript重定向，并且“Admin”部分无需身份验证即可访问。

```html
<?php
if (!$loggedin) {
    print "<script>window.location = '/login';</script>\n\n";
}
?>
<h1>Admin</h1>
<a href=/mu>Manage Users</a><br />
<a href=/ud>Update Database Settings</a>
```

# 参考
- [CWE-698：重定向后执行（EAR）](https://cwe.mitre.org/data/definitions/698.html "CWE-698：重定向后执行（EAR）")
- [警惕EAR：在及时发现并封堵重定向漏洞](http://cs.ucsb.edu/~bboe/public/pubs/fear-the-ear-ccs2011.pdf "警惕EAR：在及时发现并封堵重定向漏洞")
- [CVE-2013-1402：DigiLIBE管理控制台| 重定向（EAR）漏洞执行](https://nvd.nist.gov/vuln/detail/CVE-2013-1402 "CVE-2013-1402：DigiLIBE管理控制台| 重定向（EAR）漏洞执行")