[TOC]

*最新版本(mm/dd/yy): 04/23/2009*

翻译自[Setting Manipulation](https://www.owasp.org/index.php/Setting_Manipulation "Setting Manipulation")

# 漏洞描述
此攻击旨在修改应用程序设置，为攻击者生成误导性数据或创造优势。他可能会操纵系统中的值并管理应用程序的特定用户资源或影响其功能。攻击者可以利用这种攻击技术执行应用程序的多个功能，但由于攻击者可能使用无数的选项来控制系统值，因此无法描述所有利用方式。使用这种攻击技术，可以通过更改应用程序功能来操作设置，例如调用数据库，阻止访问外部库和/或修改日志文件。

# 风险因素
待定

# 示例
## 示例1
攻击者需要识别没有输入验证的变量或不恰当的封装以达成攻击。以下示例基于Individual CWE Dictionary Definition (Setting Manipulation-15)中的那些示例。假设有如下的JAVA代码片段

```java
 …
 conn.setCatalog(request.getParameter(“catalog”));
 ...
```
该片段从“HttpServletRequest”中读取字符串“catalog”，并将其设置为数据库连接的活动目录。 攻击者可能会操纵这些信息并导致连接错误或未经授权访问其他目录。

## 示例2 - 阻止访问库
通过执行此攻击，攻击者可以获得阻止应用程序访问外部库的特权。有必要发现应用程序访问哪些外部库并阻止它们。攻击者需要观察系统的行为是否进入不安全/不一致的状态。在此情况下应用程序使用第三方密码随机数生成库来生成用户会话ID。攻击者可以通过重命名的方式阻止访问该库。然后应用程序将使用脆弱的伪随机数字生成库。攻击者可以利用这个弱点预测用户会话ID，执行特权提升并获得对用户帐户的访问权限。更多关于此攻击的详情请参见：[http://capec.mitre.org/data/definitions/96.html](http://capec.mitre.org/data/definitions/96.html "http://capec.mitre.org/data/definitions/96.html")

# 相关威胁代理

- [类别：逻辑攻击](https://www.owasp.org/index.php?title=Category:Logical_Attacks&action=edit&redlink=1 "类别：逻辑攻击")

# 相关攻击

- [拒绝服务](https://www.andseclab.cn/2018/04/14/owasp%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%e4%ba%8c%e5%8d%81%e5%85%ad%e6%8b%92%e7%bb%9d%e6%9c%8d%e5%8a%a1/ "拒绝服务")

# 相关的漏洞

- [类别：常规逻辑错误漏洞](https://www.owasp.org/index.php/Category:General_Logic_Error_Vulnerability "类别：常规逻辑错误漏洞")

# 相关防御措施

- [类别：错误处理](https://www.owasp.org/index.php/Category:Error_Handling "类别：错误处理")

# 参考

- [http://cwe.mitre.org/data/definitions/15.html](http://cwe.mitre.org/data/definitions/15.html "http://cwe.mitre.org/data/definitions/15.html") - 设置操作
- [http://capec.mitre.org/data/definitions/13.html](http://capec.mitre.org/data/definitions/13.html "http://capec.mitre.org/data/definitions/13.html") - 覆写环境变量值
- [http://capec.mitre.org/data/definitions/96.html](http://capec.mitre.org/data/definitions/96.html "http://capec.mitre.org/data/definitions/96.html") - 阻止对库的访问