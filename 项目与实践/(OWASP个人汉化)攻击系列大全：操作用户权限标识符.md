[TOC]

翻译自[Manipulating User Permission Identifier](https://www.owasp.org/index.php/Manipulating_User_Permission_Identifier "Manipulating User Permission Identifier")

# 漏洞描述
此攻击重点在于操纵用户权限标识符，以提升其对应用程序的权限，导致未经授权的访问，欺诈性交易和应用程序中断。用户权限标识符通常与会话ID，本地Cookie，隐藏字段等相关联。为了执行本攻击，必须要知道应用程序是如何管理用户权限辨识符，存储和管理信息的位置/方式/条目（客户端，服务器端或两者）以及使用哪些数据作为标识符的一部分。基于此，攻击者可以使用会话标识符的新值伪造他的请求，并在应用程序中提升他的权限。

# 攻击示例
假设应用程序使用在客户机器上使用未加密的cookie来存储认证标识（auth=0/1）。攻击者可以篡改用户会话的这一信息，并设置“auth = 1”以获得非法访问并在应用程序中提升其权限

# 外部参考
- [http://capec.mitre.org/data/definitions/74.html](http://capec.mitre.org/data/definitions/74.html "http://capec.mitre.org/data/definitions/74.html")

# 相关威胁
- [类别：授权](https://www.owasp.org/index.php?title=Category:Authorization&action=edit&redlink=1 "类别：授权")
- [类别：信息披露](https://www.owasp.org/index.php?title=Category:Information_Disclosure&action=edit&redlink=1 "类别：信息披露")


# 相关攻击方式
- [会话劫持](https://www.andseclab.cn/2018/04/21/owasp%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%e4%ba%94%e5%8d%81%e4%b8%83%ef%bc%9a%e4%bc%9a%e8%af%9d%e5%8a%ab%e6%8c%81%e6%94%bb%e5%87%bb/ "会话劫持")

# 相关漏洞
- [类别：环境漏洞](https://www.owasp.org/index.php/Category:Environmental_Vulnerability "类别：环境漏洞")


# 相关防御措施
- [类别：访问控制](https://www.owasp.org/index.php/Category:Access_Control "类别：访问控制")

# 攻击分类
- [类别：资源操作](https://www.owasp.org/index.php/Category:Resource_Manipulation "类别：资源操作")