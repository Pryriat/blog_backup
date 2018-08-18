[TOC]

*最新版本 (mm/dd/yy): 05/30/2013*

翻译自[Blind XPath Injection](https://www.owasp.org/index.php/Blind_XPath_Injection "Blind XPath Injection")

# 漏洞描述
- XPath是一种查询语言，它描述了如何在XML文档中查找特定元素（包括属性、处理指令等）。既然是一种查询语言，XPath在一些方面与SQL相似，不过，XPath的不同之处在于它可以用来引用XML文档的几乎任何部分，而不受访问控制限制。在SQL中，一个“用户”（在XPath/XML上下文中未定义的术语）的权限被限制在一个特定的数据库，表，列或者行。使用XPath注入攻击，攻击者可以修改XPath查询语句来执行所选择的操作
- XPath盲注攻击可以从一个使用不安全方式嵌入用户信息的应用中提取数据。在输入未被过滤的情况下，攻击者可以提交并执行有效的XPath代码。这种类型的攻击适用于以下情况：攻击者不清楚XML文档的架构，或者错误消息被抑制，一次只能通过布尔化查询来获取部分信息，就像SQL盲注一样。
- 更多信息请参考[XPath注入](https://www.andseclab.cn/2018/04/20/owasp%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%e5%85%ad%e5%8d%81%e5%85%ab%ef%bc%9axpath%e6%b3%a8%e5%85%a5%e6%94%bb%e5%87%bb/ "XPath注入")

# 风险因素
待定

# 攻击示例
- 一个攻击者可以通过两种方法实施攻击：布尔攻击（暂译）和XML爬行。通过添加XPath语法，攻击者使用其他表达式（替换攻击者在注入点输入的内容）

## 布尔攻击
- 使用“布尔攻击”的方式，攻击者可以判断给定的XPath表达式的返回值是真还是假。让我们假设攻击者的目标是在网页应用登陆一个账户。一次成功的登陆会返回`True`且一次失败的尝试返回`False`。通过这种方式可以推断目标信息的一小部分数字或字符。当攻击者着眼于字符串时，他可以通过检查该字符串所属的类/字符范围内的每个单个字符来揭示它的全部内容。
- 攻击者可以通过使用`string-length(S)`函数（S是一个`String`)来获取字符串的长度。通过适当次数的`subsreing(S,N,1)`函数迭代，其中S是前面提到的字符串，N是起始字符，"1"是从N起始的下一个字符，攻击者能够枚举出整个字符串。
- 代码
```xml
<?xml version="1.0" encoding="UTF-8"?>
<data>
   <user>
   <login>admin</login>
   <password>test</password>
   <realname>SuperUser</realname>
   </user>
   <user>
   <login>rezos</login>
   <password>rezos123</password>
   <realname>Simple User</realname>
   </user>
</data>
```
函数：
- `string.stringlength(//user[position()=1]/child::node()[position()=2])`返回第一个用户第二个字串的长度(8)
  `substring((//user[position()=1]/child::node()[position()=2),1,1)`返回该用户名第一个字符（'r'）

## XML爬行
- 为了获取XML文档的架构，攻击者可能会用
  - count(表达式)
    `count(//user/child::node()`
    这条语句返回节点数
     stringlength(string)	
    `stringlength(//user[position()=1]/child::node()[position()=2])=6`
    使用这条查询语句攻击者会找出第一个节点（用户'admin'）的第二个子串（密码）由6个字符组成
  - substring(string, number, number)
    `substring((//user[position()=1]/child::node()[position()=2]),1,1)="a"`
    这条查询语句将确认或否定用户(admin)密码的第一个字符是'a'

- 如果登录表单看起来像这样
```C#
String FindUser;
FindUser = "//user[login/text()='" + Request("Username") + "' And
      password/text()='" + Request("Password") + "']";
```
攻击者可以通过以下代码注入
`Username: ' or substring((//user[position()=1]/child::node()[position()=2]),1,1)="a" or ''='`

- 这种XPath语法可能会让你回想起SQL注入攻击，不过攻击者必须明白该语言禁止注释剩余的表达式。为了绕过这一限制，攻击者应使用OR表达式来激活所有表达式，这可能导致攻击被终止。

- 由于布尔攻击的查询次数，即使在一个小的XML文档中依然非常高（成千上万）。这就是这种类型的攻击不是手动进行的原因。了解一些基本的XPath函数，攻击者可以在短时间内写出一个重建文档结构并添加自己内容的应用。

# 相关的威胁代理
待定

# 相关攻击方式

- [SQL盲注](https://tinytracer.com/archives/%ef%bc%88owasp%e4%b8%aa%e4%ba%ba%e6%b1%89%e5%8c%96%ef%bc%89%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%ef%bc%9asql%e7%9b%b2%e6%b3%a8/ "SQL盲注")
- [XPath注入](https://www.andseclab.cn/2018/04/20/owasp%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%e5%85%ad%e5%8d%81%e5%85%ab%ef%bc%9axpath%e6%b3%a8%e5%85%a5%e6%94%bb%e5%87%bb/ "XPath注入")

# 相关漏洞

- [注入漏洞](https://www.owasp.org/index.php/Injection_problem "注入漏洞")

# 相关防御措施

- [类别：输入验证](https://www.owasp.org/index.php/Category:Input_Validation "Category:Input Validation")

# 参考

- [http://dl.packetstormsecurity.net/papers/bypass/Blind_XPath_Injection_20040518.pdf](http://dl.packetstormsecurity.net/papers/bypass/Blind_XPath_Injection_20040518.pdf "http://dl.packetstormsecurity.net/papers/bypass/Blind_XPath_Injection_20040518.pdf") - by Amit Klein (更多细节，是了解XPath盲注最佳的来源)

- [http://www.ibm.com/developerworks/xml/library/x-xpathinjection/index.html](http://www.ibm.com/developerworks/xml/library/x-xpathinjection/index.html "http://www.ibm.com/developerworks/xml/library/x-xpathinjection/index.html")

- [http://projects.webappsec.org/w/page/13247005/XPath%20Injection](http://projects.webappsec.org/w/page/13247005/XPath%20Injection "http://projects.webappsec.org/w/page/13247005/XPath%20Injection")