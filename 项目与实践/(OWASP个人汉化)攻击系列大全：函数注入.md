[TOC]

*最新版本(mm/dd/yy)：09/1/2016*

翻译自[Function Injection](https://www.owasp.org/index.php/Function_Injection "Function Injection")

# 漏洞介绍
函数注入攻击包括从客户端向应用程序插入或“注入”函数名称。一次成功的函数注入攻击可以执行任何内置或者用户自定义的函数。函数注入攻击是注入攻击的一类，将带参数或无参的任意函数注入应用程序并执行。如果参数传递给注入函数，会导致远程代码执行。

# 风险因素

- 这些类型的漏洞可能很难找到，也可能很容易找到
- 如果找到，通常难以利用，取决于具体情况。
- 如果成功利用，影响可能包括机密性丧失，完整性丧失，可用性损失和/或责任丧失

# 攻击示例
## 示例1
如果应用程序将通过GET请求发送的参数传递给PHP，然后通过在变量名后面包含（）将该参数作为函数进行计算，则该变量将被视为函数并将被执行。下面的URL向脚本传递一个函数名
`http://testsite.com/index.php?action=edit`
index.php文件包含以下代码

```php
<?php
$action = $_GET['action'];
$action();
?>
```

通过这种方法攻击者可以执行在脚本中执行任意函数，如`phpinfo`
`http://testsite.com/index.php?action=phpinfo`

## 示例2
该示例是示例1的扩展且更危险的版本，本示例中，引用不仅允许提供函数名，也允许提供该函数的参数
`http://testsite.com/index.php?action=edit&pageid=1`
index.php包含以下代码

```php
<?php
$action = $_GET['action'];
$pageid = $_GET['pageid'];
$action($pageid);
?>
```
在此情况下攻击者不仅可以传递函数名，而且可以传递函数的参数，这意味着可以通过使用任意命令传递给系统函数来导致远程代码执行
`http://testsite.com/index.php?action=system&pageid=ls`
这将会在系统中执行`ls`命令

## 示例3
此示例显示了使用call_user_func而不是使用括号来评估用户函数的另一种方法。
`http://testsite.com/index.php?action=edit`
index.php包含以下代码

```php
<?php
$action = $_GET['action'];
call_user_func($action);
?>
```

与示例1相似，攻击者可以向脚本传递任意函数名，如`phpinfo`
`http://testsite.com/index.php?action=phpinfo`

## 示例4
该示例是示例3的扩展且更危险的版本，本示例中，应用将向call_user_func传递另一个参数，它将作为参数传递给call_user_func的第一个参数中提供的函数，多个参数可以用数组的形式传递给call_user_func。
`http://testsite.com/index.php?action=edit&pageid=1`
index.php包含以下代码

```php
<?php
$action = $_GET['action'];
$pageid = $_GET['pageid'];
call_user_func($action,$pageid);
?>
```

在这种情况下，攻击者不仅会传递函数名，而且还会传递该函数的参数，这意味着可以通过使用任意命令传递给系统函数来导致远程代码执行。
`http://testsite.com/index.php?action=system&pageid=ls`
这将会在系统中执行`ls`命令

# 相关攻击代理

- [类别：互联网攻击](https://www.owasp.org/index.php?title=Category:Internet_attacker&action=edit&redlink=1 "类别：互联网攻击")
- [内部软件开发商](https://www.owasp.org/index.php?title=Internal_software_developer&action=edit&redlink=1 "内部软件开发商")

# 相关攻击类型

- [命令注入](https://www.andseclab.cn/2018/04/12/owasp%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%e4%b9%9d-%e5%91%bd%e4%bb%a4%e6%b3%a8%e5%85%a5/ "命令注入")
- [SQL注入](https://www.andseclab.cn/2018/04/22/owasp%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%e5%85%ad%e5%8d%81%e4%ba%8c%ef%bc%9asql%e6%b3%a8%e5%85%a5/ "SQL注入")
- [LDAP注入](https://tinytracer.com/archives/owasp%e4%b8%aa%e4%ba%ba%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%ef%bc%9aldap%e6%b3%a8%e5%85%a5/ "LDAP注入")
- [SSI注入](https://www.andseclab.cn/2018/04/20/owasp%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%e4%ba%94%e5%8d%81%e4%ba%94%ef%bc%9a%e6%9c%8d%e5%8a%a1%e5%99%a8%e7%ab%af%e5%8c%85%e5%90%abssi%e6%b3%a8%e5%85%a5/ "SSI注射")
- [跨站点脚本（XSS）](https://tinytracer.com/archives/owasp%e4%b8%aa%e4%ba%ba%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%ef%bc%9a%e8%b7%a8%e7%ab%99%e7%82%b9%e8%84%9a%e6%9c%ac%ef%bc%88xss%ef%bc%89/ "跨站点脚本（XSS）")

# 相关防御措施

- [输入验证](https://www.owasp.org/index.php/Input_Validation "输入验证")
- [规范化](https://www.owasp.org/index.php/Canonicalization "规范化")

# 参考

[call_user_func](http://php.net/manual/en/function.call-user-func.php "call_user_func") - call_user_func的PHP文档。
[call_user_func_array](http://php.net/manual/en/function.call-user-func-array.php "call_user_func_array") - call_user_func的PHP文档