[TOC]

*最新版本: 12/9/2016*

翻译自[LDAP injection](https://www.owasp.org/index.php/LDAP_injection "LDAP injection")

# 漏洞描述
LDAP注入是一种攻击，利用了web应用程序基于用户输入构建LDAP语句的行为。当应用程序未能正确处理用户输入时，可以使用本地代理修改LDAP语句。 这可能会导致执行任意命令，如授予对未授权查询的权限以及LDAP树内的内容修改。SQL注入中可用的相同高级利用技术可以类似地应用于LDAP注入。

# 参考

- [https://www.owasp.org/index.php/LDAP_Injection_Prevention_Cheat_Sheet](https://www.owasp.org/index.php/LDAP_Injection_Prevention_Cheat_Sheet "https://www.owasp.org/index.php/LDAP_Injection_Prevention_Cheat_Sheet")