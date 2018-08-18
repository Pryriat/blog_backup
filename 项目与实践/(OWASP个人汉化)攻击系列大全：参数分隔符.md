[TOC]

*最新版本 (mm/dd/yy): 06/3/2009*

翻译自[Parameter Delimiter](https://www.owasp.org/index.php/Parameter_Delimiter "Parameter Delimiter")

# 漏洞描述
此攻击基于对Web应用程序输入向量使用的参数分隔符进行操纵，以引起意外行为，如访问控制和授权绕过以及信息泄露等。

# 风险因素
待定

# 攻击示例
为了说明这个漏洞，我们将使用一个基于PHP编程语言的发布系统Poster V2上找到的漏洞。这个应用中包含了一个危险的漏洞，允许向"mem.php"文件中负责管理应用用户的用户领域（用户名、密码、电子邮箱地址和特权）插入数据。“mem.php”文件的一个例子，其中用户Jose拥有管理权限并且拥有Alice用户访问权限

```php
<?
Jose|12345678|jose@attack.com|admin|
Alice|87654321|alice@attack.com|normal|
?>
```

当用户想要编辑他的个人资料时，他必须使用“index.php”页面中的“编辑帐户”选项并输入他的登录信息。但是，使用“|”作为电子邮件字段后跟“admin”的参数分隔符，用户可以将其权限提升为管理员。如：


```
Username: Alice
Password: 87654321
Email: alice@attack.com |admin|
```

在 “mem.php”文件中信息会被如下记录:
`Alice|87654321|alice@attack.com|admin|normal|`
在这种情况下，最后一个参数分隔符是“| admin |”，用户可以通过分配管理员配置文件来提升他的权限。

# 相关攻击代理

- [类别：授权](https://www.owasp.org/index.php?title=Category:Authorization&action=edit&redlink=1 "- 类别：授权")
- [类别：命令执行](https://www.owasp.org/index.php?title=Category:Command_Execution&action=edit&redlink=1 "类别：命令执行")

# 相关攻击类型
- [类别：注入攻击](https://www.owasp.org/index.php/Category:Injection_Attack "类别：注入攻击")

# 相关防御措施
- [类别：输入验证](https://www.owasp.org/index.php/Category:Input_Validation "类别：输入验证")

# 参考

- [http://cwe.mitre.org/data/definitions/141.html](http://cwe.mitre.org/data/definitions/141.html "http://cwe.mitre.org/data/definitions/141.html")
- [http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2003-0307](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2003-0307 "http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2003-0307")