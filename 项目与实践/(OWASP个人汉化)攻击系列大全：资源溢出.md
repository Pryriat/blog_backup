[TOC]

*最新版本 (mm/dd/yy): 12/30/2013*

翻译自[Cash Overflow](https://www.owasp.org/index.php/Cash_Overflow "Cash Overflow")

# 漏洞描述
资源溢出是一种[拒绝服务攻击](https://www.andseclab.cn/2018/04/14/owasp%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%e4%ba%8c%e5%8d%81%e5%85%ad%e6%8b%92%e7%bb%9d%e6%9c%8d%e5%8a%a1/ "拒绝服务攻击")，旨在超出云应用程序的托管成本，主要是破坏服务所有者或超出应用程序成本限制，导致云服务提供商禁用应用程序。

# 风险因素
- 在资源足够的情况下容易发起攻击
- 由立刻停机/资源消耗/日志可以很快发现攻击
- 影响通常局限于使服务器失能

# 相关的威胁代理
- 与[类别：Internet攻击者](https://www.owasp.org/index.php?title=Category:Internet_attacker&action=edit&redlink=1 "类别：Internet攻击者")最为相似

# 相关攻击方式
- [拒绝服务攻击](https://www.andseclab.cn/2018/04/14/owasp%e6%b1%89%e5%8c%96%e6%94%bb%e5%87%bb%e7%b3%bb%e5%88%97%e5%a4%a7%e5%85%a8%e4%ba%8c%e5%8d%81%e5%85%ad%e6%8b%92%e7%bb%9d%e6%9c%8d%e5%8a%a1/ "拒绝服务攻击")

# 相关漏洞
- [https://www.owasp.org/index.php/Category:Input_Validation_Vulnerability](https://www.owasp.org/index.php/Category:Input_Validation_Vulnerability "https://www.owasp.org/index.php/Category:Input_Validation_Vulnerability")
- [https://www.owasp.org/index.php/Category:API_Abuse](https://www.owasp.org/index.php/Category:API_Abuse "https://www.owasp.org/index.php/Category:API_Abuse")

# 相关防御措施
- DoS预防技术

# 参考
[http://capec.mitre.org/data/index.html](http://capec.mitre.org/data/index.html "http://capec.mitre.org/data/index.html") -资源枯竭导致的拒绝服务