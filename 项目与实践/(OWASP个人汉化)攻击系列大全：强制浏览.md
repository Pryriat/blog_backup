[TOC]

翻译自[Forced browsing](https://www.owasp.org/index.php/Forced_browsing "Forced browsing")

# 漏洞描述
强制浏览是一种攻击方式，其目的是枚举和访问未被应用程序引用，但仍可访问的资源。攻击者可以使用[暴力攻击](https://tinytracer.com/archives/%ef%bc%88owasp%e4%b8%aa%e4%ba%ba%e6%b1%89%e5%8c%96%ef%bc%89%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%ef%bc%9a%e6%9a%b4%e5%8a%9b%e6%94%bb%e5%87%bb/ "暴力攻击")技术来搜索域目录内未被链接的内容，比如临时目录和文件，旧的备份和构造文件。这些资源可能存储有关Web应用程序和操作系统的敏感信息，例如源代码，证书，内部网络地址等，因此被视为入侵者的宝贵资源。当应用程序的索引目录和页面基于数字生成或可预测的值，或者使用自动化工具生成公用文件和目录名称时，可以手动执行此攻击。此攻击也称为可预测资源位置，文件枚举，目录枚举和资源枚举

# 攻击示例
## 示例1
本示例介绍了可预测资源位置攻击的技术，该攻击基于手动识别资源并通过修改URL参数进行定位。用户1希望通过以下URL查看他的在线日程表：
` www.site-example.com/users/calendar.php/user1/20070715 `
在URL中，可以识别用户名（user1）和日期（mm / dd / yyyy）。如果用户试图进行强制浏览攻击，他可以通过预测用户身份和日期来猜测另一个用户的日程安排，如下所示：
`www.site-example.com/users/calendar.php/user6/20070716 `
在访问其他用户的议程时，攻击可被视为成功。 授权机制的确实促成了这次攻击。

## 示例2
这个例子展示了使用自动化工具对静态目录和文件枚举的攻击。像[Nikto](http://www.cirt.net/code/nikto.shtml "Nikto")这样的扫描工具能够根据众所周知的资源数据库搜索现有的文件和目录，例如：

```
/system/
/password/
/logs/
/admin/
/test/
```

当工具收到HTTP 200,意味着这种资源被发现，应该手动检查有价值的信息。

# 相关的威胁代理
- [内部软件开发商](https://www.owasp.org/index.php?title=Internal_software_developer&action=edit&redlink=1 "内部软件开发商")

# 相关攻击
- [目录遍历](https://tinytracer.com/archives/%ef%bc%88owasp%e4%b8%aa%e4%ba%ba%e6%b1%89%e5%8c%96%ef%bc%89%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%ef%bc%9a%e8%b7%af%e5%be%84%e9%81%8d%e5%8e%86/ "目录遍历")

# 相关漏洞
- [类别：访问控制漏洞](https://www.owasp.org/index.php/Category:Access_Control_Vulnerability "类别：访问控制漏洞")

# 相关防御措施
- [类别：访问控制](https://www.owasp.org/index.php/Category:Access_Control "类别：访问控制")

# 参考

- Imperva，防止强制浏览，保护应用程序数据安全性和合规性 
  [http://www.imperva.com/application_defense_center/glossary/forceful_browsing.html](http://www.imperva.com/application_defense_center/glossary/forceful_browsing.html "http://www.imperva.com/application_defense_center/glossary/forceful_browsing.html")
- 参数模糊和强制浏览 WebAppSec [http://seclists.org/webappsec/2006/q3/0182.html](http://seclists.org/webappsec/2006/q3/0182.html "http://seclists.org/webappsec/2006/q3/0182.html")
- [http://www.webappsec.org/projects/threat/classes/predictable_resource_location.shtml](http://www.webappsec.org/projects/threat/classes/predictable_resource_location.shtml "http://www.webappsec.org/projects/threat/classes/predictable_resource_location.shtml")
- [http://cwe.mitre.org/data/definitions/425.html](http://cwe.mitre.org/data/definitions/425.html "http://cwe.mitre.org/data/definitions/425.html")
